# Local validation

## Environment

- Windows 11 Pro, build 10.0.26200.
- AMD Ryzen 5 5500, approximately 16 GiB installed RAM.
- NVIDIA GeForce RTX 4060, driver version 32.0.16.1088.
- Rust stable 1.100.0, x86_64-pc-windows-msvc; Visual Studio 2026 build tools.
- DXC 1.9.2602.17 from Windows SDK 10.0.26100.0.
- Rust and Cargo were installed into a temporary, isolated directory for this
  validation; the system PATH was not modified.

## Regression reproduced before the runtime fix

With the new regression test added to the otherwise original runtime:

```text
cargo test -p xcl --lib shader_cache_coalesces_oversized_compiles -- --nocapture
0 passed; 1 failed; 0 ignored; 1126 filtered out
assertion failed: limit 0: waiters recompiled an oversized shader
left: 8
right: 1
```

This counts a synthetic compile closure for eight simultaneous requests. It is
not a GTA 6 shader benchmark or a measured frame-rate improvement.

## Results after the fix

```text
cargo test -p xcl --lib shader_cache_tests -- --test-threads=4 --nocapture
20 passed; 0 failed; 0 ignored; 1108 filtered out
No SKIPPED messages in this focused test run.
```

The set includes RAM and disk caching, context/byte comparisons, retry and deferred
requests, non-blocking probes, concurrent compiles, eviction and a synthetic
console mesh shader compiled with the local DXC and GPU path. Four added tests
cover configurable budgets, sharing oversized results, collision/error handling
and releasing the transient result after callers finish.

The prebuilt test binary's exact
`gpu::recomp::pso::shader_cache_tests::shader_cache_coalesces_oversized_compiles`
test was additionally invoked 100 times: all 100 invocations passed. Each
invocation tests both zero retention and a one-byte budget, with eight callers.

```text
cargo build --release --workspace
Exit code 0; optimized workspace build succeeded.
xcl.exe --version
xcl 0.1.0
xcl.exe --help
Exit code 0; CLI help displayed.
```

The build emitted two pre-existing `Atomic::fetch_update` deprecation warnings,
in `hle/d3d12x/real.rs` and `hle/xgameruntime/mod.rs`. Those unrelated files were
not changed. A local release executable was produced as `bin/xcl.exe` in the
original working directory; executable binaries are not tracked in this source
repository. Source, license and notices are preserved; DXC, console libraries
and game data are not bundled.

## Synthetic console-title execution

The same local release launcher inspected and executed XCL's own synthetic title:

```text
xcl.exe inspect <temporary-testgame>/xcl_testtitle.exe --json
Inspect exit code: 0
xcl.exe run <temporary-testgame> -- --hello
50 checks passed; 0 failed; title completed; exit code 0
```

Settings: `XCL_DATA_ROOT` pointed to an isolated validation directory,
`XCL_DUMP_SHADERS=1`, `XCL_SHADER_CACHE_MB=256`, `XCL_DRED=1`
and `XCL_RUN_SECONDS=30`. An external 60-second timeout was configured.

The runtime reported no crash, the real RTX 4060 D3D12 backend, one `PresentX`
call and one presented frame. The rendered triangle was a separate readback check;
the displayed frame was a cleared 640 x 360 back buffer, not gameplay. Two shader
pipelines used fallbacks; `translated_pipelines` was 0. Gaming Services was missing.
SHA-256 snapshots confirmed the test title's input files were unchanged.

This is a direct fixture execution, not the complete workspace test suite, not
GTA VI and not a retail-title regression comparison. Raw stdout, stderr,
inspection JSON, runtime report and input-file snapshots remain local.

## Not tested

- GTA 6, any retail title or title-specific console DLL/API compatibility.
- Correct game rendering, gameplay, stability or frame rates.
- The complete workspace test suite: only the focused 20-test set was run.
- A retail-title before/after smoke run.
- AMD GPUs, other NVIDIA GPUs or other driver versions.

The original `xcl-main.zip` was not modified. Its SHA-256 is:

```text
6f5878d5420e638fbf501287af526eb2f80d09dc6babc4e8319cfcb52e42df46
```

Raw command output is retained locally in the original working directory under
`validation/` for the baseline failure, focused passing tests, release build and
synthetic-title execution. Logs and runtime dumps are not published here.
