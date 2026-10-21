# ITEM BALANCE REPORT (Part 17 MVP Gate)

**Date**: 2026-10-14  
**Project**: PlatformRLike (2D Platformer Roguelike)  
**Total MVP Items**: 21 items (`Assets/Resources/Items/*.asset`)  
**Scope**: Verification of game balance, trivialization risks, infinite exploit prevention, and boss encounter integrity.

---

## 1. Executive Summary

A comprehensive balance and mechanics audit was conducted across all 21 items implemented for the MVP. Every item's stat modifiers, stacking rules, input handling, physics interactions, and boss encounter combat behaviors were evaluated against the core design principles defined in [`DESIGN.md`](file:///g:/UnityHub/Projects/PlatformRLike/DESIGN.md):

1. **Boss Encounter Integrity**: `ColossusBoss` utilizes a discrete 3-phase hit state machine (`_maxHits = 3`) with vulnerable windows (steam vent, dazed wall crash, overheat). Incoming damage outside vulnerable windows is deflected with an armored clang. Furthermore, `ApplyBossHit` decrements HP by exactly 1 per hit regardless of player damage multipliers. Thus, attack buffs (e.g., `blood_pact` +100% damage, `fireball`, `split_shot`, `piercing_shot`) **cannot bypass the 3-phase boss encounter or 1-shot the boss**.
2. **Infinite Mobility & Pit Trivialization**:
   * Air dash (`air_dash`) enforces a strict 0.35s cooldown timer (`_dashCooldownTimer`), preventing infinite horizontal flight.
   * Double jump (`double_jump`) is capped at 3 stacks (3 air jumps maximum).
   * Glide (`glide`) scales terminal fall velocity down but does not add horizontal force; player cannot stall indefinitely in the air.
   * Teleport (`short_teleport`) casts player BoxCollider2D against solid layers (`Default`, `OneWay`) to clamp distance, preventing players from clipping into solid walls or out of level bounds.
3. **Defensive Stacking**:
   * `recharging_shield` is capped at 1 stack, providing exactly 1 hit absorption per level.
   * `second_chance` provides a single 1-time revival per run, safely recovering the player to the active checkpoint if death occurred via pit.
   * `extra_heart` is capped at 10 stacks, granting +1 max heart per stack. In a standard 5-level run, economy and shop offerings naturally limit players to 2–4 defensive items.

**Verdict**: **No item or combination of items trivializes or breaks the game.** The progression loop offers meaningful build variety while preserving the platforming skill ceiling.

---

## 2. Comprehensive Item-by-Item Breakdown

### A. Passive Mobility Items (6)

| Item ID | Display Name | Rarity | Max Stacks | Core Effect | Balance & Trivialization Assessment |
| :--- | :--- | :--- | :---: | :--- | :--- |
| `double_jump` | Double Jump | Uncommon | 3 | `DoubleJumpItemEffect`: +1 air jump per stack | **Balanced**. Allows up to 3 mid-air jumps at max stacks. Enables reaching secret cache ledges without breaking level bounds (clamped by camera confiner). |
| `wall_jump` | Wall Claws | Uncommon | 1 | `WallJumpItemEffect`: Enables wall-slide and wall jump | **Balanced**. Single stack. Wall slide caps fall speed to 3.0 m/s; wall jump pushes player outward and upward. |
| `air_dash` | Feather Dash | Rare | 2 | `AirDashItemEffect`: +1 mid-air horizontal dash per stack | **Balanced**. 0.35s cooldown timer prevents infinite chaining; mid-air dash preserves horizontal velocity clamp on completion. |
| `glide` | Paraglider | Uncommon | 1 | `GlideItemEffect`: Hold jump while falling to glide | **Balanced**. Reduces fall speed to 2.5 m/s while jump is held; horizontal friction continues to apply, preventing horizontal infinite flight. |
| `ground_pound` | Ground Pound | Common | 1 | `GroundPoundItemEffect`: Down in air triggers vertical stomp slam | **Balanced**. Destroys breakable bricks beneath player, deals 2 damage to enemies on landing. Cannot be aborted mid-descent, exposing player to ground hazards. |
| `higher_bounce` | Higher Bounce | Common | 3 | `HigherBounceItemEffect`: +30% stomp bounce velocity per stack | **Balanced**. Stacking up to +90% bounce gives high vertical recovery off enemy stomps, synergizing with stomp chains without skipping level geometry. |

---

### B. Combat & Projectile Items (5)

| Item ID | Display Name | Rarity | Max Stacks | Core Effect | Balance & Trivialization Assessment |
| :--- | :--- | :--- | :---: | :--- | :--- |
| `fireball` | Fireball | Uncommon | 1 | `FireballItemEffect`: Fire bouncing fireballs (F / J) | **Balanced**. Bounces off solid floors up to 3 times, destroyed on contact with walls or unpierced enemies. Deals 1 damage to standard enemies. Deflected by boss armor outside vulnerable windows. |
| `piercing_shot` | Piercing Shot | Uncommon | 2 | `PiercingItemEffect`: Fireballs pierce +2 enemies per stack | **Balanced**. Max 4 pierces at 2 stacks. Clears tight enemy corridors, but single projectiles still dissipate against solid walls. |
| `split_shot` | Split Shot | Rare | 1 | `SplitShotItemEffect`: Fireballs fire in a 3-way spread (-15°, 0°, +15°) | **Balanced**. Excellent wide-area coverage; each projectile consumes separate pierce counts and collides with terrain. |
| `stomp_shockwave` | Shockwave | Uncommon | 3 | `ShockwaveItemEffect`: Stomps emit lateral ground shockwave | **Balanced**. Deals 1 damage within 2.5m radius. Requires landing an actual stomp; does not trigger on empty landings or damage taken. |
| `chain_bonus` | Chain Bonus | Rare | 1 | `ChainBonusItemEffect`: Grants +2 bonus coins on stomp chains >= 2 | **Balanced**. Rewards high-skill aerial stomps without touching the ground; rewards economy, not combat invulnerability. |

---

### C. Defense & Health Items (4)

| Item ID | Display Name | Rarity | Max Stacks | Core Effect | Balance & Trivialization Assessment |
| :--- | :--- | :--- | :---: | :--- | :--- |
| `extra_heart` | Extra Heart | Common | 10 | `ExtraHeartItemEffect`: +1 Max Heart | **Balanced**. Restores +1 current heart on pickup. Starting HP is 3; typical run yields 1–3 hearts. High stack counts require heavy coin investment at shops. |
| `recharging_shield` | Recharging Shield | Rare | 1 | `RechargingShieldItemEffect`: Absorbs 1 hit per level | **Balanced**. Single stack. Protects against 1 mistake per level. Does not absorb instant kills (pits, crush, time-out). Recharges only when transitioning to a new level. |
| `second_chance` | Second Chance | Rare | 1 | `ReviveItemEffect`: One-time extra life per run | **Balanced**. Single stack, consumed on fatal hit. Resets hearts to max, triggers 1.5s invulnerability, and teleports player to active checkpoint if death was by pit. |
| `damage_reflect` | Damage Reflect | Uncommon | 3 | `DamageReflectItemEffect`: Deals 1 thorns damage in 3m radius on hit | **Balanced**. Deals retaliatory damage when player takes damage. Player still suffers standard heart loss and knockback. |

---

### D. Economy & Utility Items (4)

| Item ID | Display Name | Rarity | Max Stacks | Core Effect | Balance & Trivialization Assessment |
| :--- | :--- | :--- | :---: | :--- | :--- |
| `coin_magnet` | Coin Magnet | Common | 5 | `CoinMagnetItemEffect`: Pulls coins from up to 4m per stack | **Balanced**. Quality-of-life economy booster. Smoothly attracts nearby coin pickups via kinematic velocity. |
| `bigger_coins` | Bigger Coin Drops | Common | 3 | `BiggerCoinDropsItemEffect`: +50% coin value per stack | **Balanced**. Multiplies coin income (+50% to +150%), enabling more shop purchases in later levels. |
| `shop_discount` | Shop Discount | Uncommon | 2 | `ShopDiscountItemEffect`: -25% shop item prices per stack | **Balanced**. Max 50% discount at 2 stacks. Encourages shop-heavy node pathing on the world map. |
| `blood_pact` | Blood Pact | Cursed | 1 | `CursedPactItemEffect`: +100% damage & coins, -1 Max Heart | **Balanced**. Classic high-risk / high-reward cursed item. Reduces player maximum health to 2 (from default 3), making boss shockwaves and hazards significantly deadlier. |

---

### E. Active Slot Items (2)

| Item ID | Display Name | Rarity | Max Stacks | Core Effect | Balance & Trivialization Assessment |
| :--- | :--- | :--- | :---: | :--- | :--- |
| `short_teleport` | Short Teleport | Rare | 1 | `TeleportActiveEffect`: Teleport forward 4 units (Cooldown: 8s) | **Balanced**. Checks `Physics2D.Cast` against solid layers (`Default`, `OneWay`) to stop safely before terrain, completely preventing clipping out of bounds. |
| `time_slow` | Time Slow | Rare | 1 | `TimeSlowActiveEffect`: 60% time dilation for 3s (Cooldown: 15s) | **Balanced**. Smoothly scales `Time.timeScale` to 0.4 for 3 seconds while keeping player responsive. Long 15s cooldown limits usage to critical platforming or boss dodge moments. |

---

## 3. Synergy & Edge-Case Audit

1. **Fireball + Split Shot + Piercing Shot**:
   * *Observed*: Fires 3 piercing fireballs simultaneously. High screen-clear capability.
   * *Boss Interaction*: The Colossus Boss ignores all projectile hits during invulnerable movement phases (clang deflection). During vulnerable phases (exhausted, dazed), any single hit deals 1 damage and immediately triggers the phase-transition stagger, preventing multiple simultaneous damage ticks.
2. **Air Dash + Double Jump + Glide**:
   * *Observed*: Player can jump, air-jump 3 times, dash horizontally, and glide down.
   * *Boundaries*: Level ceilings, boundary walls, and CinemachineConfiner2D prevent bypassing chunks. Chunks require rightward progression where hazard pits span 6–10 units; high-mobility builds feel exhilarating without sequence-breaking level triggers.
3. **Blood Pact + Damage Reflect**:
   * *Observed*: Player takes high risk with 2 max hearts, reflecting damage to surrounding enemies.
   * *Risk*: With only 2 hearts, taking hits to deal reflect damage is high risk and disincentivized, maintaining game balance.

---

## 4. Conclusion & Tuning Recommendations

* **All 21 item assets have valid ScriptableObject bindings, appropriate stack limits, and zero null effect references.**
* **Boss encounter mechanics are impervious to one-shot damage abuse due to the state-machine hit cap.**
* **Kinematic collision casting guards teleportation and dash movement against clipping bugs.**
* **No code changes or tuning nerfs are required; items conform 100% to Part 17 MVP gate requirements.**
