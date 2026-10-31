# Zuma — Ports

Per-OS build and release notes. Build prerequisites and the Windows walkthrough
live in the [README](README.md).

## Platforms

| OS | Build | Status |
|----|-------|--------|
| Windows (x64) | `build.ps1` (one shot: tools, SDK CLI, extract, regen, build), or by hand the `win-amd64-release` preset (clang++ GNU driver, MSVC ABI) → `out/build/win-amd64-release` | 🚧 scaffolded, not yet booted (SDK v0.9.2) |
| Steam Deck / Linux (x64) | `linux-amd64-release` preset | ⬜ untried |
| macOS (arm64) | `mac-arm64-release` preset on an Apple Silicon Mac | ⬜ untried |

The presets are shared with the sibling ports (daytona et al.): `win-amd64-*`
name `clang`/`clang++` with `-march=x86-64-v3` and target
`x86_64-pc-windows-msvc`, so configure from a VS developer shell. The `*-arm64`
presets are untested.

## SDK pin

All builds link the `thirdparty/rexglue-sdk` submodule, pinned at **v0.9.2 + 1**
(branch `zuma/fpscr-host-threads`, `4e4f4bb`) of
[springah/rexglue-sdk](https://github.com/springah/rexglue-sdk): daytona's v0.9.2
plus one fix (host-side threads got a zeroed FPSCR shadow, so the first guest
callback on the audio worker unmasked every FP exception and died with
0xC000008F). Upstream has since tagged v0.10.0; moving
to it is an open item once the port boots on the known-good pin. The manifest's
`sdk_version` must match the pinned tag.
