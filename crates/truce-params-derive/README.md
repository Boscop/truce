# truce-params-derive

Derive macros for truce parameter structs.

## Overview

Provides `#[derive(Params)]` and `#[derive(ParamEnum)]` to generate the
boilerplate needed to expose parameter structs to the host. The generated code
handles parameter enumeration, get/set by index, display formatting, and state
serialization. Plugin authors rarely depend on this crate directly -- it is
re-exported through `truce::prelude`.

## Why a separate crate (vs. `truce-derive`)

Mostly **separation of concerns** — this crate covers parameter
struct boilerplate, `truce-derive` covers plugin metadata
(`plugin_info!()`). Different axes of plugin authoring.

The deps differ (this crate stays pure `syn` + `quote` +
`proc-macro2`; `truce-derive` adds `toml` + `serde`), but in practice
every plugin uses both `#[derive(Params)]` and `plugin_info!()`, so
the heavier compile cost is universal regardless of split.

Could be merged into a single `truce-derive` carrying all four
derives + `plugin_info!()`. Costs: every plugin's `Cargo.toml` would
need to swap the direct `truce-params-derive` dep for `truce-derive`,
and `truce-loader`'s dev-dep (used in two test files for
`#[derive(Params)]` fixtures) would also rename. Today's split is the
status quo, not a hard technical requirement.

## Macros

### `#[derive(Params)]`

Applied to a struct whose fields are `FloatParam`, `IntParam`, `BoolParam`, or
`EnumParam`. Generates trait implementations for parameter discovery, indexed
access, and state round-tripping.

### `#[derive(ParamEnum)]`

Applied to an enum to make it usable as an `EnumParam` value. Generates
variant-to-index mapping and display names.

## Example

```rust
use truce::prelude::*;

#[derive(ParamEnum)]
enum FilterMode { LowPass, HighPass, BandPass }

#[derive(Params)]
struct MyParams {
    #[param(name = "Cutoff", range = log(20.0, 20000.0), unit = "Hz")]
    cutoff: FloatParam,

    #[param(name = "Mode")]
    mode: EnumParam<FilterMode>,
}
```

Part of [truce](https://github.com/truce-audio/truce).
