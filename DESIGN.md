# DESIGN.md
The source of truth for WHAT the game is. Read at the start of every
session. Do NOT change this file without my explicit approval. Any
approved change must be logged in the Change Log at the bottom with a
timestamp (YYYY-MM-DD HH:MM, from the system clock). Values marked
"PROPOSED DEFAULT" are starting points to be tuned in play, not rules.

## 1. Vision and Design Pillar
- Genre: 2D platformer roguelike with tight, momentum-based
  platforming in the spirit of Super Mario Bros. 3.
- The platforming skill is the constant; the build, the level layout
  and the risk/reward choices are the variables.
- DESIGN PILLAR: Collect enemy abilities by stomping them (an
  original take on copy-abilities).
- All characters, art, music, names and layouts are ORIGINAL.

## 2. Design Principles
1. Feel first: movement is the game. Tune it before building content.
2. Simple verbs, deep variations: run, jump, stomp, build.
3. Teach by doing, not by text: Introduce -> Develop -> Twist ->
   Conclude.
4. Telegraph everything: the player must always see why they got hurt.
5. Alternate tension and rest; never run at 100% intensity.
6. Every item must change HOW the player approaches a level, not just
   give raw stats.
7. Failure is cheap and fast: short levels, quick restarts.
8. Skill over stats: meta-progression unlocks options, never raw
   power boosts.

## 3. Core Loop
Move -> Fight (stomp/shoot) -> Collect coins and items -> Choose next
node -> Spend/choose upgrades -> Repeat -> Boss -> Run ends
(victory or death) -> Start a new run.
The loop must be fun on its own before anything else is added.

## 4. Player Movement Spec
Features (all values live in PlayerMovementConfig):
- Acceleration/deceleration (momentum), walk speed vs run speed.
- Skid/turn-around when reversing at speed.
- Separate ground and air friction; reduced but present air control.
- Variable jump height; stronger gravity on the way down; faster
  horizontal speed gives a slightly higher/longer jump.
- Terminal fall velocity.
- Forgiveness: coyote time (PROPOSED DEFAULT 6 frames), jump
  buffering (PROPOSED DEFAULT 6 frames), corner correction on head
  bumps, player hitbox smaller than the sprite.
- Ducking.
- States: Idle, Walk, Run, Skid, Jump, Fall, Duck.
- P-meter: sustained full-speed running fills it; when full, speed
  increases and flight-type items can use it.

## 5. World and Tiles
- 16x16 px tiles, Pixels Per Unit 16, 1 tile = 1 Unity unit.
- Tile types: Empty, Solid, OneWay, Breakable, QuestionBlock,
  HiddenBlock, Hazard (MVP); Slope, Water, Climbable, Pipe (post-MVP).
- Foreground (interactive) must be clearly distinct from background
  (decor) by color, contrast and outline.
- Consistent iconography: ? = reward, brick = breakable.
- Collision resolves X axis first, then Y axis.

## 6. Camera
- Follows the player with a dead zone (centered or slightly offset),
  run-direction look-ahead, and level-bounds clamping.
- Modes: Free (default, bidirectional horizontal follow), OneWay,
  VerticalFollow, AutoScroll (post-MVP).

## 7. Health and Damage
- Hearts-based health (hybrid of small/big feel). Max HP is modified
  via StatModifier. PROPOSED DEFAULT: start with 3 hearts.
- Brief invulnerability frames with blink after damage, plus
  knockback.
- Death causes: pit, hazard, enemy contact at 0 hearts, timer-out,
  crush.
- Death ends the run (no lives). One revive item exists (one-time
  extra life per run).
- Level timer: kills at zero; also used as a reward (par-time bonus).

## 8. Combat
- STOMP RULE: if the player is falling AND overlaps the enemy's top
  region, the enemy is damaged and the player bounces (higher if jump
  is held). Otherwise the player takes contact damage.
- STOMP ABILITIES: stomping an enemy drops an Ability Orb; touching
  it equips that enemy type's ability into ONE ability slot (a new
  one replaces the old one). PROPOSED DEFAULT: the ability is lost
  when the player takes damage.
- Stomp chain: consecutive stomps without touching the ground raise a
  chain count (fires OnStomp with the count).
- Momentum Combo: chaining run, stomp and bounce without touching the
  ground builds a multiplier on coin/score rewards.
- Enemy hitbox against the player is slightly smaller than its sprite.

## 9. Enemies (MVP: exactly 3 types)
1. Walker: walks, turns at walls and (optionally) ledges. Ability:
   Charge Dash.
2. Shelled Walker: stomp -> shell; kicking the shell makes it a sliding
   projectile that can damage other enemies. Ability: Shell Spin
   (kick-slide through enemies).
3. Flyer: sine or patrol movement. Ability: Flutter (limited hover).
- Each enemy is a small state machine (Walk, Stunned, Shell, Sliding,
  Dead as needed) driven by an EnemyData ScriptableObject.
- Enemies drop coins, hearts or temporary buffs via DropTables, plus
  an Ability Orb on stomp.
- Abilities are data-driven (ScriptableObjects) and use the
  StatModifier/state system.
- Enemies teach by placement: alone, then pairs, then with terrain
  complications.
- Placement in generated levels follows each chunk's enemy budget.

## 10. Items (MVP target: 15-20)
Rarities: Common, Uncommon, Rare, Cursed. Items stack up to a
per-item max. All changes go through the StatModifier system.
PASSIVES (Part 7): Double Jump, Wall Jump (with wall slide), Air Dash,
  Glide, Extra Heart, Coin Magnet.
MOVEMENT/COMBAT (Part 8): Ground Pound (breaks blocks, damages
  enemies), Shockwave (on stomp), Higher Bounce, Chain Bonus,
  Fireball (bouncing), Piercing, Split Shot.
DEFENSIVE (Part 8): Recharging Shield (restores each level), Damage
  Reflect.
ECONOMY (Part 8): Bigger Coin Drops, Shop Discount.
ACTIVE, one slot, cooldown-based (acts like SMB3's reserve item):
  Time Slow, Short Teleport.
ABILITY SLOT (replaces separate Suit slot): one slot, new replaces old.
  Stomp-acquired abilities (Charge Dash, Shell Spin, Flutter) equip here.
  Flight Suit becomes the Flyer ability; Tank Suit is cut from MVP.
CURSED (1 for MVP): an item with a powerful upside and a real
  downside.
- Item count is now 20 (suits removed). Do not exceed 20 for
  MVP. If something must be cut, cut in this order: Split Shot,
  Shop Discount, Bigger Coin Drops.
- Synergy rule: items can check other owned items' tags. MVP includes
  ONE simple synergy as the proof of concept.

## 11. Level Generation
- Levels are assembled from hand-built chunks (1-2 screens each),
  tagged by difficulty and theme, using a SEEDED random generator
  (same seed = same level/run).
- Structure: Start chunk, N middle chunks, Midpoint (checkpoint)
  chunk, Goal chunk.
- Connection rule: a chunk's exit height must match the next chunk's
  entrance height. Every level must have a guaranteed beatable path.
- MVP content: 20 chunks (about 7 easy, 8 medium, 5 hard, including 3
  breathers/rest chunks).
- Level length: roughly 1-3 minutes.
- A ChunkValidator tool must check connectors and reachability using
  the max jump distance from PlayerMovementConfig.
- Each level uses the teaching pattern, with breathers between
  intense sections. Coins act as breadcrumbs showing ideal paths.
- Chunks are authored as plain-text grids (one character per tile) and imported into Unity by an editor tool; the validator works on the text grids using the base kit only (no items or abilities).

## 12. Run Structure
- A run is 5 levels with branching choices, ending in a boss.
- After each level, choose from 2-3 nodes. MVP node types:
  Normal, Shop, Rest, Treasure.
- Run flow: Title -> Map -> Level -> Reward -> Map ... -> Boss ->
  Run Complete or Run Over -> Title.
- A death ends the run and shows a summary (seed, items, coins,
  cause of death).
- Run state: seed, node graph, inventory, coins, hearts.

## 13. Economy and Nodes
- Coins: earned from blocks, enemies and the level; spent in shops.
  100 coins grants a bonus (PROPOSED DEFAULT: one extra heart).
- Shop: limited stock from a rarity-weighted item pool; reroll costs
  coins; items can be sold.
- Rest: heal or upgrade. Treasure: choose 1 of 3 items.
- Speed bonus: finishing under par time grants extra loot.

## 14. Boss (MVP: 1)
- 3 phases, each with a clear telegraph and distinct attacks.
- Vulnerable windows tied to stomps/projectiles (3-hit style rule).
- Arena: camera locked, exit sealed until defeat.
- Reward on defeat: choose 1 of 3 items.

## 15. Difficulty Curve
- Start easy, ramp gradually within the run, with a small reset at
  the start of each new world (post-MVP).
- Optional challenges (secrets, hidden blocks) serve skilled players
  without blocking others.

## 16. Juice and Audio
- Screen shake, hit-stop, squash and stretch, particle bursts for
  stomps, coins and item reveals.
- Distinct SFX for jump, coin, stomp, power-up, damage, 1-up; pitch
  rises with combos.
- Clear item-pickup moment: brief pause, flash, name and description
  pop-up.
- Art: limited palette per sprite, 16x16 tiles. Music: loopable;
  layered dynamic music post-MVP.

## 17. UI
- HUD: hearts, coins, timer, active-item cooldown, ability icon, combo,
  P-meter.
- Screens: title, node map, shop, treasure, pause, run summary.
- UI is read-only against game state and updates via events.

## 18. Post-MVP Backlog (PARKED - do not build until I approve)
- Characters: Speedster, Bruiser, Floater, Trickster, Collector, each
  with a unique mechanic, not just stats.
- Meta-progression: unlocks new items into the pool (never raw stat
  boosts), hub, bestiary/item compendium, achievements.
- Worlds: Meadow, Desert, Ice Caverns, Underwater, Volcano, Sky
  Islands, Machine Factory, Haunted Manor, Final. Level modifiers:
  Low Gravity, Darkness, Ice Floor, Wind, Auto-scroll, Rising Lava.
- Enemies: Shooter, Charger, Shielded, Stationary hazard, Elite
  modifiers (Explosive, Armored, Fast, Splitting, Healing Aura),
  mini-bosses, extra bosses.
- Risk/reward: more cursed items, gamble rooms, greed mechanic, risk
  gates, pre-level challenge modifiers, Mystery Event nodes.
- Movement/tiles: slopes, water/swim, vines/ladders, pipes/warps,
  swinging ropes.
- Items: Homing Fireball, Summon Helper, Bomb, Grapple, Swim/Ghost/
  Bounce abilities (ability-slot abilities), more synergies.
- Spice: daily seed + leaderboard, ghost replays, mutators/ascension,
  co-op, boss rush/endless, secret rooms and alternate endings, NPCs,
  mini-games, map inventory, dynamic music.

## 19. Open Design Questions (decide with me; do not assume)
- Starting hearts and exact heart/damage values.
- Exact par-time rule for the speed bonus.
- Which synergy is the MVP example.
- Name of the game and original character concept.
- Ability loss rule, orb drop rate, and ability duration need tuning in play.

## 20. Change Log (append-only; never edit old entries)
Format: [YYYY-MM-DD HH:MM] Section changed - what and why
- [2026-10-05 20:49] Initial DESIGN.md created.
- [2026-10-05 21:15] Updated the design pillar and added notes about original character ability concept.
- [2026-10-05 21:49] Section 1 - Correction: the earlier entry mentioning character ability notes was inaccurate; no such notes were added. Pillar formatting cleaned up.
- [2026-10-05 21:52] Sections 8, 9, 10, 19 - Added stomp-acquired enemy abilities spec (Walker -> Charge Dash, Shelled Walker -> Shell Spin, Flyer -> Flutter). Ability slot replaces separate Suit slot; Flight Suit becomes Flyer ability, Tank Suit cut from MVP. Stomp ability tuning questions added.
- [2026-10-05 21:55] Sections 10, 17, 18, 20 - Updated item count to 20 (suits removed), HUD updated with ability icon, backlog suits updated to abilities, and restored original wording of 21:15 entry.
- [2026-10-06 20:24] Section 6 - Updated default camera mode from OneWay to Free (bidirectional horizontal follow) per user preference.
- [2026-10-07 23:06] Section 11 - Added specification that chunks are authored as plain-text grids (one character per tile) and imported into Unity by an editor tool; the validator works on the text grids using the base kit only (no items or abilities).
