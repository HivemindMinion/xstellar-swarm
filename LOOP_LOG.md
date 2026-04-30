# LOOP_LOG.md — XStellar: Swarm iteration journal

Recurring loop fires hourly at :07. Each entry: version + 1-line change + commit hash.

## 2026-04-29

- **v6** (initial depth pass) — stat tradeoffs (Haste -dmg, Armor -speed) + combo pierce I/II/III. Commit `c2d93db`.
- **Research synthesized** — 4 parallel agents mined XStellar-Rebuild for enemies, abilities, progression, lore. Backlog written to RESEARCH_NOTES.md.
- **v7** — crit chance + crit damage. Perception (+1%/pt, cap 50%) and Luck (+5% crit dmg/pt, base 2x). Crits yellow-flash particles and count as +2 combo, accelerating the v6 pierce ladder. Class bases tuned, save migrated v6→v7, LevelUpScene compressed to fit 7 attributes, GameOverScene + ClassSelect updated.
- **v8** — wave bands + 25 flavor strings. 8 named bands ("The Outer Walls" w1-3 through "World-Ender Epoch" w21+). Band-entry announcement on threshold waves; band name persists in the HUD wave label as ongoing context. Pure content, no save schema change.
- **v9** — hit-stop + camera flash on crit-kill. 70ms slow-mo at 8% time-scale, plus 60ms gold camera flash. Synergizes with v7 crit-as-build-identity: every crit-kill *feels* earned. ~10 lines of code.
- **v10** — Charger enemy. Wave 4+, ~18% spawn weight (overridden by spitter at w5+). Arrives in formation, locks aim at player when its 800ms windup begins (red telegraph ring + aim line + horn rotates to face), then sprints in a straight line at the locked angle. Idle wobble freezes during telegraph for readability. 2x contact damage on ram. Off-screen → respawn at top with 2.5-4.5s delay before next charge.
- **v11** — Spitter acid pools. Missed enemy bullets that cross the wall plane splash into a persistent green pool (4s life, 26px radius) at the wall surface where the player walks. Pool ticks 2 damage every 0.5s while player overlaps; respects existing invuln/shield-wall guards. Pre-seeded bubble offsets bob in place (no per-frame flicker). Spitters now dictate positioning during their wave 5+ presence — every miss is a future hazard.
- **v12** — procedural SFX via Tone.js. 8 sound events: fire (PluckSynth twang), hit (pink-noise tick), crit (FM bell two-note), kill (brown-noise squish), wave start (membrane drum two-beat), level up (FM bell arpeggio C-E-G-C), death (FM gong), special activate (FM flourish). Init lazily on first user input (browser audio policy). Silent fallback if Tone.js CDN fails. Throttled hit/kill (40/60ms) so high-pierce shots don't audibly stack.
