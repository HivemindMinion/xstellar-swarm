# RESEARCH_NOTES.md — XStellar: Swarm v7+ Backlog

Curated from 4 research agents that mined `C:\Users\tanki\XStellar-Rebuild\` for stealable mechanics, abilities, progression systems, and lore. Source-of-truth backlog for the iteration loop.

## How the loop uses this

Each loop iteration:
1. Reads this file + `LOOP_LOG.md`.
2. Picks the topmost **unchecked** item from the priority backlog.
3. Ships one focused change to `index.html` (small scope per iteration).
4. Checks the item off here, appends a one-line entry to `LOOP_LOG.md`.

**Principle (Connor):** "Strategic depth does not require complexity." Bias toward changes that add decisions per minute, not features.

---

## Priority backlog (top = ship next)

### Combat depth — small, additive
- [x] **v7: Crit chance + crit damage** — shipped. PER + LUK stats, crit kills jump combo by 2.
- [ ] **v8: Cooldown reduction stat** — Wisdom-style. Either extend Haste (specials -3% CD/pt, cap 60%) or add new stat. Cleanest: Haste already has tradeoff; piggyback there. Specials feel reactive, not hoarded.
- [x] **v10: Charger enemy (wave 4+)** — shipped. Telegraphed 800ms aim-lock + sprint at locked angle, 2x contact dmg.
- [ ] **v10: Acid pools from Spitter** — when Spitter shoots, also drops a green pool at its target spot lasting 4s, ticking 1 dmg/0.5s if player stands in it. Stateless graphics object array, like particles. Adds positioning layer.
- [ ] **v11: Screamer enemy** — runs AWAY from player toward map edge; if reaches edge or survives 12s, spawns +5 mini-crawlers as bonus wave. DPS-race target. Inverts the "always advance toward you" reflex.

### Boss waves — every 5th wave
- [ ] **v12: Mini-Queen boss (wave 5)** — beefy crawler-shape, 8x HP, slow advance, every 4s spawns 1 broodling. On death: drops 30 XP burst.
- [ ] **v13: Brood Tyrant boss (wave 10)** — armored, periodically lays 3 egg sacs; sacs hatch into mini-crawlers after 3s telegraph. Kill sacs early to thin the swarm.
- [ ] **v14: Hive Architect boss (wave 15)** — doesn't attack; teleports between 3 anchor points each spawning 2 enemies on arrival. Mid-battle spawner. Force focus-fire vs. cleared minions.

### Polish & feel — high-impact, low-risk
- [ ] **v15: Tone.js procedural SFX** — fire (twang), hit (chitin crack), crit (chime), kill (squish), wave start (drum), level up (bell), death (gong), special activate (per-class). Single CDN script tag, ~100 lines of code.
- [x] **v8: Wave bands + extended flavor** — shipped. 8 bands, 25 flavor strings, HUD subtitle.
- [ ] **v17: Faction codename + class lore on title** — "The Culigri Collective" tagline beneath subtitle. Class blurbs in select screen (1 line each, lore-grounded).
- [x] **v9: Hit-stop + camera flash on crit-kill** — shipped. 70ms slow-mo at 8% time + 60ms gold flash.
- [ ] **v19: Sprite swap for 3 base enemies** — downscale `Processed/NewDroneSprite.png`, `NewFlyerSprite.png`, `NewSpitterSprite.png` to 64×64 PNGs in `assets/enemies/`. Swap drawEnemy() to use Phaser images for those types; keep procedural fallback.

### Heavier mechanics — defer until above land
- [ ] **v20: Wraith enemy** — phase-teleport every 3s to a random spot 80-180px away, intangible while phasing.
- [ ] **v21: Burrower enemy** — 2s underground (invuln), emerges adjacent to player with telegraph ring 0.5s before.
- [ ] **v22: Carrier enemy** — drops 1 egg every 5s as it moves; eggs hatch into broodlings (1 HP, fast) after 2s.
- [ ] **v23: Shielder/Resonator support orbits** — non-attacking enemies that grant nearby bugs +damage or HP regen aura. Forces target-priority decisions.
- [ ] **v24: Elite modifiers** — 5% chance per enemy to roll one elite mod (Berserk +50% dmg, Shielded HP shield, Speedy +30% move, Regen +1 HP/s). Visual tag (colored outline). Drops 2x XP.

### Skip / defer (scope creep — Connor flagged "depth ≠ complexity")

- ❌ **Cascade economy + prestige lock-in** — too big for single-file Phaser. Existing roguelite loop is sufficient.
- ❌ **Equipment system w/ rarities + procs** — needs inventory UI, drop tables, slots. Massive lift for marginal decisions/min.
- ❌ **12 town archetypes / G.O.D. alignment** — out of scope for castle defense.
- ❌ **60 achievements** — content padding, not depth.
- ❌ **Animated 4×4 spritesheets** — 7×3×16 = 336 frames vs. negligible feel improvement over static + tween.

---

## Reference details

### Crit/Luck math (v7)
- `getCritChance() = Math.min(0.5, PLAYER.stats.per * 0.01)`
- `getCritDmg() = 2.0 + PLAYER.stats.luk * 0.05`
- Roll on each bullet hit; on crit: damage *= critDmg, spawn yellow particles (8 count), gainXP called with comboGain=2.

### Suggested base stats with PER/LUK (v7)
| Class  | FOR | MIG | AGI | HAS | ARM | PER | LUK |
|--------|-----|-----|-----|-----|-----|-----|-----|
| Ranger | 6   | 4   | 5   | 6   | 1   | 3   | 1   |
| Knight | 12  | 3   | 3   | 2   | 5   | 1   | 1   |
| Mage   | 5   | 6   | 4   | 4   | 2   | 2   | 3   |
| Cleric | 9   | 3   | 4   | 3   | 5   | 1   | 2   |

### Faction & enemy lore (lore agent canonical)
- **The Culigri Collective** — tungsten-based insectoid hive, harvests Seeds (source of magic) on a 10,000-year cycle. The invasion IS the harvest.
- **Drone** — worker caste, gathers organic matter, cannibal by necessity.
- **Flyer** — dragonfly-wasp hybrid, four tattered wings, ignores terrain.
- **Spitter** — bloated pressure sac, arcs acid creating danger zones.
- **Charger** — rhinoceros-beetle horn plate, telegraphs charge then sprints.
- **Burrower** — centipede-head, burrows underground (invulnerable), erupts near player.
- **Screamer** — oversized vocal chambers; if it survives, spawns extra wave.
- **Wraith** — light-bending carapace, near-invisible, ambush striker.
- **Carrier** — eyeless spawner, gives birth mid-combat.
- **Shielder** — projects bio-energy bubble, allies inside take -50% dmg.
- **Resonator** — vibrating wing-plates emit subsonic +25% dmg buff aura.
- **Hive Queen** — multi-phase boss; spawn → awakening → hive-song → metamorphosis.
- **Hive Architect** — termite-queen builder, teleports and spawns mid-fight.
- **Brood Tyrant** — bloated abdomen of egg sacs, architect of swarms.

### Wave flavor strings (lore agent — drop into `flavors[]` in startWave)
Early (w1-5): "The hive stirs beneath the earth.", "Mandibles scrape stone. They are coming.", "The swarm has caught our scent.", "Shadows move where no shadows should be.", "The Keep's shadow grows long. Prepare."
Mid (w6-10): "The hive awakens. Reinforcements endless.", "They pour from every crevice, an organic tide.", "No negotiation. No mercy. Only the harvest.", "The Keep glows brighter—they are drawn to it.", "We hold. We bleed. We hold."
Late (w11-15): "The ground itself writhes. Hive corruption spreads.", "Queens send their brood. Architects direct the swarm.", "The air is thick with spores. Lungs burn.", "They learn. They adapt. They hunger.", "Only the strongest bloodline will outlive this dawn."
Catastrophic (w16+): "The hive itself descends. All is shadow and chitin.", "Mothers of the swarm command the assault.", "The earth cracks. The sky darkens. The end times whisper.", "They sing now. The collective voice of the hive itself.", "This is no longer a wave. This is the harvest."

### Wave-band names
- W1-3 "The Outer Walls"
- W4-6 "Below the Crypt"
- W7-9 "The Hive Rises"
- W10-12 "The Architects"
- W13-15 "Resonance"
- W16-18 "The Singing"
- W19-20 "Mothers Descend"
- W21+ "The World-Ender Epoch"

### Class lore (1 line each, for ClassSelectScene flavor)
- **Ranger** — "The swift survivor. Rains arrows like the storm before the swarm."
- **Knight** — "The iron wall. Last line of defense; armor thickens with every life saved."
- **Mage** — "The arcane scholar. Magic flows from Seeds left by the Culigri themselves."
- **Cleric** — "The blessed protector. Holy light burns where no ordinary flame can touch."
