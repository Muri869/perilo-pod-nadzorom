# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

"PERILO POD NADZOROM" — a comic single-file browser game (laundry sorting) written
in vanilla JS + Canvas 2D. The entire game is `index.html` (~740 lines: HTML + CSS +
JS inline). No dependencies, no build step, no test suite. UI language is Slovenian.

There are **no image assets**: every character, laundry piece, and icon is an emoji
drawn with `fillText`, so on-screen appearance depends on the host platform's emoji
font. When changing visuals you edit emoji glyphs and Canvas geometry, not files.

## Run / deploy

- **Run locally:** open `index.html` directly in a browser (double-click, or
  `open index.html`). There is nothing to build, install, lint, or test.
- **Deploy:** the repo is served via GitHub Pages from `main` branch root at
  https://muri869.github.io/perilo-pod-nadzorom/ — pushing to `main` redeploys.
  After a push, the Pages build takes ~30–60s; status is `built` when live
  (`gh api repos/Muri869/perilo-pod-nadzorom/pages/builds/latest --jq '.status'`).

## Git workflow (required)

After every change: `git add` → commit with a clear message → `git push`. The repo
is the saved source of truth for reverting. The repo has no global git identity, so
commits use inline flags:
`git -c user.email="aljosa.vodan@gmail.com" -c user.name="Aljosa Vodan" commit -m "..."`

Tag each completed, self-contained milestone and push the tag
(`git tag -a vX.Y -m "..."` then `git push origin vX.Y`); bump the minor each time so
there are clear revert points. Current baseline is `v1.0`.

## Architecture

`index.html` is organized into labeled modules (search the big comment banners):

- **STANJE** (~line 62) — all game state in one `state` object (`screen`:
  `intro`/`play`/`over`, score, level, patience, combo, shake/flash timers) plus
  `sel` (chosen boss/man names), `player`, and the `pieces`/`particles` arrays. Game
  data lives here too: `KAT` (the three categories + their colors/glow), `BASKETS`
  (basket order for keys 1/2/3), `PERILO` (laundry emoji→category templates), `LINES`
  (Slovenian quips by event), and `UI` (logical-coordinate rects for every clickable
  button across all screens).
- **ZVOK** (~line 153) — procedural Web Audio; lazily creates the `AudioContext` on
  first input via `ensureAudio()`. All SFX are synthesized from `tone()`/`noiseBurst()`.
- **VNOS** (~line 216) — keyboard + unified `pointer` events (mouse and touch share
  one path). `onPointerDown` hit-tests `UI` rects per `state.screen`. Movement is
  tracked as separate keyboard/touch booleans combined by `goLeft()`/`goRight()`.
- **IGRA** (~line 283) — `update(dt)` is the simulation: player movement, spawning,
  falling physics, and catch resolution. The core rule lives in `resolveCatch()`: a
  catch counts only when the player overlaps the piece **and** the open basket matches
  the piece category; the red-sock trap triggers `catastrophe()` only when dropped into
  BELO. `startGame()` resets all state.
- **IZRIS** (~line 449) — one `drawIntro`/`drawPlay`/`drawOver` per screen, all using
  emoji via `fillText`. Helpers: `roundRect`, `text`, `strokedText`, `emoji`, `wrap`.
- **ZANKA** (~line 727) — single `requestAnimationFrame` loop with delta-time clamped
  to 0.05s; calls `update(dt)` then `render()`.

### Coordinate system (important when editing layout or input)

Everything is authored in a fixed **logical space `LW=480 × LH=800`**. `render()`
applies a transform that scales/centers this onto the real canvas and accounts for
`devicePixelRatio`; `resize()` recomputes `view.scale/ox/oy`. Any new clickable element
must be defined in logical coords and `toLogical()` maps pointer events back into that
space — never mix CSS pixels with game coords. Gameplay tuning constants (`PLAYER_Y`,
`CATCH_TOP/BOT/HALF`, `SCORE_PER_LEVEL`, `WIN_THRESHOLD`) are grouped near the top of
STANJE.
