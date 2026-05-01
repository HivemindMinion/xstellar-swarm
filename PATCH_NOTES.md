# XStellar: Swarm — Patch Notes

All notable changes ship as a numbered version. Each entry below lists what was **Added**, **Changed**, or **Removed** relative to the previous version.

The latest version is always at the top.

---

## v34 — Drop the crit-kill camera flash (eye-strain fix)

### Removed
- The 60ms gold full-screen flash that fired on every crit-kill (added in v9). At high Perception + Pierce III, a single Volley could trigger 3–5 crit-kills per second, and the flash strobed the entire screen at that rate — uncomfortable to look at.

### Changed
- The hit-stop slow-mo (70ms at 8% time) is preserved on crit-kills. Combined with the existing yellow particle burst, white spark, gold damage number, and crit SFX bell two-note, the moment still lands clearly without ever flashing the whole screen.
- Boss-kill flash kept — it fires only once per ~5 waves (when a Queen/Tyrant/Architect dies), so it's a genuine victory moment rather than a strobe. Same with the Screamer-scream camera shake (rare, intentional jolt).

---

## v33 — HiDPI text (no more blur on big screens)

### Fixed
- **Text was blurry when the browser scaled up the 480×720 game canvas.** Phaser renders Text into an off-screen canvas at the requested font size, uploads as a texture, and then `Scale.FIT` mode bitmap-upscales the entire canvas to fill the window — including the text texture, which softens serif glyphs. Particularly bad on high-DPI displays where the screen pixel ratio is 1.5×–2× already.
- Patched the `Phaser.GameObjects.GameObjectFactory.prototype.text` factory at boot so every `this.add.text(...)` call automatically calls `setResolution(max(2, devicePixelRatio))`. The off-screen text canvas now renders at 2×–3× density, then scales DOWN to display — crisp at any browser zoom level.
- Added `render: { roundPixels: true, antialias: true }` to the game config to snap text to integer pixel boundaries and avoid sub-pixel sampling blur.

---

## v32 — HUD readout for crit chance and cooldown reduction

### Added
- New tiny stat readout under the kill counter / stat-points line: `Crit X%  ◆  CDR X%`. Updates every time `updateHUD()` runs.
- Exposes both effects of the v7+v16 Perception stat so allocating points feels purposeful: investing PER produces a visible number bump in real time, and players can see how close they are to the 50% crit cap and 30% CDR cap.
- Rendered in dim italic (`#7a6a4a`, 9px) so it sits under the existing HUD without competing with the bright kill counter or the gold "points pending" line.

---

## v31 — Ambient music drone

### Added
- A slow ominous bass drone via Tone.js `AMSynth`, layered behind the existing v12 procedural SFX. Plays an 8-step minor sequence (C2 → F2 → D#2 → G2 → F2 → C2 → A#1 → D#2) on a 4.5-second interval, with each note attack/decay shaped to swell rather than punch — so it sits well under hit/crit/kill SFX without competing for attention.
- Sub-mixed at -28 dB (vs SFX at -10 to -22 dB) so combat audio stays foreground; the drone is felt more than heard, like a heartbeat for the harvest.
- Lazy-init alongside the existing SFX init (first user gesture, browser audio policy). Silent fallback if Tone.js fails to load.

---

## v30 — Pause menu (P / Esc)

### Added
- **P** or **Esc** toggles a full pause: enemies, bullets, pools, particles, charger telegraphs, queen advance, charger sprints — everything freezes mid-frame. The pause flag short-circuits `update()`, so it's a true freeze with zero ongoing simulation cost.
- A semi-transparent veil over the play area with **PAUSED** centered and a "Press P or Esc to resume" hint underneath.
- Tab special-swap and right-click special activation are gated on `!gamePaused` so the menu can't be exploited mid-pause.
- Title screen control hint updated: `Left Click: Attack ◆ Right Click: Special / WASD / Arrows: Move ◆ Tab: Swap Special ◆ P / Esc: Pause`.

---

## v29 — Red damage numbers when the player takes a hit

### Added
- Every successful hit on the player now spawns a red `-N` floater that rises and fades from the player's position over 700ms. Closes the feedback loop with v26 (enemy damage numbers): you see what you deal, you see what you take.
- Reuses the v27 dmgPool — no extra allocation per hit.
- Number reflects post-Armor damage (the actual HP delta), not the raw incoming amount, so investing in Armor visibly shows up as smaller red numbers.

---

## v28 — Wave announcement timing polish

### Changed
- **Wave / Band / Flavor / Boss announcements fade much faster.** Old durations (1.8–2.6s visible) meant text stayed on screen well into active gameplay; playtest screenshots showed "Below the Crypt" + "Wave 4" + flavor line still visible while the player had a 25× combo and 30 kills. New durations (≈0.9–1.3s visible) flash dramatically, then get out of the way.
  - Band: delay 1700→700ms, fade 900→400ms
  - Boss: delay 1500→800ms, fade 1000→500ms
  - Wave + flavor: delay 1200→500ms, fade 600→400ms
- Announcement positions shifted up ~20–30px (Boss `y/2-180`, Band `y/2-150`, Wave `y/2-110`, Flavor `y/2-72`) so the title flash sits above the formation rows instead of slicing through them.

---

## v27 — Performance pass: damage-number pool, per-frame caches

### Changed
- **Damage numbers now use an object pool of 24 pre-allocated `Text` objects** instead of `add.text() / destroy()` per hit. Phaser Text creation is expensive (font shaping + GPU upload); under Pierce III + Volley (11 arrows) you can spawn 30+ numbers in a single frame, which previously caused micro-stutters. Pool slots track a `until` timestamp so a saturated pool recycles the oldest slot (rare in practice).
- **Per-frame caches at the top of the bullet-collision phase:** `getDamage()`, `getCritChance()`, `getCritDmg()` are read once per frame (not once per hit), and the live-shielder list is built once per frame instead of walking the entire enemies group on every bullet contact. With multiple shielders + Pierce + many enemies, the inner loop now scans only the (usually small) shielder set rather than every enemy for every bullet hit.

### Removed
- `isShieldedBy(e)` helper — replaced by the inlined cached-shielder loop in the bullet-collision check. Behavior identical, fewer iterations per hit.

---

## v26 — Floating damage numbers

### Added
- Every bullet hit now spawns a damage-number text that rises and fades over ~550ms (~750ms for crits). Numbers are jittered ±6px on spawn so two simultaneous Pierce hits don't perfectly overlap.
- Crits render distinctly: 20px gold italic with `!` suffix and a longer rise; normal hits are 13px white italic. Both stroked black for readability against the field/sky.
- Numbers are rounded to one decimal so Haste-penalty fractional damage (e.g. 3.5) reads correctly without showing 3.4999.

### Changed
- The classic "make every shot satisfying" feel pass. Synergizes with v7 (crits exist), v9 (crit hit-stop + camera flash), and v6 (combo pierce) — every Pierce-III crit now pops 3 gold `!` numbers in slow-mo.
- Mage Arcane Nova still uses the existing single particle burst per hit; AoE numbers would clutter the screen with one button press, kept the ring effect instead.

---

## v25 — Wave-transition polish, charger respawn fix, hazard cleanup

### Fixed
- **"BOSS" announcement bleeding into the next wave's "Wave N" text.** When a player burst-killed wave 5 in under 2.5s, wave 5's BOSS / band / wave-name texts were still tweening when wave 6's startWave fired and stacked its own announcements on top. New waves now destroy any leftover announcement text objects from the previous wave (`this.waveAnnouncements` array tracked across `startWave` calls).
- **Chargers respawning into the SKY after diving off-screen.** Charger respawn used the original sky target (`Phaser.Math.Between(50, 160)`), but in v23 chargers became a ground type. They now respawn into the field (`Phaser.Math.Between(240, 360)`) so they crawl back into formation rather than reappearing 60px below the wave-name banner.

### Changed
- **Acid pool cap.** Spitter misses no longer pile up indefinitely — when 6 pools are already on the wall, the oldest is destroyed before the new one spawns. Keeps the wall readable instead of a continuous green carpet.
- **Wave-end hazard reset.** Previous wave's acid pools, in-flight enemy bullets, and Mage Nova rings are now wiped on wave-clear so the next wave starts visually clean. (Player bullets and particles are short-lived enough to leave alone.)
- **Stronger ground perspective.** `groundPerspective(y)` now maps `y ∈ [200, 620]` to scale `[0.30, 1.30]` (was `[0.45, 1.20]`) clamped to `[0.25, 1.30]`. Makes "small on the horizon → big at the wall" much more readable as a depth cue: ~3× scale change across the field instead of 1.7×.

---

## v24 — Bug fixes: wave chain-firing, lingering corpses, and ground perspective

### Fixed
- **Multiple waves spawning at once.** The wave-end check (`enemiesRemaining <= 0`) was firing every frame in the 250ms gap between `startWave()` running and `spawnWaveEnemies()` populating the count. Each fire scheduled another `startWave`, chain-spawning 2-3 waves in close succession. Added a `pendingSpawn` flag set true in `startWave` and cleared inside the `spawnWaveEnemies` callback; the wave-end check now requires `!pendingSpawn`.
- **Wave still moving on before all enemies were dead.** Symptom of the same bug — when a chain-fired startWave overwrote `enemiesRemaining` with the new wave's count, lingering enemies from the previous wave became "free" and the player was already in the next wave. Same fix.
- **Dead enemies' sprites lingered on the battlefield.** Kill paths set `e.alive = false` but the update-loop's early-return skipped `drawEnemy`, where the sprite teardown lived. Now the early-return itself destroys the sprite.

### Added
- **Perspective scaling for ground bugs.** Crawlers, Spitters, Chargers, Screamers, Shielders, and Resonators now scale based on their on-screen Y: ~0.45× when they appear at the horizon, ~1.2× when they reach the wall. Function `groundPerspective(y)` maps `y ∈ [200, 620]` to scale `[0.45, 1.20]`, clamped to `[0.40, 1.25]` at the edges.
- **Slow advance for ground "creepers."** Crawlers, Spitters, Shielders, Resonators now advance 9 px/s toward the wall after arriving in formation (until y=600). Combined with perspective scaling, they read as continuously approaching from depth. Speedy elites advance proportionally faster. Chargers (own dive logic) and Screamers (escape sideways) are excluded.

---

## v23 — Sprite swap (12 enemy types) + ground bugs land in the field

### Added
- **`assets/enemies/`** folder with 12 PNG sprites downscaled from the XStellar Unity project (`XStellar-Rebuild/Sprites/Enemies/Processed/`). Total 161KB across all 12. Mapped one-to-one to existing enemy types based on visual fit:
  - Drone → crawler · Flyer → flyer · Spitter → spitter · Charger → charger
  - HiveQueen → queen · Broodling → broodling · BroodTyrant → tyrant · HiveArchitect → architect
  - Screamer → screamer · Wraith → wraith · Shielder → shielder · Resonator → resonator
- `GameScene.preload()` loads all 12 sprites once; the texture cache prevents re-loads on subsequent runs.
- `SPRITE_TYPES` set + `SPRITE_SIZES` map drive the sprite-vs-procedural branch and per-type display size.

### Changed
- `spawnEnemy()` now creates a Phaser `Image` at the enemy's position when the type matches a sprite. The image is stored on the enemy object as `spriteImg` and synced to position/alpha each frame in `drawEnemy`.
- `drawEnemy()` refactored: procedural body chain is now gated on `if (!e.spriteImg)` so it only fires for eggsacs (and as fallback if a texture failed to load). Auxiliary visuals — Shielder/Resonator auras, Charger telegraph + aim line, Wraith phase outline, Screamer urgency ring — extracted into a separate "always-render" block so they fire regardless of whether the body is sprite or procedural.
- **Ground-walking enemies now form deeper into the battlefield, not in the sky.** Crawlers, Spitters, Chargers, Screamers, Shielders, Resonators target row-y of `240 + r*44` (in the grass field, near the wall). Flyers and Wraiths keep the sky formation `40 + r*50` for the classic Galaga dive. Queen-wave crawler escorts also moved to the field.
- Sprite images are destroyed on enemy death (in `drawEnemy` if `!e.alive` and on the wave-clear path) so they don't leak across waves.
- Wraith phase alpha (0.22) is now applied to the sprite directly via `setAlpha`.

### Removed
- Procedural body code is no longer the visual default for the 12 mapped types — kept inline only as fallback (e.g. CDN/asset-fetch failure).

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
