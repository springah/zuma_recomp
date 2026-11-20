# Changelog

Versions are per-repo; each release is a tag with a GitHub Release and the
per-OS zips attached to it. See [PORTS.md](PORTS.md) for the per-platform build
and packaging notes. Everything here is early: the first level of each mode
has been played, and nobody has finished a level yet.

## Unreleased

On `main`, not in a release zip yet.

- **First play past the title (2026-11-20, Windows, keyboard).** Adventure
  works through stage select, the instruction pages and the first level
  (aiming, shooting, matches, score, losing a life); Gauntlet plays through to
  its Game Over screen; the main menu, pause menu, Options (both volume
  sliders, saved across runs) and Help work. Not tested: finishing a level,
  anything after the first level, a real controller.
- **Return to Arcade no longer crashes on the way out.** Quitting from the main
  menu logged "Execution complete" and then died with heap corruption
  (0xC0000374): the SDK destroyed the window before the input drivers that
  still pointed at it. With that order fixed the exit could hang instead,
  because a guest thread had gone away holding a host lock. The SDK now exits
  the process as soon as the title has ended, as it already did for a window
  close. SDK fork commit d2e35ae. v0.1.1 still has the crash; closing the
  window there is clean.
- Known gaps: the Achievements menu entry does nothing (the game calls
  `XamShowAchievementsUI`, which the SDK only stubs), and Leaderboards shows
  the game's own "Xbox Live members only" message.
- `RUN.txt` now lists the keyboard defaults and the tested state.

## v0.1.1 — 2026-11-02 (Windows, pre-release)

First bundle on GitHub Releases. Same bring-up state as v0.1.0, with the host
VSync fix carried over from the sibling ports.

- **VSync now actually syncs the window to the monitor.** `vsync = true` paced
  the game at 60 but never reached the host swap chain: the SDK presenter
  always presented immediately with tearing allowed, which on a fixed 60Hz
  monitor shows as a tear line and, while an overlay is drawn, as roughly twice
  the refresh rate in presents per second. A new SDK cvar `present_vsync`
  (default on) makes Direct3D 12 present on the vertical blank without the
  tearing flag; the `vsync` line in `zuma.ini` and the F4 VSync checkbox set
  both the guest pacing and the host present. Variable refresh rate users can
  keep the pacing but restore immediate presentation with
  `present_vsync = false` in the F4 cvar list. SDK fork commits 185dff8 and
  a8e56b9; same fix as springah/futari_recomp#5.
- The release zip ships `RUN.txt`, `extract_game.bat` + `extract_game.py` and
  the third-party licence bundle; no game data.

## v0.1.0 — 2026-10-31 (private family-and-friends build)

- First build to leave the project. Scaffolded on the ReXGlue SDK (fork branch
  `zuma/fpscr-host-threads`, v0.9.2 + 1), boots and renders the title screen
  ("PRESS A TO START") with the Xenos D3D12 renderer.
- Two recompilation bugs fixed on the way: host-side threads started with all
  FP exceptions unmasked (first guest callback died with 0xC000008F; the SDK
  fork seeds the FPSCR shadow from the host), and libjpeg's `error_exit`
  `longjmp` into the loader's `setjmp` restored registers from the wrong frame
  (now routed through the SDK's host `jmp_buf` via `setjmp_address` /
  `longjmp_address` hints).
