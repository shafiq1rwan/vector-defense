# ⬡ VECTOR DEFENSE

A neon vector-style tower defense game in a **single HTML file** — no build step, no dependencies. Open `index.html` and play.

## Features

- **6 hero towers** — SLY (archer), GLACE (frost), BRUNO (cannon), VOLTA (tesla), REX (sniper), LUMA (support aura)
- **10 enemy types** with damage-type resistances, including splitters, shield-bearers, phasers, and the OVERMIND boss with energy shield, minion spawns, and EMP pulses
- **Tower specializations** — every hero branches into one of two upgrade paths at max level
- **Active abilities** — Orbital Strike, Frost Nova, Overdrive (Q/W/E)
- **3 maps × 3 difficulties**, wave modifiers (fast / regen / horde / wealth), combo credit system
- **Meta-progression** — earn ⬡ cores every run and buy permanent upgrades in the shop
- **Tower veterancy** — towers earn XP per kill and gain ★ ranks with passive bonuses
- **Procedural music** — WebAudio synth score that intensifies with combat and boss waves
- **DPS scoreboard**, targeting modes, medals, persistent high scores (localStorage)
- **Mobile-friendly** — responsive layout, drag-to-place touch controls, auto-landscape in portrait
- **Installable PWA** — "Add to Home Screen" for fullscreen offline play

## Controls

| Key | Action |
|-----|--------|
| 1–6 | Select hero |
| Q/W/E | Abilities |
| Space | Start wave (early call = bonus credits) |
| F | Game speed 1×/2×/3× |
| T | Cycle targeting mode of selected tower |
| P | Pause / settings |
| Shift+click | Place multiple towers |

Built with HTML5 Canvas + WebAudio.
