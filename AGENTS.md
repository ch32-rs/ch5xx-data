# AGENTS.md — working on ch5xx-data

This repository is the structured MCU database for WCH's CH5xx RISC-V BLE
parts, and the generator that turns it into `ch5xx-metapac`. It is a sibling of
[ch32-data](https://github.com/ch32-rs/ch32-data); the CH5xx register map is
not the CH32 one, so the data lives here.

## Layout

```
data/
  chips/          one YAML per chip (or per flash variant)
  family/         peripheral list: name, address, registers.kind/version/block
  interrupts/     interrupt name → number
  registers/      chiptool IR: block/<NAME> + fieldset/<NAME>
svd/
  vendor/         MounRiver SVD, untouched
  fixed/          vendor SVD after the cleanup scripts (schema-valid)
scripts/          draft helpers (not required to regenerate from committed data)
ch5xx-data-gen/   YAML → build/data JSON
ch5xx-metapac-gen/
                  JSON → ch5xx-metapac crate
```

`build/` is generated and gitignored. `scripts/` and the SVD cleanup tools are
kept local until they settle; commits focus on `data/` and the generators.

## Naming

- `data/chips/<CHIP>.yaml` is named after the chip, not the MounRiver series
  directory. Prefer `CH592.yaml`, never `CH59X.yaml`.
- A separate chip file is allowed when memory differs enough that the chip-level
  `memory:` field cannot describe every package. `CH584X.yaml` exists for that
  reason (480KB code / 512KB data vs CH584's 448KB / 32KB). Same peripherals,
  different flash.
- `data/family/<NAME>.yaml` and `data/interrupts/<NAME>.yaml` are shared lists.
  Name them after the most complete chip that uses them (`CH592.yaml` for both
  CH591 and CH592). Do not use a wildcard series name like `CH59X` — it collides
  visually with neighbouring parts such as `CH595`.
- `data/registers/<kind>_<version>.yaml` carries the register layout. The
  `version` is a lowercase chip tag (`ch585`, `ch595`, `ch59x`, `ch581`). A
  family entry may point at another chip's version when the YAML is
  byte-identical. When merging identical files, keep the **smaller chip
  number** as the version (`i2c_ch585.yaml`, not `i2c_ch595.yaml`).
- Kind / block / instance are separate:
  - **instance** is the peripheral name in the family list (`I2C0`).
  - **kind** is the type slug in the filename and family `registers.kind`
    (`i2c` → `i2c_<version>.yaml`).
  - **block** is the chiptool block name inside that file (`block/I2C`).
  Do not bake the instance number into kind or block (`i2c0` / `I2C0` as the
  type is wrong; the instance stays `I2C0`).
- Packages inside a chip file stop at the package letter (`CH585M`, `CH592F`).
  Temperature-grade digits are not part of the name.

`CH595` and `CH592` (formerly filed under MounRiver's `CH59X`) are different
chips with different SVDs. Their family files are not interchangeable.

## Sources of truth, in order

1. **Datasheet** (WCH "DS" PDF). For these parts the DS also acts as the
   reference manual — there is no separate RM. Prefer it for memory maps,
   which peripherals exist on which package, and field meanings.
2. **SFR header and linker script** from the matching MounRiver Studio 2
   project template (`…/WCH/SDK/default/RISC-V/<series>/NoneOS/<PART>.zip`).
   Use these when the datasheet is missing, or to confirm what the vendor's
   own code believes about a cut-down die (CH581 vs CH585 is the example).
3. **Vendor SVD** from the same SDK directory. Source of register offsets and
   fields. Never trust it alone for which peripherals a cut-down package has:
   one SVD often covers several part numbers.

Which SVD covers which part numbers is recorded in `docs/svd-map.md`, taken
from the template zip names next to each SVD in MounRiver Studio 2.

When sources disagree, write the disagreement into the chip YAML comment and
prefer the datasheet. Example: CH584 keeps LCD per the datasheet; CH581 drops
LCD per its SFR header, which the CH585 datasheet never mentions.

## Factual basis for CH5xx

These differ from CH32 and drive the data model:

- No RCC / AFIO. Clock, power, pin function and GPIO live in one SYS region at
  `0x40001000`, split in the SVD into SYS / RTC / System_control / TouchKey /
  ADC / GPIO by register offset. Keep the split; do not merge them back.
- Register names carry a width prefix (`R8_` / `R16_` / `R32_`) and fields use
  `RB_`. Mixed widths inside one peripheral are normal.
- Flash is not the CH32 model. There is usually no flash register block in the
  SVD. Erase and program go through an ISP library (or, on CH595, a
  memory-mapped controller at `0x40001800` that the SVD omits). Code and data
  regions can have different erase/write sizes; describe each with its own
  `memory:` entry and `modes`.
- BLE is present in the address map (`0x4000C000`) but the vendor SVD gives it
  no registers. Keep a register-less placeholder so the base address stays
  recorded.
- Instance grouping is per chip. Timers and UARTs only share a block when the
  extracted YAML is the same. On CH585 all four timers share one block; on
  CH595 / CH59x, TMR0/TMR3 have no DMA and TMR1/TMR2 do, so they are two
  blocks. Pass the grouping explicitly to the extract script; do not hard-code
  one rule for every SVD.

## Workflow for a new chip

1. Copy the vendor SVD into `svd/vendor/` and run the cleanup pipeline into
   `svd/fixed/` (see `svd/README.md`). The result must pass
   `xmllint --schema svd/CMSIS-SVD.xsd`.
2. Draft registers, family and interrupts with
   `scripts/gen-from-svd.py <fixed.svd> <CHIP> --group 'PATTERN=BLOCK:KIND'`.
   Re-insert the BLE placeholder afterwards; the script overwrites the family
   file.
3. Diff the new `data/registers/*_<version>.yaml` against existing versions.
   Where a file is byte-identical to another chip's, delete it and point the
   family entry at that `version`.
4. Write `data/chips/<CHIP>.yaml` from the datasheet (memory, packages, docs,
   core). Include the shared family and interrupt files. Cut-down dies get
   their own family list (drop missing peripherals) and, only when fields
   actually disappear, their own register version (see `sys_ch581`).
5. `cargo run -p ch5xx-data-gen` then
   `cargo run -p ch5xx-metapac-gen -- <CHIP>`.
6. `cargo check --manifest-path build/ch5xx-metapac/Cargo.toml \
     --features <chip>,pac,metadata --target riscv32imac-unknown-none-elf`.
   Only one chip feature at a time.

Pins, clocks and per-signal interrupt links are not in the SVD; leave them for
hand editing.

## Commits

Commit `data/` and generator crate changes. Leave `scripts/`, `svd/` cleanup
outputs and local helpers uncommitted until asked. End commit messages with:

```
Co-Authored-By: Claude Code <noreply@anthropic.com>
```
