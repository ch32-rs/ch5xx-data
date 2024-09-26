# ch5xx-data

The structured MCU DB of WCH MCUs, CH5xx RISC-V Series.

These MCU are different from CH32V Series, majar differences are:

- GPIO/RTC/PMU are all in SYS register block
- Most peripheral registers are 8-bit (R8_ prefix)

## MCU Family

| Part NO. | Core   | Freq   | Flash | SRAM | DataFlash | BLE | Other |
|----------|--------|--------|-------|------|-----------|-----|-------|
| CH585    | RISC-V | 78MHz  | 448K  | 128K | 32K       | 5.4 | USB HS, NFC |
| CH584    | RISC-V | 78MHz  | 448K  | 96K  | 32K       | 5.4 | NFC |
| CH592    | RISC-V | 20MHz  | 448K  | 26K  | 32K       | 5.4 | |
| CH591    | RISC-V | 20MHz  | 192K  | 26K  | 32K       | 5.4 | |
| CH583    | RISC-V | 20MHz  | 448K  | 32K  | 32K       | 5.3 | |
| CH582    | RISC-V | 20MHz  | 448K  | 32K  | 32K       | 5.3 | |
| CH581    | RISC-V | 20MHz  | 192K  | 32K  | 32K       | 5.3 | |
| CH573    | RISC-V | 20MHz  | 448K  | 18K  | 32K       | 4.2 | |
| CH571    | RISC-V | 20MHz  | 192K  | 18K  | 32K       | 4.2 | |

And CH569/5.

CH564 is not included, it's more list a CH32V series.
