# ch5xx-metapac

Peripheral Access Crate (PAC) for WCH CH5xx microcontrollers, generated from
[`ch32-rs/ch5xx-data`](https://github.com/ch32-rs/ch5xx-data).

The register, peripheral and interrupt definitions live in `ch5xx-data`. This
repository is a checkpoint of the generated crate so downstream crates can use a
git dependency; do not edit it by hand. It is regenerated from `ch5xx-data`
`main` by CI and replaced wholesale.

## Features

Select exactly one generated chip feature (for example `ch585`) and `rt` for the
runtime. `memory-x` emits `memory.x` for `qingke-rt`'s linker script, and
`metadata` exports the chip metadata for `build.rs` users. With
`default-features = false, features = ["metadata"]` only the metadata is built.

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT license ([LICENSE-MIT](LICENSE-MIT))

at your option.
