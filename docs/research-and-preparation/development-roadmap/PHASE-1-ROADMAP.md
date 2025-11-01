# Phase 1 Development Roadmap: Vertical Slice
## Hybrid Turn-Based RPG - Solarpunk Setting

**Document Version:** 2.0 (UPDATED: Pragmatic Infrastructure Approach)
**Last Updated:** 2025-11-01
**Timeline:** 15 weeks (infrastructure integrated into Week 1)
**Team Size:** Solo developer or 2-3 person team
**Infrastructure Strategy:** Minimal upfront (3-5 days), expand as needed

---

## **UPDATED: What's New in v2.0 - Pragmatic Infrastructure Approach**

### Key Changes Summary

**Decision Made:** Minimal infrastructure upfront (3-5 days) instead of complete infrastructure (2 weeks)

**Timeline Impact:** Maintains 15 weeks (no delay) - infrastructure built in parallel with gameplay

**Major Updates:**

1. **Week 1 Restructured (M1):**
   - Days 1-5: Essential data infrastructure (DataLoader, file structure, TypeScript types)
   - Days 6-10: Phaser setup and core architecture
   - Gameplay development starts Day 6 (not Week 3)

2. **Infrastructure Decision Points Added:**
   - Week 7-8: Evaluate if validation needed
   - Week 11: Evaluate if hot-reload needed
   - Checklists provided for each decision

3. **New Risk Assessment:**
   - Risk 0: Technical debt from minimal infrastructure (Medium impact, mitigated)
   - Risk 0b: Premature optimization avoidance (Success indicator added)
   - Updated monitoring metrics (complexity <4/10 Month 1)

4. **Updated Success Criteria:**
   - Added infrastructure-specific metrics
   - YAGNI principle emphasized
   - Complexity budget tracking

5. **New Section: Infrastructure Development Timeline**
   - Phase-by-phase breakdown
   - Decision point checklists
   - Monitoring metrics
   - Comparison table (Original vs. Architect vs. Pragmatic)

**Philosophy:** Convention over configuration. Build patterns, not implementations. Add complexity only when needed.

**Read if you're:**
- New to the project → Start with [Phase 1 Objectives](#phase-1-objectives)
- Reviewing infrastructure approach → Read [Milestone 1](#milestone-1-foundation-weeks-1-2) and [Infrastructure Timeline](#updated-infrastructure-development-timeline)
- Tracking risks → See [Risk Assessment](#risk-assessment)

---

## Table of Contents

1. [Phase 1 Objectives](#phase-1-objectives)
2. [Development Milestones](#development-milestones)
3. [Task Breakdown by System](#task-breakdown-by-system)
4. [Asset Creation Plan](#asset-creation-plan)
5. [Risk Assessment](#risk-assessment)
6. [Testing Strategy](#testing-strategy)
7. [Phase 2 Preview](#phase-2-preview)
8. [**NEW:** Infrastructure Development Timeline](#updated-infrastructure-development-timeline)
9. [Appendix](#appendix)

---

## Phase 1 Objectives

### Primary Goal
Build a **vertical slice** demonstrating all three game layers (world map, city exploration, combat) with a complete, playable gameplay loop focused on one city.

### Clear Deliverables

#### 1. World Map Layer
- [ ] Interactive world map with 1 playable city node
- [ ] 2-3 locked/upcoming city nodes (visual only)
- [ ] Metro-style travel system UI
- [ ] Map state persistence (unlocked locations)

#### 2. City Exploration Layer
- [ ] 1 fully explorable city district (15-20 locations)
- [ ] 5-7 interactive NPCs with branching dialogue
- [ ] 3-5 skill check encounters (D&D-style)
- [ ] Quest system with 2-3 main quests, 3-5 side quests
- [ ] Shop/vendor system with basic economy

#### 3. Combat System
- [ ] Turn-based tactical combat (Fire Emblem-inspired)
- [ ] Party of 3-4 playable characters
- [ ] 3-4 distinct character classes/types
- [ ] 5-7 enemy types with varied AI behaviors
- [ ] 15-20 combat abilities across all character types
- [ ] Status effects system (buffs/debuffs)
- [ ] Combat formation system

#### 4. Character Progression
- [ ] Level-up system (levels 1-5 for vertical slice)
- [ ] Ability unlock progression
- [ ] Basic stat growth system
- [ ] Equipment slots (weapon, armor, accessory)
- [ ] 8-12 equippable items

#### 5. Core Game Loop
- [ ] City exploration → Encounter/Quest → Combat → Rewards → Progression
- [ ] Save/load system
- [ ] Inventory management
- [ ] Party composition and switching

### Success Criteria

A successful Phase 1 vertical slice must:

1. **Be Playable End-to-End**: Player can start a new game, explore the city, engage in 3+ combat encounters, complete 1 main quest, and see character progression
2. **Demonstrate All Core Systems**: Every major system (combat, exploration, dialogue, progression) is functional
3. **Feel Satisfying**: Game juice (animations, effects, feedback) makes actions feel impactful
4. **Run Smoothly**: 60 FPS on target hardware, load times <3 seconds
5. **Be Extensible**: Architecture supports adding new cities, characters, and content without major refactoring

**UPDATED: Infrastructure Success Criteria**

6. **Be Data-Driven**: 100% of game content (characters, abilities, quests) defined in JSON, not hardcoded
7. **Maintain Low Complexity**: Infrastructure code <1000 lines, complexity rating <4/10 in Month 1, <6/10 by end
8. **Enable Fast Iteration**: Content changes take <1 hour from edit to test
9. **Support Incremental Enhancement**: Can add validation, hot-reload, schemas without refactoring gameplay code
10. **Follow YAGNI Principle**: No premature optimization - infrastructure added only when needed

### Scope Boundaries: What's NOT in Phase 1

**Explicitly Excluded from Phase 1:**
- Multiple cities (only 1 playable city)
- Multiple combat environments (1-2 battlefields max)
- Crafting system
- Multiplayer/co-op features
- Complex relationship/romance systems
- Procedural generation (use handcrafted content)
- Full audio implementation (placeholder sounds only)
- Cinematics/cutscenes (text-based dialogue only)
- Meta-progression (Hades-style unlocks) - saved for Phase 2+
- Advanced AI behaviors
- Endgame content/balancing

**UPDATED: Infrastructure Scope (Deferred to Phase 2 or Mid-Phase 1):**
- Complete JSON Schema definitions (add incrementally Week 7+)
- Full validation pipeline (add if needed Week 7-8)
- Hot-reload system (evaluate Week 11, may defer to Phase 2)
- Content authoring tools (visual editors - Phase 2+)
- Build-time validation (Phase 2)
- Asset optimization pipeline (Phase 2)
- Localization infrastructure (Phase 2+)

**Allowed Placeholders:**
- Placeholder art (procedural/geometric shapes)
- Placeholder sound effects (web audio API generated)
- Simplified UI (functional over beautiful)
- Limited enemy variety (recolor/resize for variations)

---

## Development Milestones

### Overview

| Milestone | Duration | Focus | Key Deliverable |
|-----------|----------|-------|----------------|
| M1: Foundation | 2 weeks | **UPDATED:** Essential infrastructure (Days 1-5) + Phaser setup (Days 6-10) | Running game with data-driven architecture |
| M2: Combat Core | 2 weeks | Turn-based battle system | Functional 1v1 combat |
| M3: Character Systems | 2 weeks | Classes, abilities, progression | 3 playable characters with abilities |
| M4: Exploration | 2 weeks | City navigation, dialogue + **UPDATED:** Add validation if needed | Explorable city with NPCs |
| M5: World Map & Integration | 1.5 weeks | World map, scene transitions | Full layer integration |
| M6: Game Juice & Polish | 2 weeks | Visual effects, animations + **UPDATED:** Evaluate hot-reload need | Satisfying combat feel |
| M7: Content & Balance | 2 weeks | Quests, encounters, enemies | Complete playable loop |
| M8: Testing & Bug Fixes | 1.5 weeks | QA, balance, optimization | Polished vertical slice |

**Total Timeline:** 15 weeks (infrastructure integrated, no delay from pragmatic approach)

**UPDATED Note:** The pragmatic infrastructure approach (3-5 days essential infrastructure in Week 1) maintains the original 15-week timeline. Infrastructure enhancements are built in parallel with gameplay development, not sequentially. The architect's 2-week upfront infrastructure proposal would have extended to 17 weeks.

---

### Milestone 1: Foundation (Weeks 1-2)

**Goal:** **UPDATED:** Establish essential data infrastructure (Days 1-5) and core game architecture (Days 6-10)

**Infrastructure Strategy:** Minimal, expandable approach - build patterns not implementations

---

#### **UPDATED: Week 1 - Parallel Development Tracks**

**Days 1-5: Essential Infrastructure (Track A - Priority)**

**Goal:** Establish data-driven patterns that enable gameplay development

##### Day 1-2 Deliverables: Data Loading Pattern
- [ ] Project repository initialized with version control
- [ ] Basic file structure created
- [ ] Implement DataLoader class (fetch + cache pattern)
- [ ] Error handling and logging
- [ ] Unit tests for DataLoader

**Technical Implementation:**
```typescript
// src/utils/DataLoader.ts
export class DataLoader {
  private cache: Map<string, any> = new Map();

  async load(category: string, filename: string): Promise<any> {
    const key = `${category}:${filename}`;
    if (this.cache.has(key)) return this.cache.get(key);

    const response = await fetch(`./data/${category}/${filename}.json`);
    const data = await response.json();
    this.cache.set(key, data);
    return data;
  }
}
```

##### Day 3 Deliverables: File Structure Convention
- [ ] Create `/public/data/` directory structure
- [ ] Define file organization conventions (documented)
- [ ] Create example JSON files (characters, abilities, enemies)
- [ ] Document naming conventions and patterns

**File Structure:**
```
/public/data/
├── characters/playable.json
├── abilities/fighter.json
├── enemies/common.json
├── items/ (placeholder - define Week 5-6)
├── quests/ (placeholder - define Week 7-8)
└── README.md (conventions documentation)
```

##### Day 4 Deliverables: TypeScript Types
- [ ] Define core interfaces (Character, Ability, Enemy)
- [ ] Define placeholder interfaces (Item, Quest, Location)
- [ ] Add JSDoc comments for documentation
- [ ] Configure TypeScript strict mode

**Type Strategy:**
```typescript
// Core types - stable and known
export interface Character {
  id: string;
  name: string;
  class: string;
  baseStats: BaseStats;
  abilities: string[];  // IDs, not nested objects
}

// Placeholder types - expand when building system
export interface Item {
  id: string;
  name: string;
  [key: string]: any;  // Allow flexibility, tighten later
}
```

##### Day 5 Deliverables: Integration Test
- [ ] Create starter data files (1 character, 2 abilities, 1 enemy)
- [ ] Integrate DataLoader into BootScene
- [ ] Verify data loads successfully
- [ ] Test error scenarios (missing files, malformed JSON)
- [ ] Document "How to Add Content" guide

**Days 6-10: Phaser Setup & Architecture (Track B)**

##### Day 6-7 Deliverables: Core Engine Setup
- [ ] Phaser 3 project configured and running
- [ ] Vite/Webpack build pipeline configured
- [ ] Development server with basic hot reload
- [ ] TypeScript compilation working
- [ ] Basic scene manager (3 empty scenes: WorldMap, Exploration, Combat)

##### Day 8-9 Deliverables: Game Architecture
- [ ] Implement EventBus for inter-scene communication
- [ ] Setup global GameState object
- [ ] Create AssetLoader integration with Phaser
- [ ] Basic input handling (keyboard/mouse)
- [ ] Scene transition system

##### Day 10 Deliverables: First Systems
- [ ] Basic UI components (Button, Panel, Text)
- [ ] Character sprite rendering (placeholder shapes)
- [ ] First integration: Load character data and render sprite

---

#### Week 2 Deliverables
- [ ] State management system (Redux or MobX)
- [ ] Save/load architecture (IndexedDB integration)
- [ ] Enhanced UI framework (menus, panels, dialogs)
- [ ] Tile rendering system (Tiled map support)
- [ ] Data-driven character loading (use DataLoader)

#### **UPDATED: Technical Tasks**

1. **UPDATED: Data Infrastructure (Days 1-5)**
   ```bash
   # Establish data-driven patterns
   - Implement DataLoader with caching
   - Define file structure conventions
   - Create TypeScript interfaces for data
   - Write example data files
   - Test data loading pipeline
   ```

2. **UPDATED: Core Architecture (Days 6-10)**
   - Create SceneManager class
   - Implement EventBus for inter-scene communication
   - Integrate DataLoader with Phaser asset system
   - Setup global GameState object
   - Implement basic UI components (Button, Panel, Text)

3. **UPDATED: Data Structures (Convention-Based)**
   ```javascript
   // Define core data models through examples
   // Use conventions, not rigid schemas
   - Character structure (example-based)
   - Ability structure (define as we build combat)
   - Enemy structure (example-based)
   - Item structure (defer to Week 5-6)
   - Quest structure (defer to Week 7-8)
   ```

#### **UPDATED: What We're NOT Building (Deferred)**

**❌ Defer to Mid-Phase 1:**
- Complete JSON Schema definitions (add when validating, Week 7-8)
- Full validation pipeline (add incrementally as systems stabilize)
- Hot-reload infrastructure (evaluate need in Week 11, add if painful)

**❌ Defer to Phase 2:**
- Content authoring tools (manual JSON editing is fine)
- Advanced type safety (strengthen gradually)
- Build-time validation (runtime validation sufficient)
- Asset management pipeline (Phaser handles it)

#### Testing Criteria
- [ ] Game boots without errors
- [ ] DataLoader successfully loads all example data files
- [ ] Can transition between 3 scenes
- [ ] Can save and load empty game state
- [ ] Placeholder character renders on screen using data from JSON
- [ ] Input (keyboard/mouse) responds correctly
- [ ] **UPDATED:** Data changes visible after browser refresh (<5 seconds cycle time)

#### **UPDATED: Success Metrics**

**Infrastructure Complexity Target:**
- Keep infrastructure code <500 lines (simple and maintainable)
- Data loading cycle time <1 second
- Content addition time <1 hour (create JSON, test, iterate)
- **Complexity Budget: 3/10** (simple, not overengineered)

#### Dependencies
- **None** (first milestone)

---

### Milestone 2: Combat Core (Weeks 3-4)

**Goal:** Implement functional turn-based combat system

#### Week 3 Deliverables
- [ ] Battle scene with grid/formation positioning
- [ ] Turn order system (speed-based)
- [ ] Basic attack action
- [ ] Damage calculation
- [ ] HP tracking and display
- [ ] Win/loss conditions

#### Week 4 Deliverables
- [ ] Action selection UI (Attack, Skill, Item, Defend)
- [ ] Target selection system
- [ ] 3-4 combat abilities per character
- [ ] Enemy AI (basic behavior tree)
- [ ] Battle rewards (XP, gold, items)

#### Technical Tasks
1. **Battle State Machine**
   ```javascript
   States:
   - BATTLE_START
   - TURN_START
   - ACTION_SELECT
   - TARGET_SELECT
   - EXECUTE_ACTION
   - ANIMATION_PLAY
   - TURN_END
   - VICTORY
   - DEFEAT
   ```

2. **Combat Grid System**
   - 2-row formation (front/back)
   - Position-based targeting (melee vs ranged)
   - Character/enemy sprite positioning
   - Grid visualization

3. **Action System**
   - Command pattern for actions
   - Action validation
   - Action execution pipeline
   - Action animation sequencing

4. **Damage Formula**
   ```javascript
   // Basic D&D-inspired formula
   Physical Damage = (Attack - Defense) * variance
   Magic Damage = (Magic Attack * spell power) - Magic Defense
   Critical Hit = damage * 2 (10% base chance)
   ```

#### Testing Criteria
- [ ] Can start battle with 2v2 combatants
- [ ] Turn order displays correctly
- [ ] Can select actions and targets
- [ ] Damage numbers appear and HP updates
- [ ] Battle ends when one side reaches 0 HP
- [ ] Returns to previous scene on victory/defeat

#### Dependencies
- **M1:** Scene management, state system, rendering

---

### Milestone 3: Character Systems (Weeks 5-6)

**Goal:** Create distinct character classes with unique abilities and progression

#### Week 5 Deliverables
- [ ] 3 character classes defined (Technomancer, Guardian, Bioengineer)
- [ ] Stat system (HP, MP, Attack, Defense, Magic, Speed)
- [ ] 5 unique abilities per class
- [ ] MP/resource management
- [ ] Ability cooldown system

#### Week 6 Deliverables
- [ ] Level-up system (1-5)
- [ ] Stat growth on level-up
- [ ] Ability unlock progression
- [ ] Equipment system (weapon, armor, accessory)
- [ ] Stat modifiers from equipment

#### Technical Tasks
1. **Character Class Architecture**
   ```javascript
   class Character {
     baseStats: { hp, mp, attack, defense, magic, speed }
     currentStats: { calculated with equipment }
     level: 1-5
     experience: number
     abilities: Ability[]
     equipment: { weapon, armor, accessory }
     statusEffects: StatusEffect[]
   }
   ```

2. **Ability System**
   - Ability data structure (name, description, cost, effects, targeting)
   - Ability execution
   - MP cost validation
   - Cooldown tracking
   - Area-of-effect abilities
   - Status effect application

3. **Progression System**
   - XP curve definition (exponential scaling)
   - Stat growth tables per class
   - Ability unlock levels
   - Level-up visual effect

4. **Equipment System**
   - Equipment slots
   - Stat modifiers
   - Equipment restrictions by class
   - Equip/unequip functionality

#### Character Class Design

**Technomancer** (Magic DPS)
- High Magic Attack, Low Defense
- Abilities: Lightning Strike, Energy Shield, Overcharge, System Hack, EMP Blast
- Equipment: Datapad (weapon), Light Armor

**Guardian** (Tank/Support)
- High HP and Defense, Low Speed
- Abilities: Shield Bash, Provoke, Defensive Stance, Rally, Fortify
- Equipment: Barrier Gauntlet, Heavy Armor

**Bioengineer** (Healer/Hybrid)
- Balanced stats, healing focus
- Abilities: Bio-Regeneration, Poison Cloud, Photosynthesis, Vine Whip, Nature's Blessing
- Equipment: Bio-Staff, Medium Armor

#### Testing Criteria
- [ ] Each character has distinct stats and abilities
- [ ] Abilities consume MP and execute correctly
- [ ] Level-up increases stats and unlocks abilities
- [ ] Equipment modifies stats appropriately
- [ ] Can switch equipment between battles

#### Dependencies
- **M2:** Combat system, damage calculation

---

### Milestone 4: Exploration (Weeks 7-8)

**Goal:** Create explorable city environment with NPCs and interactions

**UPDATED: Infrastructure Decision Point - Validation System**

**Decision Criteria:**
- [ ] Do we have 10+ JSON data files?
- [ ] Are typos/bugs from malformed data becoming frequent?
- [ ] Is content being added by non-programmers?

**If YES to 2+:** Add basic validation for quest and dialogue data
**If NO:** Defer validation, continue with manual testing

**Quick Validation Implementation (If Needed):**
```typescript
// Add to DataLoader.ts (2-3 hours)
private validateStructure(category: string, data: any): void {
  if (category === 'quests') {
    if (!data.id || !data.title || !data.objectives) {
      throw new Error(`Invalid quest structure: ${data.id}`);
    }
  }
}
```

#### Week 7 Deliverables
- [ ] Tiled map for city district (15-20 locations)
- [ ] Player movement (arrow keys/WASD)
- [ ] Collision detection
- [ ] NPC sprites and positioning
- [ ] Dialogue system
- [ ] 3 NPCs with branching dialogue

#### Week 8 Deliverables
- [ ] Skill check system (D&D-style: Persuade, Investigate, Strength, etc.)
- [ ] Dialogue with skill check options
- [ ] Quest system (accept, track, complete)
- [ ] Quest UI and journal
- [ ] Interactive objects (doors, chests, terminals)
- [ ] Shop/vendor system

#### Technical Tasks
1. **City Map Creation**
   - Design city district in Tiled (16x16 or 32x32 tiles)
   - Create tileset (procedural or placeholder)
   - Define collision layer
   - Add event trigger zones
   - Export as JSON

2. **Player Controller**
   ```javascript
   - Grid-based or free movement (recommend grid for consistency)
   - 4-directional movement
   - Collision with walls/NPCs
   - Interaction trigger (press E near NPC/object)
   - Movement animation (idle, walk)
   ```

3. **Dialogue System**
   - JSON dialogue tree structure
   - Dialogue box UI
   - Text typewriter effect
   - Choice selection
   - Conditional dialogue (based on flags/items)
   - Skill check integration

4. **Skill Check System**
   ```javascript
   Skills: Persuade, Intimidate, Investigate, Strength, Agility

   Check Formula:
   - Roll 1d20 + skill modifier
   - Compare to DC (Difficulty Class)
   - Success/Failure outcomes
   - Critical success (20) / Critical failure (1)
   ```

5. **Quest System**
   ```javascript
   Quest Structure:
   - ID, name, description
   - Objectives (list of conditions)
   - Rewards (XP, gold, items)
   - Prerequisites
   - State (not started, active, completed)
   ```

#### Testing Criteria
- [ ] Player can move around city without clipping through walls
- [ ] Can interact with NPCs to trigger dialogue
- [ ] Dialogue choices affect conversation flow
- [ ] Skill checks show success/failure with visual feedback
- [ ] Can accept and complete quests
- [ ] Shop allows buying/selling items

#### Dependencies
- **M1:** Tile rendering, UI framework
- **M3:** Inventory system (for shop/items)

---

### Milestone 5: World Map & Integration (Weeks 9-10)

**Goal:** Connect all three game layers with world map and transitions

#### Week 9 Deliverables
- [ ] World map scene with node-based navigation
- [ ] 1 unlocked city node (Phase 1 city)
- [ ] 2-3 locked city nodes (visual placeholders)
- [ ] Travel animation between map and city
- [ ] City entrance/exit triggers

#### Week 10 Deliverables
- [ ] Battle encounter triggers in city
- [ ] Transition from city → battle → city with state preservation
- [ ] World map state persistence
- [ ] Scene transition effects (fade, wipe)
- [ ] Full gameplay loop integration

#### Technical Tasks
1. **World Map Design**
   - Metro/node-based map (not tile-based)
   - City nodes with names and status (locked/unlocked)
   - Connection lines between cities
   - Click to select/travel
   - Display city info on hover

2. **Scene Transition System**
   ```javascript
   Transitions:
   - WorldMap → City (select city node)
   - City → WorldMap (exit city trigger)
   - City → Battle (enemy encounter)
   - Battle → City (victory/defeat)

   Transition Effects:
   - Fade to black (300ms)
   - Scene switch
   - Fade in (300ms)
   - Preserve game state across transitions
   ```

3. **Battle Encounter System**
   - Random encounters (configurable rate)
   - Story encounters (triggered by quests)
   - Encounter zones in city map
   - Enemy formation setup
   - Battle rewards distribution

4. **State Preservation**
   - Save player position before battle
   - Restore position after battle
   - Track defeated encounters (don't re-trigger)
   - Persist world map unlock status

#### Testing Criteria
- [ ] Can navigate from world map → city → battle → city → world map
- [ ] State persists across all transitions
- [ ] No memory leaks when switching scenes
- [ ] Transition animations play smoothly
- [ ] Can save/load at any point in the loop

#### Dependencies
- **M2:** Combat system
- **M4:** City exploration
- **M1:** Scene management

---

### Milestone 6: Game Juice & Polish (Weeks 11-12)

**Goal:** Make the game feel satisfying through visual effects and feedback

**UPDATED: Infrastructure Decision Point - Hot-Reload System**

**Decision Criteria:**
- [ ] Are we making 10+ content changes per hour?
- [ ] Is 30-second browser refresh cycle becoming painful?
- [ ] Are multiple team members editing content simultaneously?
- [ ] Is refresh disrupting testing flow frequently?

**If YES to 2+:** Implement hot-reload for JSON data files
**If NO:** Defer to Phase 2, browser refresh is acceptable

**Hot-Reload Implementation (If Needed):**
```typescript
// Enhance DataLoader.ts (4-6 hours)
// Uses Vite's HMR API to watch data files
if (import.meta.hot) {
  import.meta.hot.accept((newModule) => {
    this.cache.clear();
    this.reloadAll();
    EventBus.emit('data-hot-reload');
  });
}
```

**Trade-off Analysis:**
- **Cost:** 4-6 hours implementation + potential reload bugs
- **Benefit:** 1-second reload vs. 30-second refresh (29s saved per change)
- **Break-even:** ~10 changes/day = 5 minutes saved/day

#### Week 11 Deliverables
- [ ] Screen shake system
- [ ] Hit pause (freeze frames)
- [ ] Impact sparkles and flash effects
- [ ] Particle system for abilities
- [ ] Damage numbers with floating animation
- [ ] HP/MP bar animations

#### Week 12 Deliverables
- [ ] Ability-specific visual effects (5+ unique effects)
- [ ] Status effect indicators
- [ ] Level-up celebration effect
- [ ] Victory/defeat animations
- [ ] Menu transition animations
- [ ] Sound effect placeholders (Web Audio API)

#### Technical Tasks
1. **Core Juice Systems** (from Game Juice Guide)
   ```javascript
   - Camera shake with trauma-based decay
   - Hit pause (50-300ms based on impact)
   - Sprite flash on hit (white/red flash)
   - Screen flash for critical hits
   - Squash and stretch on impacts
   ```

2. **Particle System**
   - Base particle class
   - Particle emitter
   - Effect templates:
     * Fire (Technomancer abilities)
     * Ice/Frost (elemental abilities)
     * Lightning bolts
     * Healing sparkles
     * Impact sparks
     * Buff/debuff auras

3. **Damage Numbers**
   ```javascript
   Features:
   - Float upward with easing
   - Pause at peak (200ms)
   - Color based on type (physical=orange, magic=blue, heal=green, critical=yellow)
   - Size based on damage amount
   - Fade out smoothly
   ```

4. **Animation Sequencing**
   - Timeline-based combat animations
   - Timing: Move → Attack → Impact → Return (800ms total)
   - Ability cast animation (charge up → release → impact)
   - Victory pose sequence
   - Defeat animation

5. **UI Juice**
   - Button hover effects (scale up, glow)
   - Selection cursor animations
   - HP/MP bar smooth transitions
   - Menu slide-in animations
   - Quest complete celebration

#### Visual Effects by Ability Type

**Physical Attacks:**
- Slash arc effect
- Impact sparks (white/orange)
- Small screen shake (5px, 100ms)
- Damage number (orange)

**Magic Abilities:**
- Charging particles converge to caster
- Projectile with trail effect
- AoE explosion on impact
- Elemental particles (fire/ice/lightning)
- Medium screen shake (8px, 150ms)

**Healing:**
- Green sparkles float upward
- Gentle glow around target
- Healing number (green)
- Soothing pulse effect

**Buffs/Debuffs:**
- Aura ring around character
- Floating status icon
- Particle trail (color-coded)

#### Testing Criteria
- [ ] Every action has immediate visual feedback
- [ ] Critical hits feel more impactful than normal hits
- [ ] Abilities have distinct visual identities
- [ ] UI feels responsive and smooth
- [ ] No visual clutter (effects don't obscure gameplay)
- [ ] Performance: 60 FPS with all effects active

#### Dependencies
- **M2:** Combat system for effect triggers
- **M3:** Abilities to add effects to

---

### Milestone 7: Content & Balance (Weeks 13-14)

**Goal:** Create complete playable content and balance gameplay

#### Week 13 Deliverables
- [ ] 2 main quests (15-20 minutes each)
- [ ] 5 side quests (5-10 minutes each)
- [ ] 5-7 enemy types with AI behaviors
- [ ] 8-12 equippable items
- [ ] 3-5 consumable items
- [ ] Economy balancing (shop prices, rewards)

#### Week 14 Deliverables
- [ ] Combat balance pass (damage, HP, difficulty)
- [ ] 5-8 scripted battle encounters
- [ ] Boss encounter (mini-boss for Phase 1)
- [ ] Progression curve validation (levels 1-5)
- [ ] Tutorial/onboarding (first 5 minutes)

#### Content Tasks

1. **Main Quest Line**

   **Quest 1: "The Solar Nexus Disruption"**
   - Duration: 15-20 minutes
   - Objectives:
     * Talk to City Engineer
     * Investigate 3 power nodes
     * Skill checks: Investigate (DC 12), Repair (DC 15)
     * Combat: 2 encounters with corrupted maintenance drones
     * Reward: 250 XP, 100 credits, Upgraded Power Cell (equipment)

   **Quest 2: "Seeds of Discord"**
   - Duration: 15-20 minutes
   - Objectives:
     * Interview 4 NPCs about recent conflicts
     * Choose faction to support (branching dialogue)
     * Skill checks: Persuade (DC 14), Intimidate (DC 16)
     * Combat: 1 encounter (changes based on choice)
     * Reward: 300 XP, 150 credits, Faction-specific ability unlock

2. **Side Quests** (5 total)
   - "Lost Pet Bot" (5 min): Find missing companion drone
   - "Garden Parasite" (8 min): Clear bio-garden of invasive species (combat)
   - "Data Recovery" (10 min): Hack terminal, skill checks
   - "Mediation Request" (7 min): Resolve NPC dispute (dialogue-heavy)
   - "Supply Run" (5 min): Fetch quest with shop tutorial

3. **Enemy Design**

   | Enemy Type | HP | Attack | Defense | AI Behavior | Abilities |
   |------------|-----|--------|---------|-------------|-----------|
   | Maintenance Drone | 30 | 8 | 5 | Aggressive | Basic Attack, Self-Repair |
   | Security Bot | 45 | 12 | 8 | Defensive | Shield, Counter-Attack |
   | Corrupted AI | 25 | 15 | 3 | Erratic | Random Target, Overload |
   | Bio-Pest Swarm | 20 | 6 | 2 | Swarm | Multi-Hit, Poison |
   | Elite Enforcer | 60 | 18 | 12 | Tactical | Focus Fire, Summon Backup |
   | MINI-BOSS: Nexus Guardian | 150 | 20 | 15 | Pattern-based | 3 phases, special abilities |

4. **Equipment List**

   **Weapons:**
   - Standard Datapad (Technomancer): +5 Magic Attack
   - Arc Datapad (Technomancer): +10 Magic Attack, +5% Crit
   - Basic Gauntlet (Guardian): +5 Attack, +3 Defense
   - Fortified Gauntlet (Guardian): +8 Attack, +8 Defense
   - Bio-Staff (Bioengineer): +4 Magic Attack, +10 HP
   - Enhanced Bio-Staff (Bioengineer): +8 Magic Attack, +20 HP, +5% Healing

   **Armor:**
   - Light Suit, Medium Suit, Heavy Suit (3 tiers each)

   **Accessories:**
   - Speed Chip (+5 Speed)
   - Vitality Core (+30 HP)
   - Energy Capacitor (+20 MP)

5. **Consumable Items**
   - Minor Health Pack (restore 30 HP): 20 credits
   - Health Pack (restore 60 HP): 50 credits
   - Minor Energy Cell (restore 15 MP): 30 credits
   - Energy Cell (restore 30 MP): 70 credits
   - Revive Kit (resurrect with 50% HP): 100 credits

#### Balance Tasks

1. **Combat Tuning**
   ```javascript
   Target Metrics:
   - Average battle duration: 2-4 minutes
   - Player death rate: 10-20% (first attempt)
   - Resource usage: 30-50% MP per battle
   - Critical hit rate: 10%
   - Ability usage: 2-3 abilities per battle average
   ```

2. **Progression Curve**
   ```javascript
   Level 1-2: 100 XP (tutorial battles)
   Level 2-3: 250 XP (2-3 battles)
   Level 3-4: 500 XP (3-4 battles)
   Level 4-5: 750 XP (4-5 battles)

   Total XP for Phase 1: ~1600 XP
   Total battles: 8-10 encounters
   ```

3. **Economy Balance**
   - Battle rewards: 30-50 credits per encounter
   - Quest rewards: 50-150 credits
   - Equipment costs: 50-300 credits
   - Consumable costs: 20-100 credits
   - Player should afford 2-3 equipment upgrades during Phase 1

#### Tutorial Design

**First 5 Minutes:**
1. Wake up in medical bay (narrative intro)
2. Movement tutorial (walk to exit)
3. First NPC interaction (dialogue system intro)
4. First battle (combat tutorial, 1v1, guided actions)
5. Level up to level 2 (progression tutorial)
6. Receive first quest (quest system intro)

#### Testing Criteria
- [ ] Can complete both main quests
- [ ] All side quests are completable
- [ ] Enemy difficulty feels balanced
- [ ] Player reaches level 5 by end of content
- [ ] Economy feels fair (not too generous or stingy)
- [ ] Tutorial teaches all core mechanics

#### Dependencies
- **All previous milestones** (content integrates all systems)

---

### Milestone 8: Testing & Bug Fixes (Weeks 15-16)

**Goal:** Polish, optimize, and fix all critical bugs

#### Week 15 Deliverables
- [ ] Complete playtesting (3+ full playthroughs)
- [ ] Bug tracking and prioritization
- [ ] Critical bug fixes
- [ ] Performance optimization pass
- [ ] Save/load edge case handling

#### Week 16 Deliverables
- [ ] Final balance adjustments
- [ ] UI/UX improvements based on feedback
- [ ] Build optimization (minification, compression)
- [ ] Deployment to GitHub Pages/Netlify
- [ ] Release notes and documentation

#### Testing Tasks

1. **Functional Testing**
   - [ ] All quests completable
   - [ ] No soft-locks or game-breaking bugs
   - [ ] Save/load works in all scenes
   - [ ] All abilities function correctly
   - [ ] Equipment system works properly
   - [ ] Dialogue choices lead to correct outcomes

2. **Performance Testing**
   - [ ] Frame rate: 60 FPS target
   - [ ] Load time: <3 seconds per scene
   - [ ] Memory usage: Monitor for leaks
   - [ ] Asset loading: Optimize largest assets
   - [ ] Battle performance: Test with max effects

3. **Balance Testing**
   - [ ] Record combat metrics (duration, deaths, resource usage)
   - [ ] Track progression pace (XP, gold, levels)
   - [ ] Test different party compositions
   - [ ] Test equipment progression
   - [ ] Validate economy balance

4. **UX Testing**
   - [ ] Can complete game without instructions
   - [ ] UI elements are clear and readable
   - [ ] Feedback is immediate for all actions
   - [ ] No confusing interactions
   - [ ] Tutorial is helpful, not overwhelming

#### Bug Prioritization

**P0 (Critical - Must Fix):**
- Game crashes
- Soft locks (can't progress)
- Data corruption/loss
- Core mechanics broken

**P1 (High - Should Fix):**
- Visual glitches affecting gameplay
- Balance issues making game too easy/hard
- Missing feedback on actions
- Performance issues (FPS drops)

**P2 (Medium - Nice to Fix):**
- Minor visual issues
- UI polish
- Quality-of-life improvements
- Non-critical optimization

**P3 (Low - Defer to Phase 2):**
- Edge case bugs
- Nice-to-have features
- Minor polish items

#### Optimization Tasks

1. **Asset Optimization**
   - Compress images (PNG → WebP)
   - Minimize audio files
   - Bundle optimization (code splitting)
   - Lazy load non-critical assets

2. **Code Optimization**
   - Identify performance bottlenecks (Chrome DevTools)
   - Optimize particle systems (object pooling)
   - Reduce draw calls
   - Cache expensive calculations

3. **Build Optimization**
   - Minify JavaScript
   - Tree-shaking unused code
   - Gzip compression
   - CDN for common libraries

#### Deployment Checklist

- [ ] Build succeeds without errors
- [ ] All assets load correctly in production
- [ ] Game runs in multiple browsers (Chrome, Firefox, Safari, Edge)
- [ ] Mobile/tablet responsive (if targeting)
- [ ] Analytics integration (optional)
- [ ] Error tracking (optional, e.g., Sentry)

#### Testing Criteria
- [ ] Zero P0 bugs
- [ ] <5 P1 bugs
- [ ] 60 FPS maintained
- [ ] Game is beatable by playtesters
- [ ] Deployment successful

#### Dependencies
- **M7:** Complete content for testing

---

## Task Breakdown by System

### 1. Project Setup and Foundation

**Priority:** Critical
**Timeline:** Week 1-2
**Owner:** Lead Developer

#### Tasks

1. **Repository Setup**
   - [ ] Initialize Git repository
   - [ ] Create `.gitignore` for node_modules, build files
   - [ ] Setup branch strategy (main, dev, feature branches)
   - [ ] Create README with setup instructions
   - [ ] Define commit message conventions

2. **Development Environment**
   - [ ] Install Node.js (v18+)
   - [ ] Initialize npm project
   - [ ] Install Phaser 3 (`npm install phaser`)
   - [ ] Install TypeScript (`npm install typescript -D`)
   - [ ] Install bundler (Vite recommended: `npm install vite -D`)
   - [ ] Configure TypeScript (`tsconfig.json`)
   - [ ] Configure Vite (`vite.config.js`)
   - [ ] Setup hot module replacement

3. **Project Structure**
   ```
   /project-root
   ├── /src
   │   ├── /scenes
   │   │   ├── BootScene.ts
   │   │   ├── WorldMapScene.ts
   │   │   ├── ExplorationScene.ts
   │   │   └── BattleScene.ts
   │   ├── /entities
   │   │   ├── Character.ts
   │   │   ├── Enemy.ts
   │   │   └── NPC.ts
   │   ├── /systems
   │   │   ├── BattleSystem.ts
   │   │   ├── DialogueSystem.ts
   │   │   ├── QuestSystem.ts
   │   │   └── ProgressionSystem.ts
   │   ├── /ui
   │   │   ├── Button.ts
   │   │   ├── Panel.ts
   │   │   ├── DialogueBox.ts
   │   │   └── BattleUI.ts
   │   ├── /data
   │   │   ├── abilities.json
   │   │   ├── characters.json
   │   │   ├── enemies.json
   │   │   ├── items.json
   │   │   └── quests.json
   │   ├── /utils
   │   │   ├── StateManager.ts
   │   │   ├── SaveSystem.ts
   │   │   └── EventBus.ts
   │   ├── /effects
   │   │   ├── ParticleSystem.ts
   │   │   ├── ScreenShake.ts
   │   │   └── Animations.ts
   │   └── main.ts
   ├── /assets
   │   ├── /sprites
   │   ├── /tilesets
   │   ├── /maps
   │   ├── /audio
   │   └── /ui
   ├── /public
   │   └── index.html
   ├── package.json
   ├── tsconfig.json
   └── vite.config.js
   ```

4. **Core Configuration**
   ```typescript
   // main.ts - Phaser game config
   const config: Phaser.Types.Core.GameConfig = {
     type: Phaser.AUTO,
     width: 1280,
     height: 720,
     parent: 'game-container',
     backgroundColor: '#2d2d2d',
     pixelArt: true,
     physics: {
       default: 'arcade',
       arcade: {
         gravity: { y: 0 },
         debug: false
       }
     },
     scene: [
       BootScene,
       WorldMapScene,
       ExplorationScene,
       BattleScene
     ]
   };
   ```

5. **CI/CD Setup** (Optional for Phase 1, recommended)
   - [ ] Create `.github/workflows/deploy.yml`
   - [ ] Configure GitHub Actions for build
   - [ ] Setup automatic deployment to GitHub Pages
   - [ ] Add build status badge to README

#### Acceptance Criteria
- [ ] `npm run dev` starts development server
- [ ] `npm run build` creates production build
- [ ] Game window appears with blank screen
- [ ] No console errors
- [ ] Hot reload works when editing files

---

### 2. World Map Implementation

**Priority:** High
**Timeline:** Week 9-10
**Owner:** Gameplay Developer

#### Tasks

1. **Map Design**
   - [ ] Design world map layout (node-based, metro-style)
   - [ ] Create background art (procedural gradient or simple image)
   - [ ] Define city node positions (1 unlocked, 2-3 locked)
   - [ ] Create connection lines between nodes

2. **Node System**
   ```typescript
   interface CityNode {
     id: string;
     name: string;
     x: number;
     y: number;
     unlocked: boolean;
     description: string;
     level: number; // Recommended player level
     connections: string[]; // Connected city IDs
   }
   ```

3. **WorldMapScene Implementation**
   - [ ] Render background
   - [ ] Render city nodes (circles/icons)
   - [ ] Render connection lines
   - [ ] Add node labels
   - [ ] Implement node hover effect (scale, glow)
   - [ ] Implement node click detection
   - [ ] Show city info panel on hover
   - [ ] Transition to city scene on click

4. **State Management**
   - [ ] Track unlocked cities in GameState
   - [ ] Persist unlocked status in save file
   - [ ] Add method to unlock new cities
   - [ ] Trigger unlock events (from quests, story progression)

5. **UI Elements**
   - [ ] "Current Location" indicator
   - [ ] City info panel (name, level, description)
   - [ ] Legend/key for node status
   - [ ] Return to menu button

#### Acceptance Criteria
- [ ] World map displays with nodes and connections
- [ ] Can click unlocked city to enter
- [ ] Locked cities show "locked" state visually
- [ ] Hover shows city information
- [ ] Map state persists across saves

---

### 3. City Exploration Implementation

**Priority:** Critical
**Timeline:** Week 7-8
**Owner:** Gameplay Developer

#### Tasks

1. **Map Creation (Tiled)**
   - [ ] Install Tiled Map Editor
   - [ ] Create tileset (32x32 tiles, procedural or placeholder)
   - [ ] Design city layout (15-20 locations/rooms)
   - [ ] Add collision layer
   - [ ] Add event object layer (NPC positions, triggers)
   - [ ] Add decoration layer
   - [ ] Export as JSON (uncompressed)

2. **Player Movement**
   ```typescript
   class Player extends Phaser.GameObjects.Sprite {
     - Grid-based movement (4 directions)
     - Movement speed: 4 tiles/second
     - Collision detection with walls
     - Movement animation (4-frame walk cycle per direction)
     - Idle animation
   }
   ```

3. **NPC System**
   - [ ] Create NPC class
   - [ ] Position NPCs from Tiled object layer
   - [ ] NPC sprites (placeholders: colored rectangles)
   - [ ] Interaction trigger (press E when near NPC)
   - [ ] NPC dialogue data structure

4. **Interactive Objects**
   - [ ] Doors (transition between areas)
   - [ ] Chests (contain items)
   - [ ] Terminals (skill checks, info)
   - [ ] Quest objects (interact to progress quest)

5. **Camera System**
   - [ ] Follow player smoothly
   - [ ] Limit camera to map bounds
   - [ ] Zoom controls (optional)

#### Dialogue System

```typescript
interface DialogueNode {
  id: string;
  speaker: string;
  text: string;
  choices?: {
    text: string;
    nextNodeId: string;
    condition?: string; // e.g., "flag:quest1_complete"
    skillCheck?: {
      skill: string;
      dc: number;
      successNode: string;
      failureNode: string;
    };
  }[];
}
```

**Implementation:**
- [ ] Dialogue box UI (bottom third of screen)
- [ ] Typewriter text effect (30 characters/second)
- [ ] Choice buttons
- [ ] Speaker name display
- [ ] Speaker portrait (optional, placeholder)
- [ ] Dialogue skip (click to complete typewriter)
- [ ] Dialogue history (optional)

#### Skill Check System

```typescript
class SkillCheck {
  performCheck(skill: string, dc: number, modifier: number): {
    success: boolean;
    roll: number;
    total: number;
    critical: boolean;
  } {
    const roll = Math.floor(Math.random() * 20) + 1;
    const total = roll + modifier;
    return {
      success: total >= dc,
      roll: roll,
      total: total,
      critical: roll === 20 || roll === 1
    };
  }
}
```

**Skills:**
- Persuade (Charisma-based)
- Intimidate (Strength-based)
- Investigate (Intelligence-based)
- Agility (Dexterity-based)
- Strength (raw power)

**UI:**
- [ ] Show skill check prompt
- [ ] Display DC and player modifier
- [ ] Animate dice roll
- [ ] Show result (success/failure, roll total)
- [ ] Branch dialogue based on result

#### Quest System

```typescript
interface Quest {
  id: string;
  title: string;
  description: string;
  objectives: Objective[];
  rewards: {
    xp: number;
    gold: number;
    items: string[];
  };
  state: 'not_started' | 'active' | 'completed';
  prerequisites?: string[]; // Required quest IDs
}

interface Objective {
  id: string;
  description: string;
  type: 'talk' | 'collect' | 'defeat' | 'explore' | 'skill_check';
  target?: string; // NPC ID, item ID, enemy ID
  count?: number;
  currentCount?: number;
  completed: boolean;
}
```

**Implementation:**
- [ ] Quest acceptance (from NPC dialogue)
- [ ] Quest tracking in GameState
- [ ] Objective progress tracking
- [ ] Quest completion detection
- [ ] Reward distribution
- [ ] Quest journal UI (view active/completed quests)

#### Shop System

```typescript
interface Shop {
  id: string;
  name: string;
  inventory: {
    itemId: string;
    price: number;
    stock: number; // -1 for unlimited
  }[];
  buyPriceMultiplier: 1.0;
  sellPriceMultiplier: 0.5;
}
```

**Implementation:**
- [ ] Shop UI (item list, buy/sell)
- [ ] Item filtering (weapons, armor, consumables)
- [ ] Transaction handling (validate currency)
- [ ] Stock tracking
- [ ] Item tooltips (stats, description)

#### Acceptance Criteria
- [ ] Player can move around city smoothly
- [ ] NPCs are positioned correctly
- [ ] Can interact with NPCs to trigger dialogue
- [ ] Dialogue choices affect conversation flow
- [ ] Skill checks roll dice and show results
- [ ] Quests can be accepted and tracked
- [ ] Shop allows buying and selling items

---

### 4. Combat System Implementation

**Priority:** Critical
**Timeline:** Week 3-4
**Owner:** Combat Designer

#### Tasks

1. **Battle Scene Setup**
   - [ ] Create BattleScene class
   - [ ] Design battlefield layout (2 rows, front/back)
   - [ ] Position character sprites (left side)
   - [ ] Position enemy sprites (right side)
   - [ ] Background art (placeholder gradient)

2. **Battle State Machine**
   ```typescript
   enum BattleState {
     INITIALIZE,
     TURN_START,
     ACTION_SELECT,
     TARGET_SELECT,
     EXECUTE_ACTION,
     ANIMATION_PLAY,
     CHECK_VICTORY,
     TURN_END,
     VICTORY,
     DEFEAT
   }
   ```

3. **Turn Order System**
   - [ ] Calculate turn order (speed-based)
   - [ ] Display turn order UI (character portraits)
   - [ ] Highlight current actor
   - [ ] Update turn order after each action

4. **Action System**
   ```typescript
   interface BattleAction {
     type: 'attack' | 'skill' | 'item' | 'defend';
     actorId: string;
     targetIds: string[];
     abilityId?: string;
     itemId?: string;
   }
   ```

5. **Combat UI**
   - [ ] Action menu (Attack, Skill, Item, Defend buttons)
   - [ ] Skill menu (list of available abilities)
   - [ ] Item menu (list of usable items)
   - [ ] Target selection (highlight selectable targets)
   - [ ] HP/MP bars for all combatants
   - [ ] Status effect icons

6. **Damage Calculation**
   ```typescript
   function calculateDamage(attacker: Character, defender: Character, ability: Ability): number {
     const baseDamage = ability.baseDamage || (attacker.attack - defender.defense);
     const variance = 0.9 + Math.random() * 0.2; // ±10%
     const critMultiplier = isCritical() ? 2.0 : 1.0;
     const typeModifier = getTypeModifier(ability.damageType, defender.weaknesses);

     return Math.max(1, Math.floor(baseDamage * variance * critMultiplier * typeModifier));
   }
   ```

7. **Enemy AI**
   ```typescript
   class EnemyAI {
     selectAction(enemy: Enemy, allies: Enemy[], opponents: Character[]): BattleAction {
       // Simple behavior tree
       if (enemy.hp < enemy.maxHp * 0.3 && enemy.hasAbility('heal')) {
         return { type: 'skill', abilityId: 'heal', targetIds: [enemy.id] };
       }

       if (Math.random() < 0.3 && enemy.abilities.length > 0) {
         // Use random ability
         const ability = randomChoice(enemy.abilities);
         return { type: 'skill', abilityId: ability.id, targetIds: selectTargets(ability) };
       }

       // Default: basic attack weakest opponent
       const target = opponents.sort((a, b) => a.hp - b.hp)[0];
       return { type: 'attack', targetIds: [target.id] };
     }
   }
   ```

8. **Battle Flow**
   - [ ] Initialize battle (load characters, enemies)
   - [ ] Calculate initial turn order
   - [ ] Turn loop:
     * Display action menu for player-controlled character
     * Wait for action selection
     * Validate action
     * Execute action (play animation)
     * Apply effects (damage, status)
     * Check for victory/defeat
     * Next turn
   - [ ] Handle victory (rewards, XP)
   - [ ] Handle defeat (game over or return)

#### Acceptance Criteria
- [ ] Battle starts with correct characters and enemies
- [ ] Turn order is calculated correctly
- [ ] Can select actions and targets
- [ ] Actions execute and apply effects
- [ ] HP updates visually
- [ ] Battle ends on victory or defeat
- [ ] Transitions back to exploration scene

---

### 5. Character Progression Implementation

**Priority:** High
**Timeline:** Week 5-6
**Owner:** Systems Designer

#### Tasks

1. **Character Data Structure**
   ```typescript
   interface Character {
     id: string;
     name: string;
     class: 'technomancer' | 'guardian' | 'bioengineer';
     level: number;
     experience: number;
     baseStats: Stats;
     statModifiers: Stats; // From equipment
     equipment: {
       weapon: Item | null;
       armor: Item | null;
       accessory: Item | null;
     };
     abilities: Ability[];
     statusEffects: StatusEffect[];
   }

   interface Stats {
     hp: number;
     mp: number;
     attack: number;
     defense: number;
     magic: number;
     speed: number;
   }
   ```

2. **Level-Up System**
   - [ ] Define XP curve (exponential: `100 * Math.pow(1.5, level - 1)`)
   - [ ] Track experience points
   - [ ] Detect level-up condition
   - [ ] Apply stat growth
   - [ ] Check for ability unlocks
   - [ ] Play level-up animation/effect
   - [ ] Show level-up notification (new stats, abilities)

3. **Stat Growth Tables**
   ```typescript
   const statGrowth = {
     technomancer: {
       hp: 5,
       mp: 8,
       attack: 1,
       defense: 1,
       magic: 4,
       speed: 2
     },
     guardian: {
       hp: 10,
       mp: 3,
       attack: 3,
       defense: 4,
       magic: 1,
       speed: 1
     },
     bioengineer: {
       hp: 7,
       mp: 6,
       attack: 2,
       defense: 2,
       magic: 3,
       speed: 2
     }
   };
   ```

4. **Ability Unlock Progression**
   - [ ] Define ability unlock levels
   - [ ] Check for unlocks on level-up
   - [ ] Add ability to character
   - [ ] Show "New Ability Unlocked" notification

5. **Equipment System**
   - [ ] Equipment data structure
   - [ ] Equip/unequip methods
   - [ ] Stat recalculation when equipment changes
   - [ ] Equipment restrictions (class-based)
   - [ ] Equipment UI (character screen)

6. **Inventory System**
   - [ ] Inventory data structure (item ID, quantity)
   - [ ] Add/remove items
   - [ ] Item stacking (consumables)
   - [ ] Inventory UI (grid or list)
   - [ ] Item tooltips
   - [ ] Item sorting/filtering

#### Acceptance Criteria
- [ ] Characters gain XP from battles
- [ ] Level-up increases stats
- [ ] Abilities unlock at correct levels
- [ ] Equipment modifies stats correctly
- [ ] Inventory stores items properly

---

### 6. UI/UX Implementation

**Priority:** High
**Timeline:** Throughout all milestones
**Owner:** UI Developer

#### Tasks

1. **Core UI Components**
   - [ ] Button class (hover, click, disabled states)
   - [ ] Panel class (background, border)
   - [ ] Text component (typewriter effect)
   - [ ] List component (scrollable)
   - [ ] Grid component (for items/abilities)
   - [ ] Tooltip component

2. **Battle UI**
   - [ ] HP/MP bars for all combatants
   - [ ] Turn order display (portrait list)
   - [ ] Action menu (4 buttons: Attack, Skill, Item, Defend)
   - [ ] Skill list panel
   - [ ] Item list panel
   - [ ] Target selection cursor
   - [ ] Damage number display
   - [ ] Status effect icons
   - [ ] Battle log (text feed of actions)

3. **Exploration UI**
   - [ ] Player HP/MP bars (always visible)
   - [ ] Mini-map (optional)
   - [ ] Interaction prompt ("Press E to talk")
   - [ ] Quest tracker (active quest objectives)

4. **Menu UI**
   - [ ] Main menu (New Game, Continue, Options)
   - [ ] Pause menu (Resume, Save, Load, Options, Quit)
   - [ ] Character screen (stats, equipment, abilities)
   - [ ] Inventory screen (items, equipment)
   - [ ] Quest journal (active and completed quests)
   - [ ] Options screen (volume, controls, etc.)

5. **Dialogue UI**
   - [ ] Dialogue box (bottom third of screen)
   - [ ] Speaker name
   - [ ] Typewriter text
   - [ ] Choice buttons (1-4 choices)
   - [ ] Skill check display

6. **UI Themes and Styling**
   - [ ] Define color palette (solarpunk: greens, earth tones, bright accents)
   - [ ] Font selection (readable, thematic)
   - [ ] Button styles
   - [ ] Panel styles
   - [ ] Consistent spacing and alignment

#### Acceptance Criteria
- [ ] All UI elements are readable
- [ ] UI responds to input
- [ ] UI scales properly (if supporting multiple resolutions)
- [ ] Consistent visual style across all screens

---

### 7. Game Juice and Polish

**Priority:** Medium-High
**Timeline:** Week 11-12
**Owner:** VFX Developer

See Milestone 6 for detailed tasks. Key systems:
- Screen shake
- Hit pause
- Particle systems
- Damage numbers
- Animations
- Sound effects (placeholders)

---

### 8. Testing and Balancing

**Priority:** Critical
**Timeline:** Week 13-16
**Owner:** QA / All Team Members

See Milestone 7 and 8 for detailed tasks. Key activities:
- Playtesting
- Bug fixing
- Balance tuning
- Performance optimization
- Deployment

---

## Asset Creation Plan

### Zero-Touch Asset Generation Strategy

**Philosophy:** Minimize manual asset creation through procedural generation, code-generated graphics, and AI-assisted tools. Assets should be "good enough" for Phase 1, with art upgrades planned for later phases.

### Procedural Graphics Timeline

#### Week 1-2: Core Placeholders
- [ ] Character sprites: Colored rectangles with class icons
- [ ] Enemy sprites: Geometric shapes (circles, triangles, squares)
- [ ] Tilesets: Solid color tiles with borders
- [ ] UI elements: CSS-styled div elements or canvas-drawn shapes

#### Week 3-6: Combat Assets
- [ ] Ability effects: Particle systems (code-generated)
- [ ] Impact effects: Flash sprites (code-generated)
- [ ] Status icons: Simple geometric symbols
- [ ] Damage numbers: Dynamic text rendering

#### Week 7-10: Exploration Assets
- [ ] City tileset: Procedural generation (see below)
- [ ] NPC sprites: Colored shapes with unique patterns
- [ ] Interactive objects: Simple icon representations
- [ ] World map: CSS gradient background + SVG nodes

#### Week 11-14: Polish Assets
- [ ] Particle textures: Gaussian blur circles
- [ ] UI backgrounds: Gradient panels
- [ ] Icons: SVG or font icons (Font Awesome, Material Icons)
- [ ] Effects: Shader-based (WebGL)

### Procedural Generation Techniques

#### 1. Character Sprites (Code-Generated)

```javascript
function generateCharacterSprite(className, color) {
  const canvas = document.createElement('canvas');
  canvas.width = 64;
  canvas.height = 64;
  const ctx = canvas.getContext('2d');

  // Body
  ctx.fillStyle = color;
  ctx.fillRect(16, 20, 32, 40);

  // Head
  ctx.beginPath();
  ctx.arc(32, 16, 12, 0, Math.PI * 2);
  ctx.fill();

  // Class-specific icon
  ctx.fillStyle = '#FFFFFF';
  ctx.font = '20px Arial';
  ctx.textAlign = 'center';
  ctx.fillText(getClassIcon(className), 32, 40);

  return canvas;
}
```

#### 2. Tileset Generation (Procedural)

```javascript
function generateTileset() {
  const tileSize = 32;
  const tiles = {
    floor: createTile(tileSize, '#CCCCCC', '#AAAAAA'),
    wall: createTile(tileSize, '#555555', '#333333'),
    door: createTile(tileSize, '#8B4513', '#654321'),
    grass: createTile(tileSize, '#44AA44', '#339933')
  };

  return tiles;
}

function createTile(size, baseColor, shadeColor) {
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');

  // Base color
  ctx.fillStyle = baseColor;
  ctx.fillRect(0, 0, size, size);

  // Add noise/texture
  for (let i = 0; i < 20; i++) {
    ctx.fillStyle = shadeColor;
    ctx.globalAlpha = 0.3;
    const x = Math.random() * size;
    const y = Math.random() * size;
    ctx.fillRect(x, y, 2, 2);
  }

  // Border
  ctx.globalAlpha = 1;
  ctx.strokeStyle = shadeColor;
  ctx.lineWidth = 1;
  ctx.strokeRect(0, 0, size, size);

  return canvas;
}
```

#### 3. Particle Effects (Runtime Generation)

See Game Juice Guide for comprehensive particle system implementation. All effects are code-generated at runtime.

#### 4. UI Elements (CSS + Canvas)

```css
/* Use CSS for UI styling */
.button {
  background: linear-gradient(135deg, #44AA44, #339933);
  border: 2px solid #FFFFFF;
  border-radius: 8px;
  padding: 10px 20px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.1s;
}

.button:hover {
  transform: scale(1.05);
  box-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
}
```

### AI-Assisted Asset Generation (Optional)

For assets where code generation is insufficient, consider:

#### Free/Low-Cost AI Tools
1. **DALL-E Mini / Craiyon** (Free)
   - Character concept art
   - Environment concepts
   - UI icon generation

2. **Stable Diffusion** (Free, local)
   - Tile textures
   - Background art
   - Character portraits

3. **Midjourney** (Free tier)
   - Concept art
   - Promotional images

**Usage Strategy:**
- Generate concept art for reference
- Simplify AI output to 16x16 or 32x32 sprites
- Use as texture source for procedural generation
- Create sprite sheets from individual generations

### Placeholder vs Final Asset Strategy

| Asset Type | Phase 1 (Placeholder) | Phase 2+ (Final) |
|------------|----------------------|------------------|
| Character Sprites | Colored rectangles | Pixel art or vector |
| Enemy Sprites | Geometric shapes | Unique designs |
| Tilesets | Procedural solid colors | Hand-crafted or AI-assisted |
| UI | CSS-styled | Custom graphics |
| VFX | Particle systems (keep) | Enhanced particles + shaders |
| Icons | Font icons | Custom SVG |
| Audio | Web Audio API synth | Recorded/licensed SFX |
| Music | None or CC0 | Original or licensed |

### Asset Milestones

**M1-M2 (Week 1-4):**
- All placeholder assets in place
- No manual art creation required

**M3-M5 (Week 5-10):**
- Procedural generation refined
- Color palette finalized
- Visual consistency established

**M6 (Week 11-12):**
- Particle effects polished
- UI styling improved
- "Good enough" visual bar reached

**M7-M8 (Week 13-16):**
- Asset optimization (file sizes)
- Visual bugs fixed
- Accessibility improvements (contrast, size)

### Asset Storage and Organization

```
/assets
├── /generated        # Code-generated at runtime (empty folder)
├── /sprites
│   ├── placeholder_character.png  # Single 64x64 colored square
│   └── placeholder_enemy.png      # Single 64x64 shape
├── /tilesets
│   └── basic_tiles.png            # Procedural tileset (optional)
├── /maps
│   └── city_district_01.json     # Tiled JSON export
├── /audio
│   └── (empty - use Web Audio API)
└── /ui
    ├── icons.svg                  # Font Awesome or Material Icons
    └── styles.css                 # UI styling
```

**Total Asset Size Target:** <5 MB for entire Phase 1

---

## Risk Assessment

### **UPDATED: Infrastructure Risks (Pragmatic Approach)**

#### **NEW Risk 0: Technical Debt from Minimal Infrastructure**

**Likelihood:** Medium (50%)
**Impact:** Medium
**Description:** Deferring validation and schemas may lead to data quality issues

**Mitigation Strategy:**
- Add validation incrementally (Week 7-8 for stable systems)
- Manual review of new content (team is small)
- Write basic data structure tests early
- Define schemas as systems stabilize (lazy schema definition)
- Monitor "pain points" - add infrastructure when refresh cycle becomes annoying

**UPDATED Contingency:**
- If bugs from bad data exceed 5% of total bugs: Add validation immediately
- If typos are frequent: Add schema validation for that category
- If refresh is painful (>10 times/hour): Add hot-reload

**Monitoring Metrics:**
- Track % of bugs from malformed data (target: <5%)
- Track content iteration time (target: <1 hour per change)
- Track infrastructure complexity (target: <4/10 in Month 1)

**Success Indicator:** By Week 12, we have validation for critical systems without slowing initial development

---

#### **NEW Risk 0b: Premature Optimization (Inverse Risk)**

**Likelihood:** Low (avoided by pragmatic approach)
**Impact:** High (if it happens)
**Description:** Building too much infrastructure upfront slows gameplay development

**Mitigation Strategy (Already Applied):**
- Start with minimal infrastructure (3-5 days vs. 2 weeks)
- Build infrastructure as needed, not "just in case"
- Use decision points (Week 7, Week 11) to evaluate infrastructure needs
- Apply YAGNI principle (You Aren't Gonna Need It)

**Success Indicator:** Gameplay development starts Week 1 Day 6, not Week 3

---

### Technical Risks

#### Risk 1: Performance Issues with Particle Systems

**Likelihood:** Medium
**Impact:** High
**Mitigation:**
- Implement object pooling for particles (Week 11)
- Limit max concurrent particles (200-300)
- Provide graphics quality settings
- Profile early and often (Chrome DevTools)
- Fall back to simpler effects if needed

**Contingency:** Remove non-essential particle effects, use flashes instead

---

#### Risk 2: Browser Compatibility Problems

**Likelihood:** Medium
**Impact:** Medium
**Mitigation:**
- Use Phaser 3 (handles most compatibility)
- Test in Chrome, Firefox, Safari, Edge regularly
- Use Babel for ES6+ transpilation
- Avoid experimental web APIs
- Provide fallbacks (WebGL → Canvas 2D)

**Contingency:** Document minimum browser versions, focus on Chrome/Firefox

---

#### Risk 3: Save System Corruption

**Likelihood:** Low
**Impact:** Very High
**Mitigation:**
- Implement save versioning
- Validate save data on load
- Keep multiple save slots (3+)
- Auto-save frequently
- Test save/load extensively

**Contingency:** Reset save feature, export/import save as JSON

---

#### Risk 4: State Management Complexity

**Likelihood:** High
**Impact:** Medium
**Mitigation:**
- Use established state management (Redux/MobX)
- Document state structure clearly
- Centralize state mutations
- Add state validation
- Use TypeScript for type safety
- **UPDATED:** Keep data structures flexible early (tighten gradually)

**Contingency:** Simplify state structure, reduce inter-dependencies

---

### Scope Creep Prevention

#### Strategy 1: Fixed Milestone Deadlines
- Each milestone has a hard 1-2 week deadline
- If behind schedule, cut features, don't extend timeline
- Track "nice-to-have" features separately for Phase 2

#### Strategy 2: Feature Freeze Dates
- **Week 12 (M6 complete):** No new features, polish only
- **Week 14 (M7 complete):** Content complete, balance only
- **Week 15-16 (M8):** Bug fixes only, no new content

#### Strategy 3: Scope Document
Maintain a "Phase 1 Scope" document that lists:
- ✅ In Scope (must have for vertical slice)
- ⏸️ Phase 2 (nice to have, defer)
- ❌ Out of Scope (explicitly excluded)

Review scope weekly, move features to Phase 2 if needed.

#### Strategy 4: "One In, Two Out" Rule
If adding a new feature to Phase 1:
- Must remove two existing features, OR
- Must provide justification why it's critical for vertical slice

---

### Critical Path Identification

**Critical Path:** Features that block other work or are essential for playable vertical slice

```
Critical Path (Must Complete in Order):
1. [M1] Project setup → All subsequent work
2. [M2] Combat system → Character abilities, game juice, balance
3. [M3] Character classes → Combat balance, progression
4. [M4] Exploration → Quests, dialogue, integration
5. [M5] World map → Full integration, playable loop
6. [M7] Content creation → Testing
7. [M8] Testing → Launch

Parallel Paths (Can work simultaneously):
- [M6] Game juice (while content is being created)
- UI development (throughout all milestones)
- Asset generation (throughout all milestones)
```

**Risk Mitigation:**
- Front-load critical path tasks
- Identify dependencies early
- Don't start dependent tasks until prerequisites are complete
- If critical path is blocked, work on parallel paths

---

### Resource Risks

#### Risk: Solo Developer Burnout

**Likelihood:** High (for solo dev)
**Impact:** Very High
**Mitigation:**
- Realistic 15-week timeline (not 8-week)
- Schedule breaks every 4 weeks
- Set daily work hour limits (4-6 hours max)
- Celebrate milestone completions
- Join gamedev communities for support

**Contingency:** Extend timeline, reduce scope, seek collaboration

---

#### Risk: Lack of Specific Skills (Art, Audio)

**Likelihood:** High
**Impact:** Medium
**Mitigation:**
- Rely on procedural generation (see Asset Plan)
- Use free asset libraries (OpenGameArt, Kenney.nl)
- Focus on gameplay over aesthetics
- Plan art upgrade for Phase 2+

**Contingency:** Accept "programmer art," emphasize mechanics

---

## Testing Strategy

### Playtesting Schedule

#### Internal Testing (Developer)

**Daily Testing (Week 1-16):**
- Test new features as implemented
- Quick smoke tests after each commit
- Verify no regressions

**Weekly Testing (Week 1-16):**
- Full playthrough of implemented content (30-60 min)
- Test all systems in integration
- Document bugs in tracker

**Milestone Testing (End of each milestone):**
- Complete playthrough with checklist
- Verify all milestone deliverables
- Acceptance criteria validation

#### External Playtesting

**Week 10 (M5 complete):**
- First external playtest (1-2 testers)
- Focus: Can they navigate the game loop?
- Collect feedback on UX, confusion points

**Week 14 (M7 complete):**
- Second external playtest (3-5 testers)
- Focus: Balance, fun factor, content quality
- Formal feedback form

**Week 16 (M8 complete):**
- Final playtest (5-10 testers)
- Focus: Bug discovery, polish issues
- Collect quotes for marketing (optional)

### Balance Testing Approach

#### Metrics to Track

1. **Combat Metrics**
   - Average battle duration (target: 2-4 minutes)
   - Player death rate (target: 10-20%)
   - Ability usage distribution (no single ability >40% usage)
   - MP usage per battle (target: 30-50%)
   - Healing item usage (target: 1-2 per 3 battles)

2. **Progression Metrics**
   - Time to reach level 5 (target: 2-3 hours)
   - XP gain rate
   - Gold accumulation
   - Equipment purchase rate

3. **Engagement Metrics**
   - Quest completion rate (target: >80% for main quests)
   - Exploration coverage (% of city visited)
   - Dialogue skip rate (if high, dialogue too long)
   - Drop-off points (where players quit)

#### Data Collection Methods

**Manual Tracking:**
- Developer notes during playthroughs
- Tester feedback forms
- Screen recordings (OBS)

**Automated Tracking (Optional):**
- Analytics integration (e.g., Google Analytics events)
- Combat log export to CSV
- State snapshots on key events (level-up, quest complete)

#### Balance Adjustment Process

1. **Identify Issue:** Metric outside target range or player feedback
2. **Hypothesize Cause:** "Battles too long because HP too high"
3. **Make Adjustment:** Reduce enemy HP by 20%
4. **Test:** Playtest 3-5 battles
5. **Iterate:** Adjust further or revert if worse

**Balance Knobs:**
- Enemy HP, Attack, Defense
- Player stat growth rates
- XP rewards
- Ability costs and cooldowns
- Item prices
- Drop rates

### Bug Tracking Workflow

#### Tools
- **GitHub Issues** (free, integrated with repo)
- **Trello** (simple kanban board)
- **Excel/Google Sheets** (simplest)

#### Bug Report Template

```markdown
**Title:** Brief description

**Priority:** P0 / P1 / P2 / P3 (see M8 definitions)

**Severity:** Critical / Major / Minor

**Steps to Reproduce:**
1. Step 1
2. Step 2
3. Step 3

**Expected Result:** What should happen

**Actual Result:** What actually happens

**Screenshot/Video:** (if applicable)

**Environment:**
- Browser: Chrome 120
- OS: Windows 11
- Build: v0.5.0

**Additional Notes:** Any other relevant info
```

#### Bug Workflow

1. **Report:** Tester/developer files bug
2. **Triage:** Review, assign priority
3. **Assign:** Assign to developer (or self for solo)
4. **Fix:** Implement fix, commit with bug ID
5. **Verify:** Tester verifies fix
6. **Close:** Mark as resolved

#### Bug Fix Priorities

**Week 1-12 (Development):**
- Fix P0 bugs immediately
- Fix P1 bugs within 1 week
- Defer P2/P3 to later

**Week 13-14 (Content/Balance):**
- Fix P0 and P1 bugs
- Defer P2 to M8

**Week 15-16 (Testing):**
- Fix all P0 bugs
- Fix all P1 bugs
- Fix P2 bugs if time allows
- Document P3 bugs, defer to Phase 2

---

## Phase 2 Preview

### What Comes After Phase 1

Phase 1 delivers a **vertical slice**: all systems working, one city playable, demonstrating the core loop. Phase 2 expands **horizontally**: more cities, more content, more depth.

### Phase 2 Goals (Weeks 17-32, ~4 months)

1. **Add 2-3 More Cities**
   - Distinct environments (industrial, residential, agricultural)
   - Unique NPCs and storylines
   - City-specific mechanics or challenges

2. **Expand Combat**
   - 3-4 more character classes
   - Enemy type variety (15-20 total enemy types)
   - Boss battles (3-5 major bosses)
   - Advanced combat mechanics (combos, formations, weather)

3. **Deepen Progression**
   - Level cap increase (5 → 20)
   - Advanced abilities (tier 2, tier 3 skills)
   - Specialization/talent trees
   - Prestige/meta-progression (Hades-style)

4. **Enhance Story**
   - Main storyline across all cities
   - Character backstories and development
   - Branching narrative paths
   - Multiple endings

5. **Polish and Content**
   - Upgrade art assets (pixel art or commissioned)
   - Original music and sound effects
   - Cutscenes or scripted events
   - More quests, NPCs, dialogue

6. **Quality of Life**
   - Tutorial improvements
   - Difficulty settings
   - Accessibility features
   - UI/UX refinements based on feedback

### Expansion Strategy for Additional Cities

#### City Design Template

Each new city should have:
- **Unique Theme:** Visual and narrative identity
- **3-5 Story Quests:** Tie into main narrative
- **5-10 Side Quests:** Optional content
- **10-15 NPCs:** With unique dialogue
- **2-3 Boss Encounters:** Memorable battles
- **City-Specific Mechanic:** Something unique (e.g., time-based events, environmental hazards)

#### City Rollout Plan

**Phase 2.1 (Weeks 17-22): City 2 - Industrial District**
- Theme: Factory automation, labor disputes
- Mechanic: Environmental hazards in combat (toxic waste)
- Boss: Rogue Manufacturing AI

**Phase 2.2 (Weeks 23-28): City 3 - Residential Haven**
- Theme: Community, social networks
- Mechanic: Relationship/influence system
- Boss: Corrupted Council Member

**Phase 2.3 (Weeks 29-32): City 4 - Agricultural Expanse**
- Theme: Nature, sustainability
- Mechanic: Seasonal changes, crop management
- Boss: Mutated Bio-Experiment

### Feature Roadmap Beyond Vertical Slice

**Phase 2 Features:**
- Crafting system (combine items to create equipment)
- Character customization (visual appearance)
- Party camp/hub upgrades (Fire Emblem-style)
- Advanced dialogue (romance, rivalry)
- New game+ mode
- Challenge modes (arena, time trials)

**Phase 3+ Features (Long-term):**
- Procedurally generated content (dungeons, quests)
- Multiplayer/co-op battles
- Mobile version (touch controls)
- DLC/expansion content
- Modding support (expose data files)

### Metrics for Success (Phase 1 → Phase 2 Transition)

**Go/No-Go Criteria for Phase 2:**
- ✅ Phase 1 vertical slice is complete and playable
- ✅ Feedback from playtesters is positive (>7/10 average)
- ✅ No critical bugs remaining
- ✅ Developer is not burned out
- ✅ Clear vision for Phase 2 content

**If criteria not met:**
- Polish Phase 1 further before expanding
- Release as "demo" or "prologue"
- Gather wider feedback before committing to Phase 2

---

## Appendix

### A. Recommended Tools and Libraries

#### Core Development
- **Phaser 3** (v3.80+): Game engine
- **TypeScript**: Type safety, better IDE support
- **Vite**: Fast build tool and dev server
- **Redux Toolkit** or **MobX**: State management

#### Asset Creation
- **Tiled Map Editor**: Map creation
- **Aseprite** (paid) or **Piskel** (free): Pixel art
- **Inkscape**: Vector graphics (SVG icons)
- **Audacity**: Audio editing (if not using Web Audio API)

#### Development Tools
- **VS Code**: Code editor
- **Git**: Version control
- **GitHub**: Hosting, CI/CD
- **Chrome DevTools**: Debugging, profiling

#### Testing and Deployment
- **GitHub Actions**: CI/CD
- **GitHub Pages** or **Netlify**: Hosting
- **BrowserStack** (optional): Cross-browser testing

#### Libraries
- **EasyStar.js**: Pathfinding
- **Yarn Spinner** (optional): Advanced dialogue
- **Howler.js**: Audio management (if not using Phaser's)

---

### B. Key Resources

**Phaser 3:**
- Official Docs: https://photonstorm.github.io/phaser3-docs/
- Examples: https://phaser.io/examples
- Community: https://phaser.discourse.group/

**Game Design:**
- GDC Talks: https://www.gdcvault.com/
- Game Maker's Toolkit: https://www.youtube.com/c/MarkBrownGMT
- Extra Credits: https://www.youtube.com/extracredits

**Solarpunk Inspiration:**
- r/solarpunk: https://www.reddit.com/r/solarpunk/
- Solarpunk Manifesto: Research existing works

**RPG Design:**
- D&D 5E SRD: https://dnd.wizards.com/resources/systems-reference-document
- Fire Emblem Wiki: Character and mechanics references
- Final Fantasy Wiki: ATB and battle system references

---

### C. Phase 1 Scope Checklist

Use this checklist to verify Phase 1 completion:

**World Map:**
- [ ] 1 unlocked city node
- [ ] 2-3 locked city nodes (visual)
- [ ] Node selection and transition

**City Exploration:**
- [ ] 15-20 explorable locations
- [ ] 5-7 NPCs with dialogue
- [ ] 3-5 skill check encounters
- [ ] 2 main quests, 5 side quests
- [ ] 1 shop/vendor

**Combat:**
- [ ] Turn-based battle system
- [ ] 3-4 playable characters
- [ ] 5-7 enemy types
- [ ] 15-20 abilities total
- [ ] Status effects
- [ ] Victory/defeat conditions

**Progression:**
- [ ] Level 1-5 system
- [ ] Stat growth
- [ ] Ability unlocks
- [ ] Equipment (3 slots)
- [ ] 8-12 equippable items

**Core Loop:**
- [ ] Can complete full gameplay loop
- [ ] Save/load works
- [ ] Inventory management
- [ ] Party composition

**Polish:**
- [ ] Visual effects (particles, shake, flash)
- [ ] Damage numbers
- [ ] Animations
- [ ] Sound effects (placeholders)

**Technical:**
- [ ] 60 FPS performance
- [ ] Load times <3 seconds
- [ ] No critical bugs
- [ ] Deployed and playable

---

### D. Contact and Collaboration

For solo developers looking to collaborate or get feedback:

**Communities:**
- r/gamedev: General game development
- r/IndieDev: Indie game development
- r/devblogs: Share progress
- Phaser Discord: https://discord.gg/phaser

**Feedback:**
- Screenshot Saturday (Twitter: #ScreenshotSaturday)
- Feedback Friday (r/gamedev)
- Playtesting subreddits: r/playmygame

---

## **UPDATED: Infrastructure Development Timeline**

### Pragmatic Approach Summary

This roadmap uses an **incremental infrastructure strategy** instead of building everything upfront.

**Philosophy:** Build patterns, not implementations. Add complexity only when needed.

### Infrastructure Phases

#### Phase 0: Essential Foundation (Week 1, Days 1-5)
**Time Investment:** 3-5 days
**Complexity Level:** 2/10 (minimal)

✅ **Build:**
- DataLoader class (fetch + cache)
- File structure conventions
- Basic TypeScript types
- Example data files

❌ **Defer:**
- JSON schemas
- Validation pipeline
- Hot-reload
- Content authoring tools

**Success Metric:** Gameplay development starts Day 6

---

#### Phase 1: Expand as Needed (Weeks 2-10)
**Time Investment:** Parallel with gameplay (1-2 hours per week)
**Complexity Level:** 3-4/10 (still simple)

✅ **Add incrementally:**
- Week 3-4: Define ability schema (combat system needs it)
- Week 5-6: Define item schema (progression system needs it)
- Week 7-8: Define quest schema (exploration system needs it)
- Week 7-8: **Decision Point:** Add validation if data errors are frequent

❌ **Still deferred:**
- Hot-reload (unless painful)
- Build-time validation
- Advanced tooling

**Success Metric:** Infrastructure doesn't block gameplay

---

#### Phase 2: Polish Infrastructure (Weeks 11-12)
**Time Investment:** 4-8 hours (if needed)
**Complexity Level:** 5-6/10 (moderate)

✅ **Evaluate and add:**
- Week 11: **Decision Point:** Add hot-reload if refresh cycle is painful
- Week 11-12: Add validation for all critical systems
- Week 12: Schema documentation for all data types

**Success Metric:** Infrastructure enables efficient content creation

---

#### Phase 3: Advanced Features (Phase 2+, Post-Week 16)
**Time Investment:** Future work
**Complexity Level:** 7-8/10 (advanced)

✅ **Phase 2+ features:**
- Build-time validation in CI/CD
- Content authoring tools
- Hot-reload for all assets (not just JSON)
- Localization support
- Asset optimization pipeline

**Success Metric:** Professional-grade infrastructure for larger team

---

### Decision Points Checklist

Copy this checklist and evaluate at each milestone:

#### Week 7-8 Decision: Add Validation?
- [ ] We have 10+ JSON data files
- [ ] Typos/malformed data causing >5% of bugs
- [ ] Content being added by non-programmers
- [ ] Manual testing is missing errors
- **If 2+ checked:** Add validation (2-4 hours investment)
- **If not:** Continue manual testing, revisit Week 11

#### Week 11 Decision: Add Hot-Reload?
- [ ] Making 10+ content changes per hour
- [ ] 30-second refresh cycle is painful
- [ ] Multiple people editing content
- [ ] Testing flow interrupted by refresh
- **If 2+ checked:** Add hot-reload (4-6 hours investment)
- **If not:** Defer to Phase 2

#### Week 11 Decision: Add Asset Pipeline Tools?
- [ ] Asset iteration time >30 minutes
- [ ] Manual asset optimization needed
- [ ] Multiple asset formats needed
- [ ] Asset naming/organization issues
- **If 2+ checked:** Add asset tools
- **If not:** Defer to Phase 2

---

### Monitoring Metrics

Track these weekly to ensure infrastructure stays pragmatic:

**Complexity Metrics:**
- Infrastructure code lines: Target <500 (M1), <1000 (M8)
- Subjective complexity rating: Target <4/10 (Month 1), <6/10 (Month 3)
- Time to add infrastructure: Target <1 day per feature

**Quality Metrics:**
- % bugs from malformed data: Target <5%
- Content iteration time: Target <1 hour
- Data loading time: Target <1 second

**Velocity Metrics:**
- Gameplay features blocked by infrastructure: Target 0
- Infrastructure refactoring time: Target <1 day/month
- Time saved by infrastructure: Track tangible benefits

**Warning Signs (act immediately if seen):**
- 🚨 >10% of bugs from bad data → Add validation
- 🚨 >2 hours to add new content → Simplify structure
- 🚨 Gameplay blocked waiting for infrastructure → Skip that infrastructure
- 🚨 Infrastructure complexity >7/10 → Remove features, simplify

---

### Comparison: Original vs. Pragmatic Approach

| Aspect | Original Plan | Architect Proposal | **Pragmatic (This Doc)** |
|--------|--------------|-------------------|------------------------|
| **Week 1 Focus** | Project setup | 2-week infrastructure | **5 days infrastructure + 5 days setup** |
| **Validation** | Not mentioned | Week 1-2 (complete) | **Week 7-8 (if needed)** |
| **Hot-Reload** | Not mentioned | Week 1-2 | **Week 11 (if needed)** |
| **Schemas** | Not mentioned | Week 1-2 (all) | **Define when building system** |
| **First Gameplay** | Week 1 | Week 3 | **Week 1 Day 6** |
| **Infrastructure Complexity** | Low | High | **Medium (expandable)** |
| **Total Timeline** | 15 weeks | 17 weeks | **15 weeks** |
| **Refactoring Risk** | Medium | Low | **Medium** |
| **Learning Waste** | Low | High | **Low** |

**Key Difference:** Infrastructure enhances gameplay (parallel), doesn't block it (sequential)

---

## Document Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-11-01 | Initial roadmap created |
| 2.0 | 2025-11-01 | **UPDATED:** Added pragmatic infrastructure approach - 3-5 day minimal foundation, incremental expansion, decision points at Week 7 and Week 11, updated M1 with parallel tracks, added infrastructure risks and metrics |

---

**End of Phase 1 Development Roadmap**

This roadmap is a living document. Adjust timelines, scope, and priorities as development progresses. The goal is a playable, polished vertical slice that demonstrates the game's potential.

**Infrastructure Philosophy:** Start minimal (3-5 days), expand as needed (decision points), maintain low complexity (<6/10), enable fast iteration (<1 hour per change), follow YAGNI principle (You Aren't Gonna Need It).

Good luck, and happy developing!
