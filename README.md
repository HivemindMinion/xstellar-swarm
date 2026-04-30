# XStellar: Swarm — Castle Defense

**Medieval Galaga × RPG × Survival | Humans vs. Bugs**

A browser-based castle defense game set in the XStellar universe. Defend the keep against waves of insectoid invaders using mouse-aimed combat, class abilities, and deep RPG progression.

## Play It

Open `index.html` in any modern browser, or visit the GitHub Pages deployment.

## Controls

| Input | Action |
|-------|--------|
| **WASD / Arrow Keys** | Move along the castle wall |
| **Left Click** | Fire projectile toward cursor |
| **Right Click** | Use special ability |
| **Tab** | Cycle between unlocked specials |
| **Space** | Fire toward cursor (keyboard alt) |

## Features

### Classes
- **Ranger** — Fast bow, high haste, glass-ish
- **Knight** — Heavy armor, crossbow, slow but tanky
- **Mage** — Arcane bolts, devastating but fragile
- **Cleric** — Holy mace, balanced with self-sustain

### Class Mastery System
Your starting class determines base stats, but you can spec into **any** class's mastery tree. Each mastery level strengthens that class's special ability:

| Class | Special | Effect |
|-------|---------|--------|
| Ranger | **Volley** | Fires a cone of arrows toward cursor |
| Knight | **Shield Wall** | Brief invulnerability |
| Mage | **Arcane Nova** | AoE blast damages all nearby enemies |
| Cleric | **Divine Mend** | Instant heal |

Multi-spec into different classes to unlock multiple specials, then swap between them with Tab.

### Enemies
- **Crawlers** — Ground bugs in formation, animated legs
- **Flyers** — Winged bugs that dive-bomb you (Wave 3+)
- **Spitters** — Armored bugs that fire acid at you (Wave 5+)

### Progression
- XP per kill with combo multipliers
- Stat points accumulate during combat, allocated on death
- Stats persist between runs (roguelite loop)
- localStorage persistence (Firebase-ready `SaveManager` interface)

### Crit (v7)
Two new attributes split the crit dial:

| Stat | Effect |
|------|--------|
| Perception | +1% crit chance per point (cap 50%) |
| Luck | +5% crit damage per point (base 2.0×) |

Crits spawn yellow particles and **count as 2 combo** instead of 1 — fast-tracking the v6 pierce ladder. Crit-fishing builds (high PER + LUK) reach Pierce III on shorter kill streaks; consistent-damage builds (high MIG) hit harder on every shot.

### Stat Tradeoffs (v6)
Allocation is a real choice — some stats compete:

| Stat | Effect | Tradeoff |
|------|--------|----------|
| Fortitude | +5 Max HP | — |
| Might | +1 Damage | — |
| Agility | +15 Move speed | — |
| Haste | -30ms attack delay | -0.5 damage per shot |
| Armor | +2% damage reduction | -3 move speed |

Pure-Might or pure-Haste DPS-stack is strong; mixing 50/50 is a trap. Pure-Armor tank vs pure-Agility skirmisher are both viable, but stacking both fights itself.

### Combo Pierce (v6)
Combo isn't just an XP multiplier — it's a combat resource. Bullets pierce extra enemies as your kill streak climbs, rewarding tight aim and target-line setup:

| Combo | Effect |
|-------|--------|
| 5+ | Pierce 1 enemy |
| 10+ | Pierce 2 enemies |
| 15+ | Pierce 3 enemies |

Combo decays after 2 seconds of no kills. High-combo Volley = wall clearer.

## Tech Stack

- **Phaser 3** (CDN) — Game engine
- **Vanilla JS** — Zero build step
- **Single HTML file** — Drop into GitHub Pages

## Roadmap

- [ ] Firebase backend (Auth + Firestore for persistent characters)
- [ ] Multiplayer modes: Competitive, Teams (2-5 player), Invasion (co-op)
- [ ] XStellar sprite assets replacing procedural graphics
- [ ] Sound (Tone.js procedural SFX)
- [ ] Boss waves
- [ ] More enemy types (Brood Mothers, Tunnelers)
- [ ] Leaderboards
- [ ] Mobile touch controls

## License

Part of the XStellar universe. All rights reserved.
