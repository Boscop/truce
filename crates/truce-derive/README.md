# truce-derive

Proc macros for truce plugin metadata.

## Overview

Provides the `plugin_info!()` macro, which reads `truce.toml` at compile time
and generates a `PluginInfo` struct literal containing the plugin name, unique
ID, vendor, category, and version. Removes the need for hand-written metadata
constants. (Plugin crates still need a small `build.rs` calling
`truce_build::emit_plugin_env()` — that handles format-feature check-cfg and
optional env-var overrides; see the `truce-build` crate.)

The macro is re-exported through `truce::plugin_info!()`, so plugin authors do
not need to depend on this crate directly.

## Why a separate crate (vs. `truce-params-derive`)

Both proc-macro crates expose derives consumed by the `truce` facade.
The split is mostly about **separation of concerns** — this crate
covers plugin metadata (`plugin_info!()` reading `truce.toml`),
`truce-params-derive` covers parameter struct boilerplate. Different
axes of plugin authoring.

The deps differ — this crate pulls in `toml` + `serde` (with derive)
to parse `truce.toml`; `truce-params-derive` is pure `syn` + `quote`.
That doesn't actually save compile time in practice (every plugin
uses both `plugin_info!()` and `#[derive(Params)]`, so the toml /
serde cost is universal regardless of split), but it does keep the
heavier dep tree localised to one crate.

Minor build-parallelism upside: two independent proc-macro crates
can compile concurrently. Merging would serialise them behind one
proc-macro pre-build. Marginal in practice.

Could be merged into a single `truce-derive` carrying all four
derives + `plugin_info!()`. The trade-off is a `Cargo.toml` rename
for every plugin that takes a direct dep on `truce-params-derive`
(the example plugins do). Today's split is the status quo, not a
hard technical requirement.

## Key macro

- **`plugin_info!()`** -- expands to a `PluginInfo` struct populated from `truce.toml`

## Usage

```rust
use truce::prelude::*;

impl Plugin for MyPlugin {
    fn info() -> PluginInfo {
        truce::plugin_info!()
    }
}
```

## What it reads from `truce.toml`

- Plugin name and unique ID
- Vendor name and URL
- Plugin category (effect or instrument)
- AU type, subtype, and manufacturer codes
- Optional version override

Part of [truce](https://github.com/truce-audio/truce).
