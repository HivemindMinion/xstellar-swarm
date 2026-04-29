# XStellar: Swarm — Claude Code Handoff Prompt

## How to use
Copy everything below the `---` line and paste it as your initial prompt in a Claude Code session (Opus 4.7, xhigh effort).

---

You are picking up an in-progress game project called **XStellar: Swarm**. The project lives at `C:\Users\tanki\Projects\xstellar-swarm\`. Read the full `index.html` and `README.md` before doing anything else.

## Project Summary

XStellar: Swarm is a **medieval castle defense game** — imagine Galaga's wave-based shooting but set on a castle wall defending against insectoid bug enemies. It's part of a larger universe called XStellar (a Unity RPG with 1,430+ scripts and deep lore). The game is built as a **single HTML file** using **Phaser 3** (loaded from CDN), with zero build step, designed to be hosted on **GitHub Pages**.

## What's Already Built (v5)

Read the full `index.html` to understand the current architecture. Here's what exists:

### Core Combat
- **Mouse-aimed projectiles**: Left click fires toward cursor position. Projectiles travel in any direction, not just upward. Aim line rendered on player character.
- **WASD/Arrow movement**: Player moves left/right along the castle wall battlements.
- **Wave system**: Enemies spawn in formation at top, advance toward player. Waves auto-chain on completion (1.5s delay). Wave N spawns `6 + N*2` enemies.
- **Three enemy types**: Crawlers (ground formation, animated legs), Flyers (dive-bomb, appear wave 3+), Spitters (shoot acid, appear wave 5+). All procedurally drawn with Phaser graphics.
- **Combo system**: Kill streak multiplier on XP gains, resets after 2s idle.

### RPG Progression
- **Five stats**: Fortitude (+5 HP), Might (+1 damage), Agility (+15 move speed), Haste (-30ms fire delay), Armor (+2% damage reduction).
- **XP curve**: `level * 80 + 20` XP per level.
- **Death-only allocation**: Points accumulate during combat (shown in HUD as "◆ N points pending"), allocated only after dying. This is intentional design — do NOT change this.
- **Roguelite loop**: "Fight Again" keeps all stats/specs and restarts from Wave 1. "New Character" wipes save.

### Class Specialization System
- **Four starting classes**: Ranger, Knight, Mage, Cleric — each with unique base stat distributions, procedural character art, and projectile visuals.
- **Class Mastery**: Players can spend stat points into ANY class's mastery tree (not just their starting class). Each mastery has 5 levels.
- **Four specials (right-click)**:
  - **Ranger — Volley**: Fires cone of arrows toward cursor (3→11 arrows, scaling spread/damage/cooldown)
  - **Knight — Shield Wall**: Invulnerability bubble (1.5s→3.5s duration)
  - **Mage — Arcane Nova**: AoE damage ring around player (5→20 damage, 90→210px radius)
  - **Cleric — Divine Mend**: Instant heal (20%→60% max HP)
- **Multi-spec**: If multiple specials unlocked, Tab cycles between them. HUD shows active special + cooldown bar.

### Persistence
- **SaveManager**: Abstraction layer over localStorage. Versioned schema (`xstellar_swarm_v5` key). Saves: class, level, XP, stat points, all stats, class spec levels, active special, total kills, highest wave, run count.
- **Save triggers**: On death, after stat allocation, on character creation.
- **Title screen**: Detects existing save, shows "Continue" with character summary vs "Press Any Key" for new game.

### Visual Design
- **Medieval dark theme**: Stone castle wall with merlons, grass battlefield with dirt path, distant treeline, flickering torches (3 on wall, animated flames).
- **Procedural everything**: All characters, enemies, projectiles, and UI drawn with Phaser Graphics — no sprite assets yet.
- **Juice**: Screen shake on hit, particle explosions (colored by enemy type + green bug blood), invincibility flash frames, combo counter, wave announcements with flavor text.
- **HUD**: Stone-framed HP bar (color shifts green→orange→red), XP bar, level display, kill counter, pending stat points indicator, special ability indicator with cooldown bar.
- **Scenes**: TitleScene → ClassSelectScene → GameScene → LevelUpScene (on death, if points) → GameOverScene. All use serif fonts, gold accents, parchment-dark palette.

### Architecture Notes
- `PAL` object holds all color constants (hex ints for Phaser, CSS strings for text).
- `CLASSES` defines starting class data (name, desc, color, accent, base stats).
- `CLASS_SPECS` defines mastery trees with per-level scaling arrays.
- `PLAYER` is the global mutable state object. `createPlayer()` / `loadPlayer()` manage it.
- `SaveManager` is designed to be swapped to Firebase — only `save()`, `load()`, `hasSave()`, `clear()` need rewriting.
- Stat functions (`getMaxHP()`, `getDamage()`, etc.) derive from `PLAYER.stats`.
- Enemy spawning, AI, and rendering are all inline in GameScene. When this gets bigger, these should be extracted.
- Bullets use Phaser Graphics objects with `setData()` for velocity/damage. Not using Phaser physics bodies — all collision is distance-based.

## Game Modes Architecture (Not Yet Built)

The game is designed for three multiplayer modes, none implemented yet:

1. **Competitive** (FFA): Everyone plays simultaneously on their own screen. You can see other players' health bars. Survive until you die. Last one standing wins.
2. **Teams** (2v2, 3v3, 4v4, 5v5 in 9-10 player lobbies): Teammates visible on your screen. Enemy team shown as health bars only. Team elimination.
3. **Invasion** (Co-op): All players on one team, all visible on screen with individual health bars. PvE survival.

Backend plan: Firebase free tier — Auth (anonymous or Google), Firestore (player profiles, inventory, persistent stats), Realtime DB (lobby state, live match sync), Cloud Functions (anti-cheat validation, loot rolls).

## What Needs to Happen Next

### Immediate Priority: GitHub Pages Deployment
1. Initialize the git repo at `C:\Users\tanki\Projects\xstellar-swarm\`
2. Ensure the structure works for GitHub Pages (index.html at root = good)
3. The repo should be ready to push to GitHub and enable Pages on `main` branch

### Phase 1: Game Polish (Single Player)
- **Sound**: Add procedural SFX via Tone.js (fire, hit, kill, wave start, level up, death, special activation). Keep it medieval — twangy bowstrings, metallic clangs, arcane whooshes.
- **Boss waves**: Every 5th wave introduces a boss bug (large, multi-phase, unique attack patterns). Ideas: Brood Mother (spawns mini-crawlers), Tunneler (burrows underground, emerges near player), Hive Queen (shields nearby bugs).
- **More enemy variety**: Add 2-3 new bug types for mid/late waves.
- **Visual polish**: Parallax scrolling on the battlefield layers. Weather effects (rain, fog). Day/night cycle across waves.
- **Balance pass**: Playtest the stat scaling. Currently damage reduction caps at 75%, fire delay floors at 80ms. XP curve may need tuning for longer sessions.

### Phase 2: Firebase Backend
- Swap `SaveManager` internals from localStorage to Firestore
- Add Firebase Auth (anonymous first, Google later)
- Cloud-synced player profiles with the same data schema
- Leaderboards (highest wave, total kills, per-class records)

### Phase 3: Multiplayer
- Lobby system with matchmaking
- Realtime game state sync via Firebase Realtime DB
- Implement Competitive, Teams, and Invasion modes
- Spectator mode after elimination

### Phase 4: Content
- Replace procedural graphics with XStellar sprite assets (spritesheets exist in the XStellar Unity project)
- Skill trees deeper than 5 levels
- Equipment/loot system
- More classes or sub-classes

## Development Conventions

- **File naming**: Version numbers (v1, v2, v3...). NEVER use `_fixed` or `_final` suffixes.
- **Single file for now**: Keep the game as `index.html` until complexity demands a build system. When splitting, use ES modules.
- **Dark theme**: All UI must be dark-themed. No light backgrounds anywhere.
- **Privacy first**: localStorage for single-player persistence. When adding Firebase, minimize data collection. No analytics beyond what's needed for gameplay.
- **No pseudo-code**: All code must be complete and working.
- **Comments**: Brief and focused on data flow, not obvious operations.
- **Error handling**: Cover practical edge cases (~80%), don't over-engineer for fringe scenarios.

## Important Context

- This is part of the **XStellar universe** — a larger world with established lore (Book Bible, 1,430+ Unity scripts, soundtrack). The bugs are a canonical enemy faction. Medieval setting is one theater of the XStellar conflict.
- The game's theme is **Humans vs. Bugs** specifically. No other factions for now.
- The player defends from the **bottom** of the screen (standing on castle wall), enemies approach from the **top** (classic Galaga orientation but grounded in medieval aesthetics).
- The game canvas is **480×720** (portrait orientation) with `Phaser.Scale.FIT`.

Start by reading the full codebase, then ask me what I'd like to work on first.
