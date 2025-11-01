# Asset Manager Tool Specification
## Web-Based RPG Content Management System

**Project:** Hybrid Turn-Based RPG (Solarpunk Setting)
**Version:** 1.0
**Last Updated:** 2025-11-01
**Target Platform:** Web Application (Local Development Tool)
**Integration:** Phaser 3 + TypeScript Game Project

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Tool Overview](#2-tool-overview)
3. [Feature Specification](#3-feature-specification)
4. [Technical Architecture](#4-technical-architecture)
5. [UI/UX Design](#5-uiux-design)
6. [Implementation Plan](#6-implementation-plan)
7. [Code Examples](#7-code-examples)
8. [Integration Strategy](#8-integration-strategy)
9. [Appendix](#9-appendix)

---

## 1. Executive Summary

### 1.1 Problem Statement

**Current Challenge:**
- Game content (characters, abilities, items, quests, enemies) stored in JSON files
- Manual JSON editing is error-prone, slow, and requires technical knowledge
- No validation until runtime (discovering errors during gameplay)
- No preview of how content will look/behave in-game
- Difficult for non-programmers to contribute content
- Balancing requires iterative testing cycles

**Impact on Development:**
- Content creation bottleneck (50%+ of Phase 1 time is content)
- High risk of JSON syntax errors
- Slow iteration on game balance
- Cannot effectively delegate content work

### 1.2 Solution Overview

**Asset Manager: A Web-Based Content Management Tool**

A lightweight, browser-based application that provides:
- **Visual Editing:** Form-based editors for all content types
- **Real-Time Validation:** Catch errors before saving
- **Live Preview:** See how content looks/works in-game
- **Search & Filter:** Quickly find and organize content
- **Version Control Integration:** Git-aware, no conflicts
- **Zero Configuration:** Reads/writes directly to game's JSON files

**Key Benefits:**
- **50% faster content creation** (forms vs manual JSON editing)
- **95% reduction in syntax errors** (validated inputs)
- **Enables collaboration** (designers/writers can contribute)
- **Faster iteration** (preview without launching game)
- **Better balance** (see all stats in one view)

### 1.3 Recommendation

**Approach:** **React-based Web Application (Local Dev Tool)**

**Rationale:**
1. **Quick to Build:** React + TypeScript + Vite = 2-3 weeks for MVP
2. **Zero-Touch Workflow:** Runs locally, reads/writes game JSON files
3. **Familiar Tech Stack:** Same as game (TypeScript), easy context switching
4. **Hot Reload Integration:** Can trigger game reload on save
5. **No Deployment Overhead:** Local tool, no hosting required
6. **Future-Proof:** Can add advanced features (AI suggestions, analytics) later

**Alternative Considered:** In-game editor (integrated into Phaser)
- **Rejected:** More complex, couples editor to game, harder to maintain

---

## 2. Tool Overview

### 2.1 Purpose and Use Cases

#### Primary Purpose
Provide a user-friendly interface for creating, editing, and managing game content without directly editing JSON files.

#### Core Use Cases

**Use Case 1: Create New Character Class**
- **Actor:** Game Designer
- **Goal:** Add a new playable character class (e.g., "Eco-Warrior")
- **Steps:**
  1. Open Asset Manager → Characters tab
  2. Click "New Character"
  3. Fill form: name, stats, abilities, visual data
  4. Preview character sprite (procedurally generated)
  5. Validate stats against balance guidelines
  6. Save → JSON file updated, git diff shows changes
- **Outcome:** New character available in game, no JSON editing required

**Use Case 2: Balance Pass on Enemies**
- **Actor:** Game Designer
- **Goal:** Adjust enemy stats across all enemies in Act 1
- **Steps:**
  1. Open Asset Manager → Enemies tab
  2. Filter by "Act 1" tag
  3. View all enemies in table (sortable by HP, Attack, etc.)
  4. Bulk edit: increase HP by 20%
  5. Preview damage calculations with current player stats
  6. Save changes
- **Outcome:** Consistent balance adjustment, see impact before testing

**Use Case 3: Writer Adds Quest Dialogue**
- **Actor:** Writer (non-technical)
- **Goal:** Write dialogue for new side quest
- **Steps:**
  1. Open Asset Manager → Quests tab → Select quest
  2. Click "Edit Dialogue"
  3. Visual dialogue tree editor (node-based)
  4. Add dialogue nodes, choices, skill checks
  5. Preview dialogue flow with branching paths
  6. Save → quest JSON updated
- **Outcome:** Writer can contribute without learning JSON syntax

**Use Case 4: Debug Runtime Error**
- **Actor:** Developer
- **Goal:** Fix broken ability reference causing crash
- **Steps:**
  1. Game crashes: "Ability 'fireball_v2' not found"
  2. Open Asset Manager → Search for "fireball"
  3. See "fireball_v2" has typo, should be "fireball_lv2"
  4. Fix typo in character's ability list
  5. Validation shows error cleared
  6. Save, game hot-reloads
- **Outcome:** Error found and fixed in 30 seconds vs 10 minutes

**Use Case 5: Preview Item Before Playtesting**
- **Actor:** Game Designer
- **Goal:** See how new weapon looks and affects stats
- **Steps:**
  1. Open Asset Manager → Items tab → Select weapon
  2. Edit stats (e.g., +15 Attack, +5% Crit)
  3. Click "Preview on Character"
  4. See character stats with/without item (side-by-side)
  5. Adjust stats, preview updates in real-time
  6. Save when satisfied
- **Outcome:** Balanced item without launching game

### 2.2 User Personas

#### Persona 1: Solo Developer (You)
- **Role:** Developer, Designer, Content Creator
- **Technical Skill:** High (TypeScript, JSON, Git)
- **Goals:**
  - Quickly create and iterate on content
  - Reduce context switching (code ↔ JSON editing)
  - Catch errors early
  - Maintain productivity during long dev sessions
- **Pain Points:**
  - Manual JSON editing is tedious
  - Easy to make typos in ability IDs, references
  - Hard to see "big picture" of game balance
  - No quick way to preview content changes

#### Persona 2: Designer (Potential Collaborator)
- **Role:** Game Designer, Balance Specialist
- **Technical Skill:** Medium (can read code, uncomfortable editing)
- **Goals:**
  - Design character classes, abilities, enemies
  - Balance combat encounters
  - Create quest structures
  - Iterate quickly on feedback
- **Pain Points:**
  - JSON syntax is intimidating
  - No visual feedback until playtesting
  - Hard to compare stats across entities
  - Fears breaking something with typo

#### Persona 3: Writer (Potential Collaborator)
- **Role:** Quest Writer, Dialogue Author
- **Technical Skill:** Low (no coding background)
- **Goals:**
  - Write engaging dialogue
  - Create branching quest narratives
  - Add flavor text for items/abilities
  - See dialogue in context (with choices)
- **Pain Points:**
  - JSON structure is confusing
  - No way to visualize dialogue trees
  - Can't preview how dialogue looks in-game
  - Needs developer to input dialogue

### 2.3 Core Workflows

#### Workflow 1: Browse Content
```
Start → Open Asset Manager
     → Select Content Type (Characters / Abilities / Items / etc.)
     → View List (table or card view)
     → Sort / Filter / Search
     → Select Item → View Details
```

#### Workflow 2: Edit Existing Content
```
Browse → Select Item
      → Click "Edit"
      → Form Editor Opens
      → Modify Fields
      → Real-Time Validation (errors highlighted)
      → Preview (optional, see changes)
      → Save (write to JSON)
      → Git diff shows changes
      → (Optional) Trigger game hot-reload
```

#### Workflow 3: Create New Content
```
Browse → Click "New [Type]"
      → Template Selection (optional: clone existing or start blank)
      → Fill Form (guided with tooltips)
      → Validation (ensure required fields, references valid)
      → Preview
      → Save
      → New item appears in list
```

#### Workflow 4: Validate All Content
```
Main Menu → Click "Validate All"
         → Scan all JSON files
         → Check for:
            - Syntax errors
            - Missing required fields
            - Broken references (ability IDs, item IDs, etc.)
            - Balance issues (stats out of range)
         → Display Report (errors, warnings, suggestions)
         → Click error → Jump to item and fix
         → Re-validate → All clear
```

#### Workflow 5: Preview Content
```
Edit Item → Click "Preview"
         → Preview Mode (depends on type):
            - Character: Show sprite, stats, ability list
            - Ability: Show animation, damage calculation
            - Item: Show sprite, stat modifiers
            - Enemy: Show sprite, AI behavior summary
            - Quest: Show dialogue tree visualization
         → Interact with preview (test dialogue choices)
         → Return to edit or Save
```

---

## 3. Feature Specification

### 3.1 Content Browser (Priority: P0 - Critical)

**Description:** View all game content organized by type

**Features:**
- **Content Type Tabs:** Characters, Abilities, Items, Enemies, Quests, Locations
- **List View:**
  - Table view (default): Columns for key fields (name, level, stats)
  - Card view (optional): Visual cards with icons/sprites
- **Search:** Full-text search across all fields
- **Filters:**
  - By tag (e.g., "Act 1", "Technomancer", "Fire Element")
  - By stat range (e.g., HP 50-100, Level 3-5)
  - By status (Complete, WIP, Needs Review)
- **Sorting:** Sort by any column (name, level, HP, date modified)
- **Quick Actions:** Edit, Duplicate, Delete, Export (per item)
- **Bulk Actions:** Select multiple → Delete, Tag, Export

**Acceptance Criteria:**
- [ ] Can view all characters in table format
- [ ] Search finds items by name or description
- [ ] Filter by level shows correct subset
- [ ] Sorting by HP orders correctly
- [ ] Can delete item and it updates JSON

**Time Estimate:** 2-3 days

---

### 3.2 Form-Based Editors (Priority: P0 - Critical)

**Description:** Structured forms for editing each content type

#### 3.2.1 Character Editor

**Fields:**
- **Basic Info:**
  - ID (read-only, auto-generated from name)
  - Name (text input)
  - Class (dropdown: Technomancer, Guardian, Bioengineer, etc.)
  - Description (textarea)
  - Tags (multi-select chips)
- **Base Stats:**
  - HP, MP, Attack, Defense, Magic, Speed (number inputs with sliders)
  - Visual indicators: color-coded by balance guideline (red if too high, green if balanced)
- **Stat Growth:**
  - HP/level, MP/level, etc. (number inputs)
  - Preview: "At level 5, HP will be X"
- **Abilities:**
  - List of ability IDs (searchable dropdown with autocomplete)
  - Add/remove abilities
  - Set unlock levels
- **Equipment:**
  - Allowed equipment types (checkboxes: Light Armor, Heavy Armor, etc.)
  - Default equipment (dropdown)
- **Visual Data:**
  - Sprite seed (text input, generates deterministic sprite)
  - Color palette (color pickers: primary, secondary, accent)
  - Body type (dropdown: slim, average, bulky)
  - Preview sprite (live preview as you edit)

**Validation:**
- Required fields: ID, Name, Class
- HP/MP must be > 0
- Stat growth must be >= 0
- Ability IDs must exist in abilities.json
- Equipment IDs must exist in items.json

**Layout:**
```
┌─────────────────────────────────────────────────────────┐
│ Character Editor: [Aria Solstice]                 [X]  │
├─────────────────────────────────────────────────────────┤
│ ┌─────────────────┐ ┌─────────────────────────────────┐ │
│ │  Preview        │ │  Basic Info                     │ │
│ │  [Sprite]       │ │  Name: [Aria Solstice        ]  │ │
│ │                 │ │  Class: [Technomancer ▼]       │ │
│ │  [Regenerate]   │ │  Description:                   │ │
│ └─────────────────┘ │  [Textarea...              ]    │ │
│                     │  Tags: [Act1][Protagonist]      │ │
│                     └─────────────────────────────────┘ │
│                                                         │
│ Base Stats                  Stat Growth                │
│ ┌──────────┬──────┐        ┌──────────┬──────┐        │
│ │ HP       │  100 │░░░░░░  │ HP/level │   10 │        │
│ │ MP       │   80 │░░░░░   │ MP/level │    8 │        │
│ │ Attack   │   12 │░░      │ Attack   │    2 │        │
│ │ Defense  │    8 │░░      │ Defense  │    1 │        │
│ │ Magic    │   20 │░░░░░░░ │ Magic    │    4 │        │
│ │ Speed    │   14 │░░░     │ Speed    │    2 │        │
│ └──────────┴──────┘        └──────────┴──────┘        │
│                                                         │
│ Abilities                                               │
│ [Lightning Strike] [Energy Shield] [Overcharge]        │
│ [+ Add Ability]                                         │
│                                                         │
│                     [Cancel] [Save Character]           │
└─────────────────────────────────────────────────────────┘
```

**Time Estimate:** 3-4 days

#### 3.2.2 Ability Editor

**Fields:**
- **Basic Info:** ID, Name, Description, Icon
- **Mechanics:**
  - Type (Attack, Support, Heal, Buff, Debuff)
  - Target Type (Single, Multi, Area, Self, Ally)
  - MP Cost (number)
  - Cooldown (number, turns)
- **Effects:**
  - List of effects (add/remove)
  - Each effect: Type (Damage, Heal, Buff, etc.), Value, Duration
  - Damage type (Physical, Magic, Fire, Ice, etc.)
  - Formula (dropdown: Fixed, Stat-based, Level-scaled)
- **Animation Data:**
  - Effect color (color picker)
  - Particle type (dropdown: Sparkles, Lightning, Fire, etc.)
  - Sound effect (dropdown or upload)
- **Balance Info:**
  - DPR (Damage Per Round) calculator
  - Comparison to similar abilities

**Validation:**
- MP cost must be > 0
- At least one effect
- If damage ability, must have damage value

**Time Estimate:** 2-3 days

#### 3.2.3 Item Editor

**Fields:**
- **Basic Info:** ID, Name, Description, Icon
- **Type:** Weapon, Armor, Accessory, Consumable
- **Stats:**
  - Stat modifiers (HP, Attack, Defense, etc.)
  - Special effects (e.g., "+5% Crit Chance")
- **Restrictions:**
  - Allowed classes (multi-select)
  - Required level (number)
- **Economy:**
  - Buy price (number)
  - Sell price (number, auto-calculated as 50% buy price)
  - Rarity (dropdown: Common, Uncommon, Rare, Epic)
- **Visual:**
  - Sprite generation params (color, shape)
  - Icon preview

**Time Estimate:** 2 days

#### 3.2.4 Enemy Editor

**Fields:**
- **Basic Info:** ID, Name, Description, Tags
- **Stats:** HP, Attack, Defense, Magic, Speed
- **Abilities:** List of ability IDs
- **AI Behavior:**
  - AI Type (dropdown: Aggressive, Defensive, Support, Random)
  - Behavior rules (priority list)
- **Drops:**
  - XP reward (number)
  - Gold reward (number)
  - Item drops (list of item IDs + drop rates)
- **Visual:** Sprite params, color palette

**Time Estimate:** 2-3 days

#### 3.2.5 Quest Editor

**Features:**
- **Basic Info:** ID, Title, Description, Tags
- **Prerequisites:** Required quests, level, items
- **Objectives:**
  - List of objectives (add/remove/reorder)
  - Each objective: Type (Talk, Collect, Defeat, Explore), Target, Count
- **Dialogue Tree:**
  - Visual node-based editor (like Yarn Spinner, Twine)
  - Nodes: Dialogue, Choice, Skill Check, Conditional, Action
  - Connections: Link nodes with arrows
  - Preview: Walk through dialogue tree
- **Rewards:** XP, Gold, Items
- **Journal Entry:** Text shown in quest journal

**Dialogue Tree Editor:**
```
┌─────────────────────────────────────────────────────────┐
│ Dialogue Tree: [Solar Nexus Disruption]                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌────────────┐                                       │
│   │ Start      │                                       │
│   │ "Welcome!" │                                       │
│   └──────┬─────┘                                       │
│          │                                             │
│          ▼                                             │
│   ┌────────────┐         ┌────────────┐              │
│   │ Choice     │────────▶│ Response A │              │
│   │ 1. Help    │         │ "Thanks!"  │              │
│   │ 2. Refuse  │─┐       └────────────┘              │
│   └────────────┘ │                                    │
│                  │                                    │
│                  └──────▶┌────────────┐              │
│                          │ Response B │              │
│                          │ "Too bad." │              │
│                          └────────────┘              │
│                                                         │
│  [+ Add Node] [Delete] [Connect]                       │
│                                                         │
│  Selected Node:                                         │
│  ┌─────────────────────────────────────────────────┐  │
│  │ Speaker: [Engineer ▼]                           │  │
│  │ Text: [Textarea for dialogue...              ]  │  │
│  │ Choices:                                         │  │
│  │   [1. Help them] → [node_response_a]           │  │
│  │   [2. Refuse   ] → [node_response_b]           │  │
│  └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

**Time Estimate:** 5-6 days (most complex)

**Total Editor Development:** ~14-18 days

---

### 3.3 Validation System (Priority: P0 - Critical)

**Description:** Real-time validation to catch errors before saving

**Validation Types:**

#### 3.3.1 Syntax Validation
- JSON structure is valid (automatic with form-based editing)
- Required fields present
- Field types correct (number, string, array, etc.)

#### 3.3.2 Reference Validation
- Ability IDs exist in abilities.json
- Item IDs exist in items.json
- Character IDs exist in characters.json
- Quest IDs exist in quests.json
- NPC IDs exist in npcs.json

**Example:**
```
Character "Aria" has ability "lightning_strike_v2"
→ Check abilities.json for "lightning_strike_v2"
→ Not found!
→ Show error: "Ability 'lightning_strike_v2' not found. Did you mean 'lightning_strike_lv2'?"
```

#### 3.3.3 Balance Validation (Warnings, not errors)
- Stats within expected ranges
- Abilities not too powerful (DPR too high)
- Items not too cheap/expensive
- Quests give appropriate XP for level

**Guidelines:**
```javascript
const BALANCE_GUIDELINES = {
  characters: {
    level1: { hp: [80, 120], mp: [40, 100], attack: [8, 15], defense: [5, 12] },
    level5: { hp: [150, 250], mp: [100, 200], attack: [15, 30], defense: [10, 20] }
  },
  abilities: {
    maxDPR: {
      level1: 15,  // Max damage per round at level 1
      level5: 40
    }
  },
  items: {
    priceRange: {
      common: [10, 50],
      uncommon: [50, 200],
      rare: [200, 1000]
    }
  }
};
```

**Validation UI:**
```
┌─────────────────────────────────────┐
│ Validation Results          [X]     │
├─────────────────────────────────────┤
│ ❌ Errors (2)                       │
│   └─ Ability "fireball_v2" not     │
│      found in abilities.json        │
│      → Suggested fix: "fireball_lv2"│
│   └─ HP is 0 (must be > 0)         │
│                                     │
│ ⚠️  Warnings (1)                    │
│   └─ Attack stat (45) is high for  │
│      level 1 (recommended: 8-15)    │
│                                     │
│ ✅ No issues found (0)              │
└─────────────────────────────────────┘
```

**Validation Triggers:**
- On field blur (lose focus)
- On save (block save if errors)
- On demand ("Validate All" button)
- On file watch (if JSON edited externally)

**Time Estimate:** 3-4 days

---

### 3.4 Preview Capabilities (Priority: P1 - High)

**Description:** See how content looks/behaves in-game without launching game

#### 3.4.1 Character Preview
- **Sprite Preview:** Show procedurally generated sprite
- **Stats Display:** Visual stat bars, comparison to baseline
- **Ability List:** Show all abilities with tooltips
- **Equipment Preview:** Show character with/without equipment

#### 3.4.2 Ability Preview
- **Animation Preview:** Play particle effect, show damage numbers
- **Damage Calculator:**
  - Select attacker (character/enemy)
  - Select target (character/enemy)
  - Calculate: Min/Max/Average damage, DPR
  - Show formula breakdown
- **Effect Visualization:** Show buffs/debuffs as icons

**Example:**
```
┌─────────────────────────────────────┐
│ Ability Preview: Lightning Strike   │
├─────────────────────────────────────┤
│ [Animation Preview - Click to Play] │
│ ┌───────────────────────────────┐   │
│ │   ⚡⚡⚡                         │   │
│ │     ⚡     ⚡                  │   │
│ │       ⚡⚡⚡                    │   │
│ │         35 (damage number)      │   │
│ └───────────────────────────────┘   │
│                                     │
│ Damage Calculation                  │
│ Attacker: [Aria (Lv 3) ▼]          │
│ Target:   [Drone (Lv 2) ▼]         │
│                                     │
│ Formula: (Magic Attack × 1.5) - Defense │
│        = (20 × 1.5) - 3             │
│        = 30 - 3                     │
│        = 27 base damage             │
│        × variance (0.9-1.1)         │
│        = 24-30 damage range         │
│                                     │
│ DPR (Damage Per Round): 27          │
│ (Level 3 guideline: 20-35)          │
│ Status: ✅ Balanced                 │
└─────────────────────────────────────┘
```

#### 3.4.3 Item Preview
- **Sprite Preview:** Show item icon
- **Stat Comparison:**
  - Select character
  - Show stats: Before | After | Difference
  - Highlight changes in green (buff) or red (debuff)

#### 3.4.4 Dialogue Preview
- **Interactive Preview:**
  - Walk through dialogue tree
  - Make choices, see branches
  - Test skill checks (with configurable character stats)
  - See how conditionals work

**Time Estimate:** 4-5 days

---

### 3.5 Search and Filtering (Priority: P1 - High)

**Description:** Quickly find content across all types

**Features:**

#### 3.5.1 Global Search
- Search bar at top of app
- Searches across all content types
- Results grouped by type (Characters, Abilities, Items, etc.)
- Click result → Open editor for that item

**Example Search Results:**
```
Search: "fire"

Characters (1)
  └─ Pyromancer (Class, Fire-based abilities)

Abilities (3)
  └─ Fireball (Damage, Fire element)
  └─ Fire Shield (Buff, Fire resistance)
  └─ Inferno Strike (Damage, Fire element)

Items (2)
  └─ Fire Staff (Weapon, +Fire damage)
  └─ Flame Ring (Accessory, Fire resistance)

Enemies (1)
  └─ Fire Elemental (Enemy, Fire type)
```

#### 3.5.2 Advanced Filters
- **By Type:** Filter to specific content type
- **By Tag:** Filter by custom tags (Act 1, WIP, Needs Balance, etc.)
- **By Stat Range:** HP > 100, Attack 10-20, etc.
- **By Level:** Show only level 1-5 content
- **By Status:** Complete, WIP, Needs Review
- **By Last Modified:** Today, This Week, This Month

**Filter UI:**
```
┌─────────────────────────────────────┐
│ Filters                       [X]   │
├─────────────────────────────────────┤
│ Content Type                        │
│ ☑ Characters  ☑ Abilities           │
│ ☑ Items       ☑ Enemies             │
│ ☑ Quests      ☐ Locations           │
│                                     │
│ Tags                                │
│ ☑ Act 1       ☐ Act 2               │
│ ☐ Protagonist ☐ Boss                │
│                                     │
│ Level Range                         │
│ Min: [1] Max: [5]                   │
│                                     │
│ Stat Filters (Characters/Enemies)   │
│ HP:     [___] to [___]              │
│ Attack: [___] to [___]              │
│                                     │
│ [Clear Filters] [Apply]             │
└─────────────────────────────────────┘
```

**Time Estimate:** 2-3 days

---

### 3.6 Import/Export Functionality (Priority: P2 - Medium)

**Description:** Import/export content for backup, sharing, or external editing

**Features:**

#### 3.6.1 Export
- **Single Item:** Export one item as JSON
- **Bulk Export:** Select multiple items, export as single JSON array
- **Full Export:** Export all content of a type (e.g., all characters)
- **Game Pack Export:** Export all content (characters, abilities, items, etc.) as zip

**Use Cases:**
- Share content with collaborators
- Backup before major changes
- Create mod packs (Phase 2+)
- Export for external tools (spreadsheet analysis)

#### 3.6.2 Import
- **Single Item:** Import JSON file, add to content
- **Bulk Import:** Import JSON array, add multiple items
- **Merge Strategy:**
  - Replace: Overwrite existing items with same ID
  - Skip: Skip items with duplicate IDs
  - Rename: Auto-rename duplicates (e.g., "fireball" → "fireball_2")

**Validation on Import:**
- Validate JSON structure
- Check for conflicts (duplicate IDs)
- Validate references (ability IDs, item IDs, etc.)
- Show preview before importing

**Time Estimate:** 2-3 days

---

### 3.7 Version Control Integration (Priority: P2 - Medium)

**Description:** Git-aware, avoid conflicts, show diffs

**Features:**

#### 3.7.1 Git Status Awareness
- Show which JSON files have uncommitted changes
- Show diff for each file (what changed)
- Highlight edited items in browser (e.g., yellow background)

**UI:**
```
┌─────────────────────────────────────┐
│ Git Status                    [X]   │
├─────────────────────────────────────┤
│ Modified Files (3)                  │
│   src/data/characters.json (2 items)│
│     └─ Aria Solstice (edited)      │
│     └─ Zane Ironwood (new)         │
│   src/data/abilities.json (1 item)  │
│     └─ Lightning Strike (edited)    │
│                                     │
│ [View Diff] [Commit All] [Discard] │
└─────────────────────────────────────┘
```

#### 3.7.2 Commit from Asset Manager (Optional)
- Quick commit: Select items → Commit with message
- Avoid context switching to terminal

**Time Estimate:** 2 days

---

### 3.8 Additional Features (Priority: P3 - Nice to Have)

**For Phase 2+ (not MVP):**

- **Duplicate Detection:** Find similar items (e.g., abilities with same effects)
- **Batch Operations:** Bulk edit stats (e.g., increase all level 1 enemy HP by 20%)
- **Templates:** Save item as template, create new items from template
- **Analytics:** Show content statistics (total items, most used abilities, etc.)
- **Collaboration:** Multi-user editing with conflict resolution (far future)
- **AI Suggestions:** Use AI to suggest stat values, descriptions, balance tweaks
- **Localization:** Support for multiple languages (export strings for translation)

---

## 4. Technical Architecture

### 4.1 Technology Stack Recommendation

**Frontend Framework:** **React 18** + **TypeScript**

**Rationale:**
- **Familiar:** Same language as game (TypeScript)
- **Fast Development:** Rich ecosystem, UI component libraries
- **Performance:** Virtual DOM, efficient rendering
- **Tooling:** Excellent TypeScript support, hot reload
- **Future-Proof:** Large community, long-term support

**Build Tool:** **Vite**

**Rationale:**
- **Fast:** Instant dev server startup, <50ms HMR
- **Simple Config:** Minimal setup
- **TypeScript Native:** First-class TS support
- **Same as Game:** Consistency across projects

**UI Component Library:** **Shadcn/ui** (Radix UI + Tailwind CSS)

**Rationale:**
- **Zero-Touch:** Copy/paste components, no npm install bloat
- **Accessible:** ARIA-compliant, keyboard navigation
- **Customizable:** Full control over styling
- **Modern:** Tailwind CSS for rapid UI development

**Alternative:** **Ant Design** (if prefer pre-built components)

**State Management:** **Zustand** (lightweight) or **Redux Toolkit** (if complex)

**Rationale:**
- **Zustand:** Simple, minimal boilerplate, perfect for small-medium apps
- **Redux Toolkit:** If state becomes complex, easier debugging

**File System Access:** **Node.js fs module** (Electron) or **File System Access API** (browser)

**Rationale:**
- **Electron:** Full file system access, can run as desktop app
- **Browser API:** Modern browsers support, no Electron overhead

**Recommendation:** Start with **browser File System Access API**, migrate to Electron if needed

**Other Libraries:**
- **React Flow:** Dialogue tree editor (node-based UI)
- **Monaco Editor:** Code editor for JSON (fallback for advanced users)
- **Zod:** Schema validation (TypeScript-first validation library)
- **Recharts:** Charts for analytics (optional, Phase 2+)

**Tech Stack Summary:**
```
React 18 + TypeScript + Vite
├── UI: Shadcn/ui (Radix + Tailwind CSS)
├── State: Zustand
├── Validation: Zod
├── Node Editor: React Flow
├── File Access: File System Access API (browser) or fs (Electron)
└── Forms: React Hook Form
```

### 4.2 Architecture Overview

**Architecture Pattern:** **Monorepo with Asset Manager as separate app**

**Directory Structure:**
```
/project-root
├── /game                       # Phaser 3 game (existing)
│   ├── /src
│   │   ├── /data              # JSON content files (source of truth)
│   │   │   ├── characters.json
│   │   │   ├── abilities.json
│   │   │   ├── items.json
│   │   │   ├── enemies.json
│   │   │   └── quests.json
│   │   └── ...
│   └── package.json
│
├── /asset-manager              # Asset Manager tool (new)
│   ├── /src
│   │   ├── /components        # React components
│   │   │   ├── /editors       # Form editors (Character, Ability, etc.)
│   │   │   ├── /browser       # Content browser, list views
│   │   │   ├── /preview       # Preview components
│   │   │   └── /ui            # Shared UI components
│   │   ├── /lib               # Utilities
│   │   │   ├── fileSystem.ts  # Read/write JSON files
│   │   │   ├── validation.ts  # Validation logic
│   │   │   └── schemas.ts     # Zod schemas for content types
│   │   ├── /stores            # Zustand stores
│   │   │   ├── contentStore.ts # Main content state
│   │   │   └── uiStore.ts      # UI state (filters, search)
│   │   ├── /types             # TypeScript types (shared with game)
│   │   │   ├── Character.ts
│   │   │   ├── Ability.ts
│   │   │   └── ...
│   │   ├── App.tsx            # Main app component
│   │   └── main.tsx           # Entry point
│   ├── package.json
│   └── vite.config.ts
│
├── /shared                     # Shared types, utilities
│   ├── /types                 # TypeScript types (used by both game & tool)
│   │   ├── Character.ts
│   │   ├── Ability.ts
│   │   ├── Item.ts
│   │   ├── Enemy.ts
│   │   └── Quest.ts
│   └── package.json
│
└── package.json               # Root workspace config
```

**Key Architectural Decisions:**

#### 4.2.1 Single Source of Truth: JSON Files in Game Directory
- Asset Manager reads/writes JSON files in `/game/src/data/`
- Game loads JSON files at runtime
- No duplication, no sync issues

#### 4.2.2 Shared TypeScript Types
- `/shared/types/` contains all type definitions
- Both game and Asset Manager import from shared types
- Ensures consistency (type changes propagate to both)

**Example:**
```typescript
// shared/types/Character.ts
export interface Character {
  id: string;
  name: string;
  class: CharacterClass;
  baseStats: Stats;
  abilities: string[]; // Ability IDs
  equipment: Equipment;
  visualData: VisualData;
}

// game/src/entities/Character.ts
import { Character } from '@shared/types';

// asset-manager/src/components/editors/CharacterEditor.tsx
import { Character } from '@shared/types';
```

#### 4.2.3 File System Access

**Option A: File System Access API (Browser, Recommended for MVP)**

**Pros:**
- No Electron overhead
- Runs in any modern browser
- Simple deployment (just open HTML file)
- Hot reload works seamlessly

**Cons:**
- Requires user to grant directory access (one-time)
- Limited to modern browsers (Chrome, Edge, Opera)
- Some file operations slower than native

**Implementation:**
```typescript
// lib/fileSystem.ts
export class FileSystemService {
  private directoryHandle: FileSystemDirectoryHandle | null = null;

  async requestAccess(): Promise<void> {
    // User selects game directory
    this.directoryHandle = await window.showDirectoryPicker();
  }

  async readJSON(filePath: string): Promise<any> {
    const fileHandle = await this.getFileHandle(filePath);
    const file = await fileHandle.getFile();
    const text = await file.text();
    return JSON.parse(text);
  }

  async writeJSON(filePath: string, data: any): Promise<void> {
    const fileHandle = await this.getFileHandle(filePath, { create: true });
    const writable = await fileHandle.createWritable();
    await writable.write(JSON.stringify(data, null, 2));
    await writable.close();
  }

  private async getFileHandle(
    path: string,
    options?: { create?: boolean }
  ): Promise<FileSystemFileHandle> {
    const parts = path.split('/');
    let handle = this.directoryHandle!;

    for (let i = 0; i < parts.length - 1; i++) {
      handle = await handle.getDirectoryHandle(parts[i], options);
    }

    return handle.getFileHandle(parts[parts.length - 1], options);
  }
}
```

**User Flow:**
1. Open Asset Manager in browser
2. Click "Select Game Directory"
3. Browser shows directory picker → User selects `/game/src/data/`
4. Browser remembers permission (persists across sessions)
5. Asset Manager can now read/write JSON files

**Option B: Electron (Desktop App)**

**Pros:**
- Full file system access (no permission prompts)
- Better performance for file operations
- Can bundle as standalone app
- Cross-platform (Windows, Mac, Linux)

**Cons:**
- Adds complexity (Electron setup, builds)
- Larger bundle size (~100MB vs <1MB)
- Slower development (need to rebuild Electron app)
- Overkill for simple tool

**Recommendation:** **Start with File System Access API**, migrate to Electron in Phase 2+ if needed.

#### 4.2.4 Hot Reload Integration

**Goal:** When Asset Manager saves JSON, game auto-reloads content

**Approach 1: File Watcher (Simple)**
- Game watches JSON files for changes (using Chokidar or similar)
- When file changes, game reloads that content
- No communication between Asset Manager and game

**Implementation (Game side):**
```typescript
// game/src/utils/ContentHotReload.ts
import chokidar from 'chokidar';

export class ContentHotReload {
  private watcher: chokidar.FSWatcher | null = null;

  start(dataDir: string): void {
    this.watcher = chokidar.watch(`${dataDir}/**/*.json`, {
      ignoreInitial: true,
    });

    this.watcher.on('change', (path) => {
      console.log(`Content file changed: ${path}`);
      this.reloadContent(path);
    });
  }

  private reloadContent(filePath: string): void {
    // Determine content type from path
    if (filePath.includes('characters.json')) {
      this.reloadCharacters();
    } else if (filePath.includes('abilities.json')) {
      this.reloadAbilities();
    }
    // ... etc
  }

  private reloadCharacters(): void {
    // Reload characters.json
    // Update game state
    // Show toast: "Characters reloaded"
  }
}

// In game boot
const hotReload = new ContentHotReload();
if (import.meta.env.DEV) {
  hotReload.start('./src/data');
}
```

**Approach 2: WebSocket Communication (Advanced)**
- Asset Manager runs dev server with WebSocket
- Game connects to WebSocket
- Asset Manager sends "reload" message when saving
- Game reloads specific content

**Recommendation:** **Use Approach 1 (File Watcher)** for simplicity. Game dev server already has file watching.

### 4.3 Data Flow

**Read Flow (Load Content):**
```
User Opens Asset Manager
    ↓
Request File System Access
    ↓
User Grants Access to /game/src/data/
    ↓
Load all JSON files in parallel
    ↓
Parse JSON → Validate with Zod schemas
    ↓
Store in Zustand state (contentStore)
    ↓
Render UI (Content Browser)
```

**Write Flow (Save Content):**
```
User Edits Character
    ↓
Form fields update local state
    ↓
Real-time validation (Zod schema)
    ↓
User clicks "Save"
    ↓
Validate full object
    ↓
If valid:
    ↓
  Read current characters.json
    ↓
  Update/Insert character in array
    ↓
  Write back to characters.json (pretty-printed)
    ↓
  Update Zustand state
    ↓
  Show success toast
    ↓
  (Game auto-reloads via file watcher)
```

**Validation Flow:**
```
User Types in Field
    ↓
Debounced validation (300ms)
    ↓
Zod schema validation
    ↓
If error:
    ↓
  Show error below field (red text)
    ↓
If warning:
    ↓
  Show warning (yellow icon + tooltip)
    ↓
If valid:
    ↓
  Show checkmark (green)
```

### 4.4 Deployment Strategy

**Option 1: Standalone HTML File (Simplest)**
- Build Asset Manager with Vite
- Output single HTML file (inline CSS/JS)
- User opens file in browser
- Selects game directory
- Works offline

**Build:**
```bash
cd asset-manager
npm run build
# Output: dist/index.html
```

**Usage:**
1. Open `dist/index.html` in Chrome/Edge
2. Click "Select Game Directory" → Navigate to `/game/src/data/`
3. Start editing

**Option 2: Local Dev Server (Development)**
- Run Vite dev server
- Open `http://localhost:5173`
- Hot reload during Asset Manager development

**Build:**
```bash
cd asset-manager
npm run dev
# Open http://localhost:5173
```

**Option 3: Electron App (Phase 2+)**
- Package as desktop app
- Double-click to launch
- No browser required

**Recommendation:** **Option 1 for MVP** (standalone HTML), Option 2 for development.

---

## 5. UI/UX Design

### 5.1 Overall Layout

**Three-Panel Layout:**

```
┌─────────────────────────────────────────────────────────────────┐
│  Asset Manager                                  🔍 [Search]  ⚙️  │
├────────┬────────────────────────────────────────────────────────┤
│        │                                                         │
│  📁    │  ┌────────────────────────────────────────────────┐   │
│  Chars │  │ Characters (15)              [+ New]  [Filter] │   │
│        │  ├────────────────────────────────────────────────┤   │
│  ⚡    │  │ Name          Class          Level  HP    MP   │   │
│  Abil. │  │ ─────────────────────────────────────────────  │   │
│        │  │ Aria Solstice Technomancer   3      120   80   │   │
│  🎒    │  │ Zane Ironwood Guardian        3      180   40   │   │
│  Items │  │ Lyra Verdant  Bioengineer    3      150   100  │   │
│        │  │ ...                                             │   │
│  👾    │  └────────────────────────────────────────────────┘   │
│  Enemy │                                                         │
│        │  (When item selected, right panel shows editor)        │
│  📜    │                                                         │
│  Quest │                                                         │
│        │                                                         │
│  🗺️    │                                                         │
│  Locat │                                                         │
│        │                                                         │
│  ─────  │                                                         │
│  ✅    │                                                         │
│  Valid │                                                         │
│        │                                                         │
│  🔧    │                                                         │
│  Tools │                                                         │
└────────┴─────────────────────────────────────────────────────────┘

Left Panel: Navigation (Content Types)
Center Panel: Content Browser (List/Table View)
Right Panel: Editor / Details (when item selected)
```

### 5.2 Screen-by-Screen Design

#### 5.2.1 Main Screen (Content Browser)

**Layout:**
```
┌─────────────────────────────────────────────────────────────────┐
│ Asset Manager - Hybrid RPG         🔍 Search...        👤 ⚙️  │
├────────┬────────────────────────────────────────────────────────┤
│        │ Characters (15)                    [+ New]  [🔽Filter] │
│  📁    ├────────────────────────────────────────────────────────┤
│  ✓ Chars│                                                        │
│  ⚡    │  Name            Class         Lvl  HP   MP   Actions  │
│  Abil. │  ───────────────────────────────────────────────────   │
│  🎒    │  Aria Solstice   Technomancer   3   120  80   [✏️][🗑️] │
│  Items │  Zane Ironwood   Guardian        3   180  40   [✏️][🗑️] │
│  👾    │  Lyra Verdant    Bioengineer    3   150  100  [✏️][🗑️] │
│  Enemy │  ...                                                    │
│  📜    │                                                         │
│  Quest │  Showing 1-10 of 15        [< 1 2 > ] 10 per page ▼   │
│  🗺️    │                                                         │
│  Locat │                                                         │
│        │                                                         │
│  ─────  │                                                         │
│  ✅    │                                                         │
│  Valid │                                                         │
│  🔧    │                                                         │
│  Tools │                                                         │
└────────┴─────────────────────────────────────────────────────────┘
```

**Features:**
- **Left Sidebar:** Content type navigation (icons + labels)
- **Top Bar:** Search, settings, user menu
- **Table View:** Sortable columns, pagination
- **Actions:** Edit (pencil icon), Delete (trash icon)
- **New Button:** Create new item of current type
- **Filter Button:** Open filter sidebar

**Interactions:**
- Click row → Select (highlight row)
- Double-click row → Open editor (right panel or modal)
- Click sort arrow → Sort column
- Click pagination → Navigate pages

#### 5.2.2 Character Editor

```
┌─────────────────────────────────────────────────────────────────┐
│ Edit Character: Aria Solstice                              [X]  │
├────────┬────────────────────────────────────────────────────────┤
│        │ ┌────────────┐  ┌──────────────────────────────────┐  │
│  📁    │ │  Preview   │  │  Basic Info                      │  │
│  Chars │ │            │  │  ID:    aria_solstice (locked)   │  │
│        │ │   [👤]     │  │  Name:  [Aria Solstice        ]  │  │
│  ⚡    │ │            │  │  Class: [Technomancer       ▼]  │  │
│  Abil. │ │  Lv 3      │  │  Desc:  [Elite hacker...      ]  │  │
│  🎒    │ │            │  │  Tags:  [Protagonist][Act1]      │  │
│  Items │ │[🔄Regen.]  │  └──────────────────────────────────┘  │
│  👾    │ └────────────┘                                         │
│  Enemy │                                                         │
│  📜    │  Base Stats               Stat Growth (per level)      │
│  Quest │  ┌─────────┬───────┐     ┌─────────┬──────┐          │
│  🗺️    │  │ HP      │  120  │░░░  │ HP      │  10  │          │
│  Locat │  │ MP      │   80  │░░   │ MP      │   8  │          │
│        │  │ Attack  │   12  │░    │ Attack  │   2  │          │
│  ─────  │  │ Defense │    8  │░    │ Defense │   1  │          │
│  ✅    │  │ Magic   │   20  │░░░░ │ Magic   │   4  │          │
│  Valid │  │ Speed   │   14  │░░   │ Speed   │   2  │          │
│  🔧    │  └─────────┴───────┘     └─────────┴──────┘          │
│  Tools │                                                         │
│        │  Abilities                                              │
│        │  [Lightning Strike] [Energy Shield] [Overcharge]       │
│        │  [+ Add Ability ▼]                                     │
│        │                                                         │
│        │  Equipment Restrictions                                 │
│        │  ☑ Light Armor  ☐ Medium Armor  ☐ Heavy Armor         │
│        │  ☑ Datapad      ☐ Staff         ☐ Gauntlet            │
│        │                                                         │
│        │                            [Cancel]  [Save Character]  │
└────────┴─────────────────────────────────────────────────────────┘
```

**Validation Indicators:**
- **Green bar:** Stat within balanced range
- **Yellow bar:** Stat slightly high (warning)
- **Red bar:** Stat too high (error)
- **Checkmark icon:** Field valid
- **Warning icon:** Field has warning
- **Error icon:** Field has error (blocks save)

#### 5.2.3 Ability Editor

```
┌─────────────────────────────────────────────────────────────────┐
│ Edit Ability: Lightning Strike                             [X]  │
├─────────────────────────────────────────────────────────────────┤
│  Basic Info                   │  Preview                        │
│  ─────────────────────────────│─────────────────────────────   │
│  ID:   lightning_strike       │  [▶️ Play Animation]            │
│  Name: [Lightning Strike   ]  │                                 │
│  Desc: [Summons lightning...]  │   ⚡⚡⚡                        │
│  Icon: [⚡ Choose Icon ▼]      │     ⚡   ⚡                     │
│                                │       ⚡⚡⚡                     │
│  Mechanics                     │         35                      │
│  ──────────────────────────    │                                 │
│  Type:   [Attack       ▼]     │  Damage Calculator              │
│  Target: [Single Enemy ▼]     │  Attacker: [Aria Lv3 ▼]        │
│  MP Cost: [15]    ✅ Balanced │  Target:   [Drone Lv2▼]        │
│  Cooldown: [0] turns           │                                 │
│                                │  Damage: 24-30 (avg 27)        │
│  Effects                       │  DPR: 27 ✅ (guideline: 20-35) │
│  ──────────────────────────    │                                 │
│  [1] Damage                    │                                 │
│      Base: [30]                │                                 │
│      Type: [Magic   ▼]        │                                 │
│      Formula: [(Mag×1.5)-Def▼]│                                 │
│  [+ Add Effect]                │                                 │
│                                │                                 │
│  Animation                     │                                 │
│  ──────────────────────────    │                                 │
│  Effect Color: [🎨 #FFFF00]   │                                 │
│  Particle: [Lightning ▼]      │                                 │
│  Sound: [electric_zap.wav ▼]  │                                 │
│                                │                                 │
│                     [Cancel]  [Save Ability]                    │
└─────────────────────────────────────────────────────────────────┘
```

#### 5.2.4 Quest Editor (Dialogue Tree)

```
┌─────────────────────────────────────────────────────────────────┐
│ Edit Quest: Solar Nexus Disruption                         [X]  │
├─────────────────────────────────────────────────────────────────┤
│  [Basic Info] [Objectives] [Dialogue Tree] [Rewards]           │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Dialogue Tree                                  [+ Add Node]    │
│  ───────────────────────────────────────────────────────────   │
│                                                                 │
│        ┌─────────────┐                                         │
│        │ [1] Start   │                                         │
│        │ "The power  │                                         │
│        │  is out!"   │                                         │
│        └──────┬──────┘                                         │
│               │                                                 │
│        ┌──────▼──────┐        ┌─────────────┐                 │
│        │ [2] Choice  │───────▶│ [3] Accept  │                 │
│        │ 1. Help     │        │ "Thanks!"   │                 │
│        │ 2. Refuse   │──┐     └─────────────┘                 │
│        └─────────────┘  │                                      │
│                         │     ┌─────────────┐                 │
│                         └────▶│ [4] Refuse  │                 │
│                               │ "Too bad."  │                 │
│                               └─────────────┘                 │
│                                                                 │
│  Selected Node: [2] Choice                                     │
│  ─────────────────────────────────────────────────────────────  │
│  Speaker: [Engineer Nova ▼]                                   │
│  Text: [The solar nexus is malfunctioning...]                  │
│  Type: [Choice ▼]                                              │
│  Choices:                                                       │
│    [1] "I'll help you fix it."  → [Node 3: Accept]            │
│    [2] "Not my problem."        → [Node 4: Refuse]            │
│  [+ Add Choice]                                                │
│                                                                 │
│                                   [Cancel]  [Save Quest]       │
└─────────────────────────────────────────────────────────────────┘
```

#### 5.2.5 Validation Panel

```
┌─────────────────────────────────────────────────────────────────┐
│ Validation Results                                          [X]  │
├─────────────────────────────────────────────────────────────────┤
│  ❌ Errors (3) - Cannot Save                                   │
│  ─────────────────────────────────────────────────────────────  │
│  📁 characters.json                                             │
│    └─ [Aria Solstice]                                          │
│       └─ Ability "fireball_v2" not found                       │
│          → Did you mean "fireball_lv2"? [Fix]                  │
│                                                                 │
│  ⚡ abilities.json                                              │
│    └─ [Lightning Strike]                                       │
│       └─ MP cost is 0 (must be > 0)                            │
│          → Suggested: 15 MP [Apply]                            │
│                                                                 │
│  🎒 items.json                                                  │
│    └─ [Fire Staff]                                             │
│       └─ Missing required field: "buyPrice"                    │
│          → [Go to Item]                                        │
│                                                                 │
│  ⚠️  Warnings (2) - Can Save, but Review Recommended           │
│  ─────────────────────────────────────────────────────────────  │
│  📁 characters.json                                             │
│    └─ [Zane Ironwood]                                          │
│       └─ Attack stat (45) is high for level 1                  │
│          (Guideline: 8-15 for level 1)                         │
│          → Consider reducing to 12-15                          │
│                                                                 │
│  ⚡ abilities.json                                              │
│    └─ [Mega Blast]                                             │
│       └─ DPR (80) exceeds guideline (max 35 at level 5)       │
│          → This ability may be overpowered                     │
│                                                                 │
│  ✅ No Issues (157 items validated successfully)               │
│                                                                 │
│                     [Re-validate]  [Export Report]             │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 Navigation Patterns

**Primary Navigation:** Left sidebar (always visible)

**Breadcrumbs:** Not needed (only 2 levels: Type → Item)

**Shortcuts:**
- `Ctrl/Cmd + S`: Save current item
- `Ctrl/Cmd + F`: Focus search
- `Ctrl/Cmd + N`: New item (current type)
- `Esc`: Close editor/modal
- `Ctrl/Cmd + Z`: Undo (in editors)

**Back Button:** Not needed (editor opens in same view, just close to return)

### 5.4 Editing Workflow Example

**Scenario: Add new character ability**

**Steps:**
1. **Navigate:** Click "⚡ Abilities" in left sidebar
2. **Create:** Click "+ New" button → Ability editor opens
3. **Fill Form:**
   - Name: "Plasma Surge"
   - Type: Attack
   - Target: Single Enemy
   - MP Cost: 20
   - Add Effect: Damage, Base: 40, Type: Magic
4. **Validate:** See green checkmarks on all fields
5. **Preview:** Click "Preview" → See animation, damage calc shows 36-44 damage
6. **Save:** Click "Save Ability" → abilities.json updated
7. **Assign to Character:**
   - Click "📁 Characters" → Select "Aria Solstice"
   - Click "Edit" → Scroll to Abilities
   - Click "+ Add Ability" → Select "Plasma Surge"
   - Click "Save Character"
8. **Done:** Aria now has Plasma Surge ability

**Time:** ~2 minutes (vs 10+ minutes manually editing JSON, testing game)

### 5.5 Error Handling and Feedback

**Error Types:**

**1. Validation Errors (Red, blocks save):**
- Missing required field
- Invalid data type (e.g., text in number field)
- Broken reference (ID not found)
- Syntax error (if manual JSON editing)

**Display:**
- Red border on field
- Red error text below field
- Error icon next to field
- "Save" button disabled

**2. Warnings (Yellow, allows save):**
- Stat out of recommended range
- Ability too powerful/weak
- Item price unusual
- Quest missing optional fields

**Display:**
- Yellow warning icon next to field
- Tooltip on hover explaining warning
- Warning banner at top of editor
- "Save" button enabled (warning shows on click)

**3. Success Messages (Green):**
- Item saved successfully
- Validation passed
- Import successful

**Display:**
- Green toast notification (top-right)
- Auto-dismiss after 3 seconds
- Click to dismiss immediately

**Example Feedback:**

```
[Saving Character...]
  ↓
[✅ Character saved successfully!]
  (Game will auto-reload content)
```

---

## 6. Implementation Plan

### 6.1 Development Phases

**Phase 1: MVP (Minimum Viable Product) - 2-3 weeks**

**Goal:** Basic content editing with validation

**Features:**
- ✅ Content browser (list view, search, filter)
- ✅ Character editor (basic form)
- ✅ Ability editor (basic form)
- ✅ Item editor (basic form)
- ✅ Real-time validation (Zod schemas)
- ✅ Save to JSON files
- ✅ File System Access API integration

**Deliverables:**
- Functional Asset Manager
- Can create/edit characters, abilities, items
- Validation prevents broken data
- JSON files updated correctly

**Time:** 2-3 weeks (solo developer, part-time)

**Acceptance Criteria:**
- [ ] Can create new character with stats, abilities, equipment
- [ ] Can edit existing character and save changes
- [ ] Validation catches missing fields, broken references
- [ ] JSON files are formatted correctly (pretty-printed)
- [ ] No data loss or corruption

---

**Phase 2: Enhanced Editors - 1-2 weeks**

**Goal:** Add enemy, quest editors, and preview features

**Features:**
- ✅ Enemy editor
- ✅ Quest editor (basic, text-based dialogue)
- ✅ Preview capabilities (sprites, damage calc, stat comparison)
- ✅ Import/export functionality

**Deliverables:**
- All content types editable
- Preview content before saving
- Export/import for backup

**Time:** 1-2 weeks

---

**Phase 3: Advanced Features - 1-2 weeks**

**Goal:** Dialogue tree editor, git integration, polish

**Features:**
- ✅ Visual dialogue tree editor (React Flow)
- ✅ Git status awareness
- ✅ Bulk operations (multi-select, bulk edit)
- ✅ UI polish, responsive design
- ✅ Dark mode (optional)

**Deliverables:**
- Quest dialogue tree editor
- Git integration
- Polished UI

**Time:** 1-2 weeks

---

**Phase 4: Testing & Deployment - 1 week**

**Goal:** Bug fixes, testing, documentation

**Tasks:**
- Comprehensive testing (all editors)
- Bug fixes
- User documentation (README, tooltips)
- Build and package (standalone HTML or Electron)

**Deliverables:**
- Stable release
- User guide
- Deployment package

**Time:** 1 week

---

**Total Time:** **5-8 weeks** (overlaps with game Phase 1 development)

**Recommendation:** Start Asset Manager development in **Week 3-4** of game development (after basic game architecture is in place, before heavy content creation starts).

### 6.2 Timeline Integration with Game Development

**Game Phase 1:** 15 weeks
**Asset Manager Development:** Weeks 3-10 (overlapping)

```
Game Development Timeline:
Week 1-2:  M1 Foundation       → Asset Manager: Not needed yet
Week 3-4:  M2 Combat Core      → Asset Manager: Start development (MVP)
Week 5-6:  M3 Character Sys    → Asset Manager: MVP complete, start using
Week 7-8:  M4 Exploration      → Asset Manager: Phase 2 (enemy, quest editors)
Week 9-10: M5 World Map        → Asset Manager: Phase 3 (dialogue tree)
Week 11-12: M6 Game Juice      → Asset Manager: Stable, used daily
Week 13-14: M7 Content         → Asset Manager: Critical for content creation
Week 15-16: M8 Testing         → Asset Manager: Finalize, bug fixes
```

**Key Milestones:**

- **Week 5:** Asset Manager MVP ready → Start using for character creation
- **Week 8:** All editors complete → Create all content types in tool
- **Week 13:** Asset Manager critical for content creation (M7)

**Benefit:** By Week 13 (content creation phase), Asset Manager is mature and stable, enabling **50% faster content creation**.

### 6.3 Tech Stack Setup (Week 1 of Asset Manager Development)

**Day 1: Project Setup**

```bash
# Create asset-manager directory
mkdir asset-manager
cd asset-manager

# Initialize project
npm init -y
npm install react react-dom
npm install -D vite @vitejs/plugin-react typescript @types/react @types/react-dom

# Install UI library
npx shadcn-ui@latest init

# Install dependencies
npm install zustand zod react-hook-form @hookform/resolvers
npm install react-flow-renderer # For dialogue tree editor
npm install lucide-react # Icons

# Create directory structure
mkdir -p src/{components/{editors,browser,preview,ui},lib,stores,types}
```

**Day 2: Configure TypeScript and Vite**

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@/*": ["./src/*"],
      "@shared/*": ["../shared/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@shared': path.resolve(__dirname, '../shared'),
    },
  },
});
```

**Day 3: Setup Shared Types**

```bash
# Create shared types package
mkdir -p ../shared/types
```

```typescript
// shared/types/Character.ts
export interface Character {
  id: string;
  name: string;
  class: CharacterClass;
  baseStats: Stats;
  statGrowth: Stats;
  abilities: string[]; // Ability IDs
  equipment: {
    weapon: string | null;
    armor: string | null;
    accessory: string | null;
  };
  visualData: VisualData;
  tags: string[];
}

export enum CharacterClass {
  Technomancer = 'technomancer',
  Guardian = 'guardian',
  Bioengineer = 'bioengineer',
}

export interface Stats {
  hp: number;
  mp: number;
  attack: number;
  defense: number;
  magic: number;
  speed: number;
}

export interface VisualData {
  spriteSeed: string;
  colorPalette: {
    primary: string;
    secondary: string;
    accent: string;
  };
  bodyType: 'slim' | 'average' | 'bulky';
}
```

```typescript
// shared/types/Ability.ts
export interface Ability {
  id: string;
  name: string;
  description: string;
  type: AbilityType;
  targetType: TargetType;
  mpCost: number;
  cooldown: number;
  effects: Effect[];
  animationData: AnimationData;
  tags: string[];
}

export enum AbilityType {
  Attack = 'attack',
  Support = 'support',
  Heal = 'heal',
  Buff = 'buff',
  Debuff = 'debuff',
}

export enum TargetType {
  SingleEnemy = 'single_enemy',
  AllEnemies = 'all_enemies',
  SingleAlly = 'single_ally',
  AllAllies = 'all_allies',
  Self = 'self',
}

export interface Effect {
  type: EffectType;
  value: number;
  damageType?: DamageType;
  duration?: number; // For buffs/debuffs
  formula?: DamageFormula;
}

export enum EffectType {
  Damage = 'damage',
  Heal = 'heal',
  Buff = 'buff',
  Debuff = 'debuff',
}

export enum DamageType {
  Physical = 'physical',
  Magic = 'magic',
  Fire = 'fire',
  Ice = 'ice',
  Lightning = 'lightning',
}

export enum DamageFormula {
  Fixed = 'fixed',
  StatBased = 'stat_based',
  LevelScaled = 'level_scaled',
}

export interface AnimationData {
  effectColor: string;
  particleType: ParticleType;
  soundEffect?: string;
}

export enum ParticleType {
  Sparkles = 'sparkles',
  Lightning = 'lightning',
  Fire = 'fire',
  Ice = 'ice',
  Heal = 'heal',
}
```

**Day 4-5: Setup Zod Schemas**

```typescript
// src/lib/schemas.ts
import { z } from 'zod';
import { CharacterClass, AbilityType, TargetType, EffectType, DamageType } from '@shared/types';

export const StatsSchema = z.object({
  hp: z.number().min(1, 'HP must be at least 1'),
  mp: z.number().min(0, 'MP cannot be negative'),
  attack: z.number().min(0, 'Attack cannot be negative'),
  defense: z.number().min(0, 'Defense cannot be negative'),
  magic: z.number().min(0, 'Magic cannot be negative'),
  speed: z.number().min(0, 'Speed cannot be negative'),
});

export const CharacterSchema = z.object({
  id: z.string().min(1, 'ID is required'),
  name: z.string().min(1, 'Name is required'),
  class: z.nativeEnum(CharacterClass),
  baseStats: StatsSchema,
  statGrowth: StatsSchema,
  abilities: z.array(z.string()),
  equipment: z.object({
    weapon: z.string().nullable(),
    armor: z.string().nullable(),
    accessory: z.string().nullable(),
  }),
  visualData: z.object({
    spriteSeed: z.string(),
    colorPalette: z.object({
      primary: z.string().regex(/^#[0-9A-Fa-f]{6}$/, 'Invalid color'),
      secondary: z.string().regex(/^#[0-9A-Fa-f]{6}$/, 'Invalid color'),
      accent: z.string().regex(/^#[0-9A-Fa-f]{6}$/, 'Invalid color'),
    }),
    bodyType: z.enum(['slim', 'average', 'bulky']),
  }),
  tags: z.array(z.string()),
});

export const EffectSchema = z.object({
  type: z.nativeEnum(EffectType),
  value: z.number(),
  damageType: z.nativeEnum(DamageType).optional(),
  duration: z.number().optional(),
  formula: z.string().optional(),
});

export const AbilitySchema = z.object({
  id: z.string().min(1, 'ID is required'),
  name: z.string().min(1, 'Name is required'),
  description: z.string().min(1, 'Description is required'),
  type: z.nativeEnum(AbilityType),
  targetType: z.nativeEnum(TargetType),
  mpCost: z.number().min(1, 'MP cost must be at least 1'),
  cooldown: z.number().min(0, 'Cooldown cannot be negative'),
  effects: z.array(EffectSchema).min(1, 'At least one effect required'),
  animationData: z.object({
    effectColor: z.string().regex(/^#[0-9A-Fa-f]{6}$/, 'Invalid color'),
    particleType: z.string(),
    soundEffect: z.string().optional(),
  }),
  tags: z.array(z.string()),
});

// Export all schemas
export const schemas = {
  character: CharacterSchema,
  ability: AbilitySchema,
  // ... more schemas
};
```

### 6.4 Week-by-Week Implementation

**Week 1: Setup & Core Infrastructure**
- Day 1-2: Project setup, dependencies, types
- Day 3-4: Zod schemas, validation utilities
- Day 5: File System Access API integration, basic UI layout

**Week 2: Content Browser & Character Editor**
- Day 1-2: Content browser (list view, search)
- Day 3-5: Character editor form, validation, save

**Week 3: Ability & Item Editors**
- Day 1-3: Ability editor with effects system
- Day 4-5: Item editor, basic preview

**Week 4: Enemy & Quest Editors (Basic)**
- Day 1-2: Enemy editor
- Day 3-5: Quest editor (basic, text-based)

**Week 5: Preview Features**
- Day 1-2: Character sprite preview
- Day 3: Ability damage calculator
- Day 4-5: Item stat comparison preview

**Week 6: Dialogue Tree Editor**
- Day 1-5: React Flow integration, dialogue node editor

**Week 7: Git Integration & Polish**
- Day 1-2: Git status awareness
- Day 3-5: UI polish, responsive design, dark mode

**Week 8: Testing & Deployment**
- Day 1-3: Comprehensive testing, bug fixes
- Day 4-5: Documentation, build, deploy

---

## 7. Code Examples

### 7.1 File System Service

```typescript
// src/lib/fileSystem.ts
export class FileSystemService {
  private directoryHandle: FileSystemDirectoryHandle | null = null;
  private dataPath = 'src/data';

  async requestAccess(): Promise<void> {
    try {
      this.directoryHandle = await window.showDirectoryPicker({
        mode: 'readwrite',
      });

      // Store handle in IndexedDB for persistence
      const db = await this.openDB();
      await db.put('handles', this.directoryHandle, 'gameDirectory');

      console.log('Directory access granted');
    } catch (error) {
      if (error.name === 'AbortError') {
        console.log('User cancelled directory selection');
      } else {
        throw error;
      }
    }
  }

  async restoreAccess(): Promise<boolean> {
    try {
      const db = await this.openDB();
      const handle = await db.get('handles', 'gameDirectory');

      if (!handle) return false;

      // Verify permission
      const permission = await handle.queryPermission({ mode: 'readwrite' });
      if (permission === 'granted') {
        this.directoryHandle = handle;
        return true;
      }

      // Request permission again
      const newPermission = await handle.requestPermission({ mode: 'readwrite' });
      if (newPermission === 'granted') {
        this.directoryHandle = handle;
        return true;
      }

      return false;
    } catch {
      return false;
    }
  }

  async readJSON<T>(fileName: string): Promise<T> {
    if (!this.directoryHandle) {
      throw new Error('Directory access not granted');
    }

    try {
      const fileHandle = await this.getFileHandle(fileName);
      const file = await fileHandle.getFile();
      const text = await file.text();
      return JSON.parse(text);
    } catch (error) {
      console.error(`Error reading ${fileName}:`, error);
      throw error;
    }
  }

  async writeJSON(fileName: string, data: any): Promise<void> {
    if (!this.directoryHandle) {
      throw new Error('Directory access not granted');
    }

    try {
      const fileHandle = await this.getFileHandle(fileName, { create: true });
      const writable = await fileHandle.createWritable();

      // Pretty-print JSON with 2-space indentation
      const jsonString = JSON.stringify(data, null, 2);

      await writable.write(jsonString);
      await writable.close();

      console.log(`Saved ${fileName}`);
    } catch (error) {
      console.error(`Error writing ${fileName}:`, error);
      throw error;
    }
  }

  private async getFileHandle(
    fileName: string,
    options?: { create?: boolean }
  ): Promise<FileSystemFileHandle> {
    if (!this.directoryHandle) {
      throw new Error('Directory handle not available');
    }

    // Navigate to src/data/ directory
    const parts = this.dataPath.split('/');
    let dirHandle = this.directoryHandle;

    for (const part of parts) {
      dirHandle = await dirHandle.getDirectoryHandle(part, options);
    }

    return dirHandle.getFileHandle(fileName, options);
  }

  private async openDB(): Promise<IDBDatabase> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open('AssetManagerDB', 1);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result);

      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;
        if (!db.objectStoreNames.contains('handles')) {
          db.createObjectStore('handles');
        }
      };
    });
  }

  async loadAllContent(): Promise<{
    characters: Character[];
    abilities: Ability[];
    items: Item[];
    enemies: Enemy[];
    quests: Quest[];
  }> {
    const [characters, abilities, items, enemies, quests] = await Promise.all([
      this.readJSON<Character[]>('characters.json'),
      this.readJSON<Ability[]>('abilities.json'),
      this.readJSON<Item[]>('items.json'),
      this.readJSON<Enemy[]>('enemies.json'),
      this.readJSON<Quest[]>('quests.json'),
    ]);

    return { characters, abilities, items, enemies, quests };
  }
}

// Export singleton instance
export const fileSystem = new FileSystemService();
```

### 7.2 Content Store (Zustand)

```typescript
// src/stores/contentStore.ts
import { create } from 'zustand';
import { Character, Ability, Item, Enemy, Quest } from '@shared/types';
import { fileSystem } from '@/lib/fileSystem';

interface ContentState {
  // Data
  characters: Character[];
  abilities: Ability[];
  items: Item[];
  enemies: Enemy[];
  quests: Quest[];

  // Loading state
  isLoading: boolean;
  error: string | null;

  // Actions
  loadContent: () => Promise<void>;

  // Character actions
  addCharacter: (character: Character) => Promise<void>;
  updateCharacter: (id: string, updates: Partial<Character>) => Promise<void>;
  deleteCharacter: (id: string) => Promise<void>;

  // Ability actions
  addAbility: (ability: Ability) => Promise<void>;
  updateAbility: (id: string, updates: Partial<Ability>) => Promise<void>;
  deleteAbility: (id: string) => Promise<void>;

  // ... similar for items, enemies, quests
}

export const useContentStore = create<ContentState>((set, get) => ({
  // Initial state
  characters: [],
  abilities: [],
  items: [],
  enemies: [],
  quests: [],
  isLoading: false,
  error: null,

  // Load all content from JSON files
  loadContent: async () => {
    set({ isLoading: true, error: null });
    try {
      const content = await fileSystem.loadAllContent();
      set({
        characters: content.characters,
        abilities: content.abilities,
        items: content.items,
        enemies: content.enemies,
        quests: content.quests,
        isLoading: false,
      });
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },

  // Add new character
  addCharacter: async (character: Character) => {
    const { characters } = get();
    const newCharacters = [...characters, character];

    // Save to file
    await fileSystem.writeJSON('characters.json', newCharacters);

    // Update state
    set({ characters: newCharacters });
  },

  // Update existing character
  updateCharacter: async (id: string, updates: Partial<Character>) => {
    const { characters } = get();
    const newCharacters = characters.map((char) =>
      char.id === id ? { ...char, ...updates } : char
    );

    // Save to file
    await fileSystem.writeJSON('characters.json', newCharacters);

    // Update state
    set({ characters: newCharacters });
  },

  // Delete character
  deleteCharacter: async (id: string) => {
    const { characters } = get();
    const newCharacters = characters.filter((char) => char.id !== id);

    // Save to file
    await fileSystem.writeJSON('characters.json', newCharacters);

    // Update state
    set({ characters: newCharacters });
  },

  // Ability actions (similar pattern)
  addAbility: async (ability: Ability) => {
    const { abilities } = get();
    const newAbilities = [...abilities, ability];
    await fileSystem.writeJSON('abilities.json', newAbilities);
    set({ abilities: newAbilities });
  },

  updateAbility: async (id: string, updates: Partial<Ability>) => {
    const { abilities } = get();
    const newAbilities = abilities.map((ability) =>
      ability.id === id ? { ...ability, ...updates } : ability
    );
    await fileSystem.writeJSON('abilities.json', newAbilities);
    set({ abilities: newAbilities });
  },

  deleteAbility: async (id: string) => {
    const { abilities } = get();
    const newAbilities = abilities.filter((ability) => ability.id !== id);
    await fileSystem.writeJSON('abilities.json', newAbilities);
    set({ abilities: newAbilities });
  },

  // ... similar for items, enemies, quests
}));
```

### 7.3 Validation System

```typescript
// src/lib/validation.ts
import { z } from 'zod';
import { schemas } from './schemas';
import { useContentStore } from '@/stores/contentStore';

export interface ValidationError {
  field: string;
  message: string;
  type: 'error' | 'warning';
  suggestion?: string;
}

export interface ValidationResult {
  valid: boolean;
  errors: ValidationError[];
  warnings: ValidationError[];
}

export class ValidationService {
  // Validate character
  validateCharacter(character: any): ValidationResult {
    const result: ValidationResult = {
      valid: true,
      errors: [],
      warnings: [],
    };

    // Schema validation
    try {
      schemas.character.parse(character);
    } catch (error) {
      if (error instanceof z.ZodError) {
        result.valid = false;
        result.errors = error.errors.map((err) => ({
          field: err.path.join('.'),
          message: err.message,
          type: 'error',
        }));
      }
    }

    // Reference validation (check ability IDs exist)
    const { abilities } = useContentStore.getState();
    const abilityIds = new Set(abilities.map((a) => a.id));

    for (const abilityId of character.abilities) {
      if (!abilityIds.has(abilityId)) {
        result.valid = false;
        result.errors.push({
          field: 'abilities',
          message: `Ability "${abilityId}" not found`,
          type: 'error',
          suggestion: this.findSimilarId(abilityId, Array.from(abilityIds)),
        });
      }
    }

    // Balance validation (warnings)
    const balanceWarnings = this.validateCharacterBalance(character);
    result.warnings.push(...balanceWarnings);

    return result;
  }

  // Validate ability
  validateAbility(ability: any): ValidationResult {
    const result: ValidationResult = {
      valid: true,
      errors: [],
      warnings: [],
    };

    // Schema validation
    try {
      schemas.ability.parse(ability);
    } catch (error) {
      if (error instanceof z.ZodError) {
        result.valid = false;
        result.errors = error.errors.map((err) => ({
          field: err.path.join('.'),
          message: err.message,
          type: 'error',
        }));
      }
    }

    // Balance validation
    const dpr = this.calculateDPR(ability);
    const maxDPR = this.getMaxDPR(5); // Assume level 5 for now

    if (dpr > maxDPR) {
      result.warnings.push({
        field: 'effects',
        message: `DPR (${dpr}) exceeds guideline (max ${maxDPR} at level 5)`,
        type: 'warning',
      });
    }

    return result;
  }

  // Calculate damage per round for ability
  private calculateDPR(ability: any): number {
    const damageEffect = ability.effects.find((e: any) => e.type === 'damage');
    if (!damageEffect) return 0;

    // Simple calculation (can be more complex based on formula)
    return damageEffect.value;
  }

  // Get max DPR guideline for level
  private getMaxDPR(level: number): number {
    const guidelines = {
      1: 15,
      2: 20,
      3: 25,
      4: 30,
      5: 35,
    };
    return guidelines[level] || 35;
  }

  // Validate character balance
  private validateCharacterBalance(character: any): ValidationError[] {
    const warnings: ValidationError[] = [];

    const guidelines = {
      1: { hp: [80, 120], attack: [8, 15], defense: [5, 12] },
      5: { hp: [150, 250], attack: [15, 30], defense: [10, 20] },
    };

    // Check HP
    const level = 1; // Default to level 1
    const { hp, attack, defense } = character.baseStats;
    const guide = guidelines[level];

    if (hp < guide.hp[0] || hp > guide.hp[1]) {
      warnings.push({
        field: 'baseStats.hp',
        message: `HP (${hp}) outside recommended range (${guide.hp[0]}-${guide.hp[1]}) for level ${level}`,
        type: 'warning',
      });
    }

    if (attack < guide.attack[0] || attack > guide.attack[1]) {
      warnings.push({
        field: 'baseStats.attack',
        message: `Attack (${attack}) outside recommended range (${guide.attack[0]}-${guide.attack[1]}) for level ${level}`,
        type: 'warning',
      });
    }

    // ... similar for other stats

    return warnings;
  }

  // Find similar ID (for suggestions)
  private findSimilarId(target: string, ids: string[]): string | undefined {
    // Simple Levenshtein distance-based suggestion
    const distances = ids.map((id) => ({
      id,
      distance: this.levenshteinDistance(target, id),
    }));

    distances.sort((a, b) => a.distance - b.distance);

    if (distances[0].distance < 3) {
      return distances[0].id;
    }

    return undefined;
  }

  // Levenshtein distance algorithm
  private levenshteinDistance(a: string, b: string): number {
    const matrix = [];

    for (let i = 0; i <= b.length; i++) {
      matrix[i] = [i];
    }

    for (let j = 0; j <= a.length; j++) {
      matrix[0][j] = j;
    }

    for (let i = 1; i <= b.length; i++) {
      for (let j = 1; j <= a.length; j++) {
        if (b.charAt(i - 1) === a.charAt(j - 1)) {
          matrix[i][j] = matrix[i - 1][j - 1];
        } else {
          matrix[i][j] = Math.min(
            matrix[i - 1][j - 1] + 1,
            matrix[i][j - 1] + 1,
            matrix[i - 1][j] + 1
          );
        }
      }
    }

    return matrix[b.length][a.length];
  }

  // Validate all content
  validateAll(): {
    characters: ValidationResult[];
    abilities: ValidationResult[];
    // ... more
  } {
    const { characters, abilities } = useContentStore.getState();

    return {
      characters: characters.map((char) => ({
        ...this.validateCharacter(char),
        id: char.id,
        name: char.name,
      })),
      abilities: abilities.map((ability) => ({
        ...this.validateAbility(ability),
        id: ability.id,
        name: ability.name,
      })),
      // ... more
    };
  }
}

export const validationService = new ValidationService();
```

### 7.4 Character Editor Component

```typescript
// src/components/editors/CharacterEditor.tsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { Character, CharacterClass } from '@shared/types';
import { CharacterSchema } from '@/lib/schemas';
import { useContentStore } from '@/stores/contentStore';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Slider } from '@/components/ui/slider';
import { Textarea } from '@/components/ui/textarea';

interface CharacterEditorProps {
  character?: Character;
  onSave: () => void;
  onCancel: () => void;
}

export function CharacterEditor({ character, onSave, onCancel }: CharacterEditorProps) {
  const { addCharacter, updateCharacter } = useContentStore();

  const {
    register,
    handleSubmit,
    formState: { errors },
    watch,
    setValue,
  } = useForm<Character>({
    resolver: zodResolver(CharacterSchema),
    defaultValues: character || {
      id: '',
      name: '',
      class: CharacterClass.Technomancer,
      baseStats: { hp: 100, mp: 50, attack: 10, defense: 8, magic: 10, speed: 12 },
      statGrowth: { hp: 10, mp: 5, attack: 2, defense: 1, magic: 2, speed: 1 },
      abilities: [],
      equipment: { weapon: null, armor: null, accessory: null },
      visualData: {
        spriteSeed: '',
        colorPalette: { primary: '#44AA44', secondary: '#339933', accent: '#FFFF00' },
        bodyType: 'average',
      },
      tags: [],
    },
  });

  const onSubmit = async (data: Character) => {
    try {
      if (character) {
        await updateCharacter(character.id, data);
      } else {
        // Generate ID from name
        data.id = data.name.toLowerCase().replace(/\s+/g, '_');
        await addCharacter(data);
      }
      onSave();
    } catch (error) {
      console.error('Error saving character:', error);
    }
  };

  const baseStats = watch('baseStats');

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      <div className="grid grid-cols-2 gap-6">
        {/* Left Column: Preview & Basic Info */}
        <div className="space-y-4">
          <div className="border rounded-lg p-4">
            <h3 className="text-lg font-semibold mb-2">Preview</h3>
            {/* TODO: Add sprite preview component */}
            <div className="w-32 h-32 bg-gray-200 rounded flex items-center justify-center">
              <span className="text-4xl">👤</span>
            </div>
            <Button type="button" className="mt-2 w-full" variant="outline">
              Regenerate Sprite
            </Button>
          </div>

          <div className="space-y-2">
            <Label htmlFor="name">Name</Label>
            <Input id="name" {...register('name')} />
            {errors.name && <p className="text-red-500 text-sm">{errors.name.message}</p>}
          </div>

          <div className="space-y-2">
            <Label htmlFor="class">Class</Label>
            <Select
              value={watch('class')}
              onValueChange={(value) => setValue('class', value as CharacterClass)}
            >
              <SelectTrigger>
                <SelectValue />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value={CharacterClass.Technomancer}>Technomancer</SelectItem>
                <SelectItem value={CharacterClass.Guardian}>Guardian</SelectItem>
                <SelectItem value={CharacterClass.Bioengineer}>Bioengineer</SelectItem>
              </SelectContent>
            </Select>
          </div>

          <div className="space-y-2">
            <Label htmlFor="description">Description</Label>
            <Textarea id="description" {...register('description')} rows={3} />
          </div>
        </div>

        {/* Right Column: Stats */}
        <div className="space-y-4">
          <h3 className="text-lg font-semibold">Base Stats</h3>

          {/* HP */}
          <div className="space-y-2">
            <div className="flex justify-between">
              <Label htmlFor="hp">HP</Label>
              <span className="text-sm text-gray-500">{baseStats.hp}</span>
            </div>
            <Slider
              id="hp"
              min={10}
              max={300}
              step={10}
              value={[baseStats.hp]}
              onValueChange={([value]) => setValue('baseStats.hp', value)}
            />
            {errors.baseStats?.hp && (
              <p className="text-red-500 text-sm">{errors.baseStats.hp.message}</p>
            )}
          </div>

          {/* MP */}
          <div className="space-y-2">
            <div className="flex justify-between">
              <Label htmlFor="mp">MP</Label>
              <span className="text-sm text-gray-500">{baseStats.mp}</span>
            </div>
            <Slider
              id="mp"
              min={0}
              max={200}
              step={10}
              value={[baseStats.mp]}
              onValueChange={([value]) => setValue('baseStats.mp', value)}
            />
          </div>

          {/* Attack */}
          <div className="space-y-2">
            <div className="flex justify-between">
              <Label htmlFor="attack">Attack</Label>
              <span className="text-sm text-gray-500">{baseStats.attack}</span>
            </div>
            <Slider
              id="attack"
              min={1}
              max={50}
              step={1}
              value={[baseStats.attack]}
              onValueChange={([value]) => setValue('baseStats.attack', value)}
            />
          </div>

          {/* Defense */}
          <div className="space-y-2">
            <div className="flex justify-between">
              <Label htmlFor="defense">Defense</Label>
              <span className="text-sm text-gray-500">{baseStats.defense}</span>
            </div>
            <Slider
              id="defense"
              min={1}
              max={50}
              step={1}
              value={[baseStats.defense]}
              onValueChange={([value]) => setValue('baseStats.defense', value)}
            />
          </div>

          {/* Magic */}
          <div className="space-y-2">
            <div className="flex justify-between">
              <Label htmlFor="magic">Magic</Label>
              <span className="text-sm text-gray-500">{baseStats.magic}</span>
            </div>
            <Slider
              id="magic"
              min={1}
              max={50}
              step={1}
              value={[baseStats.magic]}
              onValueChange={([value]) => setValue('baseStats.magic', value)}
            />
          </div>

          {/* Speed */}
          <div className="space-y-2">
            <div className="flex justify-between">
              <Label htmlFor="speed">Speed</Label>
              <span className="text-sm text-gray-500">{baseStats.speed}</span>
            </div>
            <Slider
              id="speed"
              min={1}
              max={50}
              step={1}
              value={[baseStats.speed]}
              onValueChange={([value]) => setValue('baseStats.speed', value)}
            />
          </div>
        </div>
      </div>

      {/* Abilities Section */}
      <div className="space-y-2">
        <Label>Abilities</Label>
        <div className="flex flex-wrap gap-2">
          {watch('abilities').map((abilityId) => (
            <div key={abilityId} className="px-3 py-1 bg-blue-100 rounded-full text-sm flex items-center gap-2">
              {abilityId}
              <button
                type="button"
                onClick={() => {
                  const abilities = watch('abilities').filter((id) => id !== abilityId);
                  setValue('abilities', abilities);
                }}
                className="text-red-500 hover:text-red-700"
              >
                ×
              </button>
            </div>
          ))}
          <Button type="button" variant="outline" size="sm">
            + Add Ability
          </Button>
        </div>
      </div>

      {/* Action Buttons */}
      <div className="flex justify-end gap-2">
        <Button type="button" variant="outline" onClick={onCancel}>
          Cancel
        </Button>
        <Button type="submit">
          {character ? 'Save Character' : 'Create Character'}
        </Button>
      </div>
    </form>
  );
}
```

### 7.5 Content Browser Component

```typescript
// src/components/browser/ContentBrowser.tsx
import React, { useState } from 'react';
import { useContentStore } from '@/stores/contentStore';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { CharacterEditor } from '@/components/editors/CharacterEditor';

export function ContentBrowser() {
  const { characters } = useContentStore();
  const [search, setSearch] = useState('');
  const [selectedCharacter, setSelectedCharacter] = useState<Character | null>(null);
  const [isEditing, setIsEditing] = useState(false);
  const [isCreating, setIsCreating] = useState(false);

  const filteredCharacters = characters.filter((char) =>
    char.name.toLowerCase().includes(search.toLowerCase())
  );

  if (isEditing || isCreating) {
    return (
      <div className="p-6">
        <CharacterEditor
          character={isEditing ? selectedCharacter : undefined}
          onSave={() => {
            setIsEditing(false);
            setIsCreating(false);
            setSelectedCharacter(null);
          }}
          onCancel={() => {
            setIsEditing(false);
            setIsCreating(false);
            setSelectedCharacter(null);
          }}
        />
      </div>
    );
  }

  return (
    <div className="p-6 space-y-4">
      <div className="flex justify-between items-center">
        <h2 className="text-2xl font-bold">Characters ({filteredCharacters.length})</h2>
        <div className="flex gap-2">
          <Input
            placeholder="Search characters..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="w-64"
          />
          <Button onClick={() => setIsCreating(true)}>+ New Character</Button>
        </div>
      </div>

      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>Name</TableHead>
            <TableHead>Class</TableHead>
            <TableHead>Level</TableHead>
            <TableHead>HP</TableHead>
            <TableHead>MP</TableHead>
            <TableHead>Actions</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {filteredCharacters.map((character) => (
            <TableRow key={character.id}>
              <TableCell className="font-medium">{character.name}</TableCell>
              <TableCell>{character.class}</TableCell>
              <TableCell>1</TableCell>
              <TableCell>{character.baseStats.hp}</TableCell>
              <TableCell>{character.baseStats.mp}</TableCell>
              <TableCell>
                <div className="flex gap-2">
                  <Button
                    variant="outline"
                    size="sm"
                    onClick={() => {
                      setSelectedCharacter(character);
                      setIsEditing(true);
                    }}
                  >
                    Edit
                  </Button>
                  <Button variant="outline" size="sm" onClick={() => {}}>
                    Delete
                  </Button>
                </div>
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>
    </div>
  );
}
```

---

## 8. Integration Strategy

### 8.1 Integration with Game Development

**File Structure Integration:**

```
/project-root
├── /game                   # Game codebase
│   └── /src
│       └── /data           # Single source of truth (JSON files)
│           ├── characters.json
│           ├── abilities.json
│           ├── items.json
│           ├── enemies.json
│           └── quests.json
│
├── /asset-manager          # Asset Manager tool
│   └── src                 # Reads/writes /game/src/data/*.json
│
└── /shared                 # Shared TypeScript types
    └── /types
        ├── Character.ts
        ├── Ability.ts
        └── ...
```

**Workflow:**

1. **Developer writes game code** (Phaser scenes, systems)
2. **Developer opens Asset Manager** (browser, `file:///.../asset-manager/dist/index.html`)
3. **Grant access to `/game/src/data/` directory**
4. **Create/edit content** (characters, abilities, etc.)
5. **Save** → JSON files updated
6. **Game file watcher detects changes** → Auto-reloads content
7. **Test in game** → See changes immediately
8. **Iterate** (repeat steps 4-7)

**Benefits:**
- No manual JSON editing
- Instant feedback (hot reload)
- Type-safe (shared TypeScript types)
- Version control (git tracks JSON changes)

### 8.2 Hot Reload Setup (Game Side)

```typescript
// game/src/utils/ContentHotReload.ts
import chokidar from 'chokidar';
import { EventEmitter } from 'events';

export class ContentHotReload extends EventEmitter {
  private watcher: chokidar.FSWatcher | null = null;

  start(dataDir: string): void {
    if (this.watcher) {
      console.warn('Hot reload already started');
      return;
    }

    console.log(`[Hot Reload] Watching ${dataDir}`);

    this.watcher = chokidar.watch(`${dataDir}/**/*.json`, {
      ignoreInitial: true,
      awaitWriteFinish: {
        stabilityThreshold: 300,
        pollInterval: 100,
      },
    });

    this.watcher.on('change', (path) => {
      console.log(`[Hot Reload] ${path} changed`);
      this.handleFileChange(path);
    });

    this.watcher.on('error', (error) => {
      console.error('[Hot Reload] Error:', error);
    });
  }

  stop(): void {
    if (this.watcher) {
      this.watcher.close();
      this.watcher = null;
      console.log('[Hot Reload] Stopped');
    }
  }

  private handleFileChange(filePath: string): void {
    const fileName = filePath.split('/').pop();

    switch (fileName) {
      case 'characters.json':
        this.emit('reload:characters');
        break;
      case 'abilities.json':
        this.emit('reload:abilities');
        break;
      case 'items.json':
        this.emit('reload:items');
        break;
      case 'enemies.json':
        this.emit('reload:enemies');
        break;
      case 'quests.json':
        this.emit('reload:quests');
        break;
      default:
        console.warn(`[Hot Reload] Unknown file: ${fileName}`);
    }
  }
}

// Usage in game
export const hotReload = new ContentHotReload();

if (import.meta.env.DEV) {
  hotReload.start('./src/data');

  hotReload.on('reload:characters', async () => {
    console.log('[Game] Reloading characters...');
    const characters = await fetch('./src/data/characters.json').then((r) => r.json());
    // Update game state
    gameState.characters = characters;
    // Show toast notification
    showToast('Characters reloaded');
  });

  // Similar handlers for other content types
}
```

### 8.3 Version Control Best Practices

**Git Workflow:**

1. **Commit JSON files regularly**
   - After significant content changes
   - Before major refactoring
   - At end of work session

2. **Use descriptive commit messages**
   ```bash
   git add src/data/characters.json
   git commit -m "Add Eco-Warrior character class with 5 abilities"
   ```

3. **Review diffs before committing**
   ```bash
   git diff src/data/characters.json
   ```

4. **Avoid merge conflicts:**
   - Pretty-print JSON (2-space indentation)
   - Keep array items on separate lines
   - Sort arrays by ID (optional, but helps)

**Example `.gitignore`:**
```
# Asset Manager build artifacts
asset-manager/dist/
asset-manager/node_modules/

# Game build artifacts
game/dist/
game/node_modules/

# Editor files
.vscode/
.idea/

# OS files
.DS_Store
Thumbs.db
```

### 8.4 Deployment Checklist

**Before First Use:**

- [ ] Build Asset Manager: `cd asset-manager && npm run build`
- [ ] Test file access: Open `asset-manager/dist/index.html` in Chrome
- [ ] Grant directory access: Select `/game/src/data/`
- [ ] Verify content loads: See all characters, abilities, etc.
- [ ] Test edit & save: Edit a character, save, check JSON file
- [ ] Test game hot reload: Edit content, see game reload

**For Team Members:**

- [ ] Share built `asset-manager/dist/index.html`
- [ ] Provide setup instructions (README)
- [ ] Ensure they use Chrome/Edge (for File System Access API)
- [ ] Walk through first use (grant directory access)

---

## 9. Appendix

### 9.1 Browser Compatibility

**File System Access API Support:**

| Browser | Version | Support |
|---------|---------|---------|
| Chrome | 86+ | ✅ Full |
| Edge | 86+ | ✅ Full |
| Opera | 72+ | ✅ Full |
| Safari | ❌ | Not supported |
| Firefox | ❌ | Behind flag |

**Recommendation:** Use Chrome or Edge (most developers already use these)

**Fallback:** If File System Access API not available, show error message:
```
"Your browser doesn't support File System Access API.
Please use Chrome 86+, Edge 86+, or Opera 72+."
```

### 9.2 Performance Considerations

**File I/O:**
- Read all JSON files on startup (100-500ms)
- Write only changed files (10-50ms per file)
- Use debounced saves (auto-save 2s after last edit)

**Large Content Sets:**
- Pagination (show 10-50 items per page)
- Virtual scrolling (for 1000+ items)
- Lazy load details (load full item only when editing)

**Memory:**
- Keep all content in memory (acceptable for <10MB total)
- Avoid loading game assets (sprites, sounds) in Asset Manager

### 9.3 Future Enhancements (Phase 2+)

**Advanced Features:**
- **Undo/Redo:** Track edit history, allow reverting changes
- **AI-Powered Suggestions:** Use AI to suggest stat values, descriptions
- **Analytics Dashboard:** Show content statistics (most used abilities, average stats, etc.)
- **Collaboration:** Multi-user editing (via git branches or real-time sync)
- **Localization:** Export strings for translation, import translations
- **Templates:** Save items as templates, create from template
- **Batch Operations:** Bulk edit stats across multiple items
- **Graph Visualizations:** Show stat distributions, damage curves
- **Ability Combos:** Suggest ability combinations (synergies)
- **Balance Calculator:** Input player party, simulate battles, predict win rate

**Integration Improvements:**
- **Electron App:** Desktop app with full file system access
- **Game Preview:** Embed Phaser game preview in Asset Manager
- **Live Testing:** Test abilities/items directly in Asset Manager (mini battle simulator)
- **Git Integration:** Commit directly from Asset Manager, view git history

### 9.4 Troubleshooting

**Common Issues:**

**1. "Directory access not granted"**
- **Cause:** User didn't grant permission or permission revoked
- **Solution:** Click "Select Game Directory" again

**2. "File not found: characters.json"**
- **Cause:** Wrong directory selected, or JSON file doesn't exist
- **Solution:** Verify `/game/src/data/` contains JSON files

**3. "JSON parse error"**
- **Cause:** JSON file has syntax error (manual editing)
- **Solution:** Use JSON validator (jsonlint.com), fix syntax, re-save

**4. "Validation errors on save"**
- **Cause:** Missing required fields or invalid references
- **Solution:** Check validation panel, fix errors, re-save

**5. "Game not hot-reloading"**
- **Cause:** File watcher not running or game not in dev mode
- **Solution:** Ensure game dev server is running, check console for errors

### 9.5 FAQ

**Q: Can I use Asset Manager on Mac/Linux?**
A: Yes, File System Access API works on Chrome/Edge on all platforms.

**Q: Can multiple people edit content simultaneously?**
A: No (MVP). Use git branches for collaboration. Phase 2+ may add real-time sync.

**Q: What if I prefer manual JSON editing?**
A: That's fine! Asset Manager is optional. Git will track all changes.

**Q: Can I export content for use in other tools?**
A: Yes, use Import/Export feature (Phase 2). JSON files are already portable.

**Q: Does Asset Manager work offline?**
A: Yes, once directory access is granted. No internet required.

**Q: Can I customize validation rules?**
A: Yes, edit `/asset-manager/src/lib/schemas.ts` and balance guidelines.

**Q: What if I break something?**
A: Use git to revert changes: `git checkout src/data/characters.json`

---

## Conclusion

This Asset Manager specification provides a comprehensive blueprint for building a content management tool tailored to your web-based RPG project. The recommended approach—a **React-based web application using File System Access API**—offers the best balance of:

- **Quick Development:** 2-3 weeks for MVP
- **Zero-Touch Workflow:** Reads/writes game JSON files directly
- **Ease of Use:** Form-based editors, real-time validation, preview
- **Integration:** Hot reload with game, git-aware, type-safe
- **Flexibility:** Can extend with advanced features in Phase 2+

**Key Takeaways:**

1. **MVP First:** Focus on core editors (Character, Ability, Item) in first 2-3 weeks
2. **Shared Types:** Use `/shared/types/` to ensure consistency between game and tool
3. **Validation is Critical:** Zod schemas + reference validation prevent 95% of errors
4. **Hot Reload:** Game auto-reloads content on save = instant feedback
5. **Iterative Development:** Start simple, add features based on real usage

**Next Steps:**

1. **Review this spec** with team (if applicable)
2. **Week 3 of game dev:** Start Asset Manager project setup
3. **Week 5:** MVP complete, start using for character creation
4. **Week 13:** Asset Manager critical for M7 (content creation)
5. **Week 15:** Finalize Asset Manager alongside game testing

**Expected Impact:**

- **50% faster content creation** (2 min vs 10+ min per character)
- **95% fewer syntax errors** (validation catches mistakes)
- **Enable collaboration** (non-programmers can contribute)
- **Faster iteration** (preview without launching game)

This tool will be a **force multiplier** for your development process, allowing you to focus on game design and mechanics rather than manual JSON editing.

---

**Document End**
