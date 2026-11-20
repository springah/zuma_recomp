# Zuma — Ports

Per-OS build and release notes. Build prerequisites and the Windows walkthrough
live in the [README](README.md).

## Platforms

| OS | Build | Status |
|----|-------|--------|
| Windows (x64) | `build.ps1` (one shot: tools, SDK CLI, extract, regen, build), or by hand the `win-amd64-release` preset (clang++ GNU driver, MSVC ABI) → `out/build/win-amd64-release` | 🚧 v0.1.4: plays past the title — Adventure level 1 and Gauntlet to Game Over, menus, options and the Achievements list work; finishing a level and a controller are untested (SDK fork v0.9.2 + 7 commits) |
| Steam Deck / Linux (x64) | `linux-amd64-release` preset | ⬜ untried |
| macOS (arm64) | `mac-arm64-release` preset on an Apple Silicon Mac | ⬜ untried |

The presets are shared with the sibling ports (daytona et al.): `win-amd64-*`
name `clang`/`clang++` with `-march=x86-64-v3` and target
`x86_64-pc-windows-msvc`, so configure from a VS developer shell. The `*-arm64`
presets are untested.

## SDK pin

All builds link the `thirdparty/rexglue-sdk` submodule, pinned at **v0.9.2 + 7**
(branch `zuma/fpscr-host-threads`, `771522d`) of
[springah/rexglue-sdk](https://github.com/springah/rexglue-sdk): daytona's v0.9.2
plus

- `bd718f4` — host-side threads got a zeroed FPSCR shadow, so the first guest
  callback on the audio worker unmasked every FP exception and died with
  0xC000008F;
- `0d8924f`, `b6f3513` — the `present_vsync` cvar (host presentation honours
  VSync);
- `780d6cd` — clean exit when the title quits itself (Return to Arcade used to
  crash with 0xC0000374, or hang, in the SDK's teardown);
- `c214471` — CI only: the fork's format check pinned to `ubuntu-24.04`;
- `b07bdd8` — `XamShowAchievementsUI` opens the achievements list (was a
  stub), with pad scrolling and closing;
- `771522d` — a launch that fails (no game files) ends the process directly;
  the SDK's teardown hung about one time in ten there.

Upstream is now at v0.11.0. Moving to it is deferred: this fork line is 150+
commits past upstream v0.9.0, a trial merge with upstream v0.10.1 conflicts in
about 30 files (build system, both input drivers, the app class), and upstream's
code generator changed, so the game would have to be regenerated and re-tested.
The manifest's `sdk_version` must match the pinned tag.
