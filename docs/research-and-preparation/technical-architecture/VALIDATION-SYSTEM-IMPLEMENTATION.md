# Validation System Implementation Guide
## Production-Ready Content Validation for Solar Punk Chronicles

**Document Version:** 1.0
**Last Updated:** 2025-11-01
**Implementation Timeline:** Days 1-2 of Phase 1
**Status:** Ready for Implementation

---

## Table of Contents

1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Zod Schema Definitions](#zod-schema-definitions)
4. [ContentValidator Class](#contentvalidator-class)
5. [CLI Validation Tool](#cli-validation-tool)
6. [Runtime Validation](#runtime-validation)
7. [Reference Validation](#reference-validation)
8. [Integration with Phaser](#integration-with-phaser)
9. [Usage Examples](#usage-examples)
10. [Error Message Samples](#error-message-samples)
11. [Installation and Setup](#installation-and-setup)

---

## Overview

### The Problem

As identified in the Architecture Review, the current system has **ZERO content validation**, which is a critical blocker for Phase 1 development. Without validation:
- Malformed JSON causes silent failures
- Missing required fields discovered during gameplay
- Typos in IDs cause undefined errors
- Cross-reference validation impossible
- Designer productivity blocked

### The Solution

A three-layer validation system:

1. **Build-Time Validation** (CLI tool using Zod)
   - Validates all JSON files before deployment
   - Fails build on invalid content
   - Provides detailed error messages

2. **Runtime Validation** (Development mode)
   - Validates content as it loads
   - Shows error overlay with helpful messages
   - Only runs in development (zero production overhead)

3. **Reference Validation** (Cross-file integrity)
   - Validates all ID references exist
   - Detects circular dependencies
   - Warns about unused content

### Key Benefits

- **Fast to implement**: 2 days (critical for Phase 1 timeline)
- **Zero production overhead**: Validation runs only in dev/build
- **Type-safe**: Zod schemas generate TypeScript types
- **Comprehensive**: All content types validated
- **Developer-friendly**: Clear error messages with file/line numbers

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Content JSON Files                       │
│  /src/data/static/characters/*.json                         │
│  /src/data/static/abilities/*.json                          │
│  /src/data/static/items/*.json                              │
│  /src/data/static/enemies/*.json                            │
│  /src/data/static/quests/*.json                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
         ┌──────────────────────────────────────┐
         │    Build-Time Validation (CLI)       │
         │  scripts/validate-content.ts         │
         │  - Schema validation (Zod)           │
         │  - Cross-reference checks            │
         │  - Circular dependency detection     │
         └──────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                │                       │
         [FAIL] │                       │ [PASS]
                ▼                       ▼
      ┌──────────────┐          ┌──────────────┐
      │ Build Failed │          │ Build Success│
      │ Show Errors  │          │ Deploy Ready │
      └──────────────┘          └──────────────┘
                                       │
                                       ▼
                          ┌──────────────────────────┐
                          │  Runtime (Development)   │
                          │  ContentLoader.ts        │
                          │  - Load JSON             │
                          │  - Validate with Zod     │
                          │  - Show error overlay    │
                          └──────────────────────────┘
                                       │
                                       ▼
                          ┌──────────────────────────┐
                          │   Phaser Game Cache      │
                          │   Validated, Type-Safe   │
                          └──────────────────────────┘
```

### File Structure

```
project-root/
├── scripts/
│   ├── validate-content.ts          # CLI validation tool
│   └── generate-types.ts            # Type generation (future)
│
├── src/
│   ├── data/
│   │   ├── static/                  # Hand-authored content
│   │   │   ├── characters/
│   │   │   ├── abilities/
│   │   │   ├── items/
│   │   │   ├── enemies/
│   │   │   ├── quests/
│   │   │   ├── cities/
│   │   │   ├── npcs/
│   │   │   └── dialogs/
│   │   │
│   │   └── schemas/                 # Zod schema definitions
│   │       ├── index.ts             # Export all schemas
│   │       ├── character.schema.ts
│   │       ├── ability.schema.ts
│   │       ├── item.schema.ts
│   │       ├── enemy.schema.ts
│   │       ├── quest.schema.ts
│   │       ├── city.schema.ts
│   │       ├── npc.schema.ts
│   │       └── dialog.schema.ts
│   │
│   ├── systems/
│   │   ├── ContentValidator.ts      # Runtime validator
│   │   └── ContentLoader.ts         # Data loading with validation
│   │
│   └── types/
│       └── content.types.ts         # Generated from Zod schemas
│
└── package.json
```

---

## Zod Schema Definitions

### Installation

```bash
npm install zod
npm install -D @types/node
```

### Base Schemas (`src/data/schemas/base.schema.ts`)

```typescript
import { z } from 'zod';

// Common patterns
export const ContentIdSchema = z.string()
  .min(1)
  .regex(/^[a-z0-9_]+$/, 'ID must be lowercase letters, numbers, and underscores only');

export const ContentReferenceSchema = z.object({
  id: ContentIdSchema,
  type: z.enum([
    'character', 'ability', 'item', 'enemy', 'quest',
    'dialog', 'city', 'npc', 'status', 'loot', 'event'
  ])
});

export const VersionSchema = z.string().regex(/^\d+\.\d+\.\d+$/, 'Version must be semver format (e.g., 1.0.0)');

// D&D Ability Scores
export const AbilityScoresSchema = z.object({
  strength: z.number().int().min(1).max(30),
  dexterity: z.number().int().min(1).max(30),
  constitution: z.number().int().min(1).max(30),
  intelligence: z.number().int().min(1).max(30),
  wisdom: z.number().int().min(1).max(30),
  charisma: z.number().int().min(1).max(30)
});

export const DnDSkillSchema = z.enum([
  'athletics', 'acrobatics', 'sleight_of_hand', 'stealth',
  'arcana', 'history', 'investigation', 'nature', 'religion',
  'animal_handling', 'insight', 'medicine', 'perception', 'survival',
  'deception', 'intimidation', 'performance', 'persuasion'
]);

// Character Stats
export const CharacterStatsSchema = z.object({
  hp: z.number().int().positive(),
  sp: z.number().int().nonnegative(),
  attack: z.number().int().nonnegative(),
  magic: z.number().int().nonnegative(),
  defense: z.number().int().nonnegative(),
  resistance: z.number().int().nonnegative(),
  speed: z.number().int().positive(),
  accuracy: z.number().int().min(0).max(100),
  evasion: z.number().int().min(0).max(100),
  critRate: z.number().int().min(0).max(100),
  critDamage: z.number().int().min(100).max(500)
});

// Sprite Data
export const SpriteDataSchema = z.object({
  type: z.enum(['procedural', 'texture']),
  shape: z.enum(['rectangle', 'circle', 'triangle']).optional(),
  baseColor: z.string().regex(/^#[0-9A-Fa-f]{6}$/).optional(),
  symbolId: z.string().optional(),
  texturePath: z.string().optional(),
  frameWidth: z.number().int().positive().optional(),
  frameHeight: z.number().int().positive().optional()
});

// Export type helpers
export type ContentReference = z.infer<typeof ContentReferenceSchema>;
export type AbilityScores = z.infer<typeof AbilityScoresSchema>;
export type CharacterStats = z.infer<typeof CharacterStatsSchema>;
export type SpriteData = z.infer<typeof SpriteDataSchema>;
```

### Character Class Schema (`src/data/schemas/character.schema.ts`)

```typescript
import { z } from 'zod';
import {
  ContentIdSchema,
  VersionSchema,
  ContentReferenceSchema,
  CharacterStatsSchema,
  AbilityScoresSchema,
  DnDSkillSchema,
  SpriteDataSchema
} from './base.schema';

const StatGrowthSchema = z.object({
  hp: z.number().int().nonnegative(),
  sp: z.number().int().nonnegative(),
  attack: z.number().nonnegative(),
  magic: z.number().nonnegative(),
  defense: z.number().nonnegative(),
  resistance: z.number().nonnegative(),
  speed: z.number().nonnegative(),
  accuracy: z.number().nonnegative(),
  evasion: z.number().nonnegative(),
  critRate: z.number().nonnegative(),
  critDamage: z.number().nonnegative()
});

const AbilityProgressionSchema = z.object({
  ability: ContentReferenceSchema,
  unlockLevel: z.number().int().min(1).max(30),
  prerequisite: ContentReferenceSchema.optional()
});

const EquipmentSlotSchema = z.object({
  slot: z.enum(['weapon', 'armor', 'accessory']),
  required: z.boolean()
});

const UnlockMethodSchema = z.object({
  type: z.enum(['story', 'quest', 'discovery', 'challenge']),
  requirement: ContentReferenceSchema.optional(),
  automatic: z.boolean().optional()
});

export const CharacterClassSchema = z.object({
  // Metadata
  id: ContentIdSchema,
  version: VersionSchema,
  name: z.string().min(1).max(50),
  description: z.string().min(1).max(500),
  flavor: z.string().max(200),

  // Visuals
  sprite: SpriteDataSchema,
  primaryColor: z.string().regex(/^#[0-9A-Fa-f]{6}$/),
  secondaryColor: z.string().regex(/^#[0-9A-Fa-f]{6}$/),
  icon: z.string(),

  // Base Stats
  baseStats: CharacterStatsSchema,
  statGrowth: StatGrowthSchema,

  // D&D Abilities
  abilityScores: AbilityScoresSchema,
  skillProficiencies: z.array(DnDSkillSchema),
  savingThrowProficiencies: z.array(z.enum(['strength', 'dexterity', 'constitution', 'intelligence', 'wisdom', 'charisma'])),

  // Combat Role
  role: z.enum(['tank', 'damage_dealer', 'support', 'healer', 'crowd_control', 'hybrid']),
  preferredRow: z.enum(['front', 'back']),

  // Abilities
  startingAbilities: z.array(ContentReferenceSchema),
  learnableAbilities: z.array(AbilityProgressionSchema),

  // Equipment
  equipmentSlots: z.array(EquipmentSlotSchema),
  weaponProficiencies: z.array(z.enum(['sword', 'staff', 'bow', 'dagger', 'hammer', 'gun', 'wand'])),
  armorProficiencies: z.array(z.enum(['light', 'medium', 'heavy', 'cloth'])),

  // Progression
  xpCurve: z.enum(['fast', 'standard', 'slow']),
  maxLevel: z.number().int().min(1).max(100),

  // Unlock
  unlockMethod: UnlockMethodSchema,

  // Tags
  tags: z.array(z.string())
});

export type CharacterClass = z.infer<typeof CharacterClassSchema>;
```

### Ability Schema (`src/data/schemas/ability.schema.ts`)

```typescript
import { z } from 'zod';
import { ContentIdSchema, VersionSchema, ContentReferenceSchema } from './base.schema';

const TargetCountSchema = z.object({
  min: z.number().int().positive(),
  max: z.number().int().positive()
}).refine(data => data.max >= data.min, {
  message: 'max must be >= min'
});

const AbilityEffectSchema = z.object({
  type: z.enum(['damage', 'heal', 'status', 'stat_change', 'position_change', 'summon', 'remove_status', 'revive']),
  value: z.number().optional(),
  duration: z.number().int().positive().optional(),
  statusEffect: ContentReferenceSchema.optional(),
  chance: z.number().int().min(0).max(100).optional(),
  onTrigger: z.enum(['hit', 'kill', 'crit', 'turn_start', 'turn_end']).optional()
});

const ParticleConfigSchema = z.object({
  type: z.enum(['spark', 'flame', 'frost', 'heal', 'buff', 'debuff']),
  color: z.string().regex(/^#[0-9A-Fa-f]{6}$/),
  count: z.number().int().positive(),
  duration: z.number().int().positive()
});

const AnimationDataSchema = z.object({
  castAnimation: z.string(),
  projectileAnimation: z.string().optional(),
  hitAnimation: z.string(),
  soundEffect: z.string().optional(),
  screenShake: z.boolean().optional(),
  duration: z.number().int().positive(),
  particles: ParticleConfigSchema.optional()
});

const AbilityRequirementSchema = z.object({
  type: z.enum(['level', 'weapon', 'status', 'resource']),
  value: z.union([z.number(), z.string()])
});

const UpgradeEffectSchema = z.object({
  type: z.enum(['increase_power', 'reduce_cost', 'add_effect', 'increase_targets']),
  value: z.union([z.number(), AbilityEffectSchema])
});

const AbilityUpgradeSchema = z.object({
  id: z.string(),
  name: z.string(),
  description: z.string(),
  effect: UpgradeEffectSchema
});

export const AbilitySchema = z.object({
  // Metadata
  id: ContentIdSchema,
  version: VersionSchema,
  name: z.string().min(1).max(50),
  description: z.string().min(1).max(500),
  flavorText: z.string().max(200).optional(),

  // Type
  type: z.enum(['attack', 'heal', 'buff', 'debuff', 'utility', 'summon', 'field']),
  element: z.enum(['physical', 'fire', 'ice', 'lightning', 'earth', 'wind', 'light', 'dark', 'neutral']),
  damageType: z.enum(['physical', 'magical']).optional(),

  // Costs
  spCost: z.number().int().nonnegative(),
  cooldown: z.number().int().nonnegative().optional(),
  charges: z.number().int().positive().optional(),

  // Targeting
  targeting: z.enum(['single_enemy', 'all_enemies', 'random_enemies', 'single_ally', 'all_allies', 'self', 'area']),
  range: z.number().int().positive(),
  targetCount: TargetCountSchema,

  // Effects
  effects: z.array(AbilityEffectSchema),

  // Power
  basePower: z.number().int().nonnegative().optional(),
  scaling: z.enum(['attack', 'magic', 'speed', 'hp_percent', 'target_missing_hp', 'fixed']),
  scalingRatio: z.number().positive(),

  // Hit Calculation
  accuracy: z.number().int().min(0).max(100),
  canCrit: z.boolean(),
  critBonus: z.number().int().min(0).max(100).optional(),

  // Animation
  animation: AnimationDataSchema,

  // Requirements
  requirements: z.array(AbilityRequirementSchema).optional(),

  // Upgrades
  upgradeable: z.boolean(),
  upgradeOptions: z.array(AbilityUpgradeSchema).optional(),

  // Tags
  tags: z.array(z.string())
});

export type Ability = z.infer<typeof AbilitySchema>;
```

### Item Schema (`src/data/schemas/item.schema.ts`)

```typescript
import { z } from 'zod';
import { ContentIdSchema, VersionSchema, ContentReferenceSchema, CharacterStatsSchema } from './base.schema';

const ItemEffectSchema = z.object({
  type: z.enum(['damage', 'heal', 'status', 'stat_change', 'position_change', 'summon', 'remove_status', 'revive']),
  value: z.number(),
  target: z.enum(['user', 'enemy', 'all_allies']),
  duration: z.number().int().positive().optional()
});

const PassiveEffectSchema = z.object({
  type: z.enum(['stat_boost', 'element_resist', 'status_immune', 'ability_boost']),
  value: z.union([z.number(), z.string()]),
  condition: z.string().optional()
});

const ConsumableDataSchema = z.object({
  usableInCombat: z.boolean(),
  usableInExploration: z.boolean(),
  targetType: z.enum(['self', 'single_ally', 'all_allies']),
  effectOnUse: z.array(ItemEffectSchema)
});

const CraftingIngredientSchema = z.object({
  item: ContentReferenceSchema,
  quantity: z.number().int().positive()
});

const CraftingRecipeSchema = z.object({
  ingredients: z.array(CraftingIngredientSchema),
  craftingStation: z.string().optional(),
  skillRequired: z.object({
    skill: z.string(),
    dc: z.number().int().min(1).max(30)
  }).optional(),
  craftingTime: z.number().int().positive().optional(),
  yield: z.number().int().positive()
});

const EquipmentRequirementSchema = z.object({
  type: z.enum(['class', 'level', 'stat', 'quest']),
  value: z.union([z.string(), z.number()])
});

export const ItemSchema = z.object({
  // Metadata
  id: ContentIdSchema,
  version: VersionSchema,
  name: z.string().min(1).max(50),
  description: z.string().min(1).max(500),
  flavorText: z.string().max(200).optional(),

  // Type
  category: z.enum(['equipment', 'consumable', 'material', 'key_item', 'quest_item']),
  subtype: z.enum(['weapon', 'armor', 'accessory', 'potion', 'food', 'scroll', 'ore', 'plant', 'tech']),

  // Rarity
  rarity: z.enum(['common', 'uncommon', 'rare', 'epic', 'legendary']),

  // Value
  value: z.number().int().nonnegative(),
  stackable: z.boolean(),
  maxStack: z.number().int().positive().optional(),

  // Equipment
  equipmentSlot: z.enum(['weapon', 'armor', 'accessory']).optional(),
  stats: CharacterStatsSchema.partial().optional(),
  requirements: z.array(EquipmentRequirementSchema).optional(),

  // Weapon
  weaponType: z.enum(['sword', 'staff', 'bow', 'dagger', 'hammer', 'gun', 'wand']).optional(),
  attackPower: z.number().int().nonnegative().optional(),
  attackSpeed: z.number().optional(),
  range: z.number().int().positive().optional(),

  // Armor
  armorType: z.enum(['light', 'medium', 'heavy', 'cloth']).optional(),
  defensePower: z.number().int().nonnegative().optional(),
  resistancePower: z.number().int().nonnegative().optional(),

  // Effects
  effects: z.array(ItemEffectSchema).optional(),
  passiveEffects: z.array(PassiveEffectSchema).optional(),

  // Consumable
  consumable: ConsumableDataSchema.optional(),

  // Crafting
  craftable: z.boolean(),
  craftingRecipe: CraftingRecipeSchema.optional(),
  ingredients: z.boolean().optional(),

  // Visuals
  icon: z.string(),
  color: z.string().regex(/^#[0-9A-Fa-f]{6}$/),

  // Unlock
  unlockRequirements: z.array(z.object({
    type: z.enum(['quest', 'level', 'item', 'flag']),
    id: ContentIdSchema.optional(),
    value: z.number().optional(),
    flag: z.string().optional()
  })).optional(),

  // Tags
  tags: z.array(z.string())
});

export type Item = z.infer<typeof ItemSchema>;
```

### Enemy Schema (`src/data/schemas/enemy.schema.ts`)

```typescript
import { z } from 'zod';
import {
  ContentIdSchema,
  VersionSchema,
  ContentReferenceSchema,
  CharacterStatsSchema,
  AbilityScoresSchema,
  SpriteDataSchema
} from './base.schema';

const AbilityUsagePatternSchema = z.object({
  ability: ContentReferenceSchema,
  priority: z.number().int().min(0).max(100),
  condition: z.string().optional()
});

const SpecialBehaviorSchema = z.object({
  type: z.enum(['summon', 'transform', 'enrage', 'heal_allies']),
  trigger: z.string(),
  action: z.union([ContentReferenceSchema, z.string()])
});

const AIPatternSchema = z.object({
  strategy: z.enum(['aggressive', 'defensive', 'balanced', 'support', 'random']),
  priorityTarget: z.enum(['lowest_hp', 'highest_damage', 'back_row', 'front_row', 'random', 'healer_first']),
  abilityUsage: z.array(AbilityUsagePatternSchema),
  specialBehaviors: z.array(SpecialBehaviorSchema).optional()
});

const FormationPreferenceSchema = z.object({
  row: z.enum(['front', 'back']),
  position: z.number().int().min(0).max(2).optional()
});

const ElementalResistancesSchema = z.object({
  fire: z.number().int().min(-100).max(100).optional(),
  ice: z.number().int().min(-100).max(100).optional(),
  lightning: z.number().int().min(-100).max(100).optional(),
  earth: z.number().int().min(-100).max(100).optional(),
  wind: z.number().int().min(-100).max(100).optional(),
  light: z.number().int().min(-100).max(100).optional(),
  dark: z.number().int().min(-100).max(100).optional()
});

const StatusResistancesSchema = z.object({
  stun: z.number().int().min(0).max(100).optional(),
  poison: z.number().int().min(0).max(100).optional(),
  burn: z.number().int().min(0).max(100).optional(),
  freeze: z.number().int().min(0).max(100).optional(),
  blind: z.number().int().min(0).max(100).optional(),
  silence: z.number().int().min(0).max(100).optional(),
  slow: z.number().int().min(0).max(100).optional()
});

const BossPhaseSchema = z.object({
  phaseNumber: z.number().int().positive(),
  hpThreshold: z.number().int().min(0).max(100),
  abilities: z.array(ContentReferenceSchema),
  statChanges: CharacterStatsSchema.partial().optional(),
  dialogue: z.string().optional(),
  transformation: SpriteDataSchema.optional()
});

const SpawnConditionSchema = z.object({
  type: z.enum(['time', 'weather', 'quest', 'event']),
  value: z.string()
});

export const EnemySchema = z.object({
  // Metadata
  id: ContentIdSchema,
  version: VersionSchema,
  name: z.string().min(1).max(50),
  description: z.string().min(1).max(500),

  // Type
  type: z.enum(['construct', 'beast', 'humanoid', 'undead', 'elemental', 'aberration']),
  category: z.enum(['minion', 'normal', 'elite', 'mini_boss', 'boss']),

  // Stats
  stats: CharacterStatsSchema,
  level: z.number().int().positive(),
  abilityScores: AbilityScoresSchema.optional(),

  // Combat
  abilities: z.array(ContentReferenceSchema),
  aiPattern: AIPatternSchema,
  formation: FormationPreferenceSchema,

  // Resistances
  elementalResistances: ElementalResistancesSchema,
  statusResistances: StatusResistancesSchema,

  // Loot
  lootTable: ContentReferenceSchema,
  xpReward: z.number().int().nonnegative(),
  goldReward: z.object({
    min: z.number().int().nonnegative(),
    max: z.number().int().nonnegative()
  }).refine(data => data.max >= data.min, {
    message: 'max must be >= min'
  }),

  // Visual
  sprite: SpriteDataSchema,
  color: z.string().regex(/^#[0-9A-Fa-f]{6}$/),
  size: z.enum(['small', 'medium', 'large', 'boss']),

  // Behavior
  aggressive: z.boolean(),
  flees: z.boolean(),
  fleeThreshold: z.number().min(0).max(1).optional(),

  // Special
  isBoss: z.boolean(),
  phases: z.array(BossPhaseSchema).optional(),

  // Spawn
  spawnConditions: z.array(SpawnConditionSchema).optional(),

  // Tags
  tags: z.array(z.string())
});

export type Enemy = z.infer<typeof EnemySchema>;
```

### Quest Schema (`src/data/schemas/quest.schema.ts`)

```typescript
import { z } from 'zod';
import { ContentIdSchema, VersionSchema, ContentReferenceSchema } from './base.schema';

const ObjectiveSchema = z.object({
  id: z.string(),
  type: z.enum(['kill', 'collect', 'talk', 'explore', 'skill_check', 'escort', 'defend', 'craft']),
  description: z.string(),
  target: ContentReferenceSchema.optional(),
  quantity: z.number().int().positive().optional(),
  currentProgress: z.number().int().nonnegative().optional(),
  requiredProgress: z.number().int().positive().optional(),
  completed: z.boolean(),
  optional: z.boolean(),
  hidden: z.boolean().optional()
});

const OutcomeSchema = z.object({
  type: z.enum(['xp', 'item', 'quest', 'dialog', 'event', 'unlock']),
  value: z.any(),
  message: z.string().optional()
});

const SkillCheckSchema = z.object({
  skill: z.string(),
  dc: z.number().int().min(1).max(30),
  successOutcome: OutcomeSchema,
  failureOutcome: OutcomeSchema,
  canRetry: z.boolean()
});

const ConditionalStageSchema = z.object({
  stage: z.string(),
  condition: z.string()
});

const QuestStageSchema = z.object({
  id: z.string(),
  name: z.string(),
  description: z.string(),
  objectives: z.array(ObjectiveSchema),
  startDialog: ContentReferenceSchema.optional(),
  completeDialog: ContentReferenceSchema.optional(),
  rewards: z.object({
    xp: z.number().int().nonnegative(),
    gold: z.number().int().nonnegative(),
    items: z.array(ContentReferenceSchema).optional(),
    abilities: z.array(ContentReferenceSchema).optional()
  }).partial().optional(),
  nextStage: z.union([z.string(), z.array(ConditionalStageSchema)]).optional()
});

const QuestBranchSchema = z.object({
  id: z.string(),
  name: z.string(),
  triggerCondition: z.string(),
  stages: z.array(z.string()),
  exclusiveWith: z.array(z.string()).optional()
});

const QuestPrerequisiteSchema = z.object({
  type: z.enum(['quest', 'level', 'class', 'item', 'flag']),
  value: z.union([z.string(), z.number()])
});

const UnlockRewardSchema = z.object({
  type: z.enum(['character', 'location', 'shop', 'feature']),
  id: ContentReferenceSchema
});

const ReputationRewardSchema = z.object({
  faction: z.string(),
  amount: z.number().int()
});

const QuestRewardSchema = z.object({
  xp: z.number().int().nonnegative(),
  gold: z.number().int().nonnegative(),
  items: z.array(ContentReferenceSchema).optional(),
  abilities: z.array(ContentReferenceSchema).optional(),
  unlocks: z.array(UnlockRewardSchema).optional(),
  reputation: z.array(ReputationRewardSchema).optional()
});

const FailureConditionSchema = z.object({
  type: z.enum(['time_limit', 'character_death', 'objective_failed']),
  value: z.union([z.string(), z.number()]).optional(),
  consequence: z.string().optional()
});

export const QuestSchema = z.object({
  // Metadata
  id: ContentIdSchema,
  version: VersionSchema,
  name: z.string().min(1).max(100),
  description: z.string().min(1).max(1000),

  // Type
  type: z.enum(['main', 'side', 'character', 'daily', 'challenge']),
  category: z.enum(['story', 'combat', 'exploration', 'social', 'crafting', 'collection']),

  // Structure
  stages: z.array(QuestStageSchema),
  branches: z.array(QuestBranchSchema).optional(),

  // Requirements
  prerequisites: z.array(QuestPrerequisiteSchema),
  recommendedLevel: z.number().int().positive(),

  // Objectives
  trackable: z.boolean(),
  hidden: z.boolean(),

  // Rewards
  rewards: QuestRewardSchema,

  // Location
  questGiver: ContentReferenceSchema.optional(),
  startLocation: ContentReferenceSchema,
  relevantLocations: z.array(ContentReferenceSchema).optional(),

  // Time
  timeLimit: z.number().int().positive().optional(),
  expiresOn: z.string().optional(),

  // State
  repeatable: z.boolean(),
  repeatCooldown: z.number().int().positive().optional(),

  // Failure
  canFail: z.boolean(),
  failureConditions: z.array(FailureConditionSchema).optional(),

  // Tags
  tags: z.array(z.string())
});

export type Quest = z.infer<typeof QuestSchema>;
```

### Schema Index (`src/data/schemas/index.ts`)

```typescript
export * from './base.schema';
export * from './character.schema';
export * from './ability.schema';
export * from './item.schema';
export * from './enemy.schema';
export * from './quest.schema';

// Export a schema registry for dynamic access
import { z } from 'zod';
import { CharacterClassSchema } from './character.schema';
import { AbilitySchema } from './ability.schema';
import { ItemSchema } from './item.schema';
import { EnemySchema } from './enemy.schema';
import { QuestSchema } from './quest.schema';

export const SchemaRegistry = {
  character: CharacterClassSchema,
  ability: AbilitySchema,
  item: ItemSchema,
  enemy: EnemySchema,
  quest: QuestSchema
} as const;

export type ContentType = keyof typeof SchemaRegistry;

export function getSchema(type: ContentType): z.ZodSchema {
  return SchemaRegistry[type];
}
```

---

## ContentValidator Class

### Core Validator (`src/systems/ContentValidator.ts`)

```typescript
import { z } from 'zod';
import { SchemaRegistry, ContentType, ContentReference } from '../data/schemas';

export interface ValidationError {
  file: string;
  field: string;
  message: string;
  severity: 'error' | 'warning';
  line?: number;
}

export interface ValidationResult {
  valid: boolean;
  errors: ValidationError[];
  warnings: ValidationError[];
}

export class ContentValidator {
  private contentCache: Map<string, any> = new Map();

  /**
   * Validate a single content file
   */
  validateContent(
    content: unknown,
    type: ContentType,
    fileName: string
  ): ValidationResult {
    const schema = SchemaRegistry[type];
    const errors: ValidationError[] = [];
    const warnings: ValidationError[] = [];

    try {
      // Parse with Zod
      schema.parse(content);

      return {
        valid: true,
        errors: [],
        warnings: []
      };
    } catch (error) {
      if (error instanceof z.ZodError) {
        // Transform Zod errors to our format
        error.errors.forEach(err => {
          errors.push({
            file: fileName,
            field: err.path.join('.'),
            message: err.message,
            severity: 'error'
          });
        });
      } else {
        errors.push({
          file: fileName,
          field: 'unknown',
          message: error instanceof Error ? error.message : String(error),
          severity: 'error'
        });
      }

      return {
        valid: false,
        errors,
        warnings
      };
    }
  }

  /**
   * Validate an entire collection of content files
   */
  validateCollection(
    contents: Record<string, unknown>,
    type: ContentType
  ): ValidationResult[] {
    const results: ValidationResult[] = [];

    for (const [fileName, content] of Object.entries(contents)) {
      const result = this.validateContent(content, type, fileName);
      results.push(result);

      // Cache valid content for reference validation
      if (result.valid) {
        this.contentCache.set(content.id as string, content);
      }
    }

    return results;
  }

  /**
   * Validate cross-references between content files
   */
  validateReferences(): ValidationError[] {
    const errors: ValidationError[] = [];
    const allIds = new Set(this.contentCache.keys());

    for (const [id, content] of this.contentCache) {
      const refs = this.extractReferences(content);

      for (const ref of refs) {
        if (!allIds.has(ref.id)) {
          errors.push({
            file: id,
            field: 'reference',
            message: `Reference to non-existent content: ${ref.type}:${ref.id}`,
            severity: 'error'
          });
        }
      }
    }

    return errors;
  }

  /**
   * Detect circular dependencies
   */
  detectCircularDependencies(): ValidationError[] {
    const errors: ValidationError[] = [];
    const visited = new Set<string>();
    const recursionStack = new Set<string>();

    const visit = (id: string, path: string[]): void => {
      if (recursionStack.has(id)) {
        errors.push({
          file: id,
          field: 'dependencies',
          message: `Circular dependency detected: ${[...path, id].join(' -> ')}`,
          severity: 'error'
        });
        return;
      }

      if (visited.has(id)) {
        return;
      }

      visited.add(id);
      recursionStack.add(id);

      const content = this.contentCache.get(id);
      if (content) {
        const refs = this.extractReferences(content);
        for (const ref of refs) {
          visit(ref.id, [...path, id]);
        }
      }

      recursionStack.delete(id);
    };

    for (const id of this.contentCache.keys()) {
      visit(id, []);
    }

    return errors;
  }

  /**
   * Find unused content (content that is never referenced)
   */
  findUnusedContent(): ValidationError[] {
    const warnings: ValidationError[] = [];
    const referencedIds = new Set<string>();

    // Collect all referenced IDs
    for (const content of this.contentCache.values()) {
      const refs = this.extractReferences(content);
      refs.forEach(ref => referencedIds.add(ref.id));
    }

    // Find content that is never referenced
    for (const id of this.contentCache.keys()) {
      if (!referencedIds.has(id)) {
        warnings.push({
          file: id,
          field: 'usage',
          message: `Content is never referenced and may be unused: ${id}`,
          severity: 'warning'
        });
      }
    }

    return warnings;
  }

  /**
   * Extract all ContentReferences from an object recursively
   */
  private extractReferences(obj: any): ContentReference[] {
    const refs: ContentReference[] = [];

    const traverse = (value: any): void => {
      if (!value || typeof value !== 'object') {
        return;
      }

      // Check if this is a ContentReference
      if (value.id && value.type) {
        refs.push(value as ContentReference);
      }

      // Recurse into arrays and objects
      if (Array.isArray(value)) {
        value.forEach(traverse);
      } else {
        Object.values(value).forEach(traverse);
      }
    };

    traverse(obj);
    return refs;
  }

  /**
   * Clear the content cache
   */
  clearCache(): void {
    this.contentCache.clear();
  }

  /**
   * Get cached content by ID
   */
  getCached(id: string): any | undefined {
    return this.contentCache.get(id);
  }

  /**
   * Get all cached content
   */
  getAllCached(): Map<string, any> {
    return new Map(this.contentCache);
  }
}
```

---

## CLI Validation Tool

### Main CLI Script (`scripts/validate-content.ts`)

```typescript
#!/usr/bin/env node

import * as fs from 'fs';
import * as path from 'path';
import { glob } from 'glob';
import { ContentValidator, ValidationError, ValidationResult } from '../src/systems/ContentValidator';
import { ContentType } from '../src/data/schemas';

// Color output helpers
const colors = {
  reset: '\x1b[0m',
  red: '\x1b[31m',
  green: '\x1b[32m',
  yellow: '\x1b[33m',
  blue: '\x1b[34m',
  magenta: '\x1b[35m',
  cyan: '\x1b[36m'
};

function colorize(text: string, color: keyof typeof colors): string {
  return `${colors[color]}${text}${colors.reset}`;
}

interface ContentFileMap {
  type: ContentType;
  pattern: string;
}

const CONTENT_MAP: ContentFileMap[] = [
  { type: 'character', pattern: 'src/data/static/characters/**/*.json' },
  { type: 'ability', pattern: 'src/data/static/abilities/**/*.json' },
  { type: 'item', pattern: 'src/data/static/items/**/*.json' },
  { type: 'enemy', pattern: 'src/data/static/enemies/**/*.json' },
  { type: 'quest', pattern: 'src/data/static/quests/**/*.json' }
];

async function loadJsonFile(filePath: string): Promise<any> {
  try {
    const content = fs.readFileSync(filePath, 'utf-8');
    return JSON.parse(content);
  } catch (error) {
    throw new Error(`Failed to parse ${filePath}: ${error.message}`);
  }
}

async function validateContentType(
  type: ContentType,
  pattern: string,
  validator: ContentValidator
): Promise<{ results: ValidationResult[]; fileCount: number }> {
  console.log(colorize(`\nValidating ${type}...`, 'cyan'));

  const files = await glob(pattern);
  const contents: Record<string, any> = {};

  if (files.length === 0) {
    console.log(colorize(`  ⚠ No ${type} files found`, 'yellow'));
    return { results: [], fileCount: 0 };
  }

  // Load all files
  for (const file of files) {
    try {
      contents[file] = await loadJsonFile(file);
    } catch (error) {
      console.error(colorize(`  ✗ ${file}: ${error.message}`, 'red'));
    }
  }

  // Validate
  const results = validator.validateCollection(contents, type);

  // Report results
  let errorCount = 0;
  let warningCount = 0;

  results.forEach(result => {
    errorCount += result.errors.length;
    warningCount += result.warnings.length;

    if (!result.valid) {
      result.errors.forEach(err => {
        console.error(
          colorize(`  ✗ ${err.file}`, 'red') +
          colorize(` [${err.field}]`, 'magenta') +
          `: ${err.message}`
        );
      });
    }
  });

  if (errorCount === 0) {
    console.log(
      colorize(`  ✓ ${files.length} ${type} file(s) validated`, 'green')
    );
  } else {
    console.log(
      colorize(`  ✗ ${errorCount} error(s) in ${type} files`, 'red')
    );
  }

  if (warningCount > 0) {
    console.log(
      colorize(`  ⚠ ${warningCount} warning(s) in ${type} files`, 'yellow')
    );
  }

  return { results, fileCount: files.length };
}

async function main(): Promise<number> {
  console.log(colorize('='.repeat(60), 'blue'));
  console.log(colorize('Content Validation Tool', 'blue'));
  console.log(colorize('='.repeat(60), 'blue'));

  const validator = new ContentValidator();
  let totalErrors = 0;
  let totalWarnings = 0;
  let totalFiles = 0;

  // Validate each content type
  for (const { type, pattern } of CONTENT_MAP) {
    const { results, fileCount } = await validateContentType(type, pattern, validator);

    totalFiles += fileCount;
    results.forEach(result => {
      totalErrors += result.errors.length;
      totalWarnings += result.warnings.length;
    });
  }

  // Cross-reference validation
  console.log(colorize('\nValidating cross-references...', 'cyan'));
  const refErrors = validator.validateReferences();

  if (refErrors.length === 0) {
    console.log(colorize('  ✓ All references are valid', 'green'));
  } else {
    refErrors.forEach(err => {
      console.error(
        colorize(`  ✗ ${err.file}`, 'red') +
        `: ${err.message}`
      );
    });
    totalErrors += refErrors.length;
  }

  // Circular dependency detection
  console.log(colorize('\nChecking for circular dependencies...', 'cyan'));
  const circularErrors = validator.detectCircularDependencies();

  if (circularErrors.length === 0) {
    console.log(colorize('  ✓ No circular dependencies found', 'green'));
  } else {
    circularErrors.forEach(err => {
      console.error(
        colorize(`  ✗ ${err.file}`, 'red') +
        `: ${err.message}`
      );
    });
    totalErrors += circularErrors.length;
  }

  // Unused content detection
  console.log(colorize('\nChecking for unused content...', 'cyan'));
  const unusedWarnings = validator.findUnusedContent();

  if (unusedWarnings.length === 0) {
    console.log(colorize('  ✓ All content is referenced', 'green'));
  } else {
    unusedWarnings.forEach(warn => {
      console.warn(
        colorize(`  ⚠ ${warn.file}`, 'yellow') +
        `: ${warn.message}`
      );
    });
    totalWarnings += unusedWarnings.length;
  }

  // Summary
  console.log(colorize('\n' + '='.repeat(60), 'blue'));
  console.log(colorize('Validation Summary', 'blue'));
  console.log(colorize('='.repeat(60), 'blue'));
  console.log(`Files validated: ${totalFiles}`);

  if (totalErrors === 0) {
    console.log(colorize(`Errors: ${totalErrors} ✓`, 'green'));
  } else {
    console.log(colorize(`Errors: ${totalErrors} ✗`, 'red'));
  }

  if (totalWarnings === 0) {
    console.log(colorize(`Warnings: ${totalWarnings} ✓`, 'green'));
  } else {
    console.log(colorize(`Warnings: ${totalWarnings} ⚠`, 'yellow'));
  }

  console.log(colorize('='.repeat(60), 'blue'));

  if (totalErrors > 0) {
    console.log(colorize('\n❌ Validation FAILED', 'red'));
    return 1; // Exit with error code
  } else {
    console.log(colorize('\n✅ Validation PASSED', 'green'));
    return 0; // Exit successfully
  }
}

// Run if executed directly
if (require.main === module) {
  main()
    .then(exitCode => process.exit(exitCode))
    .catch(error => {
      console.error(colorize('Validation tool crashed:', 'red'), error);
      process.exit(1);
    });
}

export { main as validateContent };
```

### Package.json Scripts

```json
{
  "scripts": {
    "validate": "ts-node scripts/validate-content.ts",
    "validate:watch": "nodemon --watch 'src/data/**/*.json' --exec 'npm run validate'",
    "prebuild": "npm run validate",
    "lint:data": "npm run validate"
  },
  "devDependencies": {
    "ts-node": "^10.9.1",
    "nodemon": "^3.0.1",
    "glob": "^10.3.10",
    "@types/node": "^20.10.0",
    "typescript": "^5.3.0",
    "zod": "^3.22.0"
  }
}
```

---

## Runtime Validation

### Content Loader with Validation (`src/systems/ContentLoader.ts`)

```typescript
import { ContentValidator, ValidationError } from './ContentValidator';
import { ContentType, SchemaRegistry } from '../data/schemas';

export class ContentLoader {
  private validator: ContentValidator;
  private cache: Map<string, any> = new Map();
  private isDevMode: boolean;

  constructor() {
    this.validator = new ContentValidator();
    this.isDevMode = import.meta.env.DEV;
  }

  /**
   * Load and validate a content file
   */
  async loadContent<T>(
    fileName: string,
    type: ContentType
  ): Promise<T> {
    // Check cache first
    const cacheKey = `${type}:${fileName}`;
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey) as T;
    }

    try {
      // Fetch JSON file
      const response = await fetch(`./data/static/${type}/${fileName}.json`);

      if (!response.ok) {
        throw new Error(`Failed to load ${fileName}: ${response.statusText}`);
      }

      const content = await response.json();

      // Validate in development mode
      if (this.isDevMode) {
        const result = this.validator.validateContent(
          content,
          type,
          fileName
        );

        if (!result.valid) {
          this.showValidationError(fileName, result.errors);
          throw new Error(`Validation failed for ${fileName}`);
        }
      }

      // Cache and return
      this.cache.set(cacheKey, content);
      return content as T;

    } catch (error) {
      if (this.isDevMode) {
        this.showLoadError(fileName, error);
      }
      throw error;
    }
  }

  /**
   * Load multiple content files of the same type
   */
  async loadContentCollection<T>(
    type: ContentType,
    fileNames: string[]
  ): Promise<T[]> {
    const promises = fileNames.map(name => this.loadContent<T>(name, type));
    return Promise.all(promises);
  }

  /**
   * Show validation error overlay (development mode only)
   */
  private showValidationError(fileName: string, errors: ValidationError[]): void {
    const overlay = document.createElement('div');
    overlay.id = 'validation-error-overlay';
    overlay.style.cssText = `
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.95);
      color: #ff4444;
      padding: 40px;
      overflow: auto;
      z-index: 999999;
      font-family: 'Courier New', monospace;
      font-size: 14px;
      line-height: 1.6;
    `;

    const title = document.createElement('h1');
    title.textContent = '❌ Content Validation Error';
    title.style.cssText = 'color: #ff4444; margin-bottom: 20px;';

    const subtitle = document.createElement('h2');
    subtitle.textContent = `File: ${fileName}`;
    subtitle.style.cssText = 'color: #ffaa44; margin-bottom: 30px;';

    const errorList = document.createElement('ul');
    errorList.style.cssText = 'list-style: none; padding: 0;';

    errors.forEach(error => {
      const item = document.createElement('li');
      item.style.cssText = `
        background: rgba(255, 68, 68, 0.1);
        border-left: 4px solid #ff4444;
        padding: 15px;
        margin-bottom: 15px;
      `;

      item.innerHTML = `
        <strong style="color: #ffaa44;">${error.field || 'Unknown field'}</strong><br>
        <span style="color: #ffffff;">${error.message}</span>
      `;

      errorList.appendChild(item);
    });

    const instructions = document.createElement('p');
    instructions.textContent = 'Fix the errors in the JSON file and save to continue. The page will reload automatically.';
    instructions.style.cssText = 'color: #aaaaaa; margin-top: 30px; font-size: 16px;';

    overlay.appendChild(title);
    overlay.appendChild(subtitle);
    overlay.appendChild(errorList);
    overlay.appendChild(instructions);

    // Remove existing overlay if present
    const existing = document.getElementById('validation-error-overlay');
    if (existing) {
      existing.remove();
    }

    document.body.appendChild(overlay);
  }

  /**
   * Show load error overlay (development mode only)
   */
  private showLoadError(fileName: string, error: unknown): void {
    const overlay = document.createElement('div');
    overlay.id = 'load-error-overlay';
    overlay.style.cssText = `
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.95);
      color: #ff4444;
      padding: 40px;
      overflow: auto;
      z-index: 999999;
      font-family: 'Courier New', monospace;
      font-size: 14px;
      line-height: 1.6;
    `;

    const title = document.createElement('h1');
    title.textContent = '❌ Failed to Load Content';
    title.style.cssText = 'color: #ff4444; margin-bottom: 20px;';

    const subtitle = document.createElement('h2');
    subtitle.textContent = `File: ${fileName}`;
    subtitle.style.cssText = 'color: #ffaa44; margin-bottom: 30px;';

    const errorText = document.createElement('pre');
    errorText.textContent = error instanceof Error ? error.message : String(error);
    errorText.style.cssText = `
      background: rgba(255, 68, 68, 0.1);
      border-left: 4px solid #ff4444;
      padding: 15px;
      white-space: pre-wrap;
      word-wrap: break-word;
    `;

    overlay.appendChild(title);
    overlay.appendChild(subtitle);
    overlay.appendChild(errorText);

    // Remove existing overlay if present
    const existing = document.getElementById('load-error-overlay');
    if (existing) {
      existing.remove();
    }

    document.body.appendChild(overlay);
  }

  /**
   * Clear the cache
   */
  clearCache(): void {
    this.cache.clear();
    this.validator.clearCache();
  }

  /**
   * Get validator instance (for reference validation)
   */
  getValidator(): ContentValidator {
    return this.validator;
  }
}
```

---

## Reference Validation

### Example: Validating Character Equipment References

```typescript
import { ContentLoader } from './ContentLoader';
import { ContentValidator } from './ContentValidator';

export class ReferenceValidator {
  private loader: ContentLoader;
  private validator: ContentValidator;

  constructor(loader: ContentLoader) {
    this.loader = loader;
    this.validator = loader.getValidator();
  }

  /**
   * Validate all references in characters
   */
  async validateCharacterReferences(): Promise<string[]> {
    const errors: string[] = [];

    // Load all characters
    const characters = await this.loadAllCharacters();

    // Load all items for reference checking
    const items = await this.loadAllItems();
    const abilities = await this.loadAllAbilities();

    const itemIds = new Set(items.map(i => i.id));
    const abilityIds = new Set(abilities.map(a => a.id));

    // Check each character
    for (const char of characters) {
      // Check starting abilities
      for (const abilityRef of char.startingAbilities || []) {
        if (!abilityIds.has(abilityRef.id)) {
          errors.push(
            `Character '${char.id}' references non-existent ability '${abilityRef.id}'`
          );
        }
      }

      // Check learnable abilities
      for (const progression of char.learnableAbilities || []) {
        if (!abilityIds.has(progression.ability.id)) {
          errors.push(
            `Character '${char.id}' references non-existent learnable ability '${progression.ability.id}'`
          );
        }

        // Check prerequisites
        if (progression.prerequisite && !abilityIds.has(progression.prerequisite.id)) {
          errors.push(
            `Character '${char.id}' ability progression has invalid prerequisite '${progression.prerequisite.id}'`
          );
        }
      }
    }

    return errors;
  }

  private async loadAllCharacters(): Promise<any[]> {
    // Implementation depends on your file structure
    // This is a simplified example
    const fileNames = ['technomancer', 'guardian', 'bioengineer'];
    return this.loader.loadContentCollection('character', fileNames);
  }

  private async loadAllItems(): Promise<any[]> {
    // Similar to loadAllCharacters
    return [];
  }

  private async loadAllAbilities(): Promise<any[]> {
    // Similar to loadAllCharacters
    return [];
  }
}
```

---

## Integration with Phaser

### Boot Scene with Validation (`src/scenes/BootScene.ts`)

```typescript
import Phaser from 'phaser';
import { ContentLoader } from '../systems/ContentLoader';
import { EventBus } from '../utils/EventBus';

export class BootScene extends Phaser.Scene {
  private contentLoader!: ContentLoader;
  private loadingText!: Phaser.GameObjects.Text;
  private errorOccurred: boolean = false;

  constructor() {
    super({ key: 'BootScene' });
  }

  preload() {
    this.showLoadingScreen();
    this.contentLoader = new ContentLoader();

    // Load all content with validation
    this.loadGameContent();
  }

  create() {
    if (this.errorOccurred) {
      return; // Error overlay is shown, don't proceed
    }

    // Validate references in development
    if (import.meta.env.DEV) {
      this.validateReferences();
    }

    // Transition to main menu or world map
    this.scene.start('WorldMapScene');
  }

  private showLoadingScreen() {
    const { width, height } = this.cameras.main;

    // Background
    this.add.rectangle(
      width / 2,
      height / 2,
      width,
      height,
      0x2d2d2d
    );

    // Title
    const title = this.add.text(
      width / 2,
      height / 2 - 50,
      'Solar Punk Chronicles',
      {
        fontSize: '32px',
        color: '#44AA44',
        fontFamily: 'Arial'
      }
    );
    title.setOrigin(0.5);

    // Loading text
    this.loadingText = this.add.text(
      width / 2,
      height / 2 + 50,
      'Loading content...',
      {
        fontSize: '18px',
        color: '#AAAAAA',
        fontFamily: 'Arial'
      }
    );
    this.loadingText.setOrigin(0.5);
  }

  private async loadGameContent() {
    try {
      this.updateLoadingText('Loading character classes...');
      const characters = await this.contentLoader.loadContentCollection(
        'character',
        ['technomancer', 'guardian', 'bioengineer']
      );
      this.cache.json.add('characters', characters);

      this.updateLoadingText('Loading abilities...');
      const abilities = await this.contentLoader.loadContentCollection(
        'ability',
        ['lightning_strike', 'static_shield', 'chain_lightning']
      );
      this.cache.json.add('abilities', abilities);

      this.updateLoadingText('Loading items...');
      const items = await this.contentLoader.loadContentCollection(
        'item',
        ['solar_blade', 'photon_vest', 'energy_core']
      );
      this.cache.json.add('items', items);

      this.updateLoadingText('Loading enemies...');
      const enemies = await this.contentLoader.loadContentCollection(
        'enemy',
        ['corrupted_drone', 'security_bot', 'bio_pest_swarm']
      );
      this.cache.json.add('enemies', enemies);

      this.updateLoadingText('Loading quests...');
      const quests = await this.contentLoader.loadContentCollection(
        'quest',
        ['solar_nexus_disruption', 'seeds_of_discord']
      );
      this.cache.json.add('quests', quests);

      this.updateLoadingText('Content loaded successfully!');

    } catch (error) {
      console.error('Failed to load game content:', error);
      this.errorOccurred = true;
      this.updateLoadingText('Failed to load content. See console for details.');
    }
  }

  private validateReferences() {
    console.log('🔍 Validating content references...');

    const validator = this.contentLoader.getValidator();

    // Validate cross-references
    const refErrors = validator.validateReferences();
    if (refErrors.length > 0) {
      console.error('❌ Reference validation errors:', refErrors);
    } else {
      console.log('✅ All references valid');
    }

    // Check for circular dependencies
    const circularErrors = validator.detectCircularDependencies();
    if (circularErrors.length > 0) {
      console.error('❌ Circular dependencies detected:', circularErrors);
    } else {
      console.log('✅ No circular dependencies');
    }

    // Check for unused content
    const unusedWarnings = validator.findUnusedContent();
    if (unusedWarnings.length > 0) {
      console.warn('⚠️ Unused content detected:', unusedWarnings);
    } else {
      console.log('✅ All content is used');
    }
  }

  private updateLoadingText(message: string) {
    if (this.loadingText) {
      this.loadingText.setText(message);
    }
    console.log(message);
  }
}
```

---

## Usage Examples

### Example 1: Creating a Valid Character

```json
{
  "id": "class_technomancer",
  "version": "1.0.0",
  "name": "Technomancer",
  "description": "Masters of electrical manipulation and digital warfare",
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

  "skillProficiencies": ["arcana", "investigation", "nature", "perception"],
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
    }
  ],

  "equipmentSlots": [
    { "slot": "weapon", "required": true },
    { "slot": "armor", "required": true },
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

### Example 2: Validating Content Programmatically

```typescript
import { ContentValidator } from './systems/ContentValidator';
import { CharacterClassSchema } from './data/schemas';

// Create validator
const validator = new ContentValidator();

// Load and validate character
const characterData = {
  id: "class_technomancer",
  version: "1.0.0",
  name: "Technomancer",
  // ... rest of data
};

const result = validator.validateContent(
  characterData,
  'character',
  'technomancer.json'
);

if (result.valid) {
  console.log('✅ Character is valid!');
} else {
  console.error('❌ Validation errors:', result.errors);
}
```

### Example 3: Running CLI Validation

```bash
# Validate all content
npm run validate

# Watch for changes and validate automatically
npm run validate:watch

# Validate before building
npm run build
# (automatically runs validation due to prebuild script)
```

---

## Error Message Samples

### Schema Validation Error

```
❌ Content Validation Error
File: technomancer.json

● baseStats.hp
  Expected number, received string

● startingAbilities[0].id
  Invalid content reference: ability ID not found

● primaryColor
  Invalid hex color format (must be #RRGGBB)
```

### Cross-Reference Error

```
❌ Reference Validation Error

File: class_technomancer

Reference to non-existent content: ability:ability_chain_lightning
  → startingAbilities[2] references an ability that doesn't exist

Suggestion: Check that 'ability_chain_lightning.json' exists in /abilities/
```

### Circular Dependency Error

```
❌ Circular Dependency Detected

Dependency chain:
  quest_main_01 → npc_elder → dialog_quest_start → quest_main_01

This creates a circular reference that will cause loading issues.
```

### Unused Content Warning

```
⚠️ Unused Content Warning

File: item_rusty_sword.json

This item is never referenced by any character, quest, or enemy.
Consider removing it or adding it to a shop/loot table.
```

---

## Installation and Setup

### Step 1: Install Dependencies

```bash
npm install zod
npm install -D @types/node ts-node nodemon glob
```

### Step 2: Create Directory Structure

```bash
mkdir -p src/data/schemas
mkdir -p src/data/static/characters
mkdir -p src/data/static/abilities
mkdir -p src/data/static/items
mkdir -p src/data/static/enemies
mkdir -p src/data/static/quests
mkdir -p src/systems
mkdir -p scripts
```

### Step 3: Copy Schema Files

1. Create `src/data/schemas/base.schema.ts` (copy from above)
2. Create `src/data/schemas/character.schema.ts` (copy from above)
3. Create `src/data/schemas/ability.schema.ts` (copy from above)
4. Create `src/data/schemas/item.schema.ts` (copy from above)
5. Create `src/data/schemas/enemy.schema.ts` (copy from above)
6. Create `src/data/schemas/quest.schema.ts` (copy from above)
7. Create `src/data/schemas/index.ts` (copy from above)

### Step 4: Copy System Files

1. Create `src/systems/ContentValidator.ts` (copy from above)
2. Create `src/systems/ContentLoader.ts` (copy from above)

### Step 5: Copy CLI Tool

1. Create `scripts/validate-content.ts` (copy from above)
2. Make it executable: `chmod +x scripts/validate-content.ts`

### Step 6: Update package.json

Add scripts from the package.json section above.

### Step 7: Test the System

```bash
# Create a test character file
mkdir -p src/data/static/characters
cat > src/data/static/characters/technomancer.json << 'EOF'
{
  "id": "class_technomancer",
  "version": "1.0.0",
  "name": "Technomancer"
}
EOF

# Run validation (should fail with missing fields)
npm run validate

# Fix the file by adding all required fields
# Run validation again (should pass)
npm run validate
```

---

## Implementation Timeline

### Day 1 (4-6 hours)

**Morning:**
- [ ] Install dependencies
- [ ] Create directory structure
- [ ] Copy base schema files (base.schema.ts, index.ts)
- [ ] Copy ContentValidator.ts

**Afternoon:**
- [ ] Create character.schema.ts
- [ ] Create ability.schema.ts
- [ ] Create item.schema.ts
- [ ] Test validation with sample files

### Day 2 (4-6 hours)

**Morning:**
- [ ] Create enemy.schema.ts
- [ ] Create quest.schema.ts
- [ ] Copy CLI tool (validate-content.ts)
- [ ] Test CLI validation

**Afternoon:**
- [ ] Copy ContentLoader.ts
- [ ] Integrate with BootScene
- [ ] Create sample content files
- [ ] Full end-to-end test

---

## Success Criteria

By the end of Day 2, you should have:

- [ ] All Zod schemas defined and working
- [ ] CLI validation tool running successfully
- [ ] Runtime validation in BootScene
- [ ] Error overlays showing helpful messages
- [ ] Cross-reference validation working
- [ ] Circular dependency detection working
- [ ] At least 3 test content files of each type validating successfully

---

## Next Steps After Implementation

1. **Generate TypeScript Types** (Day 3)
   - Use Zod's `.infer<>` to generate types
   - Export types for use in game code
   - Enable IDE autocomplete

2. **Hot-Reload Integration** (Day 3-4)
   - Add Vite plugin to watch JSON files
   - Implement data reload without page refresh
   - Test with content changes

3. **Content Creation** (Week 2+)
   - Create real game content
   - Use validation to catch errors
   - Iterate based on designer feedback

---

## Troubleshooting

### Issue: "Cannot find module 'zod'"

**Solution:**
```bash
npm install zod
```

### Issue: "ts-node not found"

**Solution:**
```bash
npm install -D ts-node @types/node
```

### Issue: Validation passing but content not loading in game

**Solution:**
- Check file paths in BootScene match actual files
- Verify JSON files are in correct directories
- Check browser console for loading errors

### Issue: Zod validation too strict

**Solution:**
- Use `.optional()` for optional fields
- Use `.nullable()` for nullable fields
- Adjust min/max constraints as needed

---

## Conclusion

This validation system provides:

1. **Build-time validation**: Catch errors before deployment
2. **Runtime validation**: See errors immediately during development
3. **Reference validation**: Ensure all IDs exist
4. **Circular dependency detection**: Prevent infinite loops
5. **Developer-friendly errors**: Clear messages with context

**Total implementation time:** 2 days (12-16 hours)

**Impact:**
- Eliminates entire class of runtime errors
- Enables non-programmers to safely edit content
- Provides immediate feedback during development
- Prevents invalid content from reaching production

This system is the **foundation** for all content creation in Phase 1 and beyond. Implement it first, before writing any game content.

---

**Document Status:** Ready for Implementation
**Last Updated:** 2025-11-01
**Author:** Content Validation System Guide
**Version:** 1.0
