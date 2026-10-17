# SHOW design

## Layers

- **Immutable:** fixed, always searched first, not numbered. Contains `builtin`, `builtout`, and other core commands. Cannot be overridden or reordered.
- **Layer 1, builtin:** SHOW's internal commands.
- **Layer 2, extended builtin:** additional internal commands.
- **Layer 3, path:** external executables found through `PATH`.

Layers 1 to 3 can be reordered by the user.

## Resolution

- `NAME ...`: immutable, then layers 1, 2, and 3 in the order given by `SHOW_CMD_ORDER`. Stops at the first match.
- `builtin NAME ...`: immutable, then layer 1, then layer 2. Never checks layer 3.
- `builtout NAME ...`: layer 3 only. Never checks the immutable layer or layers 1 and 2.

## SHOW_CMD_ORDER

- Lists layers as numbers from 1 to 3, for example `3,2,1`.
- Partial lists are allowed. Layers left out are not searched for ordinary `NAME ...` lookups.
- Duplicates keep their first position.
- An empty value means the default `1,2,3`.
- Unknown entries are an error.

## Open decisions

These are proposed but not yet confirmed:

1. Whether `builtin` and `builtout` ignore `SHOW_CMD_ORDER` and always use their fixed scopes. Proposed: yes.
2. Whether an invalid `SHOW_CMD_ORDER` is an error or a fallback. Proposed: print a warning naming the bad entry, then fall back to `1,2,3`.
3. Whether `builtin` and `builtout` with no arguments print usage and exit with status 2. Proposed: yes.

## Test cases

- `SHOW_CMD_ORDER=3,2,1`, with `grep` on PATH and a user function `grep` in layer 1: `grep` runs the PATH version.
- Same setting, `builtin grep`: runs the layer 1 or 2 version, never PATH.
- `builtout grep`: runs the PATH version even when an immutable or internal `grep` exists.
- `builtout builtin` with no `builtin` executable on PATH: fails with a clear error.
- Names in the immutable layer, such as `builtin` and `builtout`, cannot be overridden by any layer.
