# GTA 6 development target — local XCL changes

This is a modified development snapshot of XCL, not a GTA VI compatibility release.
The repository name identifies the intended development target only. No GTA VI
executable, game data, shader corpus or runtime report was available for this
change. No game-specific support or frame-rate result is claimed.

## Shader-cache improvement

The previous in-flight lock serialized concurrent requests but kept no result.
If a compiled shader did not fit the RAM cache, each waiting caller compiled it
again. The change retains one successful result for the lifetime of that flight,
so its callers can share it even when the LRU budget is zero or too small.

- Full console-blob bytes are compared, not only their hash.
- Failed compiles remain retryable.
- Cached-only probes still do not wait for an in-flight compile.
- Flight results are released after the last active caller leaves; they do not
  become an unbounded second cache.
- Independent shader keys remain independently compilable.

`XCL_SHADER_CACHE_MB` now sets the retained shader-cache budget in MiB. The default
is unchanged at 64; accepted values are 0 through 4096. Invalid values use 64.
Zero disables retention in RAM, not the existing disk cache. The budget is read
once, on the first use, and logged. It does not limit the whole game's memory,
active compiles or copies of shader code owned by pipelines.

A larger budget is optional. It may reduce repeated disk lookups if a game's
working set outgrows 64 MiB, but also consumes additional RAM. No performance gain
on a retail game has been measured for this copy.

## Build and focused tests

From the project folder, with the MSVC Rust toolchain and Windows SDK:

```powershell
cargo test -p xcl --lib shader_cache_tests -- --test-threads=4 --nocapture
cargo build --release --workspace
```

See `VALIDATION.md` for the commands and results actually produced locally.

## First run with an available game

These examples assume that `D:\GTA6` contains the unpacked game's files and
`MicrosoftGame.config`. They do not unpack or decrypt a game package.

```powershell
$Game = 'D:\GTA6'
# Optional RAM-cache budget; this is not a GTA 6 compatibility preset.
$env:XCL_SHADER_CACHE_MB = '256'
# Bounded first diagnostic run, with activity and GPU-removal diagnostics.
$env:XCL_ACTIVITY = '1'
$env:XCL_DRED = '1'
$env:XCL_RUN_SECONDS = '60'
.\target\release\xcl.exe run $Game
```

Inspect the actual game executable before a run when its name is known:

```powershell
.\target\release\xcl.exe inspect 'D:\GTA6\actual-game-executable.exe' --json
```

Use the real executable name; the name above is a placeholder. XCL prints the
actual log and report paths at the end of a run. By default, title data is under
`%LOCALAPPDATA%\XCL\<title>\`, including `logs\xcl.log`, `logs\report.json`,
shader caches, saves and the writable overlay. Subsequent compatibility work
requires those reports and the game version: a startup result alone is not proof
of correct rendering or playable gameplay.

For an unrestricted later run, remove the diagnostic time limit first:

```powershell
Remove-Item Env:XCL_RUN_SECONDS -ErrorAction SilentlyContinue
```

No existing graphics backends, CPU capabilities, memory allocator settings or
experimental translation modes were forced on by this change.
