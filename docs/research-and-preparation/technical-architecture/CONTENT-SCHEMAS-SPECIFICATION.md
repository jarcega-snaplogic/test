# Content Schemas Specification: Solar Punk Chronicles
**Version 1.0** | **Data Engineer** | **Last Updated: 2025-11-01**

---

## Table of Contents

1. [Overview](#overview)
2. [Content ID System](#content-id-system)
3. [Core Schema Definitions](#core-schema-definitions)
   - 3.1 [Cities](#31-cities-schema)
   - 3.2 [Character Classes](#32-character-classes-schema)
   - 3.3 [Abilities & Skills](#33-abilities--skills-schema)
   - 3.4 [Items & Equipment](#34-items--equipment-schema)
   - 3.5 [Enemies](#35-enemies-schema)
   - 3.6 [Quests](#36-quests-schema)
   - 3.7 [Dialogs](#37-dialogs-schema)
4. [Supporting Schemas](#supporting-schemas)
   - 4.1 [NPCs](#41-npcs-schema)
   - 4.2 [Status Effects](#42-status-effects-schema)
   - 4.3 [Loot Tables](#43-loot-tables-schema)
   - 4.4 [Events](#44-events-schema)
5. [File Structure & Organization](#file-structure--organization)
6. [Naming Conventions](#naming-conventions)
7. [Versioning Strategy](#versioning-strategy)
8. [Loading System](#loading-system)
9. [Validation Rules](#validation-rules)
10. [Reference Resolution](#reference-resolution)
11. [Implementation Examples](#implementation-examples)

---

## Overview

This specification defines the complete data schema for Solar Punk Chronicles RPG content. All game content is stored as JSON files, loaded dynamically at runtime, and validated against TypeScript interfaces.

**Design Principles:**
- **Type Safety**: Full TypeScript support with strict types
- **Modularity**: Each content type in separate files
- **Relationships**: Reference system using content IDs
- **Validation**: Schema validation on load with helpful error messages
- **Extensibility**: Easy to add new content without code changes
- **Versioning**: Forward-compatible versioning system

**Data Flow:**
```
JSON Files → Asset Manifest → Content Loader → Validation → Type-Safe Objects → Game Systems
```

---

## Content ID System

All content uses a structured ID system for references and lookups.

### ID Format

```typescript
type ContentID = string; // Format: "type_category_name" or "type_name"

// Examples:
// - Classes: "class_technomancer", "class_biohacker"
// - Abilities: "ability_lightning_strike", "ability_heal"
// - Items: "item_weapon_solar_blade", "item_armor_photon_vest"
// - Enemies: "enemy_corrupted_drone", "enemy_boss_megacorp_executive"
// - Quests: "quest_main_01_awakening", "quest_side_warehouse_mystery"
// - Cities: "city_verdant_heights", "city_industrial_core"
```

### ID Rules

1. **Lowercase only**: `technomancer` not `Technomancer`
2. **Underscores**: Use `_` as separator, not hyphens or camelCase
3. **Prefixes**: Always start with content type (class_, ability_, item_, etc.)
4. **Descriptive**: IDs should be human-readable
5. **Unique**: No two pieces of content can share an ID
6. **Immutable**: Once published, IDs never change (use aliases for renames)

### Reference System

```typescript
// References to other content use IDs
interface ContentReference {
  id: ContentID;
  type: ContentType; // For validation
}

type ContentType =
  | "class"
  | "ability"
  | "item"
  | "enemy"
  | "quest"
  | "dialog"
  | "city"
  | "npc"
  | "status"
  | "loot"
  | "event";
```

---

## Core Schema Definitions

### 3.1 Cities Schema

Cities are explorable hubs with locations, NPCs, events, and quests.

#### TypeScript Interface

```typescript
interface City {
  // Metadata
  id: ContentID; // "city_verdant_heights"
  version: string; // "1.0.0"
  name: string; // "Verdant Heights"
  description: string; // "A thriving solarpunk district..."

  // Visual Data
  theme: CityTheme;
  mapLayout: MapLayout;

  // Content
  districts: District[];
  locations: Location[];
  npcs: ContentReference[]; // References to NPC IDs
  events: ContentReference[]; // References to Event IDs

  // World Map Integration
  worldMapPosition: { x: number; y: number };
  unlockRequirements: UnlockRequirement[];
  connectedCities: ContentID[]; // IDs of adjacent cities

  // Metadata
  tags: string[]; // ["residential", "vertical_farms", "early_game"]
}

type CityTheme =
  | "residential"
  | "industrial"
  | "commercial"
  | "agricultural"
  | "research"
  | "wild";

interface MapLayout {
  width: number; // Grid width
  height: number; // Grid height
  tileSize: number; // Pixels per tile
  backgroundColor: string; // Hex color
  tilemap?: string; // Path to Tiled JSON (optional)
}

interface District {
  id: string; // "district_solar_plaza"
  name: string; // "Solar Plaza"
  description: string;
  bounds: {
    x: number;
    y: number;
    width: number;
    height: number;
  };
  theme: string; // "commercial", "residential"
}

interface Location {
  id: string; // "location_traders_market"
  name: string; // "Trader's Market"
  type: LocationType;
  position: { x: number; y: number };
  interactable: boolean;
  icon?: string; // Icon identifier

  // Interactions
  shopInventory?: ContentReference[]; // Item IDs if shop
  dialogTree?: ContentReference; // Dialog ID
  skillChecks?: SkillCheck[]; // Available skill checks

  // State
  unlockRequirements?: UnlockRequirement[];
  oneTimeOnly?: boolean; // Disappears after interaction
}

type LocationType =
  | "shop"
  | "quest_giver"
  | "rest_point"
  | "travel_hub"
  | "skill_check"
  | "combat_trigger"
  | "decoration";

interface SkillCheck {
  skill: DnDSkill; // "investigation", "persuasion", etc.
  dc: number; // Difficulty class (5-30)
  successOutcome: Outcome;
  failureOutcome: Outcome;
  canRetry: boolean;
}

interface Outcome {
  type: "xp" | "item" | "quest" | "dialog" | "event" | "unlock";
  value: any; // Depends on type
  message?: string; // Player feedback
}

interface UnlockRequirement {
  type: "quest" | "level" | "item" | "flag";
  id?: ContentID; // For quest/item requirements
  value?: number; // For level requirements
  flag?: string; // For flag-based unlocks
}

type DnDSkill =
  | "athletics" | "acrobatics" | "sleight_of_hand" | "stealth"
  | "arcana" | "history" | "investigation" | "nature" | "religion"
  | "animal_handling" | "insight" | "medicine" | "perception" | "survival"
  | "deception" | "intimidation" | "performance" | "persuasion";
```

#### JSON Example

```json
{
  "id": "city_verdant_heights",
  "version": "1.0.0",
  "name": "Verdant Heights",
  "description": "A thriving residential district where vertical farms cascade down gleaming towers. Solar panels catch the morning light while community gardens flourish on every rooftop.",

  "theme": "residential",
  "mapLayout": {
    "width": 40,
    "height": 30,
    "tileSize": 32,
    "backgroundColor": "#4a7c59"
  },

  "districts": [
    {
      "id": "district_solar_plaza",
      "name": "Solar Plaza",
      "description": "The heart of Verdant Heights, where residents gather.",
      "bounds": { "x": 10, "y": 10, "width": 20, "height": 15 },
      "theme": "commercial"
    },
    {
      "id": "district_hanging_gardens",
      "name": "Hanging Gardens",
      "description": "Lush vertical farms produce food for the entire district.",
      "bounds": { "x": 5, "y": 5, "width": 10, "height": 10 },
      "theme": "agricultural"
    }
  ],

  "locations": [
    {
      "id": "location_traders_market",
      "name": "Trader's Market",
      "type": "shop",
      "position": { "x": 15, "y": 12 },
      "interactable": true,
      "icon": "shop",
      "shopInventory": [
        { "id": "item_potion_health_minor", "type": "item" },
        { "id": "item_weapon_solar_blade", "type": "item" },
        { "id": "item_armor_fiber_vest", "type": "item" }
      ]
    },
    {
      "id": "location_mysterious_door",
      "name": "Locked Warehouse",
      "type": "skill_check",
      "position": { "x": 25, "y": 18 },
      "interactable": true,
      "icon": "mystery",
      "skillChecks": [
        {
          "skill": "athletics",
          "dc": 15,
          "successOutcome": {
            "type": "unlock",
            "value": "location_warehouse_interior",
            "message": "You break down the door!"
          },
          "failureOutcome": {
            "type": "dialog",
            "value": "dialog_warehouse_fail",
            "message": "The door won't budge."
          },
          "canRetry": false
        },
        {
          "skill": "sleight_of_hand",
          "dc": 12,
          "successOutcome": {
            "type": "unlock",
            "value": "location_warehouse_interior",
            "message": "You pick the lock successfully!"
          },
          "failureOutcome": {
            "type": "xp",
            "value": 10,
            "message": "You fumble the lockpick but learn from the attempt."
          },
          "canRetry": true
        }
      ]
    },
    {
      "id": "location_rest_fountain",
      "name": "Community Fountain",
      "type": "rest_point",
      "position": { "x": 20, "y": 15 },
      "interactable": true,
      "icon": "rest"
    }
  ],

  "npcs": [
    { "id": "npc_merchant_jade", "type": "npc" },
    { "id": "npc_quest_giver_marcus", "type": "npc" },
    { "id": "npc_recruitable_technomancer_aria", "type": "npc" }
  ],

  "events": [
    { "id": "event_market_dispute", "type": "event" },
    { "id": "event_drone_attack", "type": "event" }
  ],

  "worldMapPosition": { "x": 2, "y": 3 },
  "unlockRequirements": [],
  "connectedCities": [
    "city_industrial_core",
    "city_transit_nexus"
  ],

  "tags": ["residential", "vertical_farms", "early_game", "tutorial"]
}
```

---

### 3.2 Character Classes Schema

Character classes define base stats, progression, and available abilities.

#### TypeScript Interface

```typescript
interface CharacterClass {
  // Metadata
  id: ContentID; // "class_technomancer"
  version: string;
  name: string; // "Technomancer"
  description: string;
  flavor: string; // Short tagline

  // Visuals
  sprite: SpriteData;
  primaryColor: string; // Hex color
  secondaryColor: string;
  icon: string; // Icon identifier

  // Base Stats (Level 1)
  baseStats: CharacterStats;

  // Growth (per level)
  statGrowth: StatGrowth;

  // D&D Abilities
  abilityScores: AbilityScores; // STR, DEX, CON, INT, WIS, CHA
  skillProficiencies: DnDSkill[]; // Proficient skills
  savingThrowProficiencies: Ability[]; // "strength", "intelligence"

  // Combat Role
  role: CombatRole;
  preferredRow: "front" | "back"; // Default formation position

  // Abilities
  startingAbilities: ContentReference[]; // Ability IDs
  learnableAbilities: AbilityProgression[];

  // Equipment
  equipmentSlots: EquipmentSlot[];
  weaponProficiencies: WeaponType[];
  armorProficiencies: ArmorType[];

  // Progression
  xpCurve: XPCurve; // "standard", "fast", "slow"
  maxLevel: number;

  // Unlock
  unlockMethod: UnlockMethod;

  // Tags
  tags: string[]; // ["magic", "ranged", "support"]
}

interface CharacterStats {
  hp: number; // Health points
  sp: number; // Skill points (mana)
  attack: number; // Physical damage modifier
  magic: number; // Magical damage modifier
  defense: number; // Physical damage reduction
  resistance: number; // Magical damage reduction
  speed: number; // Turn order priority
  accuracy: number; // Hit chance modifier (percentage)
  evasion: number; // Dodge chance (percentage)
  critRate: number; // Critical hit chance (percentage)
  critDamage: number; // Critical damage multiplier (percentage)
}

interface StatGrowth {
  hp: number; // +hp per level
  sp: number;
  attack: number;
  magic: number;
  defense: number;
  resistance: number;
  speed: number;
  accuracy: number;
  evasion: number;
  critRate: number;
  critDamage: number;
}

interface AbilityScores {
  strength: number; // 8-20 (10 = average)
  dexterity: number;
  constitution: number;
  intelligence: number;
  wisdom: number;
  charisma: number;
}

type Ability = "strength" | "dexterity" | "constitution" | "intelligence" | "wisdom" | "charisma";

type CombatRole =
  | "tank"
  | "damage_dealer"
  | "support"
  | "healer"
  | "crowd_control"
  | "hybrid";

interface AbilityProgression {
  ability: ContentReference; // Ability ID
  unlockLevel: number; // Level required to learn
  prerequisite?: ContentReference; // Required ability (optional)
}

interface SpriteData {
  type: "procedural" | "texture";
  // For procedural
  shape?: "rectangle" | "circle" | "triangle";
  baseColor?: string;
  symbolId?: string; // "lightning", "leaf", "gear"
  // For texture
  texturePath?: string;
  frameWidth?: number;
  frameHeight?: number;
}

interface EquipmentSlot {
  slot: "weapon" | "armor" | "accessory";
  required: boolean; // Can character function without it?
}

type WeaponType = "sword" | "staff" | "bow" | "dagger" | "hammer" | "gun" | "wand";
type ArmorType = "light" | "medium" | "heavy" | "cloth";

type XPCurve = "fast" | "standard" | "slow";

interface UnlockMethod {
  type: "story" | "quest" | "discovery" | "challenge";
  requirement?: ContentReference; // Quest ID or challenge ID
  automatic?: boolean; // Unlocked by default
}
```

#### JSON Example

```json
{
  "id": "class_technomancer",
  "version": "1.0.0",
  "name": "Technomancer",
  "description": "Masters of electrical manipulation and digital warfare. Technomancers blend ancient electromagnetic understanding with cutting-edge neural interfaces.",
  "flavor": "Lightning in the circuits, power in the grid",

  "sprite": {
    "type": "procedural",
    "shape": "rectangle",
    "baseColor": "#3498db",
    "symbolId": "lightning"
  },
  "primaryColor": "#3498db",
  "secondaryColor": "#f39c12",
  "icon": "lightning_bolt",

  "baseStats": {
    "hp": 45,
    "sp": 30,
    "attack": 8,
    "magic": 15,
    "defense": 6,
    "resistance": 10,
    "speed": 12,
    "accuracy": 85,
    "evasion": 8,
    "critRate": 10,
    "critDamage": 150
  },

  "statGrowth": {
    "hp": 8,
    "sp": 6,
    "attack": 1,
    "magic": 3,
    "defense": 1,
    "resistance": 2,
    "speed": 2,
    "accuracy": 1,
    "evasion": 1,
    "critRate": 0.5,
    "critDamage": 2
  },

  "abilityScores": {
    "strength": 10,
    "dexterity": 12,
    "constitution": 11,
    "intelligence": 16,
    "wisdom": 13,
    "charisma": 10
  },

  "skillProficiencies": [
    "arcana",
    "investigation",
    "nature",
    "perception"
  ],

  "savingThrowProficiencies": ["intelligence", "wisdom"],

  "role": "damage_dealer",
  "preferredRow": "back",

  "startingAbilities": [
    { "id": "ability_lightning_strike", "type": "ability" },
    { "id": "ability_static_shield", "type": "ability" }
  ],

  "learnableAbilities": [
    {
      "ability": { "id": "ability_chain_lightning", "type": "ability" },
      "unlockLevel": 3,
      "prerequisite": { "id": "ability_lightning_strike", "type": "ability" }
    },
    {
      "ability": { "id": "ability_emp_blast", "type": "ability" },
      "unlockLevel": 5
    }
  ],

  "equipmentSlots": [
    { "slot": "weapon", "required": true },
    { "slot": "armor", "required": true },
    { "slot": "accessory", "required": false },
    { "slot": "accessory", "required": false }
  ],

  "weaponProficiencies": ["staff", "wand", "gun"],
  "armorProficiencies": ["light", "cloth"],

  "xpCurve": "standard",
  "maxLevel": 30,

  "unlockMethod": {
    "type": "story",
    "automatic": true
  },

  "tags": ["magic", "ranged", "electric", "tech"]
}
```

---

### 3.3 Abilities & Skills Schema

Abilities are combat actions with effects, costs, and animations.

#### TypeScript Interface

```typescript
interface Ability {
  // Metadata
  id: ContentID; // "ability_lightning_strike"
  version: string;
  name: string; // "Lightning Strike"
  description: string;
  flavorText?: string;

  // Type
  type: AbilityType;
  element: Element;
  damageType?: DamageType; // For attack abilities

  // Costs
  spCost: number; // Skill points consumed
  cooldown?: number; // Turns before reuse (0 = no cooldown)
  charges?: number; // Limited uses per battle

  // Targeting
  targeting: TargetingType;
  range: number; // 1 = adjacent, 99 = unlimited
  targetCount: TargetCount;

  // Effects
  effects: AbilityEffect[];

  // Power
  basePower?: number; // Base damage/healing value
  scaling: ScalingType; // What stat scales this ability
  scalingRatio: number; // Multiplier for scaling stat

  // Hit Calculation
  accuracy: number; // Base accuracy (percentage)
  canCrit: boolean;
  critBonus?: number; // Extra crit chance for this ability

  // Animation
  animation: AnimationData;

  // Requirements
  requirements?: AbilityRequirement[];

  // Upgrades
  upgradeable: boolean;
  upgradeOptions?: AbilityUpgrade[];

  // Tags
  tags: string[]; // ["aoe", "lightning", "offensive"]
}

type AbilityType =
  | "attack"
  | "heal"
  | "buff"
  | "debuff"
  | "utility"
  | "summon"
  | "field";

type Element =
  | "physical"
  | "fire"
  | "ice"
  | "lightning"
  | "earth"
  | "wind"
  | "light"
  | "dark"
  | "neutral";

type DamageType = "physical" | "magical";

type TargetingType =
  | "single_enemy"
  | "all_enemies"
  | "random_enemies"
  | "single_ally"
  | "all_allies"
  | "self"
  | "area";

interface TargetCount {
  min: number;
  max: number;
}

interface AbilityEffect {
  type: EffectType;
  value?: number; // Damage, healing, stat change
  duration?: number; // Turns (for buffs/debuffs)
  statusEffect?: ContentReference; // Status effect ID
  chance?: number; // Probability (0-100)
  onTrigger?: "hit" | "kill" | "crit" | "turn_start" | "turn_end";
}

type EffectType =
  | "damage"
  | "heal"
  | "status"
  | "stat_change"
  | "position_change"
  | "summon"
  | "remove_status"
  | "revive";

type ScalingType =
  | "attack"
  | "magic"
  | "speed"
  | "hp_percent"
  | "target_missing_hp"
  | "fixed";

interface AnimationData {
  castAnimation: string; // "cast_lightning", "swing_sword"
  projectileAnimation?: string; // For ranged
  hitAnimation: string; // "lightning_impact", "slash_effect"
  soundEffect?: string; // Sound ID
  screenShake?: boolean;
  duration: number; // Milliseconds
  particles?: ParticleConfig;
}

interface ParticleConfig {
  type: "spark" | "flame" | "frost" | "heal" | "buff" | "debuff";
  color: string;
  count: number;
  duration: number;
}

interface AbilityRequirement {
  type: "level" | "weapon" | "status" | "resource";
  value: number | string;
}

interface AbilityUpgrade {
  id: string; // "upgrade_increased_damage"
  name: string;
  description: string;
  effect: UpgradeEffect;
}

interface UpgradeEffect {
  type: "increase_power" | "reduce_cost" | "add_effect" | "increase_targets";
  value: number | AbilityEffect;
}
```

#### JSON Example

```json
{
  "id": "ability_lightning_strike",
  "version": "1.0.0",
  "name": "Lightning Strike",
  "description": "Channel electrical energy into a devastating bolt that strikes a single enemy. High damage with a chance to stun.",
  "flavorText": "The air crackles with barely contained power.",

  "type": "attack",
  "element": "lightning",
  "damageType": "magical",

  "spCost": 12,
  "cooldown": 0,

  "targeting": "single_enemy",
  "range": 99,
  "targetCount": { "min": 1, "max": 1 },

  "effects": [
    {
      "type": "damage",
      "value": 0,
      "onTrigger": "hit"
    },
    {
      "type": "status",
      "statusEffect": { "id": "status_stunned", "type": "status" },
      "chance": 30,
      "duration": 1,
      "onTrigger": "hit"
    }
  ],

  "basePower": 35,
  "scaling": "magic",
  "scalingRatio": 1.5,

  "accuracy": 95,
  "canCrit": true,
  "critBonus": 5,

  "animation": {
    "castAnimation": "cast_lightning",
    "projectileAnimation": "lightning_bolt",
    "hitAnimation": "lightning_impact",
    "soundEffect": "sfx_lightning_strike",
    "screenShake": true,
    "duration": 800,
    "particles": {
      "type": "spark",
      "color": "#f39c12",
      "count": 20,
      "duration": 500
    }
  },

  "upgradeable": true,
  "upgradeOptions": [
    {
      "id": "upgrade_chain_lightning",
      "name": "Chain Lightning",
      "description": "Lightning bounces to 1 additional target for 50% damage.",
      "effect": {
        "type": "add_effect",
        "value": {
          "type": "damage",
          "value": 0.5,
          "onTrigger": "hit"
        }
      }
    },
    {
      "id": "upgrade_reduced_cost",
      "name": "Efficient Casting",
      "description": "Reduce SP cost by 3.",
      "effect": {
        "type": "reduce_cost",
        "value": 3
      }
    }
  ],

  "tags": ["single_target", "lightning", "offensive", "stun"]
}
```

---

### 3.4 Items & Equipment Schema

Items include equipment, consumables, and crafting materials.

#### TypeScript Interface

```typescript
interface Item {
  // Metadata
  id: ContentID; // "item_weapon_solar_blade"
  version: string;
  name: string; // "Solar Blade"
  description: string;
  flavorText?: string;

  // Type
  category: ItemCategory;
  subtype: ItemSubtype;

  // Rarity
  rarity: Rarity;

  // Value
  value: number; // Gold/currency value
  stackable: boolean;
  maxStack?: number;

  // Equipment (if category is "equipment")
  equipmentSlot?: EquipmentSlot["slot"];
  stats?: Partial<CharacterStats>; // Stat bonuses
  requirements?: EquipmentRequirement[];

  // Weapon (if subtype is weapon)
  weaponType?: WeaponType;
  attackPower?: number;
  attackSpeed?: number; // Modifier to speed
  range?: number;

  // Armor (if subtype is armor)
  armorType?: ArmorType;
  defensePower?: number;
  resistancePower?: number;

  // Effects
  effects?: ItemEffect[];
  passiveEffects?: PassiveEffect[]; // While equipped

  // Consumable (if category is "consumable")
  consumable?: ConsumableData;

  // Crafting
  craftable: boolean;
  craftingRecipe?: CraftingRecipe;
  ingredients?: boolean; // Is this a crafting material?

  // Visuals
  icon: string; // Icon identifier
  color: string; // Hex color for rarity tint

  // Unlock
  unlockRequirements?: UnlockRequirement[];

  // Tags
  tags: string[];
}

type ItemCategory = "equipment" | "consumable" | "material" | "key_item" | "quest_item";
type ItemSubtype = "weapon" | "armor" | "accessory" | "potion" | "food" | "scroll" | "ore" | "plant" | "tech";

type Rarity = "common" | "uncommon" | "rare" | "epic" | "legendary";

interface EquipmentRequirement {
  type: "class" | "level" | "stat" | "quest";
  value: string | number;
}

interface ItemEffect {
  type: EffectType;
  value: number;
  target: "user" | "enemy" | "all_allies";
  duration?: number; // For temporary effects
}

interface PassiveEffect {
  type: "stat_boost" | "element_resist" | "status_immune" | "ability_boost";
  value: number | string;
  condition?: string; // "in_combat", "hp_below_50"
}

interface ConsumableData {
  usableInCombat: boolean;
  usableInExploration: boolean;
  targetType: "self" | "single_ally" | "all_allies";
  effectOnUse: ItemEffect[];
}

interface CraftingRecipe {
  ingredients: CraftingIngredient[];
  craftingStation?: string; // "forge", "alchemy_lab"
  skillRequired?: { skill: DnDSkill; dc: number };
  craftingTime?: number; // Seconds (for timed crafting)
  yield: number; // How many produced
}

interface CraftingIngredient {
  item: ContentReference; // Item ID
  quantity: number;
}
```

#### JSON Example

```json
{
  "id": "item_weapon_solar_blade",
  "version": "1.0.0",
  "name": "Solar Blade",
  "description": "A sleek sword with photo-reactive cells along the blade. Stores solar energy during the day, unleashing it in combat.",
  "flavorText": "Forged in the first solarpunk foundries of Verdant Heights.",

  "category": "equipment",
  "subtype": "weapon",
  "rarity": "uncommon",

  "value": 450,
  "stackable": false,

  "equipmentSlot": "weapon",
  "stats": {
    "attack": 15,
    "speed": 3,
    "critRate": 5
  },

  "requirements": [
    { "type": "level", "value": 3 },
    { "type": "stat", "value": "strength:10" }
  ],

  "weaponType": "sword",
  "attackPower": 15,
  "attackSpeed": 3,
  "range": 1,

  "passiveEffects": [
    {
      "type": "element_resist",
      "value": "fire:10"
    },
    {
      "type": "ability_boost",
      "value": "light_element:15",
      "condition": "in_combat"
    }
  ],

  "craftable": true,
  "craftingRecipe": {
    "ingredients": [
      { "item": { "id": "item_material_steel_ingot", "type": "item" }, "quantity": 3 },
      { "item": { "id": "item_material_solar_crystal", "type": "item" }, "quantity": 1 },
      { "item": { "id": "item_material_polymer_grip", "type": "item" }, "quantity": 1 }
    ],
    "craftingStation": "forge",
    "skillRequired": { "skill": "arcana", "dc": 12 },
    "yield": 1
  },

  "icon": "sword_01",
  "color": "#f39c12",

  "tags": ["weapon", "sword", "solar", "light_element", "melee"]
}
```

---

### 3.5 Enemies Schema

Enemies define combat opponents with stats, AI, and loot.

#### TypeScript Interface

```typescript
interface Enemy {
  // Metadata
  id: ContentID; // "enemy_corrupted_drone"
  version: string;
  name: string; // "Corrupted Drone"
  description: string;

  // Type
  type: EnemyType;
  category: EnemyCategory;

  // Stats
  stats: CharacterStats;
  level: number;

  // D&D Stats (for skill checks)
  abilityScores?: AbilityScores;

  // Combat
  abilities: ContentReference[]; // Ability IDs
  aiPattern: AIPattern;
  formation: FormationPreference;

  // Resistances
  elementalResistances: ElementalResistances;
  statusResistances: StatusResistances;

  // Loot
  lootTable: ContentReference; // Loot table ID
  xpReward: number;
  goldReward: { min: number; max: number };

  // Visual
  sprite: SpriteData;
  color: string;
  size: "small" | "medium" | "large" | "boss";

  // Behavior
  aggressive: boolean; // Engages on sight
  flees: boolean; // Runs at low HP
  fleeThreshold?: number; // HP percentage

  // Special
  isBoss: boolean;
  phases?: BossPhase[]; // For multi-phase bosses

  // Spawn
  spawnConditions?: SpawnCondition[];

  // Tags
  tags: string[];
}

type EnemyType =
  | "construct"
  | "beast"
  | "humanoid"
  | "undead"
  | "elemental"
  | "aberration";

type EnemyCategory =
  | "minion"
  | "normal"
  | "elite"
  | "mini_boss"
  | "boss";

interface AIPattern {
  strategy: AIStrategy;
  priorityTarget: TargetPriority;
  abilityUsage: AbilityUsagePattern[];
  specialBehaviors?: SpecialBehavior[];
}

type AIStrategy =
  | "aggressive"
  | "defensive"
  | "balanced"
  | "support"
  | "random";

type TargetPriority =
  | "lowest_hp"
  | "highest_damage"
  | "back_row"
  | "front_row"
  | "random"
  | "healer_first";

interface AbilityUsagePattern {
  ability: ContentReference;
  priority: number; // Higher = use more often
  condition?: string; // "hp_below_50", "ally_dead", "first_turn"
}

interface SpecialBehavior {
  type: "summon" | "transform" | "enrage" | "heal_allies";
  trigger: string; // "hp_below_30", "turn_3"
  action: ContentReference | string; // Ability ID or custom behavior
}

interface FormationPreference {
  row: "front" | "back";
  position?: number; // 0-2 (left, center, right)
}

interface ElementalResistances {
  fire?: number; // -50 = weak, 0 = neutral, 50 = resist, 100 = immune
  ice?: number;
  lightning?: number;
  earth?: number;
  wind?: number;
  light?: number;
  dark?: number;
}

interface StatusResistances {
  stun?: number; // 0-100 (percentage resistance)
  poison?: number;
  burn?: number;
  freeze?: number;
  blind?: number;
  silence?: number;
  slow?: number;
}

interface BossPhase {
  phaseNumber: number;
  hpThreshold: number; // Percentage
  abilities: ContentReference[]; // New abilities in this phase
  statChanges?: Partial<CharacterStats>;
  dialogue?: string;
  transformation?: SpriteData;
}

interface SpawnCondition {
  type: "time" | "weather" | "quest" | "event";
  value: string;
}
```

#### JSON Example

```json
{
  "id": "enemy_corrupted_drone",
  "version": "1.0.0",
  "name": "Corrupted Drone",
  "description": "A maintenance drone infected with malicious code. Sparks fly from its damaged circuits.",

  "type": "construct",
  "category": "normal",

  "stats": {
    "hp": 60,
    "sp": 20,
    "attack": 12,
    "magic": 8,
    "defense": 10,
    "resistance": 5,
    "speed": 10,
    "accuracy": 80,
    "evasion": 5,
    "critRate": 5,
    "critDamage": 150
  },

  "level": 3,

  "abilities": [
    { "id": "ability_shock_beam", "type": "ability" },
    { "id": "ability_self_repair", "type": "ability" }
  ],

  "aiPattern": {
    "strategy": "balanced",
    "priorityTarget": "lowest_hp",
    "abilityUsage": [
      {
        "ability": { "id": "ability_shock_beam", "type": "ability" },
        "priority": 70,
        "condition": "hp_above_30"
      },
      {
        "ability": { "id": "ability_self_repair", "type": "ability" },
        "priority": 90,
        "condition": "hp_below_40"
      }
    ],
    "specialBehaviors": [
      {
        "type": "enrage",
        "trigger": "hp_below_20",
        "action": "stat_boost:attack:5"
      }
    ]
  },

  "formation": {
    "row": "front",
    "position": 1
  },

  "elementalResistances": {
    "lightning": 50,
    "ice": -25,
    "fire": 0
  },

  "statusResistances": {
    "stun": 30,
    "poison": 100,
    "burn": 25
  },

  "lootTable": { "id": "loot_corrupted_drone", "type": "loot" },
  "xpReward": 85,
  "goldReward": { "min": 20, "max": 35 },

  "sprite": {
    "type": "procedural",
    "shape": "circle",
    "baseColor": "#7f8c8d",
    "symbolId": "drone"
  },
  "color": "#c0392b",
  "size": "medium",

  "aggressive": true,
  "flees": false,
  "isBoss": false,

  "tags": ["construct", "electric", "tech", "early_game"]
}
```

---

### 3.6 Quests Schema

Quests define objectives, branches, rewards, and prerequisites.

#### TypeScript Interface

```typescript
interface Quest {
  // Metadata
  id: ContentID; // "quest_main_01_awakening"
  version: string;
  name: string; // "The Awakening"
  description: string;

  // Type
  type: QuestType;
  category: QuestCategory;

  // Structure
  stages: QuestStage[];
  branches?: QuestBranch[]; // For multi-path quests

  // Requirements
  prerequisites: QuestPrerequisite[];
  recommendedLevel: number;

  // Objectives
  trackable: boolean; // Show in quest log
  hidden: boolean; // Hidden until discovered

  // Rewards
  rewards: QuestReward;

  // Location
  questGiver?: ContentReference; // NPC ID
  startLocation: ContentReference; // City or location ID
  relevantLocations?: ContentReference[];

  // Time
  timeLimit?: number; // Minutes (optional)
  expiresOn?: string; // Quest flag or date

  // State
  repeatable: boolean;
  repeatCooldown?: number; // Days

  // Failure
  canFail: boolean;
  failureConditions?: FailureCondition[];

  // Tags
  tags: string[];
}

type QuestType = "main" | "side" | "character" | "daily" | "challenge";
type QuestCategory =
  | "story"
  | "combat"
  | "exploration"
  | "social"
  | "crafting"
  | "collection";

interface QuestStage {
  id: string; // "stage_01_investigate"
  name: string;
  description: string;

  objectives: Objective[];

  // Dialog
  startDialog?: ContentReference;
  completeDialog?: ContentReference;

  // Rewards (optional per-stage rewards)
  rewards?: Partial<QuestReward>;

  // Branching
  nextStage?: string | ConditionalStage[];
}

interface Objective {
  id: string;
  type: ObjectiveType;
  description: string;

  // Target
  target?: ContentReference; // Enemy, item, NPC, location
  quantity?: number;

  // Completion
  completed: boolean; // Runtime state
  optional: boolean;
  hidden?: boolean; // Revealed later

  // Tracking
  currentProgress?: number;
  requiredProgress?: number;
}

type ObjectiveType =
  | "kill"
  | "collect"
  | "talk"
  | "explore"
  | "skill_check"
  | "escort"
  | "defend"
  | "craft";

interface ConditionalStage {
  stage: string;
  condition: string; // "skill_check_success:persuasion", "choice:A"
}

interface QuestBranch {
  id: string;
  name: string;
  triggerCondition: string;
  stages: string[]; // Stage IDs in this branch
  exclusiveWith?: string[]; // Other branch IDs
}

interface QuestPrerequisite {
  type: "quest" | "level" | "class" | "item" | "flag";
  value: string | number;
}

interface QuestReward {
  xp: number;
  gold: number;
  items?: ContentReference[];
  abilities?: ContentReference[];
  unlocks?: UnlockReward[];
  reputation?: ReputationReward[];
}

interface UnlockReward {
  type: "character" | "location" | "shop" | "feature";
  id: ContentReference;
}

interface ReputationReward {
  faction: string;
  amount: number; // Positive or negative
}

interface FailureCondition {
  type: "time_limit" | "character_death" | "objective_failed";
  value?: string | number;
  consequence?: string; // Description of failure
}
```

#### JSON Example

```json
{
  "id": "quest_side_warehouse_mystery",
  "version": "1.0.0",
  "name": "The Warehouse Mystery",
  "description": "A locked warehouse in Solar Plaza has been abandoned for months. Strange noises emanate from within.",

  "type": "side",
  "category": "exploration",

  "stages": [
    {
      "id": "stage_01_investigate",
      "name": "Investigate the Warehouse",
      "description": "Find a way to enter the locked warehouse.",
      "objectives": [
        {
          "id": "obj_01_enter_warehouse",
          "type": "explore",
          "description": "Enter the warehouse (multiple solutions possible)",
          "target": { "id": "location_warehouse_interior", "type": "location" },
          "completed": false,
          "optional": false
        }
      ],
      "nextStage": "stage_02_discover_source"
    },
    {
      "id": "stage_02_discover_source",
      "name": "Discover the Source",
      "description": "Investigate the interior to find what's causing the disturbance.",
      "objectives": [
        {
          "id": "obj_02_find_drone",
          "type": "explore",
          "description": "Locate the source of the noise",
          "target": { "id": "location_warehouse_back_room", "type": "location" },
          "completed": false,
          "optional": false
        },
        {
          "id": "obj_03_scan_drone",
          "type": "skill_check",
          "description": "Scan the malfunctioning drone (Investigation DC 12)",
          "completed": false,
          "optional": true
        }
      ],
      "completeDialog": { "id": "dialog_warehouse_discovery", "type": "dialog" },
      "nextStage": [
        {
          "stage": "stage_03_repair",
          "condition": "skill_check_success:investigation"
        },
        {
          "stage": "stage_03_destroy",
          "condition": "choice:destroy"
        },
        {
          "stage": "stage_03_report",
          "condition": "choice:report"
        }
      ]
    },
    {
      "id": "stage_03_repair",
      "name": "Repair the Drone",
      "description": "Use your technical skills to repair the drone.",
      "objectives": [
        {
          "id": "obj_04_repair",
          "type": "skill_check",
          "description": "Repair the drone (Arcana DC 14)",
          "completed": false,
          "optional": false
        }
      ],
      "rewards": {
        "xp": 150,
        "items": [
          { "id": "item_drone_companion", "type": "item" }
        ]
      },
      "nextStage": null
    },
    {
      "id": "stage_03_destroy",
      "name": "Destroy the Drone",
      "description": "Eliminate the malfunctioning drone.",
      "objectives": [
        {
          "id": "obj_04_combat",
          "type": "kill",
          "description": "Defeat the corrupted drone",
          "target": { "id": "enemy_corrupted_drone", "type": "enemy" },
          "quantity": 1,
          "completed": false,
          "optional": false
        }
      ],
      "rewards": {
        "xp": 100,
        "gold": 50
      },
      "nextStage": null
    }
  ],

  "branches": [
    {
      "id": "branch_technical",
      "name": "Technical Solution",
      "triggerCondition": "skill_check_success:investigation",
      "stages": ["stage_03_repair"],
      "exclusiveWith": ["branch_combat", "branch_report"]
    },
    {
      "id": "branch_combat",
      "name": "Combat Solution",
      "triggerCondition": "choice:destroy",
      "stages": ["stage_03_destroy"],
      "exclusiveWith": ["branch_technical", "branch_report"]
    }
  ],

  "prerequisites": [
    { "type": "level", "value": 2 },
    { "type": "flag", "value": "discovered_warehouse" }
  ],

  "recommendedLevel": 3,
  "trackable": true,
  "hidden": false,

  "rewards": {
    "xp": 200,
    "gold": 100,
    "items": [
      { "id": "item_warehouse_key", "type": "item" }
    ],
    "unlocks": [
      {
        "type": "location",
        "id": { "id": "location_warehouse_interior", "type": "location" }
      }
    ]
  },

  "questGiver": { "id": "npc_quest_giver_marcus", "type": "npc" },
  "startLocation": { "id": "city_verdant_heights", "type": "city" },
  "relevantLocations": [
    { "id": "location_warehouse_exterior", "type": "location" }
  ],

  "repeatable": false,
  "canFail": false,

  "tags": ["exploration", "skill_check", "branching", "tech"]
}
```

---

### 3.7 Dialogs Schema

Dialogs define conversation trees with choices, skill checks, and outcomes.

#### TypeScript Interface

```typescript
interface Dialog {
  // Metadata
  id: ContentID; // "dialog_merchant_greeting"
  version: string;
  name: string; // "Merchant Greeting"

  // Structure
  nodes: DialogNode[];
  startNode: string; // ID of first node

  // Context
  speaker?: ContentReference; // NPC ID
  location?: ContentReference; // Where this dialog occurs

  // Conditions
  prerequisites?: DialogPrerequisite[];

  // State
  repeatable: boolean;
  priority?: number; // For multiple dialogs from same NPC

  // Tags
  tags: string[];
}

interface DialogNode {
  id: string; // "node_greeting"

  // Content
  speaker: "npc" | "player" | "system";
  text: string;

  // Presentation
  emotion?: Emotion;
  portrait?: string; // Portrait variant
  animation?: string; // Portrait animation

  // Responses
  responses: DialogResponse[];

  // Auto-continue
  autoAdvance?: boolean; // Automatically proceed after delay
  advanceDelay?: number; // Milliseconds

  // Effects
  effects?: DialogEffect[];

  // Branching
  conditionalDisplay?: string; // Show only if condition met
}

type Emotion =
  | "neutral"
  | "happy"
  | "sad"
  | "angry"
  | "surprised"
  | "worried"
  | "excited";

interface DialogResponse {
  id: string;
  text: string; // What player says

  // Next node
  nextNode: string | null; // null = end dialog

  // Skill Check
  skillCheck?: SkillCheck;

  // Conditions
  condition?: string; // "flag:completed_quest", "stat:intelligence:15"
  disabled?: boolean; // Shown but grayed out
  disabledReason?: string; // "Requires 15 Intelligence"

  // Effects
  effects?: DialogEffect[];

  // UI
  icon?: string; // Icon for special responses
  color?: string; // Color for special responses (skill checks)
}

interface DialogEffect {
  type: DialogEffectType;
  value: any;
}

type DialogEffectType =
  | "set_flag"
  | "give_item"
  | "give_xp"
  | "give_gold"
  | "start_quest"
  | "complete_quest_objective"
  | "unlock_location"
  | "unlock_character"
  | "change_reputation"
  | "start_combat"
  | "play_animation";

interface DialogPrerequisite {
  type: "flag" | "quest" | "item" | "stat" | "reputation";
  value: string | number;
}
```

#### JSON Example

```json
{
  "id": "dialog_warehouse_discovery",
  "version": "1.0.0",
  "name": "Warehouse Discovery",

  "nodes": [
    {
      "id": "node_01_enter",
      "speaker": "system",
      "text": "You push open the heavy door. The warehouse interior is dim, lit only by flickering LED strips. In the corner, a maintenance drone lies on its side, sparking intermittently.",
      "responses": [
        {
          "id": "response_approach",
          "text": "Approach the drone carefully.",
          "nextNode": "node_02_examine"
        },
        {
          "id": "response_scan",
          "text": "[Investigation DC 12] Scan the drone from a distance.",
          "nextNode": "node_02_scan_success",
          "skillCheck": {
            "skill": "investigation",
            "dc": 12,
            "successOutcome": {
              "type": "xp",
              "value": 25,
              "message": "You identify the malfunction from afar."
            },
            "failureOutcome": {
              "type": "dialog",
              "value": "node_02_scan_failure",
              "message": "You can't determine the problem from here."
            },
            "canRetry": false
          },
          "icon": "magnifying_glass",
          "color": "#3498db"
        }
      ]
    },
    {
      "id": "node_02_examine",
      "speaker": "system",
      "text": "As you approach, the drone's optical sensor flickers to life. Its motors whir erratically, trying to right itself.",
      "emotion": "neutral",
      "responses": [
        {
          "id": "response_repair",
          "text": "[Arcana DC 14] Attempt to repair the drone.",
          "nextNode": "node_03_repair_attempt",
          "skillCheck": {
            "skill": "arcana",
            "dc": 14,
            "successOutcome": {
              "type": "quest",
              "value": "complete_repair_branch"
            },
            "failureOutcome": {
              "type": "dialog",
              "value": "node_03_repair_failure"
            },
            "canRetry": true
          },
          "icon": "wrench",
          "color": "#f39c12"
        },
        {
          "id": "response_destroy",
          "text": "It's too dangerous. Destroy it.",
          "nextNode": "node_03_combat_start",
          "effects": [
            {
              "type": "start_combat",
              "value": { "id": "enemy_corrupted_drone", "type": "enemy" }
            }
          ]
        },
        {
          "id": "response_leave",
          "text": "This is beyond me. I should report this.",
          "nextNode": null,
          "effects": [
            {
              "type": "set_flag",
              "value": "warehouse_drone_reported"
            }
          ]
        }
      ]
    },
    {
      "id": "node_02_scan_success",
      "speaker": "system",
      "text": "Your scanner reveals the issue: corrupted firmware, likely from a malicious code injection. The drone's neural pathways are scrambled, causing erratic behavior. It's repairable, but delicate work.",
      "emotion": "neutral",
      "effects": [
        {
          "type": "set_flag",
          "value": "identified_drone_malfunction"
        },
        {
          "type": "give_xp",
          "value": 25
        }
      ],
      "responses": [
        {
          "id": "response_repair_informed",
          "text": "[Arcana DC 12] Repair the drone (DC reduced due to scan).",
          "nextNode": "node_03_repair_success",
          "skillCheck": {
            "skill": "arcana",
            "dc": 12,
            "successOutcome": {
              "type": "quest",
              "value": "complete_repair_branch"
            },
            "failureOutcome": {
              "type": "dialog",
              "value": "node_03_repair_failure"
            },
            "canRetry": true
          },
          "icon": "wrench",
          "color": "#2ecc71"
        },
        {
          "id": "response_destroy_informed",
          "text": "Even knowing the issue, it's too risky. Destroy it.",
          "nextNode": "node_03_combat_start"
        }
      ]
    },
    {
      "id": "node_03_repair_success",
      "speaker": "system",
      "text": "Your fingers dance across the drone's maintenance panel. Lines of code scroll past as you isolate and purge the malicious firmware. With a satisfied beep, the drone rights itself and hovers at eye level. Its optical sensor pulses with a soft green light—gratitude, perhaps?",
      "emotion": "happy",
      "autoAdvance": false,
      "effects": [
        {
          "type": "give_item",
          "value": { "id": "item_drone_companion", "type": "item" }
        },
        {
          "type": "give_xp",
          "value": 150
        },
        {
          "type": "complete_quest_objective",
          "value": "obj_04_repair"
        }
      ],
      "responses": [
        {
          "id": "response_end",
          "text": "Welcome back, little friend.",
          "nextNode": null
        }
      ]
    },
    {
      "id": "node_03_repair_failure",
      "speaker": "system",
      "text": "Your attempt at repairs only makes things worse. The drone's motors screech louder, and sparks intensify. It's going to explode if you keep this up!",
      "emotion": "worried",
      "responses": [
        {
          "id": "response_retry",
          "text": "Try again more carefully.",
          "nextNode": "node_02_examine"
        },
        {
          "id": "response_abort",
          "text": "Back away and destroy it before it explodes.",
          "nextNode": "node_03_combat_start"
        }
      ]
    },
    {
      "id": "node_03_combat_start",
      "speaker": "system",
      "text": "The drone's optical sensor flares red. It recognizes you as a threat. Combat is inevitable.",
      "emotion": "angry",
      "autoAdvance": true,
      "advanceDelay": 2000,
      "effects": [
        {
          "type": "start_combat",
          "value": {
            "enemies": [
              { "id": "enemy_corrupted_drone", "type": "enemy" }
            ],
            "onVictory": "node_03_combat_victory"
          }
        }
      ],
      "responses": []
    },
    {
      "id": "node_03_combat_victory",
      "speaker": "system",
      "text": "The corrupted drone sparks one final time before powering down. It's beyond repair now, but at least the threat is neutralized.",
      "emotion": "neutral",
      "effects": [
        {
          "type": "give_xp",
          "value": 100
        },
        {
          "type": "give_gold",
          "value": 50
        },
        {
          "type": "complete_quest_objective",
          "value": "obj_04_combat"
        }
      ],
      "responses": [
        {
          "id": "response_end_combat",
          "text": "Leave the warehouse.",
          "nextNode": null
        }
      ]
    }
  ],

  "startNode": "node_01_enter",
  "location": { "id": "location_warehouse_interior", "type": "location" },

  "prerequisites": [
    {
      "type": "flag",
      "value": "entered_warehouse"
    }
  ],

  "repeatable": false,
  "priority": 1,

  "tags": ["quest", "branching", "skill_check", "combat"]
}
```

---

## Supporting Schemas

### 4.1 NPCs Schema

```typescript
interface NPC {
  id: ContentID;
  version: string;
  name: string;
  description: string;

  // Appearance
  sprite: SpriteData;
  portraits?: { [emotion: string]: string }; // Portrait variants

  // Personality
  personality: string;
  voice?: string; // Voice archetype for text styling

  // Interactions
  dialogs: ContentReference[]; // Dialog IDs
  quests?: ContentReference[]; // Quests this NPC gives

  // Shop (if merchant)
  shopType?: "general" | "weapons" | "armor" | "potions" | "materials";
  shopInventory?: ContentReference[];
  shopRestockTime?: number; // Hours

  // Recruitment (if recruitable)
  recruitable: boolean;
  recruitClass?: ContentReference; // Character class
  recruitRequirements?: UnlockRequirement[];

  // Location
  location: ContentReference; // Default location
  schedule?: NPCSchedule[]; // Movement schedule

  // Relationships
  faction?: string;
  reputation?: number; // Player reputation with this NPC

  tags: string[];
}

interface NPCSchedule {
  time: "morning" | "afternoon" | "evening" | "night";
  location: ContentReference;
}
```

### 4.2 Status Effects Schema

```typescript
interface StatusEffect {
  id: ContentID;
  version: string;
  name: string;
  description: string;

  type: "buff" | "debuff" | "neutral";

  // Duration
  duration: number; // Turns (-1 = permanent)
  stackable: boolean;
  maxStacks?: number;

  // Effects
  statModifiers?: Partial<CharacterStats>;
  dotDamage?: number; // Damage over time per turn
  hotHeal?: number; // Heal over time per turn

  // Special
  disablesActions?: boolean; // Stun, freeze, etc.
  preventedActions?: string[]; // "abilities", "movement", "items"

  // Visual
  icon: string;
  color: string;
  particles?: ParticleConfig;

  // Removal
  removedBy?: string[]; // "dispel", "death", "combat_end"

  tags: string[];
}
```

### 4.3 Loot Tables Schema

```typescript
interface LootTable {
  id: ContentID;
  version: string;
  name: string;

  drops: LootDrop[];

  // Roll rules
  rollMode: "all" | "pick_one" | "pick_multiple";
  maxDrops?: number;

  // Rarity modifiers
  rarityWeights?: { [rarity: string]: number };
  luckScaling?: boolean; // Player luck stat affects drops

  tags: string[];
}

interface LootDrop {
  item: ContentReference;
  chance: number; // 0-100 percentage
  quantity: { min: number; max: number };
  condition?: string; // "boss_killed", "perfect_victory"
}
```

### 4.4 Events Schema

```typescript
interface GameEvent {
  id: ContentID;
  version: string;
  name: string;
  description: string;

  type: "combat" | "dialog" | "cutscene" | "skill_check" | "choice";

  // Trigger
  triggerConditions: TriggerCondition[];
  triggerLocation?: ContentReference;

  // Content
  dialog?: ContentReference;
  enemies?: ContentReference[];
  cutsceneAnimation?: string;

  // Outcomes
  outcomes: EventOutcome[];

  // State
  oneTimeOnly: boolean;
  cooldown?: number; // Days before can trigger again

  tags: string[];
}

interface TriggerCondition {
  type: "enter_location" | "quest_stage" | "time" | "item_equipped" | "random";
  value: string | number;
  chance?: number; // For random triggers
}

interface EventOutcome {
  condition?: string;
  effects: DialogEffect[];
}
```

---

## File Structure & Organization

### Directory Layout

```
/data/
├── manifest.json                # Master index of all content
├── cities/
│   ├── city_verdant_heights.json
│   ├── city_industrial_core.json
│   └── city_transit_nexus.json
├── classes/
│   ├── class_technomancer.json
│   ├── class_biohacker.json
│   ├── class_guardian.json
│   └── class_harmonist.json
├── abilities/
│   ├── offensive/
│   │   ├── ability_lightning_strike.json
│   │   ├── ability_flame_burst.json
│   │   └── ability_frost_spike.json
│   ├── defensive/
│   │   ├── ability_static_shield.json
│   │   └── ability_regeneration.json
│   └── utility/
│       ├── ability_haste.json
│       └── ability_invisibility.json
├── items/
│   ├── weapons/
│   │   ├── item_weapon_solar_blade.json
│   │   ├── item_weapon_plasma_staff.json
│   │   └── item_weapon_photon_bow.json
│   ├── armor/
│   │   ├── item_armor_fiber_vest.json
│   │   └── item_armor_photon_shield.json
│   ├── accessories/
│   │   ├── item_accessory_speed_boots.json
│   │   └── item_accessory_power_ring.json
│   ├── consumables/
│   │   ├── item_potion_health_minor.json
│   │   ├── item_potion_health_major.json
│   │   └── item_food_energy_bar.json
│   └── materials/
│       ├── item_material_steel_ingot.json
│       └── item_material_solar_crystal.json
├── enemies/
│   ├── minions/
│   │   ├── enemy_corrupted_drone.json
│   │   └── enemy_rogue_bot.json
│   ├── elites/
│   │   ├── enemy_elite_enforcer.json
│   │   └── enemy_elite_technician.json
│   └── bosses/
│       ├── enemy_boss_megacorp_executive.json
│       └── enemy_boss_corrupted_ai.json
├── quests/
│   ├── main/
│   │   ├── quest_main_01_awakening.json
│   │   ├── quest_main_02_first_steps.json
│   │   └── quest_main_03_corporate_shadow.json
│   ├── side/
│   │   ├── quest_side_warehouse_mystery.json
│   │   └── quest_side_missing_supplies.json
│   └── character/
│       ├── quest_character_aria_backstory.json
│       └── quest_character_marcus_redemption.json
├── dialogs/
│   ├── npcs/
│   │   ├── dialog_merchant_jade_greeting.json
│   │   └── dialog_quest_giver_marcus.json
│   ├── quests/
│   │   ├── dialog_warehouse_discovery.json
│   │   └── dialog_quest_complete_warehouse.json
│   └── events/
│       ├── dialog_market_dispute.json
│       └── dialog_drone_attack.json
├── npcs/
│   ├── merchants/
│   │   ├── npc_merchant_jade.json
│   │   └── npc_merchant_tools.json
│   ├── quest_givers/
│   │   ├── npc_quest_giver_marcus.json
│   │   └── npc_quest_giver_elder.json
│   └── recruitable/
│       ├── npc_recruitable_technomancer_aria.json
│       └── npc_recruitable_biohacker_kai.json
├── status_effects/
│   ├── buffs/
│   │   ├── status_haste.json
│   │   └── status_strength_up.json
│   └── debuffs/
│       ├── status_stunned.json
│       ├── status_poisoned.json
│       └── status_burned.json
├── loot_tables/
│   ├── loot_corrupted_drone.json
│   ├── loot_treasure_chest_common.json
│   └── loot_boss_rewards.json
└── events/
    ├── event_market_dispute.json
    ├── event_drone_attack.json
    └── event_power_outage.json
```

### Manifest Structure

```json
{
  "version": "1.0.0",
  "lastUpdated": "2025-11-01T12:00:00Z",
  "contentVersion": "1.0.0",

  "collections": {
    "cities": {
      "path": "cities/",
      "count": 3,
      "files": [
        "city_verdant_heights.json",
        "city_industrial_core.json",
        "city_transit_nexus.json"
      ]
    },
    "classes": {
      "path": "classes/",
      "count": 4,
      "files": [
        "class_technomancer.json",
        "class_biohacker.json",
        "class_guardian.json",
        "class_harmonist.json"
      ]
    },
    "abilities": {
      "path": "abilities/",
      "count": 24,
      "subdirectories": [
        "offensive",
        "defensive",
        "utility"
      ]
    },
    "items": {
      "path": "items/",
      "count": 45,
      "subdirectories": [
        "weapons",
        "armor",
        "accessories",
        "consumables",
        "materials"
      ]
    },
    "enemies": {
      "path": "enemies/",
      "count": 18,
      "subdirectories": [
        "minions",
        "elites",
        "bosses"
      ]
    },
    "quests": {
      "path": "quests/",
      "count": 12,
      "subdirectories": [
        "main",
        "side",
        "character"
      ]
    },
    "dialogs": {
      "path": "dialogs/",
      "count": 35,
      "subdirectories": [
        "npcs",
        "quests",
        "events"
      ]
    },
    "npcs": {
      "path": "npcs/",
      "count": 22,
      "subdirectories": [
        "merchants",
        "quest_givers",
        "recruitable"
      ]
    },
    "status_effects": {
      "path": "status_effects/",
      "count": 15,
      "subdirectories": [
        "buffs",
        "debuffs"
      ]
    },
    "loot_tables": {
      "path": "loot_tables/",
      "count": 8
    },
    "events": {
      "path": "events/",
      "count": 10
    }
  },

  "dependencies": {
    "phaser": "^3.80.0",
    "typescript": "^5.0.0"
  }
}
```

---

## Naming Conventions

### Files

- **Lowercase with underscores**: `ability_lightning_strike.json`
- **Prefix with type**: `class_`, `item_`, `enemy_`, `quest_`, etc.
- **Descriptive names**: Avoid abbreviations
- **No spaces**: Use underscores
- **Extension**: Always `.json`

### IDs

- **Same as filename** (without extension): `ability_lightning_strike`
- **Globally unique**: No two content pieces share an ID
- **Hierarchical** (optional): `item_weapon_solar_blade`, `item_armor_fiber_vest`
- **Immutable**: Once published, never change IDs

### Properties

- **camelCase**: `attackPower`, `hitPoints`, `questGiver`
- **Clear and explicit**: `hp` vs `hitPoints` (prefer `hp` for brevity in stats)
- **Consistent terminology**: Always use same term (e.g., `sp` not `mana` or `mp`)

### Enums/Types

- **snake_case for values**: `"damage_dealer"`, `"single_enemy"`
- **No abbreviations**: `"physical"` not `"phys"`
- **Descriptive**: `"single_enemy"` not `"target1"`

---

## Versioning Strategy

### Content Versioning

Each content file has a `version` field following [Semantic Versioning](https://semver.org/):

```
MAJOR.MINOR.PATCH

1.0.0 → Initial release
1.0.1 → Bug fix (typo, balance tweak)
1.1.0 → New feature (add ability, new dialog branch)
2.0.0 → Breaking change (stat system overhaul, ID change)
```

### Compatibility

```typescript
interface VersionCompatibility {
  minEngineVersion: string; // Minimum game version required
  maxEngineVersion?: string; // Maximum compatible version (optional)
  deprecated?: boolean; // Will be removed in future version
  deprecationWarning?: string;
  replacedBy?: ContentID; // If deprecated, what replaces it
}
```

### Migration Strategy

```typescript
interface ContentMigration {
  from: string; // Old version
  to: string; // New version
  changes: Change[];
}

interface Change {
  type: "rename" | "add" | "remove" | "transform";
  field: string;
  oldValue?: any;
  newValue?: any;
  transformer?: string; // Function name for complex transforms
}
```

**Example Migration:**

```json
{
  "from": "1.0.0",
  "to": "2.0.0",
  "changes": [
    {
      "type": "rename",
      "field": "hitPoints",
      "oldValue": "hitPoints",
      "newValue": "hp"
    },
    {
      "type": "add",
      "field": "critDamage",
      "newValue": 150
    },
    {
      "type": "remove",
      "field": "legacyStatSystem"
    }
  ]
}
```

---

## Loading System

### Asset Manifest Loader

```typescript
class ContentLoader {
  private manifest: Manifest;
  private cache: Map<ContentID, any>;
  private validators: Map<ContentType, Validator>;

  constructor() {
    this.cache = new Map();
    this.validators = new Map();
    this.registerValidators();
  }

  // Load manifest first
  async loadManifest(): Promise<void> {
    const response = await fetch('/data/manifest.json');
    this.manifest = await response.json();
    console.log(`Manifest loaded: ${this.manifest.version}`);
  }

  // Load specific content by ID
  async loadContent<T>(id: ContentID, type: ContentType): Promise<T> {
    // Check cache first
    if (this.cache.has(id)) {
      return this.cache.get(id) as T;
    }

    // Determine file path
    const filePath = this.resolveFilePath(id, type);

    // Load from file
    const response = await fetch(filePath);
    if (!response.ok) {
      throw new Error(`Failed to load ${id}: ${response.statusText}`);
    }

    const content = await response.json();

    // Validate
    const validator = this.validators.get(type);
    if (validator) {
      const errors = validator.validate(content);
      if (errors.length > 0) {
        console.error(`Validation errors for ${id}:`, errors);
        throw new Error(`Invalid content: ${id}`);
      }
    }

    // Cache
    this.cache.set(id, content);

    return content as T;
  }

  // Load entire collection
  async loadCollection(type: ContentType): Promise<any[]> {
    const collection = this.manifest.collections[type];
    if (!collection) {
      throw new Error(`Unknown collection: ${type}`);
    }

    const promises = collection.files.map(file =>
      this.loadContentByFile(file, type)
    );

    return Promise.all(promises);
  }

  // Preload specific content (eager loading)
  async preload(ids: ContentID[]): Promise<void> {
    const promises = ids.map(id => {
      const type = this.inferTypeFromID(id);
      return this.loadContent(id, type);
    });

    await Promise.all(promises);
    console.log(`Preloaded ${ids.length} content items`);
  }

  // Lazy load on demand
  async lazyLoad<T>(id: ContentID, type: ContentType): Promise<T> {
    // Same as loadContent but with lower priority
    return this.loadContent<T>(id, type);
  }

  // Resolve references
  async resolveReference(ref: ContentReference): Promise<any> {
    return this.loadContent(ref.id, ref.type);
  }

  // Helper: infer type from ID prefix
  private inferTypeFromID(id: ContentID): ContentType {
    const prefix = id.split('_')[0];
    const typeMap: { [key: string]: ContentType } = {
      'city': 'city',
      'class': 'class',
      'ability': 'ability',
      'item': 'item',
      'enemy': 'enemy',
      'quest': 'quest',
      'dialog': 'dialog',
      'npc': 'npc',
      'status': 'status',
      'loot': 'loot',
      'event': 'event'
    };

    return typeMap[prefix] || 'unknown';
  }

  // Helper: resolve file path from ID
  private resolveFilePath(id: ContentID, type: ContentType): string {
    const collection = this.manifest.collections[type];
    const basePath = `/data/${collection.path}`;

    // Check subdirectories if present
    if (collection.subdirectories) {
      // Try to find file in subdirectories
      // Implementation depends on file organization
    }

    return `${basePath}${id}.json`;
  }

  // Register validators for each content type
  private registerValidators(): void {
    this.validators.set('city', new CityValidator());
    this.validators.set('class', new ClassValidator());
    this.validators.set('ability', new AbilityValidator());
    // ... register all validators
  }
}
```

### Preloading Strategy

```typescript
class PreloadStrategy {
  // Essential content loaded at boot
  static BOOT_PRELOAD = [
    'class_technomancer',
    'class_biohacker',
    'city_verdant_heights',
    'status_stunned',
    'status_poisoned'
  ];

  // Content loaded per scene
  static SCENE_PRELOAD = {
    'WorldMapScene': [
      // All city metadata
      'city_verdant_heights',
      'city_industrial_core'
    ],
    'CityScene': [
      // Current city data + NPCs
    ],
    'CombatScene': [
      // Enemy data + abilities
    ]
  };

  // Lazy load on demand
  static LAZY_LOAD = [
    'dialogs',  // Load when triggered
    'quests',   // Load when started
    'loot'      // Load when dropped
  ];
}
```

### Dependency Resolution

```typescript
class DependencyResolver {
  private loader: ContentLoader;
  private resolved: Set<ContentID>;

  constructor(loader: ContentLoader) {
    this.loader = loader;
    this.resolved = new Set();
  }

  // Resolve all dependencies for a content item
  async resolveDependencies(id: ContentID, type: ContentType): Promise<void> {
    if (this.resolved.has(id)) {
      return; // Already resolved
    }

    // Load the content
    const content = await this.loader.loadContent(id, type);

    // Extract references
    const references = this.extractReferences(content);

    // Recursively resolve dependencies
    for (const ref of references) {
      await this.resolveDependencies(ref.id, ref.type);
    }

    this.resolved.add(id);
  }

  // Extract all ContentReference from an object
  private extractReferences(obj: any): ContentReference[] {
    const refs: ContentReference[] = [];

    const traverse = (o: any) => {
      if (o && typeof o === 'object') {
        // Check if this is a ContentReference
        if (o.id && o.type) {
          refs.push(o as ContentReference);
        }

        // Recurse into properties
        for (const key in o) {
          if (o.hasOwnProperty(key)) {
            traverse(o[key]);
          }
        }
      }
    };

    traverse(obj);
    return refs;
  }

  // Get dependency tree
  getDependencyTree(id: ContentID): ContentID[] {
    // Returns all dependencies in load order
    return Array.from(this.resolved);
  }
}
```

---

## Validation Rules

### Schema Validator

```typescript
interface ValidationError {
  field: string;
  message: string;
  severity: "error" | "warning";
}

abstract class Validator {
  abstract validate(content: any): ValidationError[];

  // Common validation helpers
  protected requireField(obj: any, field: string): ValidationError | null {
    if (!(field in obj)) {
      return {
        field,
        message: `Required field "${field}" is missing`,
        severity: "error"
      };
    }
    return null;
  }

  protected requireType(obj: any, field: string, type: string): ValidationError | null {
    if (typeof obj[field] !== type) {
      return {
        field,
        message: `Field "${field}" must be type ${type}, got ${typeof obj[field]}`,
        severity: "error"
      };
    }
    return null;
  }

  protected requireRange(value: number, min: number, max: number, field: string): ValidationError | null {
    if (value < min || value > max) {
      return {
        field,
        message: `Field "${field}" must be between ${min} and ${max}, got ${value}`,
        severity: "error"
      };
    }
    return null;
  }

  protected requireEnum(value: string, allowed: string[], field: string): ValidationError | null {
    if (!allowed.includes(value)) {
      return {
        field,
        message: `Field "${field}" must be one of [${allowed.join(', ')}], got "${value}"`,
        severity: "error"
      };
    }
    return null;
  }

  protected requireReference(ref: ContentReference, field: string): ValidationError | null {
    if (!ref.id || !ref.type) {
      return {
        field,
        message: `Reference "${field}" must have "id" and "type" properties`,
        severity: "error"
      };
    }
    return null;
  }

  protected warnDeprecated(obj: any, field: string, replacement: string): ValidationError | null {
    if (field in obj) {
      return {
        field,
        message: `Field "${field}" is deprecated, use "${replacement}" instead`,
        severity: "warning"
      };
    }
    return null;
  }
}

class AbilityValidator extends Validator {
  validate(ability: Ability): ValidationError[] {
    const errors: ValidationError[] = [];

    // Required fields
    errors.push(this.requireField(ability, 'id'));
    errors.push(this.requireField(ability, 'name'));
    errors.push(this.requireField(ability, 'type'));
    errors.push(this.requireField(ability, 'spCost'));

    // Type checks
    errors.push(this.requireType(ability, 'spCost', 'number'));
    errors.push(this.requireType(ability, 'basePower', 'number'));

    // Range checks
    if (ability.spCost !== undefined) {
      errors.push(this.requireRange(ability.spCost, 0, 100, 'spCost'));
    }
    if (ability.accuracy !== undefined) {
      errors.push(this.requireRange(ability.accuracy, 0, 100, 'accuracy'));
    }

    // Enum checks
    if (ability.type) {
      const validTypes = ['attack', 'heal', 'buff', 'debuff', 'utility', 'summon', 'field'];
      errors.push(this.requireEnum(ability.type, validTypes, 'type'));
    }

    // Reference checks
    if (ability.effects) {
      ability.effects.forEach((effect, i) => {
        if (effect.statusEffect) {
          errors.push(this.requireReference(effect.statusEffect, `effects[${i}].statusEffect`));
        }
      });
    }

    // Business logic checks
    if (ability.type === 'attack' && !ability.basePower) {
      errors.push({
        field: 'basePower',
        message: 'Attack abilities must have basePower defined',
        severity: 'error'
      });
    }

    // Filter out null errors
    return errors.filter(e => e !== null) as ValidationError[];
  }
}

// Similar validators for other content types...
class ClassValidator extends Validator { /* ... */ }
class ItemValidator extends Validator { /* ... */ }
class EnemyValidator extends Validator { /* ... */ }
class QuestValidator extends Validator { /* ... */ }
class DialogValidator extends Validator { /* ... */ }
class CityValidator extends Validator { /* ... */ }
```

### Runtime Validation

```typescript
class RuntimeValidator {
  // Validate references exist
  static async validateReferences(loader: ContentLoader): Promise<ValidationError[]> {
    const errors: ValidationError[] = [];

    // Get all content
    const allContent = loader.getAllCached();

    for (const [id, content] of allContent) {
      // Extract all references
      const refs = this.extractReferences(content);

      // Check each reference exists
      for (const ref of refs) {
        try {
          await loader.loadContent(ref.id, ref.type);
        } catch (e) {
          errors.push({
            field: `${id} -> ${ref.id}`,
            message: `Reference to "${ref.id}" not found`,
            severity: "error"
          });
        }
      }
    }

    return errors;
  }

  // Validate circular dependencies
  static detectCircularDependencies(loader: ContentLoader): ValidationError[] {
    const errors: ValidationError[] = [];
    const visiting = new Set<ContentID>();
    const visited = new Set<ContentID>();

    const visit = (id: ContentID, path: ContentID[]) => {
      if (visiting.has(id)) {
        // Circular dependency detected
        errors.push({
          field: id,
          message: `Circular dependency: ${path.join(' -> ')} -> ${id}`,
          severity: "error"
        });
        return;
      }

      if (visited.has(id)) {
        return; // Already processed
      }

      visiting.add(id);

      // Get references
      const content = loader.getCached(id);
      const refs = this.extractReferences(content);

      for (const ref of refs) {
        visit(ref.id, [...path, id]);
      }

      visiting.delete(id);
      visited.add(id);
    };

    // Visit all content
    const allContent = loader.getAllCached();
    for (const [id] of allContent) {
      visit(id, []);
    }

    return errors;
  }

  private static extractReferences(obj: any): ContentReference[] {
    // Same as DependencyResolver.extractReferences
    const refs: ContentReference[] = [];
    // ... implementation
    return refs;
  }
}
```

---

## Reference Resolution

### Reference Resolver

```typescript
class ReferenceResolver {
  private loader: ContentLoader;

  constructor(loader: ContentLoader) {
    this.loader = loader;
  }

  // Resolve a single reference
  async resolve<T>(ref: ContentReference): Promise<T> {
    return this.loader.loadContent<T>(ref.id, ref.type);
  }

  // Resolve array of references
  async resolveAll<T>(refs: ContentReference[]): Promise<T[]> {
    const promises = refs.map(ref => this.resolve<T>(ref));
    return Promise.all(promises);
  }

  // Resolve references in nested object
  async resolveNested<T extends object>(obj: T): Promise<T> {
    const resolved = { ...obj };

    for (const key in resolved) {
      const value = resolved[key];

      // Check if ContentReference
      if (this.isReference(value)) {
        resolved[key] = await this.resolve(value as ContentReference);
      }
      // Recurse into arrays
      else if (Array.isArray(value)) {
        resolved[key] = await Promise.all(
          value.map(item =>
            this.isReference(item)
              ? this.resolve(item)
              : item
          )
        );
      }
      // Recurse into objects
      else if (typeof value === 'object' && value !== null) {
        resolved[key] = await this.resolveNested(value);
      }
    }

    return resolved;
  }

  private isReference(obj: any): boolean {
    return obj && typeof obj === 'object' && 'id' in obj && 'type' in obj;
  }
}
```

### Usage Example

```typescript
// Load a quest with all references resolved
async function loadQuestFull(questId: ContentID): Promise<Quest> {
  const loader = new ContentLoader();
  await loader.loadManifest();

  // Load quest
  const quest = await loader.loadContent<Quest>(questId, 'quest');

  // Resolve all references
  const resolver = new ReferenceResolver(loader);
  const questResolved = await resolver.resolveNested(quest);

  // Now questResolved.questGiver is full NPC object, not just reference
  // questResolved.rewards.items are full Item objects, not just references

  return questResolved;
}
```

---

## Implementation Examples

### Example 1: Loading a Character Class

```typescript
async function loadCharacterClass(classId: ContentID): Promise<CharacterClass> {
  const loader = new ContentLoader();
  await loader.loadManifest();

  // Load class
  const characterClass = await loader.loadContent<CharacterClass>(classId, 'class');

  // Preload starting abilities
  const abilityRefs = characterClass.startingAbilities;
  const abilities = await Promise.all(
    abilityRefs.map(ref => loader.loadContent<Ability>(ref.id, ref.type))
  );

  console.log(`Loaded ${characterClass.name} with ${abilities.length} starting abilities`);

  return characterClass;
}
```

### Example 2: Loading a City with All Content

```typescript
async function loadCityComplete(cityId: ContentID): Promise<City> {
  const loader = new ContentLoader();
  await loader.loadManifest();

  // Load city
  const city = await loader.loadContent<City>(cityId, 'city');

  // Resolve all dependencies
  const resolver = new DependencyResolver(loader);
  await resolver.resolveDependencies(cityId, 'city');

  // Get dependency tree
  const deps = resolver.getDependencyTree(cityId);
  console.log(`Loaded ${city.name} with ${deps.length} dependencies`);

  return city;
}
```

### Example 3: Validating All Content

```typescript
async function validateAllContent(): Promise<void> {
  const loader = new ContentLoader();
  await loader.loadManifest();

  // Load all content
  const collections: ContentType[] = [
    'city', 'class', 'ability', 'item',
    'enemy', 'quest', 'dialog', 'npc'
  ];

  for (const type of collections) {
    console.log(`Validating ${type}...`);
    const content = await loader.loadCollection(type);
    console.log(`  ✓ Loaded ${content.length} ${type} items`);
  }

  // Validate references
  console.log('Validating references...');
  const refErrors = await RuntimeValidator.validateReferences(loader);
  if (refErrors.length > 0) {
    console.error('Reference validation errors:', refErrors);
  } else {
    console.log('  ✓ All references valid');
  }

  // Check circular dependencies
  console.log('Checking circular dependencies...');
  const circularErrors = RuntimeValidator.detectCircularDependencies(loader);
  if (circularErrors.length > 0) {
    console.error('Circular dependency errors:', circularErrors);
  } else {
    console.log('  ✓ No circular dependencies');
  }

  console.log('Validation complete!');
}
```

### Example 4: Hot-Reloading Content (Development)

```typescript
class HotReloadManager {
  private loader: ContentLoader;
  private watchers: Map<ContentID, Function[]>;

  constructor(loader: ContentLoader) {
    this.loader = loader;
    this.watchers = new Map();
  }

  // Watch for changes to a content item
  watch(id: ContentID, callback: (newContent: any) => void): void {
    if (!this.watchers.has(id)) {
      this.watchers.set(id, []);
    }
    this.watchers.get(id)!.push(callback);
  }

  // Reload content and notify watchers
  async reload(id: ContentID, type: ContentType): Promise<void> {
    // Clear cache
    this.loader.clearCache(id);

    // Reload
    const newContent = await this.loader.loadContent(id, type);

    // Notify watchers
    const callbacks = this.watchers.get(id) || [];
    callbacks.forEach(cb => cb(newContent));

    console.log(`Hot-reloaded: ${id}`);
  }
}

// Usage in Phaser scene
class CombatScene extends Phaser.Scene {
  private hotReload: HotReloadManager;

  create() {
    this.hotReload = new HotReloadManager(contentLoader);

    // Watch for ability changes during development
    this.hotReload.watch('ability_lightning_strike', (ability) => {
      console.log('Ability updated:', ability);
      this.updateAbilityUI(ability);
    });
  }
}
```

---

## Summary

This specification provides a complete, production-ready content schema system for Solar Punk Chronicles RPG. Key features:

**✓ Type Safety**: Full TypeScript interfaces for all content types
**✓ Validation**: Comprehensive validation at load time
**✓ Modularity**: Each content type in separate files
**✓ References**: Clean reference system with automatic resolution
**✓ Versioning**: Semantic versioning with migration support
**✓ Performance**: Lazy loading and caching strategies
**✓ Developer Experience**: Hot-reloading for rapid iteration

**Next Steps:**
1. Generate TypeScript type definitions from this spec
2. Implement validators for each content type
3. Build ContentLoader with caching and preloading
4. Create initial content files for Phase 1
5. Integrate with Phaser 3 scene system

**File Location**: `/home/user/test/docs/research-and-preparation/technical-architecture/CONTENT-SCHEMAS-SPECIFICATION.md`

Developers can now implement the content system directly from this specification. All schemas are complete with examples, all systems are architected, and all patterns are documented.

🌱⚡🎮

---

*End of Content Schemas Specification*
