# Changelog

Versions are per-repo; each release is a tag with a GitHub Release and the
per-OS zips attached to it. See [PORTS.md](PORTS.md) for the per-platform build
and packaging notes. Everything here is early: the first level of each mode
has been played, and nobody has finished a level yet.

## Unreleased

- **Building from source now works for everyone.** The SDK fork this repo pins
  as a submodule (`springah/rexglue-sdk`) was private, so a fresh
  `git clone --recurse-submodules` could not fetch it. It has been republished
  as a public repository and `main` is re-pinned to it: the same SDK code, under
  new commit hashes (the hashes quoted in this file and in PORTS.md are the new
  ones). The tags v0.1.1 to v0.1.3 still pin the old hashes and cannot be built
  from source; build from `main`, or use the release zips.

## v0.1.3 — 2026-11-20 (Windows, pre-release)

The Achievements menu entry works. This is the build to test with a
controller.

- **The Achievements menu entry now opens the achievements list.** The game
  asks the system for its achievements screen (`XamShowAchievementsUI`), which
  the SDK only stubbed. It now opens the SDK's achievements list as a system
  UI: sized for a TV, scrolled with the D-pad or left stick, closed with B, A,
  Start, Back or Esc. The game gets no input while it is open, and the press
  that closes it never reaches the game. Works from the main menu and from the
  pause menu. Tested with the keyboard only (which feeds the same pad state);
  not yet with a real controller. SDK fork commit b07bdd8.
- Keyboard menu presses are no longer replayed into the game after an overlay
  that owned the input is closed.

## v0.1.2 — 2026-11-20 (Windows, pre-release)

First build known to play past the title, with the crash on Return to Arcade
fixed.

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
  close. SDK fork commit 780d6cd. v0.1.1 has the crash; closing the window
  there is clean.
- Found after the release: the same teardown bug also made v0.1.1 crash
  (0xC0000374) after the "Entrypoint XEX not found" box, which is what a first
  run without game files shows. v0.1.2 exits cleanly there (exit code 1).
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
  `present_vsync = false` in the F4 cvar list. SDK fork commits 0d8924f and
  b6f3513; same fix as springah/futari_recomp#5.
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
