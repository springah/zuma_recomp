# Zuma ReXGlue Recompilation

> ⚠️ **Disclaimer — built with heavy use of AI.** The recompilation, port code,
> tooling, and documentation in this repo were developed largely with the assistance
> of AI coding tools (large language models). It's a hobby project: verify things
> yourself before relying on them, and expect rough edges. No warranty.

A static recompilation of **Zuma** (PopCap / Oberon Media, Xbox 360 XBLA, 2005)
for native PC execution, built on the [ReXGlue SDK](https://github.com/rexglue/rexglue-sdk)
(a Xenia-derived runtime + PPC→C++ recompiler).

**Status: bring-up.** The repository is scaffolded (manifest, build scripts,
pinned SDK) and the shell `default.xex` recompiles; the port has not been booted
yet. Nothing here is playable until the first boot is banked.

No copyrighted game files are included. This tree excludes the extracted game
directory, `default.xex`, images and build outputs — you supply your own legally
obtained Zuma XBLA package. See [Legal](#legal) below.

This is one of a family of Xbox 360 recomps built on the ReXGlue SDK; zuma pins a
tagged SDK release as a submodule, like its sibling ports (daytona, futari, gh,
guwange, sdoj, sotn). Per-OS build and packaging notes are in [PORTS.md](PORTS.md).

## Prerequisites

| Tool | Version | Notes |
|------|---------|-------|
| CMake | 3.25+ | |
| Ninja | any | `winget install Ninja-build.Ninja` |
| LLVM / clang | 20+ | Standalone [LLVM](https://github.com/llvm/llvm-project/releases); zuma builds with the **clang++** (GNU) driver targeting the MSVC ABI |
| Python | 3 | runs `scripts/extract_game.py` |
| Git + Bash | any | `scripts/regen.sh` is a bash script |

**Windows:** the clang++ build targets `x86_64-pc-windows-msvc`, so it needs the
MSVC headers/libs + Windows SDK on `INCLUDE`/`LIB` (a "x64 Native Tools" command
prompt sets these, or let `build.ps1` enter one for you).

**Linux:** Clang 20+, GTK3 dev headers (`libgtk-3-dev` / `gtk3`), and X11/XCB
interop (`libx11-xcb-dev` / `libx11` / `libX11-devel`). The SDK renders via XCB;
on Wayland, XWayland must be running.

## Game files

Zuma on Xbox 360 is an XBLA title delivered as a single LIVE/STFS content
package. On your console's storage it sits at
`Content\0000000000000000\584107EF\000D0000\<40 hex digits>` (a file with no
extension whose first four bytes read `LIVE`). Copy that file into `game/` and
extract it from the repository root:

```sh
python3 scripts/extract_game.py
```

This unpacks the package into `extracted/` (the runtime game data, passed via
`--game_data_root`) and copies the shell executable to `assets/default.xex` (the
codegen input). `game/`, `extracted/`, and `assets/` are gitignored. On Windows,
dragging the package onto `extract_game.bat` does the same, except that it
unpacks everything into `assets/` beside the executable (the release-bundle layout).

The package this tree was prepared against: title ID `584107EF`, 27557888 bytes,
sha256 `10026be53823dcf3bf9718850cfa1802a2b1c6430d1847e3c89c74371e1d45b6`
(`default.xex` 4038656 bytes).

## Build

### The short road (Windows): `build.ps1`

```powershell
git clone -c core.symlinks=true --recurse-submodules https://github.com/springah/zuma_recomp
cd zuma_recomp
.\build.ps1 -GameSource "D:\path\to\your\content\package" -Run
```

`build.ps1` (or a double-click on `build.bat`) does steps 1–4 below in one go:
it fetches missing submodules and repairs libmspack's symlinks, installs any
missing tool through winget, enters the x64 developer environment, builds the
ReXGlue CLI, extracts your package into `extracted/` + `assets/default.xex`
(skipped when both already exist), runs `scripts/regen.sh`, and builds the
`win-amd64-release` preset. `-Config Debug` or `RelWithDebInfo` picks another
preset; `-NoInstall` reports missing tools instead of installing them.

### 1. Clone (SDK included)

```sh
git clone -c core.symlinks=true --recurse-submodules https://github.com/springah/zuma_recomp
```

The ReXGlue SDK is a submodule at `thirdparty/rexglue-sdk`, **pinned at a tagged
release** (currently v0.9.2 from the [springah/rexglue-sdk](https://github.com/springah/rexglue-sdk)
fork, the same pin daytona ships with). To develop against a live SDK checkout
instead, set `REXSDK_DIR` at configure time; when unset, the build uses the submodule.

> **Windows:** clone with `-c core.symlinks=true` (as above) or the SDK's
> `libmspack` submodule checks out its symlinks as plain text files and the build
> fails in `lzxd.c`.

### 2. Build the ReXGlue CLI (for codegen)

```sh
cmake --preset win-amd64 -S thirdparty/rexglue-sdk -B thirdparty/rexglue-sdk/out/build/win-amd64
cmake --build thirdparty/rexglue-sdk/out/build/win-amd64 --config Release --target rexglue
# -> thirdparty/rexglue-sdk/out/win-amd64/Release/rexglue.exe (regen.sh finds it
#    automatically, falling back to a ../rexglue-sdk sibling build)
```

### 3. Regenerate the recompiled code

```sh
scripts/regen.sh
```

This runs `rexglue codegen zuma_manifest.toml`, emitting `generated/`
(`rexglue.cmake` + `generated/default/` with the recompiled C++). zuma is a
single-module title, so this is one codegen pass. Function-entry seeds the
analyzer can't discover on its own go in `zuma_functions.toml`. `generated/` is
gitignored.

### 4. Configure & build

```sh
cmake --preset win-amd64-release     # or linux-amd64-release / mac-arm64-release
cmake --build --preset win-amd64-release
out/build/win-amd64-release/zuma.exe --game_data_root=extracted
```

A `zuma.ini` is written next to the executable on first run (display, graphics,
storage and log settings).

## Legal

This repository contains no game data. The recompiled output in `generated/` is
derived from your own copy of `default.xex` and is gitignored. Zuma is a trademark
of PopCap Games / Electronic Arts; this project is not affiliated with or endorsed
by PopCap, EA, Oberon Media or Microsoft. Port code is licensed under the
BSD 3-Clause License (see [LICENSE](LICENSE)).
