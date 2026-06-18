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
3. **Verify behavior headless** (this Windows box has Edge, no Chrome/gh CLI):
   `"/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe" --headless=new --disable-gpu
   --no-first-run --user-data-dir=_pTMP --window-size=W,H --virtual-time-budget=MS
   --screenshot=out.png file:///.../page.html` — inject a `<script>` before `</body>` that drives the
   game (set `SAVE`, call `launchMission(0)` / `openSquad(...)` / etc.), then Read the PNG. For
   assertions use `document.title='RESULT:'+JSON.stringify(r)` + `--dump-dom` and grep the title.
   Temp files use a `_` prefix (gitignored); clean up `_p*` profiles after (kill stray `msedge` if a
   profile folder is "busy"). On another OS, use Chrome/Chromium with the same flags.
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
  `glow/noGlow/poly/spray/ring/shatter`. Static scenery baked to `bgCanvas` via `rebuildBg()`.
- **Maps/lanes**: `m.paths` → `LANES[{PATH,segs,len,portal,core}]`; `pointAt(d,lane)`; convenience
  globals `PORTAL`/`CORE` (lane 0).
- **Screens**: DOM overlays toggled with the `.hidden` class (title, campaign, qpOverlay, dailyOverlay,
  squadOverlay, shopOverlay, codexOverlay, settings, pause, end). Canvas `#game` + HUD + `#dock`
  (hero cards). `$=id=>getElementById`.

## Key systems (where to look)
- **Heroes / placement**: `TOWERS`; `placeTower` guarded by `deployBlocked` (tower cap `towerCap()` 8→16,
  bastion `solo:true` one-per-match, daily roster caps). REACTOR is a HERO (`isStructure` = footprint
  only); BASTION is the only structure.
- **Line-up** (Quick Play + daily pool/handicap; NOT campaign): `openSquad` →
  `renderSquadZones`/`renderSquadDock`; free drag-and-drop via `sqGrab` + window pointer handlers +
  `sqDropIndex` (floating `.sqGhost`, `#sqInsert` bar). `SQUAD_CAP=6`. `applySquad` filters/orders the
  in-game dock to the saved squad. Campaign shows the full unlocked arsenal (no line-up).
- **Tutorial** (Mission 1 / OUTPOST only): coach intro `tutIntro`/`TUT_INTRO`/`drawTutIntro` (dim +
  spotlight + NEXT) → gated lessons `tutStep`/`TUT_ACTIONS`/`tutDid`/`updateTut` (drag→upgrade→place→
  sell→ability), wave locked until done.
- **Daily**: `todayDaily`/`genDailyRule` seeds a rule (SOLO/STRIKE/POOL/HANDICAP) via
  `mulberry32(todayKey())`; runs CASUAL, 5 waves; `#dailyOverlay` = condition (left) + battlefield (right).
- **Difficulty**: `DIFFS` = [CASUAL, NORMAL, BRUTAL, EXPERT]. `QP_DIFF=0` → Quick Play + daily are CASUAL
  (the squad/tower caps are the challenge). Campaign missions set their own `diff`; BRUTAL is act-3 only.
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

---
Keep this file current when conventions or architecture change — it's the cross-machine source of truth.
Detailed version history is in `git log`.
