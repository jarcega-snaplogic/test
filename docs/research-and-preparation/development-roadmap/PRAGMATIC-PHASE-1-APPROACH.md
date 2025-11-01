# Pragmatic Phase 1 Approach: Essential Infrastructure First

**Document Type:** Development Strategy Revision
**Status:** Proposed
**Date:** 2025-11-01
**Revision:** 1.0

---

## Executive Summary

**The Problem:**
- **Architects recommend:** 2 weeks upfront infrastructure (validation, hot-reload, complete schemas)
- **Developers respond:** "Too much, too early. We don't know final schemas yet."
- **Original timeline:** 15 weeks
- **Architect timeline:** 17 weeks (15 + 2 weeks infrastructure)

**The Pragmatic Solution:**
- **3-5 days** essential infrastructure (patterns, not implementations)
- **Convention over configuration** (expandable architecture from day 1)
- **Build infrastructure independently** (doesn't block gameplay development)
- **Define schemas as we go** (when we actually need them)
- **Defer complexity** (hot-reload is Phase 2)

**Revised Timeline:** **15 weeks** (same as original - infrastructure built in parallel)

---

## Table of Contents

1. [Philosophy: Pragmatic vs. Perfect](#philosophy-pragmatic-vs-perfect)
2. [Comparison: Three Approaches](#comparison-three-approaches)
3. [Essential Infrastructure (Week 1: Days 1-5)](#essential-infrastructure-week-1-days-1-5)
4. [Expandable Architecture Patterns](#expandable-architecture-patterns)
5. [What We're Deferring](#what-were-deferring)
6. [Independent Development Strategy](#independent-development-strategy)
7. [Week 1 Tactical Plan (Day-by-Day)](#week-1-tactical-plan-day-by-day)
8. [Risk Assessment](#risk-assessment)
9. [Success Criteria](#success-criteria)

---

## Philosophy: Pragmatic vs. Perfect

### The Architect Mindset (Valid but Premature)

**"Build it right the first time"**
- Define complete JSON schemas before writing any content
- Implement full validation pipeline
- Build hot-reload infrastructure
- Create content authoring tools
- **Result:** 2 weeks before any gameplay code

**Why this is valuable:**
- Prevents technical debt
- Catches errors early
- Enables non-programmers
- Professional-grade infrastructure

**Why this is premature:**
- We don't know what schemas we need yet
- We don't have content to validate yet
- We're solo/small team (not 20-person studio)
- We need to learn what we need by building

### The Pragmatic Developer Mindset

**"Build what you need, when you need it"**
- Start with simple JSON loading
- Add structure as patterns emerge
- Expand infrastructure when it hurts
- Keep it minimal and working
- **Result:** 3-5 days, then start building gameplay

**Why this works:**
- Faster time to first gameplay
- Learn requirements through usage
- Lower upfront complexity
- Can pivot easily

**Why this has risks:**
- Might accumulate tech debt
- Validation gaps can cause bugs
- Refactoring later is harder

### The Hybrid Approach (This Document)

**"Build patterns, not implementations"**
- Define architectural patterns (how things will work)
- Build minimal viable infrastructure (data loading + basic structure)
- Create conventions (how to add new content)
- Defer complete implementations (validation, hot-reload)
- Build infrastructure in parallel with gameplay
- **Result:** 3-5 days essential, rest built as needed

**This balances:**
- ✅ Fast start (like pure pragmatic)
- ✅ Solid foundation (like architect approach)
- ✅ Flexibility (can expand without refactoring)
- ✅ Learning (discover needs through building)

---

## Comparison: Three Approaches

| Aspect | Pure Pragmatic | Architect Approach | **Hybrid (This Doc)** |
|--------|---------------|-------------------|---------------------|
| **Upfront Time** | 1-2 days | 2 weeks | **3-5 days** |
| **First Gameplay** | Week 1 | Week 3 | **Week 1** |
| **Schemas** | None, hardcode | All defined upfront | **Define as needed** |
| **Validation** | None | Full pipeline | **Basic + expand later** |
| **Hot-Reload** | None | Day 1 | **Phase 2** |
| **Type Safety** | Weak | Strong | **Medium (improve over time)** |
| **Refactoring Risk** | High | Low | **Medium** |
| **Learning Curve** | Low | High | **Low** |
| **Timeline** | 15 weeks | 17 weeks | **15 weeks** |
| **Team Size** | Solo only | 3+ team | **Solo or 2-3 team** |

### What We Keep from Architects

✅ **Keep: Architectural Patterns**
- How data should be structured (conventions)
- Separation of static vs. generated content
- Clear file organization strategy
- Cross-reference patterns

✅ **Keep: Foundation for Expansion**
- Data loading abstraction (can add validation later)
- Cache architecture (can add hot-reload later)
- File structure that supports schemas

✅ **Keep: The Vision**
- End goal is still fully validated, hot-reloadable content
- Just build it incrementally, not all at once

### What We Defer

❌ **Defer to Phase 2: Hot-Reload**
- Valuable but adds complexity
- Full page refresh is fine for Phase 1
- 30-second cycle time acceptable for solo dev
- Build when content changes become frequent

❌ **Defer to Mid-Phase 1: Complete Schemas**
- Don't define item schema until we build items system (Week 5-6)
- Don't define quest schema until we build quest system (Week 7-8)
- **Reason:** We don't know what we need yet

❌ **Defer to Phase 2: Full Validation Pipeline**
- Build-time validation: Phase 2
- Runtime validation: Basic only (JSON parse errors)
- Cross-reference validation: Phase 2
- **Reason:** Content is small in Phase 1, manual testing sufficient

❌ **Defer to Phase 3: Content Authoring Tools**
- Visual editors: Not needed for solo/small team
- VSCode + manual JSON editing is fine
- **Reason:** We're the only content creators

### What We Skip Entirely

🚫 **Skip: Over-Engineering**
- Don't build asset manager (Phaser handles it)
- Don't build complex state versioning (simple save/load is fine)
- Don't build telemetry/analytics (not needed for Phase 1)
- Don't build modding support (post-launch feature)

---

## Essential Infrastructure (Week 1: Days 1-5)

### What "Essential" Means

**Essential = Cannot build gameplay without it**

- ✅ Data loading (need to load characters, enemies, etc.)
- ✅ Basic file structure (need to know where to put data)
- ✅ Simple conventions (need to know how to structure JSON)
- ❌ Validation (can manually test)
- ❌ Hot-reload (can refresh browser)
- ❌ Complete schemas (define when we build the system)

### The 3-5 Day Plan

#### Day 1-2: Data Loading Pattern

**Goal:** Establish how we load JSON files

**Deliverable:**
```typescript
// src/utils/DataLoader.ts

export class DataLoader {
  private cache: Map<string, any> = new Map();

  /**
   * Load a JSON data file
   * Convention: Files are in /public/data/{category}/{filename}.json
   */
  async load(category: string, filename: string): Promise<any> {
    const key = `${category}:${filename}`;

    if (this.cache.has(key)) {
      return this.cache.get(key);
    }

    try {
      const response = await fetch(`./data/${category}/${filename}.json`);

      if (!response.ok) {
        throw new Error(`Failed to load ${category}/${filename}: ${response.statusText}`);
      }

      const data = await response.json();
      this.cache.set(key, data);

      return data;
    } catch (error) {
      console.error(`❌ Data loading error:`, error);
      throw error;
    }
  }

  /**
   * Load all files from a category
   * Example: loadCategory('characters') loads all files in /data/characters/
   */
  async loadCategory(category: string, files: string[]): Promise<any> {
    const promises = files.map(f => this.load(category, f));
    const results = await Promise.all(promises);

    // Merge results into single object
    return results.reduce((acc, data) => ({ ...acc, ...data }), {});
  }

  /**
   * Get cached data (after loading)
   */
  get(category: string, filename: string): any {
    const key = `${category}:${filename}`;
    return this.cache.get(key);
  }

  /**
   * Clear cache (useful for testing)
   */
  clearCache(): void {
    this.cache.clear();
  }
}

// Singleton instance
export const dataLoader = new DataLoader();
```

**Usage Example:**
```typescript
// In BootScene
async preload() {
  // Load character data
  const characters = await dataLoader.load('characters', 'playable');

  // Load abilities
  const abilities = await dataLoader.loadCategory('abilities', ['fighter', 'mage']);

  // Data is cached, subsequent calls are instant
  const cachedCharacters = dataLoader.get('characters', 'playable');
}
```

**What this gives us:**
- ✅ Simple, predictable data loading
- ✅ Caching built-in
- ✅ Error handling
- ✅ Convention: `/data/{category}/{filename}.json`
- ✅ Easy to add validation later (just add it in `load()`)
- ✅ Easy to add hot-reload later (just add watch + clearCache)

**Time:** 4 hours

#### Day 3: File Structure Convention

**Goal:** Establish where data lives and how it's organized

**Deliverable:**
```
/public/data/
├── characters/
│   └── playable.json          # Playable character templates
├── abilities/
│   ├── fighter.json           # Fighter abilities
│   ├── mage.json              # Mage abilities
│   └── shared.json            # Class-agnostic abilities
├── enemies/
│   └── common.json            # Common enemy types
├── items/
│   └── equipment.json         # Weapons, armor (defined when we build items system)
├── quests/
│   └── main.json              # Main story quests (defined when we build quest system)
├── world/
│   └── locations.json         # Cities, locations (defined when we build world map)
└── README.md                  # Documentation of conventions
```

**Documentation (data/README.md):**
```markdown
# Game Data Conventions

## File Organization

- `characters/`: Character definitions (playable, NPCs)
- `abilities/`: Abilities, spells, skills (organized by class)
- `enemies/`: Enemy definitions and behaviors
- `items/`: Items, equipment, consumables
- `quests/`: Quest definitions
- `world/`: World map, cities, locations

## Naming Conventions

- **File names:** lowercase, descriptive (e.g., `playable.json`, not `PlayerCharacters.json`)
- **IDs:** snake_case (e.g., `"fighter_01"`, not `"Fighter01"`)
- **Keys:** camelCase for properties (e.g., `"baseStats"`, not `"base_stats"`)

## Adding New Content

1. Determine category (characters, abilities, etc.)
2. Create or edit appropriate JSON file
3. Follow existing structure as template
4. Reload page to see changes

## Structure Examples

### Character
```json
{
  "fighter_01": {
    "id": "fighter_01",
    "name": "Aria",
    "class": "fighter",
    "baseStats": { ... },
    "abilities": ["basic_attack", "second_wind"]
  }
}
```

### Ability
```json
{
  "second_wind": {
    "id": "second_wind",
    "name": "Second Wind",
    "cost": 1,
    "effects": [
      { "type": "heal", "formula": "1d10 + 5" }
    ]
  }
}
```

(More examples added as we define systems)
```

**What this gives us:**
- ✅ Clear conventions (everyone knows where to put things)
- ✅ Documentation (easy to onboard)
- ✅ Expandable (just add new categories)
- ✅ Examples (learn by copying)

**Time:** 2 hours

#### Day 4: Basic TypeScript Types

**Goal:** Define interfaces for data structures we know we need

**Deliverable:**
```typescript
// src/types/game-data.types.ts

/**
 * Base types - these are stable and unlikely to change
 */

export interface BaseStats {
  strength: number;
  dexterity: number;
  constitution: number;
  intelligence: number;
  wisdom: number;
  charisma: number;
}

export interface Character {
  id: string;
  name: string;
  class: string;  // Will be enum later when we define classes
  baseStats: BaseStats;
  level: number;
  abilities: string[];  // Ability IDs
  equipment?: string[]; // Item IDs (optional for now)
}

export interface Ability {
  id: string;
  name: string;
  description: string;
  cost: number;
  effects: AbilityEffect[];
}

export interface AbilityEffect {
  type: string;  // 'damage', 'heal', 'buff', etc. (define as we go)
  formula?: string;  // e.g., "1d10 + 5"
  target?: string;   // 'self', 'enemy', 'ally'
}

export interface Enemy {
  id: string;
  name: string;
  stats: {
    hp: number;
    ac: number;
    attack: number;
    // Add more as needed
  };
  abilities: string[];  // Ability IDs
}

/**
 * Placeholder types - define when we build these systems
 */

export interface Item {
  id: string;
  name: string;
  // TODO: Define when we build item system (Week 5-6)
  [key: string]: any;
}

export interface Quest {
  id: string;
  title: string;
  // TODO: Define when we build quest system (Week 7-8)
  [key: string]: any;
}

export interface Location {
  id: string;
  name: string;
  // TODO: Define when we build world map (Week 9-10)
  [key: string]: any;
}
```

**What this gives us:**
- ✅ Type safety for core systems (characters, abilities, enemies)
- ✅ Placeholder types for future systems (items, quests)
- ✅ Auto-complete in IDE
- ✅ Catch some typos at compile-time
- ✅ Easy to expand (add properties as we discover needs)

**What this doesn't do:**
- ❌ Validate JSON at runtime (that's optional)
- ❌ Define everything upfront (we add as we go)
- ❌ Force rigid structure (we can refactor later)

**Time:** 3 hours

#### Day 5: Starter Data + Integration Test

**Goal:** Create minimal data files and verify loading works

**Deliverable:**

**`/public/data/characters/playable.json`:**
```json
{
  "fighter_01": {
    "id": "fighter_01",
    "name": "Test Fighter",
    "class": "fighter",
    "baseStats": {
      "strength": 16,
      "dexterity": 12,
      "constitution": 14,
      "intelligence": 10,
      "wisdom": 11,
      "charisma": 8
    },
    "level": 1,
    "abilities": ["basic_attack"]
  }
}
```

**`/public/data/abilities/fighter.json`:**
```json
{
  "basic_attack": {
    "id": "basic_attack",
    "name": "Basic Attack",
    "description": "A standard melee attack",
    "cost": 0,
    "effects": [
      {
        "type": "damage",
        "formula": "1d8 + 3",
        "target": "enemy"
      }
    ]
  }
}
```

**`/public/data/enemies/common.json`:**
```json
{
  "goblin": {
    "id": "goblin",
    "name": "Goblin",
    "stats": {
      "hp": 15,
      "ac": 13,
      "attack": 8
    },
    "abilities": ["basic_attack"]
  }
}
```

**Integration Test (src/scenes/BootScene.ts):**
```typescript
import { dataLoader } from '@/utils/DataLoader';
import type { Character, Ability, Enemy } from '@/types/game-data.types';

export class BootScene extends Phaser.Scene {
  async create() {
    console.log('🔄 Loading game data...');

    try {
      // Load data
      const characters = await dataLoader.load('characters', 'playable') as Record<string, Character>;
      const abilities = await dataLoader.load('abilities', 'fighter') as Record<string, Ability>;
      const enemies = await dataLoader.load('enemies', 'common') as Record<string, Enemy>;

      console.log('✅ Data loaded successfully');
      console.log('Characters:', Object.keys(characters));
      console.log('Abilities:', Object.keys(abilities));
      console.log('Enemies:', Object.keys(enemies));

      // Verify structure
      const fighter = characters['fighter_01'];
      console.log(`Fighter: ${fighter.name} (Level ${fighter.level})`);
      console.log(`Stats:`, fighter.baseStats);

      // Boot complete, start game
      this.scene.start('WorldMapScene');

    } catch (error) {
      console.error('❌ Failed to load game data:', error);
      // Show error screen
      this.add.text(400, 300, 'Failed to load game data.\nCheck console for details.', {
        fontSize: '24px',
        color: '#ff0000'
      });
    }
  }
}
```

**What this gives us:**
- ✅ Proof that data loading works
- ✅ Example data to reference
- ✅ Error handling tested
- ✅ Ready to start building gameplay

**Time:** 3 hours

---

### Summary: Days 1-5 Deliverables

**Total Time:** ~16 hours (2 full days or 3-5 partial days)

✅ **We have:**
- Data loading pattern that works
- File structure convention
- Basic TypeScript types
- Example data files
- Documented conventions
- Integration test proving it works

✅ **We can now:**
- Start building combat system (references abilities)
- Start building character system (loads character data)
- Add new content by creating JSON files
- Expand data structures as we discover needs

✅ **We deferred:**
- JSON Schema definitions (build when we need validation)
- Hot-reload (build if refresh becomes painful)
- Complete type definitions (add properties as we go)
- Cross-reference validation (Phase 2)

---

## Expandable Architecture Patterns

### Pattern 1: Convention Over Configuration

**Instead of:** Rigid schemas that must be defined upfront
**We use:** Conventions that guide structure but allow flexibility

**Example:**
```typescript
// Convention: All entities have id, name, description
// Convention: Stats are always in a 'stats' or 'baseStats' object
// Convention: References use string IDs, not nested objects

// This works (follows convention):
{
  "id": "healing_potion",
  "name": "Potion of Healing",
  "type": "consumable",
  "effect": { "type": "heal", "amount": 50 }
}

// This also works (adds new properties as needed):
{
  "id": "super_healing_potion",
  "name": "Super Potion",
  "type": "consumable",
  "effect": { "type": "heal", "amount": 100 },
  "stackable": true,  // New property, no schema change needed
  "maxStack": 20
}
```

**Why this works:**
- ✅ Easy to add new properties
- ✅ Doesn't require schema updates
- ✅ Self-documenting (follow existing patterns)
- ✅ TypeScript can still type-check core properties

### Pattern 2: Gradual Type Strengthening

**Instead of:** Full type safety from day 1
**We use:** Loose types that tighten over time

**Evolution:**
```typescript
// Week 1: Loose (anything goes)
export interface Item {
  id: string;
  name: string;
  [key: string]: any;  // Allow any additional properties
}

// Week 5: Tighter (we know more about items now)
export interface Item {
  id: string;
  name: string;
  type: 'weapon' | 'armor' | 'consumable';
  stats: ItemStats;  // Define stats structure
  [key: string]: any;  // Still allow extensions
}

// Week 10: Strong (structure is stable)
export interface Item {
  id: string;
  name: string;
  type: ItemType;  // Enum
  stats: ItemStats;
  rarity: Rarity;   // All properties defined
  // No [key: string]: any - structure is locked
}
```

**Why this works:**
- ✅ Fast start (loose types don't block progress)
- ✅ Safety improves over time (as we learn structure)
- ✅ No big refactor (gradual tightening)
- ✅ Can still pivot if needed

### Pattern 3: Lazy Schema Definition

**Instead of:** Define all schemas upfront
**We use:** Define schemas when we build the system

**Timeline:**
```
Week 1-2:   No schemas (just TypeScript interfaces)
Week 3-4:   Combat system → Define ability schema (we now know what abilities need)
Week 5-6:   Progression → Define item schema (we now know what items need)
Week 7-8:   Exploration → Define quest schema (we now know what quests need)
Week 9-10:  World map → Define location schema (we now know what locations need)
Week 11-12: Add validation (now we have schemas to validate against)
```

**Example:**
```typescript
// Week 3: Building combat, discover we need abilities with this structure
const abilityExample = {
  id: "fireball",
  name: "Fireball",
  cost: 10,
  effects: [
    { type: "damage", formula: "8d6", damageType: "fire", aoe: 20 }
  ],
  animation: "fire_explosion"
};

// Week 4: NOW we write the schema (based on what we learned)
// src/data/schemas/ability.schema.json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Ability",
  "properties": {
    "id": { "type": "string" },
    "name": { "type": "string" },
    "cost": { "type": "number" },
    "effects": {
      "type": "array",
      "items": {
        "properties": {
          "type": { "enum": ["damage", "heal", "buff", "debuff"] },
          "formula": { "type": "string" },
          "damageType": { "enum": ["fire", "ice", "physical"] },
          "aoe": { "type": "number" }
        }
      }
    },
    "animation": { "type": "string" }
  }
}
```

**Why this works:**
- ✅ Schema reflects reality (not guesses)
- ✅ No wasted work (only define what we use)
- ✅ Better schema quality (based on actual usage)
- ✅ Can add validation retroactively

### Pattern 4: Parallel Infrastructure Development

**Instead of:** Block gameplay until infrastructure is complete
**We use:** Build infrastructure in parallel with gameplay

**Two-Track Development:**

**Track A: Gameplay (Main Developer)**
```
Week 1:   Foundation + data loading ←─┐
Week 2:   Combat system              │ Using basic data loading
Week 3:   Character classes           │ No validation yet, just works
Week 4:   Exploration                 │
Week 5:   World map                   │
```

**Track B: Infrastructure (Build As Needed)**
```
Week 1:   Basic data loader          ←─┘
Week 3:   Add ability schema (combat needs it)
Week 5:   Add item schema (progression needs it)
Week 7:   Add quest schema (exploration needs it)
Week 11:  Add hot-reload (content changes frequent now)
Week 13:  Add validation pipeline (content is stable, need quality checks)
```

**How they integrate:**
```typescript
// Week 1: Basic data loader
class DataLoader {
  async load(file) {
    return await fetch(file).then(r => r.json());
  }
}

// Week 11: Same API, now with hot-reload (no gameplay code changes!)
class DataLoader {
  async load(file) {
    const data = await fetch(file).then(r => r.json());

    // NEW: Setup hot-reload listener
    if (import.meta.hot) {
      this.watchForChanges(file);
    }

    return data;
  }

  private watchForChanges(file) {
    // Hot-reload logic added without breaking existing code
  }
}
```

**Why this works:**
- ✅ Gameplay development unblocked
- ✅ Infrastructure added when needed
- ✅ Same API, enhanced implementation
- ✅ No refactoring required

### Pattern 5: Content Structure Evolution

**Instead of:** Fixed structure that can't change
**We use:** Evolvable structure with backward compatibility

**Example: Ability Effects**
```typescript
// Week 3: Simple structure
{
  "fireball": {
    "id": "fireball",
    "damage": 20,  // Just a number
    "type": "fire"
  }
}

// Week 5: Discovered we need formulas
{
  "fireball": {
    "id": "fireball",
    "damage": "8d6",  // Now a formula (backward compatible - code checks type)
    "type": "fire"
  }
}

// Week 7: Discovered we need multiple effects
{
  "fireball": {
    "id": "fireball",
    "effects": [  // New structure
      { "type": "damage", "formula": "8d6", "damageType": "fire" },
      { "type": "status", "name": "burning", "duration": 3 }
    ]
  }
}

// Code handles all three versions:
function applyAbility(ability) {
  // Version 1: Simple damage number
  if (typeof ability.damage === 'number') {
    return ability.damage;
  }

  // Version 2: Formula string
  if (typeof ability.damage === 'string') {
    return rollFormula(ability.damage);
  }

  // Version 3: Effect array
  if (ability.effects) {
    return ability.effects.map(e => applyEffect(e));
  }
}
```

**Why this works:**
- ✅ Can evolve structure without breaking old content
- ✅ Migrate content gradually
- ✅ No "big bang" refactor
- ✅ Learn and adapt

---

## What We're Deferring

### Defer to Mid-Phase 1: Validation System

**Why defer:** We don't have enough content yet to justify validation overhead

**When to add:**
- Week 7-8: When content volume increases (10+ JSON files)
- When bugs from typos become frequent
- When non-programmers start adding content

**How to add later:**
```typescript
// Week 1: No validation
async load(file) {
  return await fetch(file).then(r => r.json());
}

// Week 8: Add validation (simple wrapper, no refactor)
async load(file) {
  const data = await fetch(file).then(r => r.json());

  if (ENABLE_VALIDATION) {  // Feature flag
    this.validate(file, data);
  }

  return data;
}

private validate(file, data) {
  // Add schema validation here
  // Only runs if ENABLE_VALIDATION = true
}
```

**Incremental approach:**
1. Week 8: Add schema for abilities (combat is stable)
2. Week 9: Add schema for items (progression is stable)
3. Week 10: Add schema for quests (exploration is stable)
4. Week 12: Enable validation by default (all schemas defined)

### Defer to Phase 2: Hot-Reload

**Why defer:**
- Adds complexity (Vite plugin, cache invalidation, state management)
- Browser refresh is acceptable for solo developer
- 30-second iteration time vs. 1-second (not 10x difference for small team)
- Risk of bugs (data reload mid-gameplay)

**When to add:**
- Phase 2: When content changes are frequent (multiple times per hour)
- When team grows (multiple content creators)
- When 30-second refresh becomes painful

**How it works when we add it:**
```typescript
// Same DataLoader API, enhanced implementation
class DataLoader {
  constructor() {
    if (import.meta.hot) {
      import.meta.hot.on('data-changed', (file) => {
        this.reload(file);
      });
    }
  }

  private async reload(file) {
    // Clear cache
    this.cache.delete(file);

    // Reload data
    const data = await this.load(file);

    // Emit event (game systems can respond)
    EventBus.emit('data-reloaded', { file, data });
  }
}
```

**No refactoring needed** - same API, enhanced internals

### Defer to Phase 2: Complete Type Definitions

**Why defer:**
- We don't know what properties items need until we build item system
- We don't know what quest structure needs until we write quests
- Premature type definitions lead to refactoring

**Evolution strategy:**
```typescript
// Week 1: Minimal types
interface Item {
  id: string;
  name: string;
  [key: string]: any;  // Anything goes
}

// Week 6: Add known properties (discovered through usage)
interface Item {
  id: string;
  name: string;
  type: 'weapon' | 'armor' | 'consumable';
  stats: {
    damage?: string;
    ac?: number;
    healing?: number;
  };
  [key: string]: any;  // Still allow extensions
}

// Week 12: Lock down (structure is stable)
interface Item {
  id: string;
  name: string;
  type: ItemType;
  stats: ItemStats;
  rarity: Rarity;
  // No [key: string]: any
}
```

### Defer to Phase 3: Advanced Features

**Not needed for Phase 1 vertical slice:**
- Content authoring tools (visual editors)
- Localization system
- Asset management pipeline
- Procedural generation scripts
- Mod support
- Cloud save sync

**These are Phase 2+ features** - don't let them block core development

---

## Independent Development Strategy

### How Infrastructure and Gameplay Work in Parallel

**Key Insight:** Infrastructure work doesn't block gameplay if APIs are stable

**Stable APIs = Parallel Development**

### Example: Ability System Development

**Week 3: Gameplay Developer (Track A)**
```typescript
// Uses basic data loader
class CombatScene {
  async create() {
    const abilities = await dataLoader.load('abilities', 'fighter');
    this.setupAbilities(abilities);
  }

  useAbility(abilityId: string) {
    const ability = this.abilities[abilityId];
    // Apply ability effects
    this.applyDamage(ability.damage);
  }
}
```

**Week 3: Infrastructure Developer (Track B) - Working Independently**
```typescript
// Defines schema (doesn't affect gameplay code)
// src/data/schemas/ability.schema.json
{
  "type": "object",
  "properties": {
    "id": { "type": "string" },
    "damage": { "type": "number" }
  }
}
```

**Week 11: Infrastructure Enhancement (No Gameplay Changes)**
```typescript
// Enhanced data loader (same API!)
class DataLoader {
  async load(category, file) {
    const data = await fetch(`/data/${category}/${file}.json`);

    // NEW: Validation (gameplay code unchanged)
    if (VALIDATION_ENABLED) {
      this.validate(category, data);
    }

    // NEW: Hot-reload (gameplay code unchanged)
    if (import.meta.hot) {
      this.watchFile(category, file);
    }

    return data;  // Same return value
  }
}
```

**Gameplay code from Week 3 still works** - no refactoring needed!

### Integration Points

**Point 1: Data Loading API**
```typescript
// Stable contract:
interface IDataLoader {
  load(category: string, file: string): Promise<any>;
  get(category: string, file: string): any;
}

// Gameplay depends on interface, not implementation
// Infrastructure can enhance implementation without breaking gameplay
```

**Point 2: File Structure Convention**
```
/public/data/{category}/{file}.json

// As long as this convention is stable:
// - Gameplay code knows where to load from
// - Infrastructure code knows where to watch
// - Both can work independently
```

**Point 3: Data Shape Convention**
```typescript
// Stable convention: All entities have id, name
interface BaseEntity {
  id: string;
  name: string;
}

// Gameplay uses this convention
// Infrastructure validates this convention
// Both agree on contract, implement independently
```

### Weekly Sync Points

**Every Friday: 30-minute sync**
- Infrastructure updates gameplay team on new capabilities
- Gameplay team reports pain points
- Decide what infrastructure to prioritize next

**Example:**
```
Week 3 Sync:
  Gameplay: "Ability system working, but hard to test changes (need refresh)"
  Infrastructure: "Noted. Hot-reload can be added Week 11 if still painful"

Week 7 Sync:
  Gameplay: "Quest system working, but typos causing bugs"
  Infrastructure: "Quest schema ready. Want validation enabled?"
  Decision: "Yes, add validation for quests. Defer for other systems."
```

### Independence Through Abstraction

**Pattern: Abstraction Layer**
```typescript
// src/core/GameData.ts
// Stable API that gameplay uses

export class GameData {
  static async loadAbilities(): Promise<Record<string, Ability>> {
    return dataLoader.load('abilities', 'all');
  }

  static getAbility(id: string): Ability {
    return dataLoader.get('abilities', 'all')[id];
  }
}

// Gameplay uses this:
const ability = GameData.getAbility('fireball');

// Infrastructure enhances dataLoader implementation
// Gameplay code never changes
```

**Why this works:**
- ✅ Gameplay depends on stable abstraction
- ✅ Infrastructure implements abstraction
- ✅ Both teams work independently
- ✅ Integration is seamless

---

## Week 1 Tactical Plan (Day-by-Day)

### Day 1 (Monday): Project Setup + Data Loader

**Morning (4 hours):**
- ✅ Initialize project (Phaser 3 + Vite + TypeScript)
- ✅ Create basic file structure
- ✅ Configure build system
- ✅ Get "Hello World" running

**Afternoon (4 hours):**
- ✅ Implement DataLoader class (see Essential Infrastructure section)
- ✅ Write unit tests for DataLoader
- ✅ Document API in code comments

**Deliverable:** DataLoader.ts with working implementation

**Validation:** Can load JSON file and cache it

---

### Day 2 (Tuesday): File Structure + Documentation

**Morning (4 hours):**
- ✅ Create `/public/data/` structure
- ✅ Create example JSON files (characters, abilities, enemies)
- ✅ Write data/README.md with conventions

**Afternoon (4 hours):**
- ✅ Integrate DataLoader into BootScene
- ✅ Test loading all example files
- ✅ Add error handling and logging

**Deliverable:** Complete file structure with example data

**Validation:** BootScene successfully loads all data files

---

### Day 3 (Wednesday): TypeScript Types

**Morning (4 hours):**
- ✅ Define core types (Character, Ability, Enemy)
- ✅ Define placeholder types (Item, Quest, Location)
- ✅ Add JSDoc comments

**Afternoon (4 hours):**
- ✅ Update example JSON to match types
- ✅ Test type checking with TypeScript compiler
- ✅ Fix any type errors

**Deliverable:** game-data.types.ts with comprehensive types

**Validation:** No TypeScript errors, auto-complete works

---

### Day 4 (Thursday): Integration + Examples

**Morning (4 hours):**
- ✅ Create comprehensive example data (10+ characters, 20+ abilities)
- ✅ Write helper functions (getCharacter, getAbility, etc.)
- ✅ Test data access patterns

**Afternoon (4 hours):**
- ✅ Create DevTools helper for data inspection
- ✅ Write "How to Add Content" guide
- ✅ Document common patterns

**Deliverable:** Rich example dataset + documentation

**Validation:** Can inspect data in browser console

---

### Day 5 (Friday): Testing + Cleanup

**Morning (4 hours):**
- ✅ End-to-end test of data loading
- ✅ Performance check (all data loads in <1 second)
- ✅ Error scenario testing (missing files, malformed JSON)

**Afternoon (4 hours):**
- ✅ Code review and cleanup
- ✅ Final documentation pass
- ✅ Demo to team (if applicable)
- ✅ Plan Week 2 (combat system development)

**Deliverable:** Production-ready data loading system

**Validation:** All tests pass, team can start gameplay development

---

### Optional Day 6-7 (Weekend): Get Ahead (If Motivated)

**Optional tasks if you want to get ahead:**
- Define ability schema (for when we add validation later)
- Sketch out item data structure (even if not implementing yet)
- Research formula parsing libraries
- Prototype hot-reload (experiment, not production)

**Not required** - Week 1 is complete after Day 5

---

## Risk Assessment

### Risks of Pragmatic Approach

#### Risk 1: Technical Debt Accumulation

**Scenario:**
- Week 1-10: No validation, typos accumulate in JSON
- Week 12: Add validation, discover 50+ errors
- Must fix all errors before deploying

**Mitigation:**
- Add validation incrementally (Week 7-8 for stable systems)
- Manual review of new content (small team can catch typos)
- Write data tests early (basic structure checks)

**Probability:** Medium (40%)
**Impact:** Medium (1-2 days to fix)
**Acceptable:** Yes (cost is manageable)

#### Risk 2: Refactoring Pain

**Scenario:**
- Week 3: Build combat with simple ability structure
- Week 7: Realize we need complex ability effects
- Must refactor combat code + all ability JSON

**Mitigation:**
- Design for change from day 1 (loose interfaces)
- Use adapter pattern (convert old→new format)
- Refactor incrementally (both formats work during transition)

**Probability:** High (70%)
**Impact:** Low-Medium (2-4 hours per system)
**Acceptable:** Yes (learning by doing has this cost)

#### Risk 3: Schema Mismatch

**Scenario:**
- Developer adds property to JSON (`"cooldown": 5`)
- TypeScript type doesn't have `cooldown`
- Runtime works, but no type safety

**Mitigation:**
- Periodic type sync (update types when JSON changes)
- Use `[key: string]: any` for extensibility
- Add linter rule to catch common mismatches

**Probability:** High (80% will happen)
**Impact:** Low (minor inconvenience)
**Acceptable:** Yes (trade-off for flexibility)

### Risks of Architect Approach (For Comparison)

#### Risk A: Premature Optimization

**Scenario:**
- Spend 2 weeks building complete validation system
- Discover in Week 3 we designed schemas wrong
- Must refactor schemas + validation code

**Probability:** High (60%)
**Impact:** High (3-5 days wasted)
**Not Acceptable:** This is why we defer

#### Risk B: Analysis Paralysis

**Scenario:**
- Spend days debating "perfect" item schema
- Can't start building item system without schema
- Progress blocked by overthinking

**Probability:** High (50%)
**Impact:** High (delays timeline)
**Not Acceptable:** This is why we use conventions

### Risk Comparison

| Risk | Pragmatic Approach | Architect Approach |
|------|-------------------|-------------------|
| **Technical Debt** | Medium | Low |
| **Refactoring Cost** | Medium | Low |
| **Timeline Delay** | Low | **High** |
| **Over-Engineering** | Low | **High** |
| **Learning Waste** | Low | **High** |
| **Flexibility Loss** | Low | **High** |

**Conclusion:** Pragmatic risks are manageable. Architect risks block progress.

---

## Success Criteria

### Week 1 Success (End of Day 5)

**Must Have:**
- ✅ DataLoader works and is tested
- ✅ File structure established and documented
- ✅ TypeScript types defined for core systems
- ✅ Example data files load successfully
- ✅ Team can start Week 2 gameplay development

**Nice to Have:**
- ✅ Helper functions for common data access patterns
- ✅ DevTools integration for data inspection
- ✅ "How to Add Content" documentation

**Not Required:**
- ❌ Schemas (defer to Week 7+)
- ❌ Validation (defer to Week 8+)
- ❌ Hot-reload (defer to Phase 2)

### Month 1 Success (End of Week 4)

**Content Created:**
- 10+ characters defined and working
- 30+ abilities implemented in combat
- 10+ enemy types in battle system
- Basic combat loop playable

**Infrastructure Status:**
- Data loading stable and performant
- No major refactoring needed
- Content additions take <1 hour (create JSON, test)

**Team Velocity:**
- Gameplay development unblocked
- Infrastructure enhancements happening in parallel
- No technical debt preventing progress

### Phase 1 Success (End of Week 15)

**Data-Driven Achievement:**
- 100% of content in JSON (no hardcoded content)
- Designers can add content without programmer support
- Data structure is stable (no major refactors needed)
- Infrastructure is "good enough" (validation + schemas for critical systems)

**Quality Metrics:**
- <5% of bugs from bad data (most caught by manual review or basic validation)
- 95%+ of content changes work first try (conventions are clear)
- Average time to add new content: <2 hours

**Infrastructure Maturity:**
- Validation for critical systems (abilities, items, quests)
- Schemas documented for all major systems
- Hot-reload added if needed (team decision)
- Type safety is strong (80%+ of data is typed)

---

## Timeline Comparison

### Original Phase 1 Plan (15 weeks)
```
Week 1-2:   Foundation (basic setup)
Week 3-4:   Combat core
Week 5-6:   Character systems
Week 7-8:   Exploration
Week 9-10:  World map
Week 11-12: Game juice
Week 13-14: Content & balance
Week 15-16: Testing
```

### Architect Plan (17 weeks)
```
Week 1-2:   Infrastructure (validation, hot-reload, schemas) ← BLOCKS GAMEPLAY
Week 3-4:   Foundation (basic setup)
Week 5-6:   Combat core
Week 7-8:   Character systems
Week 9-10:  Exploration
Week 11-12: World map
Week 13-14: Game juice
Week 15-16: Content & balance
Week 17-18: Testing
```

### Pragmatic Plan (15 weeks - same as original!)
```
Week 1:     Foundation + Essential Infrastructure (3-5 days)
            ↓ Gameplay can start Day 6!
Week 2:     Combat core (Track A: Gameplay)
            Infrastructure: Define ability schema (Track B: Parallel)
Week 3-4:   Character systems (Track A)
            Infrastructure: Define item schema (Track B)
Week 5-6:   Exploration (Track A)
            Infrastructure: Define quest schema (Track B)
Week 7-8:   World map (Track A)
            Infrastructure: Add basic validation (Track B)
Week 9-10:  Game juice (Track A)
            Infrastructure: Enhance validation (Track B)
Week 11-12: Content & balance (Track A)
            Infrastructure: Hot-reload if needed (Track B)
Week 13-15: Testing

OPTIONAL Phase 2:
Week 16+:   Hot-reload, advanced validation, content tools
```

**Key Difference:**
- **Architect:** Infrastructure blocks gameplay (sequential)
- **Pragmatic:** Infrastructure enhances gameplay (parallel)

---

## Conclusion

### The Pragmatic Philosophy

**"Perfect is the enemy of good"**
- We don't need perfect infrastructure to start
- We don't need complete schemas to be data-driven
- We don't need validation to avoid bugs
- We need **just enough** to move forward, then iterate

**"Build what you need, when you need it"**
- Week 1: Need data loading → Build data loader
- Week 3: Need combat → Define ability structure
- Week 5: Need items → Define item structure
- Week 11: Need speed → Add hot-reload
- Week 13: Need quality → Add validation

**"Convention over configuration"**
- Use patterns, not rigid rules
- Document conventions, not schemas
- Learn by example, not specification
- Evolve structure, don't freeze it

### What Makes This Work

**Small Team Reality:**
- Solo or 2-3 developers
- Can communicate verbally
- Can coordinate manually
- Don't need enterprise-grade infrastructure

**Learning Through Building:**
- We discover needs by building
- Requirements emerge from usage
- Premature design is wasted effort
- Iterate, don't perfect

**Solid Foundation:**
- Data loading pattern is sound
- File structure is logical
- TypeScript types provide safety
- Conventions guide development

### Final Recommendation

**✅ Approve Pragmatic Approach If:**
- Team is 1-3 people
- Timeline pressure is real
- Learning by doing is acceptable
- Incremental refinement is OK

**❌ Use Architect Approach If:**
- Team is 5+ people
- Multiple non-programmers need to add content
- Data quality is mission-critical
- Timeline has 2+ week buffer

**For Phase 1 Vertical Slice:**
→ **Pragmatic approach is the right choice**

---

**Next Steps:**
1. **Day 1:** Start implementing DataLoader
2. **Day 2-3:** Establish file structure and types
3. **Day 4-5:** Create example data and integration test
4. **Week 2:** Start combat system (infrastructure is ready!)

**Questions to Answer:**
- Is 3-5 days for infrastructure acceptable?
- Do we agree on convention-over-configuration approach?
- Are we comfortable with incremental validation?
- Do we want hot-reload in Phase 1 or Phase 2?

**Decision Required:**
- ☐ Approve pragmatic approach (3-5 days infrastructure)
- ☐ Adjust architect approach (reduce from 2 weeks to 1 week)
- ☐ Hybrid (discuss specific concerns)

---

**Document Status:** Ready for Review
**Recommended Reviewers:** Lead Developer, Project Manager
**Expected Review Time:** 30 minutes
**Decision Timeline:** End of week (start implementation Monday)
