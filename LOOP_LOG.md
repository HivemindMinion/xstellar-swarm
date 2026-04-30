# LOOP_LOG.md — XStellar: Swarm iteration journal

Recurring loop fires hourly at :07. Each entry: version + 1-line change + commit hash.

## 2026-04-29

- **v6** (initial depth pass) — stat tradeoffs (Haste -dmg, Armor -speed) + combo pierce I/II/III. Commit `c2d93db`.
- **Research synthesized** — 4 parallel agents mined XStellar-Rebuild for enemies, abilities, progression, lore. Backlog written to RESEARCH_NOTES.md.
- **v7** — crit chance + crit damage. Perception (+1%/pt, cap 50%) and Luck (+5% crit dmg/pt, base 2x). Crits yellow-flash particles and count as +2 combo, accelerating the v6 pierce ladder. Class bases tuned, save migrated v6→v7, LevelUpScene compressed to fit 7 attributes, GameOverScene + ClassSelect updated.
- **v8** — wave bands + 25 flavor strings. 8 named bands ("The Outer Walls" w1-3 through "World-Ender Epoch" w21+). Band-entry announcement on threshold waves; band name persists in the HUD wave label as ongoing context. Pure content, no save schema change.
- **v9** — hit-stop + camera flash on crit-kill. 70ms slow-mo at 8% time-scale, plus 60ms gold camera flash. Synergizes with v7 crit-as-build-identity: every crit-kill *feels* earned. ~10 lines of code.
