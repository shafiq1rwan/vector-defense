# VECTOR DEFENSE — project guide for Claude

Portable project context so any machine's Claude has the essentials (this file travels with git;
the per-user `.claude/` memory does not).

## What this is
A single-file HTML5 canvas tower-defense PWA. The whole game lives in **`index.html`** (~5.7k lines:
inline `<style>` + one big `<script>`). Zero build, zero dependencies. Other files: `sw.js`,
`manifest.webmanifest`, `icon-*.png`, `README.md`. Live on GitHub Pages
(https://shafiq1rwan.github.io/vector-defense/) — pushing to `main` auto-deploys in ~30s. Repo:
`shafiq1rwan/vector-defense` (local folder is named `vector-claude`).

## Hard constraints — never violate
- **No CSS gradients** — the user's display dithers them into rainbow banding. Flat colors only.
- **No emoji** in UI — use the inline SVG icon set (`ICONS` / `ic(name,size)`). Plain glyphs (✓ ▶ · ×) are fine.
- **Credit is "Saiss"** — never put "shafiq" in any UI/credit/public file.
- **Mobile-first** — the user tests on a phone. Every screen must fit BOTH desktop and mobile with no
  off-screen content. Big touch targets; use pointer events (not HTML5 drag); put `touch-action:none`
  on any draggable element that sits inside a scrollable container.
- **Commits** — end the message with `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`. Prefer a
  Bash heredoc; if you must use PowerShell here-strings, put NO double quotes in the message.

## Workflow for every change
1. Edit `index.html` (bump `const VERSION` near the top — humble minor bumps, e.g. 1.13.x).
2. **Syntax check**: extract the `<script>` and run `node --check` (catches the easy breaks).
3. **Verify behavior.** Edge 149 BROKE the old screenshot workflow on this box: `--headless=new` now
   ignores `--screenshot` AND `--dump-dom` (both return empty), and `--headless=old` `--screenshot`
   crashes silently on ANY page containing a `<canvas>` (no PNG, regardless of GPU flags) — so the whole
   game can no longer be screenshotted headless here. `--virtual-time-budget` also breaks old-headless
   screenshots. **Working verification path = a Node VM harness** (no browser): extract the `<script>`,
   `vm.runInContext(script + testCode, sandbox)` with a Proxy-based DOM/canvas/localStorage/AudioContext
   stub (every element method is a chainable no-op; `getContext` returns a stub 2d ctx). Init runs to
   completion under the stubs, then call the real functions (`makeWave(n)`, `acquire(t)`,
   `damageEnemy(e,..,{src})`, `spawnEnemy(...)`+`update(dt)`) and assert on the returns. This exercises the
   actual game code, not a reimplementation. (A tiny non-canvas page still screenshots in `--headless=old`
   if you ever need pixels; on another OS use Chrome/Chromium with the classic flags.) Temp files use a `_`
   prefix (gitignored); clean up `_p*` profiles after (kill stray `msedge` holding a profile lock).
4. Commit + `git push origin main` (auto-deploys). Bump `sw.js` `CACHE` (`vectordef-vN`) when assets change.

## Architecture (all inside index.html `<script>`)
- **State**: scattered top-level globals (`money, lives, wave, towers, enemies, projs, beams, parts,
  zones, occ`; flags `running/paused/gameOver/drafting`). `SAVE` = localStorage object, written by
  `persist()`; `doResetSave()` wipes it.
- **Data tables**: `TOWERS` (heroes + structures), `ENEMIES`, `SPECS`, `ABILITIES`, `DIFFS`, `MAPS`,
  `CAMPAIGN`, `KILLFX`, `THEMES`, `SKINS`, `RANKS`, `MODS`.
- **Loop**: `frame(now)` (rAF, dt-capped, adaptive perf governor `perfLite/perfThrottle`) → `update(dt)`
  + `draw()`. `reset()` initializes a match.
- **Render**: `draw()` (large), `drawTowerBody` (per-type), `drawEnemy`/`drawEnemyShape`; helpers
  `glow/noGlow/poly/spray/ring/shatter`. Static scenery baked to `bgCanvas` via `rebuildBg()`. **DPR crisp**:
  renders at `RS=min(devicePixelRatio,2)` supersample — `cvs`/`bgCanvas` backing is W*RS×H*RS, `draw()` and
  `rebuildBg()` start with `ctx.setTransform(RS,...)`, the bg blit is `drawImage(bgCanvas,0,0,W,H)`. All draw
  code stays in logical W,H coords; `setMouse` maps input in logical coords (unaffected).
- **Maps/lanes**: `m.paths` → `LANES[{PATH,segs,len,portal,core}]`; `pointAt(d,lane)`; convenience
  globals `PORTAL`/`CORE` (lane 0). 5 playable maps + OUTPOST (tutorial, `hidden`); `mapSel` auto-lists
  every non-hidden map (no hardcoded count). Two segment gimmicks, both flagged per-seg in `buildMap`:
  RIFT `warps:[{lane,seg}]` (teleport, no road laid) and CONDUIT `conduit:[{lane,seg}]`.
- **Surge conduit** (v1.18.0, CONDUIT map): a road segment that periodically DISCHARGES. `buildMap`
  fills `surgeBands=[{lane,start,len,a,b}]`; `update()` ticks `conduitT` and, once per `CONDUIT_CYCLE`,
  `damageEnemy(e,28+wave*5,'energy',{quiet:true})` every ground enemy whose `dist` is in a band (flyers
  excepted). `draw()` renders the band charge-ramp + white discharge flash + emitter nodes (beside the
  RIFT-gate block); the `mapSel` + `drawMiniRoad` thumbnails overlay the conduit segment in cyan.
  `reset()` zeroes `conduitT`. Tuning: `CONDUIT_CYCLE/CONDUIT_TELE/CONDUIT_FLASH` + the damage formula.
- **Screens**: DOM overlays toggled with the `.hidden` class (title, campaign, qpOverlay, dailyOverlay,
  squadOverlay, shopOverlay, codexOverlay, settings, pause, end). Canvas `#game` + HUD + `#dock`
  (hero cards). `$=id=>getElementById`.

## Key systems (where to look)
- **Heroes / placement**: `TOWERS` = 9 attack/support heroes + REACTOR (economy). `placeTower` guarded by
  `deployBlocked` (tower cap `towerCap()` 8→16, daily roster caps). All `TOWERS` are single-cell now —
  BASTION + the whole `structure` category (footprint/solo/`isStructure`/`#structs`) were removed in
  v1.16.0 (it was a redundant BRUNO+REX hybrid). Generic `def.footprint?…` ternaries remain inert in a few
  draw/UI sites (default to the 1×1 branch); a future structure could reuse them.
- **FLYER / anti-air** (v1.15.0): `ENEMIES.flyer` (`air:true, antiAirOnly:true, shape:'wing'`) crosses the
  map in a STRAIGHT line gate→core (ignores the road & rift warps) — the movement fork is in `update()` at
  the per-frame position set (keeps `e.dist+=sp*dt` + the leak check; only forks x/y to a portal→core lerp).
  Only anti-air heroes hit it: `TOWERS.archer/sniper/tesla` carry `antiAir:true`. Two gates enforce it —
  `acquire()` (`if(e.def.antiAirOnly && !t.def.antiAir) continue;`) and the `damageEnemy()` chokepoint
  (`if(e.def.antiAirOnly && o.src && !o.src.def.antiAir) return;`, so splash/chain from ground towers can't
  clip it but player abilities — no `o.src` — still can). `drawEnemy` adds a faint ground-shadow for `air`.
  `makeWave`: `if(n>=6) add('flyer', Math.floor(n/6), 1.8)`.
- **Share card** (v1.15.0): `#shareBtn` on `#endOverlay` → `shareResult()` renders an offscreen 640×360 ×RS
  canvas (wordmark + VICTORY/DEFEAT/DAILY + WAVE/KILLS/COMBO + diff·map + inline rank emblem + footer URL)
  and exports via `navigator.canShare({files})`/`navigator.share` on mobile, else `downloadCard()` anchor.
  `recordEnd()` stashes `lastWon` so the card knows the result.
- **Line-up** (Quick Play + daily pool/handicap; NOT campaign): `openSquad` →
  `renderSquadZones`/`renderSquadDock`; free drag-and-drop via `sqGrab` + window pointer handlers +
  `sqDropIndex` (floating `.sqGhost`, `#sqInsert` bar). `SQUAD_CAP=6`. `applySquad` filters/orders the
  in-game dock to the saved squad. Campaign shows the full unlocked arsenal (no line-up).
- **Tutorial** (Mission 1 / OUTPOST only): `drawTutCoach()` renders BOTH the coach intro
  (`tutIntro`/`TUT_INTRO`, tap-anywhere NEXT) AND the gated lessons (`tutStep`/`TUT_ACTIONS`/`tutDid`)
  in the same dim+spotlight+panel style — lessons spotlight a canvas target via `tutLessonTarget()` (build
  cell / your tower) with a SKIP button; the dock/ability buttons sit outside the canvas so they stay bright.
  `updateTut` only gates the wave now (the old DOM `#tutBanner` is unused). Wave locked until done.
  **Onboarding outside Mission 1** (v1.17.0): a player's FIRST match in ANY mode (Quick Play/daily/non-M1
  campaign) runs the 3-card goal intro only (`reset()` sets `tutIntro=0` when `!tut1 && !SAVE.seenCoach`;
  `tutStep` stays 99 = no forced lessons). `advanceIntro` sets `SAVE.seenCoach` on completion so it shows
  once ever. There's also a first-launch text `#howOverlay` (`SAVE.seenIntro`) and a wave-0 bottom hint in
  `draw()` ("DRAG A HERO CARD…"/"PRESS SPACE…", suppressed while the coach intro is up).
- **Daily**: `todayDaily`/`genDailyRule` seeds a rule (SOLO/STRIKE/POOL/HANDICAP) via
  `mulberry32(todayKey())`; runs CASUAL, 5 waves; `#dailyOverlay` = condition (left) + battlefield (right).
- **Difficulty**: `DIFFS` = [CASUAL, NORMAL, BRUTAL, EXPERT] (all `money:100` now). `QP_DIFF=2` → Quick Play
  runs BRUTAL; daily still runs CASUAL. **Campaign HP ramps smoothly** per mission via `makeWave` (`hpBase=0.9+campNode.id*0.1`
  → mission 1≈0.9× … mission 12≈2.0×), NOT the old 3 flat tiers; each mission's `diff` still sets
  lives/reward. Tune the ramp by editing that `hpBase` formula.
- **Progression/cosmetics**: Commander rank (`SAVE.commander`, `RANKS`, `#rankCard`); cores shop (tabbed,
  `buildShop`/`shopTab`), Collection codex tab, `equipCosmetic`, `killBurst` reads
  `SAVE.cosmetics.killfx`.
- **Audio**: `ac()` unlocks the AudioContext on first gesture (mobile suspends it); `musicTick` scheduler;
  `SND.*`. If "no music" but SFX work, it's the Music toggle/volume, not a bug.

## Common gotcha
`el.classList.toggle('x', cond)` — make `cond` a strict boolean. JS `&&`/`||` can yield `undefined`, and
`toggle(name, undefined)` FLIPS the class every call (caused the "dim heroes" and "flickering dock" bugs).

## itch.io
Upload `press/vector-defense-itch.zip` (regen: zip index.html + manifest + sw.js + icons, **index.html at
root**), tick "play in browser". Cover `press/cover.png` (630×500). Embed: 1280×720, fullscreen on,
mobile-friendly on, orientation Default (the game self-rotates portrait→landscape).

## Current focus / backlog (as of v1.18.0 — open/optional, not committed work)
- **v1.18.0 shipped**: new CONDUIT map (Surge Conduit gimmick — a road band that periodically zaps ground
  enemies on it). 5 playable maps now. Regenerate `press/vector-defense-itch.zip` before the next upload.
- **v1.17.0 shipped**: first-match goal coach-marks in any mode (`SAVE.seenCoach`).
- **v1.16.1 shipped**: fixed mouse drag-to-deploy placing a tile high (touch-only `lift` was applied to mouse).
- **Open tester note**: BALANCE pass still deferred (early difficulty/economy curve; QP runs BRUTAL).
- **v1.16.0 shipped**: removed BASTION + the whole structure category (felt OP/redundant). Mission 8
  (VOID SPIRAL) now unlocks WARDEN instead, and WARDEN lost its commander-level-8 gate. Roster is now 9
  heroes + REACTOR.
- **v1.15.0 shipped**: FLYER enemy (anti-air) + shareable result card. The `press/vector-defense-itch.zip`
  on disk is stale (pre-1.15) — regenerate before the next itch re-upload.
- **itch.io launch**: press assets are ready (`press/` screenshots + `cover.png`; `vector-defense-itch.zip`
  on disk, not committed). User is setting up the page. Marketing not done — offered: a Show HN post,
  r/playmygame + r/WebGames posts, and a short gameplay clip/GIF.
- **No-SW itch build** (offered, not built): a copy of `index.html` with the service-worker registration
  stripped, for the itch zip — avoids any stale-cache weirdness inside itch's sandboxed iframe. Make this
  if a re-upload ever shows an old version.
- **Balance (playtest-driven, all one-number tweaks)**: Quick Play + daily run CASUAL on purpose (the squad
  cap + tower cap are the challenge). Campaign act-3 = BRUTAL with eased economy (money 150, reward .95).
  Watch: `SQUAD_CAP=6` may feel tight now that REACTOR competes for a slot (consider 7); SOLO daily on CASUAL.
- **Known minor**: the line-up floating drag ghost is a DOM `.sqGhost` element, so on portrait phones (where
  `#wrap` is CSS-rotated 90°) it renders upright instead of rotated — drops still land correctly. Could swap
  to a canvas-drawn ghost (like the in-game placement ghost) if it bothers the user.
- **DESIGN.md / tuning tables** (offered, not built): a one-stop balance/tuning reference. CLAUDE.md covers
  most of it for now.

---
Keep this file current when conventions or architecture change — it's the cross-machine source of truth.
Detailed version history is in `git log`.
