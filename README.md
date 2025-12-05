

> `AS_ABOVE_SO_BELOW_SOS_MASTER_GUIDE.md`
**all-in-one info center + instruction manual for AI**: structure, systems, how to build modules, and ready-made prompts.

---

````markdown
# AS ABOVE, SO BELOW – SOS MASTER GUIDE
### System Operations Specification & AI Work Protocol

_Engine: Godot 4 (2D)_  
_Style: Top-down / action-RPG / D&D-inspired_  
_Core Idea: Space frontier life-sim + dungeon-delving + living world run by a GameMaster Engine_

---

## 0. HOW TO USE THIS DOCUMENT (READ ME FIRST)

This document is an **instruction manual for AI assistants** (and future humans) working on the game **“As Above, So Below”** (working title).

- “**As Above**” = orbit, stations, ships, politics, contracts.  
- “**So Below**” = planets, mines, ruins, war-scars, anomalies, deep weirdness.  
- “**SOS**” = System Operations Specification – this document.

If you are an AI assistant:

1. **Read this entire document**  before editing code or assets.
2. Treat everything here as **design constraints** unless the user explicitly overrides them.
3. When adding or changing systems, keep them aligned with:
   - The **architecture** in Section 2–3  
   - The **Godot structure** in Section 4  
   - The **systems guidelines** in Section 5–7

If you are the human owner:

- You can paste **parts of this doc** into new AI sessions as context.  
- Sections 8–9 contain **ready-to-use prompts** for specific tasks (combat engine, economy, scenes, etc.).

---

## 1. PROJECT IDENTITY & VISION

### 1.1 Short Pitch

> A 2D top-down, D&D-style space frontier RPG where the game itself acts like a Dungeon Master: you sign contracts in stations and bars, manage a fragile ship and limited resources, explore dangerous planets and depths, and slowly uncover rare, dangerous magic and AI-war secrets as a living world shifts above and below you.

### 1.2 Design Pillars

1. **Living Frontier, Not Static Levels**  
   The world changes between missions: factions rise, prices shift, systems fall into chaos or stability.

2. **Careers & Life Choices**  
   You live as a frontier contractor with unlockable careers and advanced paths, not a fixed class.

3. **D&D-Inspired Rules**  
   Attributes, skills, checks, rests, exhaustion, conditions, rare spell-like powers.

4. **Rare, Dangerous Magic**  
   Magic is scarce, potent, and risky. It’s tied to anomalies, relics, AI shards, deep ruins.

5. **Resource Loops & Logistics**  
   Taps → inventory → converters → traders → drains.  
   Everything you do interacts with an economy of fuel, ammo, ore, repairs, debt, reputation, and time.

6. **GameMaster Engine**  
   An internal “DM” simulates factions, clocks, events, contracts, and news.  
   The player is one person; the world is many minds.

---

## 2. HIGH-LEVEL ARCHITECTURE

The game is built as **layers**. AI should always respect these layers when coding:

1. **Lore Layer** – Setting, factions, region data (markdown + JSON).  
2. **Rules Layer** – TTRPG rules engine (attributes, checks, combat math, rests, magic resources).  
3. **GameMaster Engine (GM Engine)** – World simulation and mission generator.  
4. **Game Systems Layer** – Inventory, contracts, economy, quest tracking.  
5. **Presentation Layer (Godot 4)** – Scenes, sprites, animations, input, audio, UI.

### 2.1 Layer Responsibilities

**Lore Layer**

- Markdown + data files describing:
  - Region map, important locations, factions.
  - AI war backstory, anomalies, magic logic.
- Consumed by GM Engine and Rules as reference, not code.

**Rules Layer (Engine-agnostic)**

- Implements:
  - Attributes, skills, derived stats.
  - XP and leveling.
  - Combat math (hit, damage, crit, conditions).
  - Rest types and exhaustion logic.
  - Magic resource: Focus / corruption, costs and consequences.
- Written as **pure code** with no Godot API calls where possible (easy to test).

**GameMaster Engine (GM Engine)**

- Tracks:
  - Locations, factions, clocks, events, contracts, global economy parameters.
- Provides:
  - `world_tick()`, `generate_hooks()`, `apply_contract_outcome()`, `get_news_feed()`.
- Uses the Rules Layer for:
  - Difficulty scaling and checks (e.g., mission difficulty, DCs, expected rewards).

**Game Systems Layer**

- Godot scripts that connect:
  - Rules + GM Engine to in-scene behavior.
- Examples:
  - `inventory.gd`, `contracts.gd`, `economy.gd`, `quests.gd`, `save_load.gd`.

**Presentation Layer (Godot 4)**

- Scenes:
  - Station / hub, ship interior, planet, dungeon, UI overlays.
- Handles:
  - Rendering, input, camera, physics, animations, SFX, VFX.
- Calls into:
  - Rules Layer to resolve actions.
  - GM Engine for world state and missions.

---

## 3. GODOT 4 PROJECT STRUCTURE (TARGET)

AI should strive to keep or refactor the project into a structure like this:

```text
res://
  scenes/
    main_menu/
      MainMenu.tscn
    hub/
      HubScene.tscn
      Bar.tscn
      ContractBoard.tscn
    ship/
      ShipScene.tscn
    planet/
      PlanetScene.tscn
    dungeon/
      DungeonScene.tscn
    ui/
      HUD.tscn
      InventoryUI.tscn
      ContractUI.tscn
      DialogUI.tscn

  autoload/
    GameState.gd    # player data, current location, meta
    Rules.gd        # wrapper around Rules core (or bridging to separate module)
    DMEngine.gd     # world state, factions, clocks, hooks
    Config.gd       # global config, constants

  scripts/
    systems/
      combat.gd
      inventory.gd
      economy.gd
      contracts.gd
      quests.gd
      save_load.gd
      input_map.gd
    entities/
      player.gd
      npc.gd
      enemy.gd
      projectile.gd
      interactable.gd
    world/
      location.gd
      map_generator.gd
      dungeon_generator.gd
    ui/
      hud_controller.gd
      dialog_controller.gd
      menu_controller.gd

  data/
    careers.json
    paths.json
    items.json
    weapons.json
    armor.json
    resources.json
    factions.json
    locations.json
    contracts.json
    economy_config.json

  docs/
    AS_ABOVE_SO_BELOW_SOS_MASTER_GUIDE.md
    (other design docs as they appear)

  art/
    sprites/
    tilesets/
    fx/

  audio/
    sfx/
    music/
````

**AI constraint:**
When adding new scripts or scenes, place them in the appropriate folder and follow the naming pattern.

---

## 4. CORE SYSTEMS OVERVIEW

This section tells AI what **systems** exist and what they should roughly do. Detailed prompts follow later.

### 4.1 Input & Controls

* Top-down 2D movement:

  * Keyboard/Controller:

    * Movement (WASD or left stick).
    * Aim (mouse or right stick) for shooting.
* Actions:

  * Primary fire.
  * Secondary fire / ability.
  * Interact (talk, loot, activate).
  * Inventory & character sheet.
  * Pause menu.
* AI goals:

  * Keep input mappings in one place (`input_map.gd`).
  * Use Godot InputMap for flexibility.

### 4.2 Player Character & Camera

Player:

* Holds:

  * Attributes, skills, HP, Focus, XP, career/path.
  * Weapons, armor, inventory reference.
* Capable of:

  * Moving, aiming, firing, interacting.
  * Triggering ability use (Rift, Relic, Code abilities later).

Camera:

* Follow player with smoothing.
* Optional screen shake on hits/explosions.
* Different zoom for hub vs planet vs dungeon, but consistent style.

### 4.3 NPCs & Enemies

NPCs:

* Non-combat (bartender, quest giver, ship tech).
* Scriptable dialog trees (data-driven).
* Expose hooks for DM Engine (e.g., “this NPC offers faction X contracts”).

Enemies:

* Configurable via data:

  * HP, damage, speed, behavior type (charger, shooter, burrower, etc.).
* Use simple state machines: idle, patrol, chase, attack, retreat, flee.

### 4.4 Combat (Guns, Lasers, Swords, AOE)

* Supports:

  * Hitscan weapons (instant line shots).
  * Projectiles (bullets, rockets, energy bolts).
  * Melee arcs (swords, blunt weapons).
  * AOE patterns:

    * Cones, lines, circles, zones.

* Integration with Rules:

  * Weapon defines base damage, crit chance, tags.
  * Rules decide hit/miss, crit, damage final.
  * Conditions (stun, slow, bleed, burn, shock) applied via Rules.

### 4.5 Inventory, Items & Equipment

Inventory:

* Player:

  * Weight/slot based capacity.
* Ship cargo:

  * Larger capacity, for bulk ore and trade goods.

Items:

* Categories:

  * Consumables (meds, stims, grenades).
  * Equipment (weapons, armor, tools).
  * Resources (ore, refined materials, anomaly samples).
  * Special (relics, AI shards, quest items).

Equipment:

* Slots:

  * Main weapon, secondary, melee.
  * Armor (body), accessories (2–3).
  * Tool slot (scanner, mining laser, etc.).

### 4.6 Economy & Resource Loops

Resources:

* Credits, fuel, ammo, meds, repair kits.
* Common ore, refined materials, rare minerals.
* Relics & anomaly samples.
* Reputation (per faction).
* Focus / corruption.

Loops model: **Taps → Inventory → Converters → Traders → Drains**

* Taps: mission rewards, loot, mining.
* Inventory: player & ship storage.
* Converters: crafting, refining, research, ship upgrades.
* Traders: station merchants, black markets, corp reps.
* Drains: fuel usage, ammo usage, repairs, docking fees, debt interest, time.

The DM Engine adjusts:

* Prices, availability, contract rewards based on faction control, clocks, and player history.

### 4.7 Contracts, Quests & DM Hooks

Contracts:

* Generated by GM Engine from templates + world state.
* Types: survey, escort, raid, salvage, rescue, smuggle, anomaly dive, relic hunt.
* Each has:

  * Objective(s), location, faction, reward, risk summary.

Quests:

* Story-driven arcs:

  * Background (personal), faction arcs, path unlocks.
* Can be triggered by:

  * GM Engine events.
  * Player choices, reputation thresholds, or discoveries.

GM Hooks:

* GM Engine uses:

  * WorldTick events to generate new contracts and update news.
  * Player actions to adjust clocks and faction state.

### 4.8 Save & Load

Save slots:

* Save:

  * Player state (stats, inventory, equipment).
  * GameState (current location, time, discovered locations).
  * DM Engine state (factions, clocks, ongoing events).
* Use:

  * `user://` path.
  * One file per slot, optionally with a meta file listing.

---

## 5. RULES LAYER SUMMARY (FOR AI IMPLEMENTATION)

### 5.1 Attributes and Skills (Minimum Required)

Attributes (scores + modifiers):

* Might (MIG)
* Agility (AGI)
* Intellect (INT)
* Grit (GRT)
* Presence (PRE)
* Luck (LCK)

Skills (examples):

* Guns, Heavy Weapons, Piloting, Stealth, Acrobatics
* Engineering, Hacking, Survey, Medicine, Science
* Endurance, Survival, Hazard Resist
* Persuasion, Intimidation, Deception, Streetwise
* Gambling, Scavenging, Serendipity

### 5.2 Checks, Saves, Combat Rolls

Skill Check:

* `d20 + Attribute_Mod + Skill_Bonus + Proficiency (if any) ≥ DC` => success.

Saving Throw:

* `d20 + relevant Attribute_Mod (+ Proficiency sometimes)`.

Combat:

* Attack:

  * `Attack_Roll = d20 + Attack_Bonus` vs Target Defense (Armor, Evasion, etc).
* Damage:

  * Based on weapon + attribute, modified by resistances, crits, conditions.

### 5.3 Rests & Exhaustion

Rest Types:

* Field Rest (Short)
* Ship Rest (Medium)
* Station Rest (Long + Downtime)

Exhaustion:

* 6 tiers; each adds penalties.
* Cleared mostly by Station Rest + med/therapy.

### 5.4 Magic & Weirdness

Magic implemented as:

* **Focus Points**: spells/abilities spend Focus.
* **Corruption**: track from overchanneling or exposure to weirdness.

Rift / Relic / Code abilities are:

* High impact, limited use, risk of backlash.
* Unlocked via advanced paths and story events.

---

## 6. GAMEMASTER ENGINE SUMMARY (FOR AI IMPLEMENTATION)

### 6.1 Core Data

GM Engine should manage:

* `Factions[]`
* `Locations[]`
* `Clocks[]`
* `Events[]`
* `Contracts[]`
* Global economic modifiers (e.g., base ore price, fuel scarcity).

### 6.2 Key Functions (API Style)

AI should aim to implement something like:

```gdscript
class_name DMEngine
extends Node

func init_world(seed: int) -> void:
    # initialize factions, locations, clocks from data + RNG

func world_tick(tick_type: String) -> void:
    # "ship_rest", "station_rest", "story_beat", etc.
    # advance clocks, resolve events, update economy

func generate_hooks_for_player(player_state) -> Dictionary:
    # returns {
    #   "contracts": [contract1, contract2, ...],
    #   "personal_hooks": [hook1, ...],
    #   "news": [headline1, ...]
    # }

func apply_contract_outcome(contract_id: String, outcome: Dictionary) -> void:
    # adjust factions, clocks, player rep, economy based on player’s success/failure

func get_news_feed() -> Array:
    # returns recent headlines/events for UI
```

---

## 7. GODOT SCENES & FLOWS

### 7.1 Minimum Scenes & Flow

Initial flow for a basic vertical slice:

1. **MainMenu.tscn**

   * New Game → Character Creation
   * Continue → Load last save

2. **CharacterCreation.tscn** (could be part of Hub scene or separate)

   * Choose background and career.
   * Initialize player attributes and skills.

3. **HubScene.tscn**

   * Player walks around station bar/hub.
   * Interact with:

     * Bartender (flavor / rumors).
     * Contract Board or Faction Rep (choose mission).
     * Ship Dock (enter Ship Scene).
     * Medbay, merchant, etc. (later).

4. **ShipScene.tscn**

   * View ship interior (simple at first).
   * Trigger:

     * Ship Rest.
     * Travel to chosen mission / planet.

5. **PlanetScene.tscn**

   * Top-down exploration, combat, resource gathering.
   * When done, return to extraction point → ShipScene → HubScene.

AI should implement this barebones flow before adding more.

---

## 8. AI PROMPT LIBRARY – GLOBAL STARTER

This section is **for you to copy-paste into AI sessions**. Adjust filenames/names as needed.

### 8.1 Global “Start Work on This Repo” Prompt

```text
You are helping me build a 2D top-down space frontier RPG in Godot 4 called "As Above, So Below".

Before you answer:
1. Assume the design spec is in AS_ABOVE_SO_BELOW_SOS_MASTER_GUIDE.md.
2. Obey that document as the source of truth for:
   - Architecture (Rules Layer, GM Engine, Game Systems, Presentation Layer)
   - Godot project structure
   - Core systems (combat, inventory, economy, contracts, GM Engine)

Your job:
- When I ask for a feature, first map it to:
  - which layer(s) it belongs to (Rules, GM Engine, Game Systems, Presentation),
  - which Godot files or new scripts it should touch,
  - how it connects to existing systems.
- Then produce focused code, scenes, or data definitions that fit the structure.

Do NOT invent a totally new architecture or rename layers unless I explicitly tell you to.
Keep code modular, well-commented, and aligned with the SOS guide.
```

---

## 9. AI PROMPT LIBRARY – SPECIFIC MODULES

Below are **ready prompts** you can use when you want AI to build specific parts.

### 9.1 Build / Refine Rules Layer Core

```text
Goal: Implement the core Rules system for "As Above, So Below" following the SOS master guide.

Tasks:
1. Create or refine a Godot-agnostic Rules module (can be GDScript or pseudo-module used by autoload/Rules.gd) that supports:
   - Attributes (MIG, AGI, INT, GRT, PRE, LCK)
   - Skills (Guns, Piloting, Engineering, etc.)
   - Character creation: background + career
   - Skill checks and saves
   - Basic combat math (hit, damage, crit)
   - Rest types and exhaustion track

2. Explain:
   - Where in the project structure this code should live.
   - How DMEngine and GameState should call into it.

3. Provide example usages:
   - A function doing a Guns check to shoot a raider.
   - A function applying a Ship Rest and reducing exhaustion.

Use the SOS master guide as reference. Keep it modular and easy to expand.
```

### 9.2 Build / Refine DM Engine Skeleton

```text
Goal: Implement the DMEngine.gd skeleton for "As Above, So Below".

Read and obey the SOS master guide section about the GameMaster Engine.

Implement DMEngine.gd with:
- Data structures for: factions, locations, clocks, events, contracts.
- Functions:
  - init_world(seed)
  - world_tick(tick_type)
  - generate_hooks_for_player(player_state)
  - apply_contract_outcome(contract_id, outcome)
  - get_news_feed()

Use placeholder data (e.g., one corp, one pirate faction, one mining colony, one hub station) and show:
- How clocks advance and trigger events.
- How contracts are generated and returned to a Contract UI.

Do NOT overcomplicate combat or scene logic here; focus on world state and hooks.
```

### 9.3 Build Inventory & Items System

```text
Goal: Implement the Inventory and Items system in Godot 4 for "As Above, So Below", following SOS.

Requirements:
- Player inventory (limited capacity).
- Ship cargo (bigger capacity).
- Item definitions loaded from data/items.json and data/weapons.json, etc.
- Item categories: consumable, weapon, armor, resource, special.
- Functions to:
  - add_item, remove_item, transfer_to_ship, transfer_to_player
  - equip/unequip weapons and armor
  - use consumables (hook into Rules layer for healing, buffs, etc.)

Also:
- Describe how Inventory UI should interact with this system.
- Show example item JSON and the corresponding GDScript parsing.

Keep everything aligned with the resource loops described in SOS.
```

### 9.4 Build Combat System Integration (Guns, Lasers, Melee)

```text
Goal: Wire up the combat system in Godot 4 according to the SOS guide.

Focus:
- Player shooting (guns/lasers)
- Enemy shooting/melee
- Using the Rules layer for:
  - Attack rolls, damage, crits, conditions

Tasks:
1. Show how player.gd calls a Rules.attack() function when firing a gun.
2. Show how enemy.gd does the same for AI attacks.
3. Implement simple projectile or hitscan logic in the Presentation layer, but keep all math in the Rules layer.
4. Explain how different weapon types (hitscan vs projectile vs melee vs AOE) plug into the same pattern.

Do not hardcode magic abilities yet—just standard weapons.
```

### 9.5 Build Economy & Traders

```text
Goal: Implement a basic economy & trader system according to the Taps→Converters→Traders→Drains model in SOS.

Steps:
1. Create data/economy_config.json to define:
   - base prices for resources (ore, fuel, meds, ammo)
   - modifiers per location type (hub, mining, pirate, war-scar)
2. Implement economy.gd with functions:
   - get_buy_price(item_id, location_id)
   - get_sell_price(item_id, location_id)
   - apply_global_modifier(key, value)  # used by DMEngine for events
3. Implement a simple Trader script (trader.gd or part of an NPC script) that:
   - Shows the player a list of items for sale and their prices.
   - Allows buying/selling using player inventory and credits.

Also:
- Describe how DMEngine can change global modifiers based on clocks (e.g., pirate activity = fuel price up).
```

### 9.6 Build the Vertical Slice Flow

```text
Goal: Implement the basic game flow for "As Above, So Below" in Godot 4, following SOS:

Flow:
MainMenu → CharacterCreation → HubScene (bar) → Contract selection → ShipScene → PlanetScene → Objective → back to Ship → HubScene → contract turn-in.

Tasks:
1. Define which scenes exist (MainMenu, CharacterCreation, HubScene, ShipScene, PlanetScene).
2. Show scene transitions and what data is passed between them via GameState autoload.
3. Implement a basic Contract UI in HubScene that:
   - Fetches hooks/contracts from DMEngine.generate_hooks_for_player
   - Lets the player pick one as "active_contract"
4. Implement in PlanetScene:
   - Simple map, one type of enemy, one objective (e.g., scan 3 survey points).
5. Implement returning to HubScene and turning in contract:
   - Player gets credits + XP via Rules.
   - DMEngine.apply_contract_outcome is called.
   - world_tick("contract_completed") is triggered.

Keep everything small and placeholder; focus on wiring the loop correctly.
```

---

## 10. LAST NOTES TO AI (AND FUTURE YOU)

* **Do not reinvent the architecture** unless explicitly instructed.
* **Keep systems modular**:

  * Rules shouldn’t know about scenes.
  * DM Engine shouldn’t know about input or sprites.
* **Document as you go**:

  * When adding a major function or system, add a short comment referencing this SOS doc section.
* **Small steps > mega refactors**:

  * Build the vertical slice first.
  * Then deepen combat, economy, DM behavior, magic, etc.

This document is allowed to **evolve**, but any major change to the game’s structure should be reflected here so that every future AI session is working from the same map.

*AS ABOVE, SO BELOW – SOS*
*End of Master Guide v1*

```

::contentReference[oaicite:0]{index=0}
```
