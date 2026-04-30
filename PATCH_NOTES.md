# XStellar: Swarm — Patch Notes

All notable changes ship as a numbered version. Each entry below lists what was **Added**, **Changed**, or **Removed** relative to the previous version.

The latest version is always at the top.

---

## v22 — QoL pass: HP bar refresh, specials unlocked at start, combo carries between waves

### Changed
- **HP bar now refreshes on every hit.** `takeDamage()` was only calling `updateHUD()` on death, so mid-fight damage didn't update the bar visually. Single-line fix.
- **Starting class's mastery is now unlocked at character creation** (level 1 of the matching spec, `activeSpecial` set automatically). Previously the player couldn't use right-click until they earned a stat point on level-up, died, and spent the point in the LevelUpScene — typically several waves of waiting. New characters now have their right-click special available from wave 1.
- **Inter-wave timing tightened so an active combo can survive into the next wave.** Wave-end → next-wave-start gap reduced from 1500ms → 150ms. Spawn delay inside `startWave` reduced from 800ms → 250ms. Total time before new enemies are hittable is ~400ms (well inside the 2s combo timer).
- Wave-complete check is unchanged in logic — wave still only ends when `enemiesRemaining` reaches zero, which already counts every dynamically-spawned enemy (broodlings from Queen, broodlings from Screamer scream, Architect teleport-spawns, eggsacs from Tyrant) thanks to existing increment/decrement pairs.

---

## v21 — Shielder + Resonator support enemies

### Added
- **Shielder** (wave 8+, 8% spawn, 6+wave HP, 22 XP) — projects a 60px defensive aura visible as a pulsing translucent blue circle. Allies inside the aura take 50% reduced damage from player bullets. Stays at top in formation, no contact damage.
- **Resonator** (wave 8+, 8% spawn, 6+wave HP, 22 XP) — projects a 60px healing aura visible as a pulsing translucent green-teal circle. Heals nearby allies 1 HP/sec, with a teal sparkle particle on each tick. Stays at top in formation, no contact damage.
- Helper `isShieldedBy(e)` walks the enemies group looking for a live Shielder within 60px of the queried enemy.

### Changed
- Bullet→enemy collision now multiplies damage by 0.5 if the hit enemy is within a Shielder's aura.

---

## v20 — Wraith enemy (phase-teleport)

### Added
- **Wraith** (wave 7+, 10% spawn, 4+wave HP, 20 XP) — phase-cycles every 2-4s. Phase-out lasts 600ms; on phase, teleports 80-180px in a random direction (clamped on-screen). Bullets pass through wraiths during phase-out via a single-line guard at the top of the bullet-enemy collision check. Visual: translucent teal-purple body with inner ghostly glow, cold blue eyes that stay visible while phased so the player can read position and time the next shot.

---

## v19 — Elite modifiers

### Added
- **Berserk** elite modifier — +50% contact damage, red pulsing halo. 6% roll on regular enemies wave 4+.
- **Speedy** elite modifier — +40% movement on every motion source (idle wobble, dive, broodling drift, charger sprint, screamer escape, queen advance). Blue pulsing halo.
- **Regen** elite modifier — +1 HP/sec while below max, with a green sparkle particle on each tick. Green pulsing halo.
- Elite kills drop 2x XP. Pulse phase varies per spawn so multiple elites in the same wave don't blink in sync.

### Changed
- Bosses, eggsacs, and broodlings are exempt from elite rolls (too short-lived or already too dramatic).

---

## v18 — Faction codename and class lore

### Added
- Title screen now reads "Defend the Keep" / "The Culigri Collective approaches." in dim crimson italic.

### Changed
- Class descriptions on the Class Select cards replaced with full Culigri-canon lines:
  - Ranger — "The swift survivor. Rains arrows like the storm before the swarm."
  - Knight — "The iron wall. Last line of defense; armor thickens with every life saved."
  - Mage — "The arcane scholar. Magic flows from Seeds left by the Culigri themselves."
  - Cleric — "The blessed protector. Holy light burns where no ordinary flame can touch."

---

## v17 — Screamer enemy

### Added
- **Screamer** (wave 6+, 10% spawn, 5+wave HP, 18 XP) — runs AWAY from the player at 80 px/s. If she reaches a side edge OR survives 12 seconds, she screams: spawns 5 broodlings, 220ms camera shake, gong SFX, yellow particle burst. Last 3s of timer draws a tightening yellow urgency ring. First "kill it now or pay later" decision in the loop.

### Changed
- Idle wobble disabled for screamers so the escape vector stays clean.

---

## v16 — Perception reduces special cooldown

### Added
- Perception now grants -0.5% special cooldown per point (cap 30% at PER 60), in addition to its existing crit chance. PER becomes a "perception/focus" build-identity stat that rewards casters with reactive specials.

### Changed
- `useSpecial()` refactored: 4 duplicate `this.specialCooldown = data.cd` sites collapsed into a single end-of-method assignment that applies the multiplier.
- HUD cooldown bar normalized to effective cooldown so it always starts full and drains.

---

## v15 — Hive Architect boss (round-robin)

### Added
- **Hive Architect** (wave 15, 30, 45..., 80+wave*9 HP, 100 XP) — third boss. Doesn't directly attack. Teleports between 3 anchor points (left/center/right at y=80) every 3.5s; each warp spawns a crawler+broodling pair. Won't pick the same anchor twice in a row.
- Visual: hexagonal-pattern carapace, spade-like fore-mandibles, six articulated limbs, glowing orange compound eyes.
- Death drama uses an amber camera flash to distinguish from Queen/Tyrant.

### Changed
- Boss rotation is now round-robin: w5/20/35→Queen, w10/25/40→Tyrant, w15/30/45→Architect.

---

## v14 — Brood Tyrant boss

### Added
- **Brood Tyrant** (wave 10/20/30, 100+wave*10 HP, 120 XP) — second boss. Stationary at top, no escorts. Every 8 seconds lays a volley of 3 eggsacs across her abdomen.
- **Eggsac** (4 HP, summoned only by Tyrant) — telegraph 3s, then hatches into a broodling at the same spot. Killing the sac before hatch prevents the broodling spawn.
- Visual: bloated abdomen with decorative pulsing sacs, heavy thorax, crushing claws, two large glowing eyes.

### Changed
- Boss rotation: every 10th wave is now Tyrant; Mini-Queen still appears on every other 5x wave.
- Boss-aware checks (HP bar size, contact damage 3x, death drama) extended to cover Tyrant.

---

## v13 — Mini-Queen boss every 5th wave

### Added
- Boss-wave path: `wave % 5 === 0` now spawns a Mini-Queen + 4 supporting crawlers instead of the normal grid spawn.
- **Mini-Queen** (60+wave*8 HP, 80 XP) — slow advance toward the player line, spawns a broodling every 4 seconds at her position, 3x contact damage, survives contact (escorts/non-bosses still die on touch).
- **Broodling** (1 HP, 5 XP, summoned only by Queen) — feral 1-HP scout that drifts toward the player at 36 px/s vertical + 22 px/s horizontal tracking.
- "— B O S S —" announcement above the wave count on boss waves.
- Boss death drama: 20 pink + 10 gold particles, 180ms magenta camera flash, 140ms slow-mo, level-up bell sting.

### Changed
- Despawn rule: anything that falls past `y > H+30` (except flyers and chargers) decrements `enemiesRemaining` so the wave can still end if minions escape.

---

## v12 — Procedural SFX (Tone.js)

### Added
- Tone.js loaded from CDN.
- 8 synthesized sound events: fire (PluckSynth twang), hit (pink-noise tick, throttled 40ms), crit (FM bell two-note A5→E6), kill (brown-noise squish, throttled 60ms), wave start (membrane drum two-beat), level up (FM bell arpeggio C-E-G-C), death (FM gong), special activate (FM flourish).
- Lazy init on first user gesture (browser audio policy). Silent fallback if the CDN fails or the browser blocks audio.

---

## v11 — Spitter acid pools

### Added
- Missed enemy bullets that cross the wall plane splash into a persistent green acid pool (4s life, 26px radius) at the wall surface.
- Pools tick 2 damage every 0.5s while the player overlaps; respect existing invuln/shield-wall guards.
- Pre-seeded bubble offsets bob in place (no per-frame flicker). Pools fade alpha in their last 1.2s of life.

---

## v10 — Charger enemy

### Added
- **Charger** (wave 4+, ~18% spawn, 4+wave*1.5 HP, 14 XP) — telegraphed 800ms aim-lock window (red ring + aim line + horn rotates to face), then sprint at the locked angle (380+wave*15 px/s). Off-screen → respawn at top with 2.5-4.5s delay.
- 2x contact damage on ram (vs. crawler's 1x).

### Changed
- Idle wobble freezes during the charger's aim-lock telegraph for clearer dodge readability.

---

## v9 — Hit-stop on crit-kill

### Added
- Crit-kills now trigger a 60ms gold camera flash (255, 230, 110) and a 70ms slow-mo window where world `dt` scales to 8% of real time. Input/aim keep ticking at full speed inside the slow-mo so the player never feels input lag.

---

## v8 — Wave bands and 25 lore-grounded flavor strings

### Added
- 8 named wave bands sourced from XStellar canon: "The Outer Walls" (w1-3) → "Below the Crypt" → "The Hive Rises" → "The Architects" → "Resonance" → "The Singing" → "Mothers Descend" → "The World-Ender Epoch" (w21+).
- "~ Band Name ~" announcement on threshold waves.
- 25 Culigri Collective lore-grounded flavor lines (one per wave 1-25, capped after).

### Changed
- HUD wave label now shows current band name as ongoing context: `— Wave N · Band Name —`.

---

## v7 — Crit chance + crit damage

### Added
- **Perception** stat — +1% crit chance per point (cap 50% at PER 50).
- **Luck** stat — +5% crit damage per point (base 2.0×).
- Crits show yellow particles + white spark on hit.
- Crit-kills count as +2 combo (vs. +1 for normal kills), accelerating the v6 pierce ladder.

### Changed
- Save schema migrated v6→v7. `loadPlayer()` now merges class base under saved stats so older saves get sane PER/LUK defaults instead of NaN.
- LevelUpScene compressed to fit 7 attribute rows (40→32 row spacing).
- ClassSelectScene stat bars compressed (62→45 px) to fit 7 stats.

---

## v6 — Stat tradeoffs and combo pierce

### Added
- **Combo pierce** — kill-streak now grants pierce I/II/III at combo 5/10/15. Bullets get a colored halo (blue/orange/gold) matching the pierce tier.
- Combo HUD shows pierce tier with matching color.

### Changed
- **Haste** now also -0.5 damage per point (rate-vs-burst tradeoff, floor 1 dmg).
- **Armor** now also -3 move speed per point (tank-vs-mobile tradeoff, floor 60 speed).
- Save schema v5→v6 with migration helper preserving existing characters.

---

## v5 — Initial baseline (pre-loop)

Initial Phaser 3 single-file game shipped to GitHub Pages. Mouse-aimed projectiles, 4 classes (Ranger, Knight, Mage, Cleric) with one special each (Volley / Shield Wall / Arcane Nova / Divine Mend), 5 stats, class mastery system, RPG progression with death-allocated stats, 3 enemy types (Crawler, Flyer, Spitter), wave system, localStorage persistence.
