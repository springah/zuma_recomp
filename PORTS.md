# Zuma — Ports

Per-OS build and release notes. Build prerequisites and the Windows walkthrough
live in the [README](README.md).

## Platforms

| OS | Build | Status |
|----|-------|--------|
| Windows (x64) | `build.ps1` (one shot: tools, SDK CLI, extract, regen, build), or by hand the `win-amd64-release` preset (clang++ GNU driver, MSVC ABI) → `out/build/win-amd64-release` | 🚧 v0.1.2: plays past the title — Adventure level 1 and Gauntlet to Game Over, menus and options work; finishing a level and a controller are untested (SDK fork v0.9.2 + 5 commits) |
| Steam Deck / Linux (x64) | `linux-amd64-release` preset | ⬜ untried |
| macOS (arm64) | `mac-arm64-release` preset on an Apple Silicon Mac | ⬜ untried |

The presets are shared with the sibling ports (daytona et al.): `win-amd64-*`
name `clang`/`clang++` with `-march=x86-64-v3` and target
`x86_64-pc-windows-msvc`, so configure from a VS developer shell. The `*-arm64`
presets are untested.

## SDK pin

All builds link the `thirdparty/rexglue-sdk` submodule, pinned at **v0.9.2 + 5**
(branch `zuma/fpscr-host-threads`, `3fe4824`) of
[springah/rexglue-sdk](https://github.com/springah/rexglue-sdk): daytona's v0.9.2
plus

- `4e4f4bb` — host-side threads got a zeroed FPSCR shadow, so the first guest
  callback on the audio worker unmasked every FP exception and died with
  0xC000008F;
- `185dff8`, `a8e56b9` — the `present_vsync` cvar (host presentation honours
  VSync);
- `d2e35ae` — clean exit when the title quits itself (Return to Arcade used to
  crash with 0xC0000374, or hang, in the SDK's teardown);
- `3fe4824` — CI only: the fork's format check pinned to `ubuntu-24.04`.

Upstream has since tagged v0.10.0; moving
to it is an open item now that the port boots on the known-good pin. The manifest's
`sdk_version` must match the pinned tag.
