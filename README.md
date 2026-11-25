# gtaVI

**Experimental XCL-based compatibility research, with GTA VI as a development target.**

> **GTA VI has not been run or verified with this project.** There is no promise that
> it will launch, render correctly, become playable, or reach any particular frame
> rate. The successful demonstration is XCL's own synthetic test title, not GTA VI.

This is a derivative development snapshot of [XCL](https://github.com/xcl-dev/xcl),
an Xbox Series X|S / GDK to Windows compatibility layer. It runs x64 game code
natively and implements console APIs; it is not a full Xbox hardware emulator.
The project name describes the research target, not established game support.
It is not affiliated with Rockstar Games or Take-Two Interactive.

## What was actually tested

On Windows 11 with a Ryzen 5 5500 and NVIDIA RTX 4060:

- XCL's synthetic console title finished with **50 checks passed, 0 failed** and
  exit code **0**, using the real Direct3D 12 backend.
- Checks covered the loader, console APIs, memory, virtual filesystem, input,
  storage, audio, GPU rendering/readback and presentation.
- The presentation check displayed one cleared 640 x 360 frame. It is not gameplay.
- Two console shader pipelines used fallback shaders; no translated console
  pipeline was recorded in that end-to-end run.
- Focused shader-cache tests: **20 passed, 0 failed, 0 ignored**, 1108 filtered out.
  The oversized-shader concurrency regression also passed 100 repeated invocations.
- `cargo build --release --workspace` succeeded.

**Not tested:** GTA VI, any retail title in this local validation, the complete
workspace test suite, or a real-title before/after smoke comparison. Gaming
Services was missing, limiting Xbox service features.

See [VALIDATION.md](VALIDATION.md) for commands, measurements and exclusions.
Upstream claims in [README_XCL.md](README_XCL.md) are upstream documentation,
not independently reproduced results of this derivative project.

## Changes in this snapshot

- Share successful shader compilations between overlapping requests even when a
  shader is too large to retain in the RAM cache.
- Configurable `XCL_SHADER_CACHE_MB` budget: 0-4096 MiB, default 64 MiB.
- Regression tests for budget parsing, concurrency, collision/error handling and
  transient-result lifetime.

These are general XCL improvements, not GTA VI-specific compatibility fixes.

## Build and open

Windows 10/11 x64, Rust MSVC, Windows SDK/DXC and a compatible Direct3D 12 GPU
are required. Refer to [README_XCL.md](README_XCL.md) for the full requirements.

```powershell
cargo build --release --workspace
.\target\release\xcl.exe
```

Opening `xcl.exe` from Explorer shows XCL's game-selection menu. Only use unpacked
Xbox Series X|S / GDK files you lawfully own. There are **no GTA VI files, console
SDK components, keys, DRM bypasses or downloads** in this repository. It cannot
run a game that has not been supplied.

For inspection and bounded diagnostics, see [GTA6_NOTES.md](GTA6_NOTES.md).

## License and attribution

**GPL-2.0-only**, inherited from XCL. Original authorship, license and third-party
notices are preserved in [LICENSE](LICENSE), [THIRD_PARTY.md](THIRD_PARTY.md) and
throughout the source. The native launcher remains named `xcl.exe`.

AI assistance: an OpenAI assistant through pi prepared the cache changes, tests,
validation and publication documentation. The exact model identifier was not
supplied in the session; no model name is inferred.
