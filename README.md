# ⬡ VECTOR DEFENSE

A neon vector-style tower defense game in a **single HTML file** — no build step, no dependencies.

**▶ Play it now: https://shafiq1rwan.github.io/vector-defense/** (works on phone — add it to your home screen for fullscreen offline play)

## Features

- **Story campaign** — 12 missions across 3 acts on a constellation world map, with briefings delivered by LUMA herself — an animated android storyteller portrait that talks as the text types — 1–3 star ratings, star-milestone rewards, and heroes earned by clearing missions, ending in a showdown with the OVERMIND PRIME
- **Quick Play** — mission-prep screen with road-topology previews of all 3 maps (spawn/exit tags like "2 SPAWNS · 1 EXIT") and color-coded threat levels showing the real numbers; endless mode after wave 20
- **Arknights-style stages** — short multi-lane roads (two spawns converging, a fork into two exits, crossing lanes) rendered as flat tiled lanes with a neon rim; hero cards dock at the bottom and deploy by dragging straight onto the grid
- **7 hero towers** — SLY (archer), GLACE (frost), BRUNO (cannon), VOLTA (tesla), REX (sniper), LUMA (support aura with a visible buff glow on boosted towers), JUNO (field mender who repairs damaged towers and revives offline ones), each with two specialization branches at max level, veterancy ranks earned from kills, and an android portrait in the codex, tower popup and unlock screens
- **12 enemy types** with damage-type resistances — splitters, shield-bearers, phasers, the STATIK walking EMP, the BREAKER siege sapper that plants itself and bombards your towers offline (repair them, or let JUNO mend them), and the OVERMIND war golem boss with energy shield, minion spawns and EMP pulses
- **Structures** — multi-cell relics of the old grid: the 2×1 REACTOR (generates credits automatically every 10s — a passive economy engine with a visible payout countdown) and the 2×2 BASTION siege fortress, both reawakened through the campaign
- **Active abilities** — Orbital Strike, Frost Nova, Overdrive (Q/W/E)
- **Relic drafts** — every boss you down lets you pick 1-of-3 run-long relics (damage, range, economy, ability recharge and more); campaign star milestones widen the draft and open every run with a relic cache
- **Elite enemies** — gold-crowned champions with affixes (shielded vanguard, swift, regenerating, fission) and triple bounties, from wave 4 onward
- **Wave modifiers** (fast / regen / horde / wealth), kill-combo credits, wave MVP damage rankings
- **Core shop** — earn ⬡ cores every run and spend them on cosmetics: grid themes that restyle the whole battlefield (ember/frost/jade/void) and alternate color skins for every hero
- **Codex** — in-game encyclopedia of heroes, enemies, biome lore and controls, plus a RECORDS tab with lifetime stats and 14 achievements
- **Game feel** — towers assemble behind a rising scan line when placed, enemies shatter into their own outline fragments, heavy hits land with a micro hit-stop, bounty chips arc into your credits, the core beats audibly when integrity runs low, and phones get haptic feedback throughout
- **Procedural music** — WebAudio synth score that intensifies with combat and boss waves
- **Mobile-first** — drag-to-place touch controls with a lifted ghost, auto-landscape rotation in portrait, safe-area aware layout, installable PWA with offline support

## Controls

| Input | Action |
|-------|--------|
| 1–7 | Select hero (8/9 for structures) |
| Drag a card from the dock onto the grid (any direction) | Place tower |
| Hold a card | Inspect its stats and specs |
| Tap a tower | Details, upgrade, repair, sell, targeting mode |
| Q/W/E | Abilities |
| Space | Start wave (early call = bonus credits) |
| F | Game speed 1×/2×/3× |
| T | Cycle targeting mode |
| P | Pause |
| Shift+click | Place multiple towers |

Built with HTML5 Canvas + WebAudio. Single file, ~4k lines, zero dependencies.

By **Saiss & Claude**.
