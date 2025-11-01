# Technical Architecture Blueprint
## Web-Based Hybrid RPG with Zero-Touch Development Workflow

**Project Type:** Browser-based Tactical RPG
**Development Paradigm:** Code-only, zero-touch asset pipeline
**Target Platforms:** Desktop browsers (primary), Mobile browsers (responsive)
**Architecture Philosophy:** Modular, data-driven, scalable

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Technology Stack](#2-technology-stack)
3. [Architecture Patterns](#3-architecture-patterns)
4. [Data Architecture](#4-data-architecture)
5. [Zero-Touch Development Workflow](#5-zero-touch-development-workflow)
6. [System Implementations](#6-system-implementations)
7. [Performance Strategy](#7-performance-strategy)
8. [Development Setup](#8-development-setup)
9. [Deployment Pipeline](#9-deployment-pipeline)
10. [Testing Strategy](#10-testing-strategy)

---

## 1. Executive Summary

### 1.1 Project Overview

This architecture supports a web-based hybrid RPG combining:
- **Fire Emblem-style** city exploration and hub management
- **Final Fantasy-style** turn-based tactical combat
- **Hades-style** roguelike progression and reward systems
- **D&D-style** skill checks and dice mechanics

### 1.2 Core Technical Requirements

**Game Structure:**
- 3-layer navigation: World Map → City Exploration → Tactical Combat
- Complex state management across 50+ characters
- Persistent save system (local + optional cloud)
- Turn-based combat with visual effects and animations
- D&D mechanics implementation (d20 rolls, skill checks, ability scores)

**Development Constraints:**
- Zero manual asset creation (procedural + AI-generated)
- Fully automated CI/CD pipeline
- Code-only workflow
- Scalable architecture for future city additions

### 1.3 Key Architectural Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Game Engine** | Phaser 3 (v3.80+) | Mature ecosystem, excellent WebGL support, strong RPG community |
| **Renderer** | WebGL with Canvas fallback | Performance for particle effects and complex scenes |
| **State Management** | Redux + Immer | Predictable state, time-travel debugging, serialization |
| **Bundler** | Vite | Fast HMR, optimized builds, modern ES modules |
| **Language** | TypeScript | Type safety, better IDE support, maintainability |
| **Save System** | IndexedDB (primary) | No 5MB localStorage limit, async operations |
| **Asset Pipeline** | Procedural generation + Canvas API | Zero-touch, deterministic, version-controllable |
| **Deployment** | GitHub Pages + Actions | Free, automated, version-controlled |

---

## 2. Technology Stack

### 2.1 Core Framework

```typescript
// Phaser 3 Configuration
const gameConfig: Phaser.Types.Core.GameConfig = {
  type: Phaser.AUTO, // Auto-select WebGL with Canvas fallback
  width: 1280,
  height: 720,
  parent: 'game-container',
  backgroundColor: '#000000',
  pixelArt: false, // Smooth scaling for procedural graphics
  physics: {
    default: 'arcade',
    arcade: {
      gravity: { y: 0 }, // Top-down RPG
      debug: false
    }
  },
  scene: [
    BootScene,
    WorldMapScene,
    CityExplorationScene,
    CombatScene,
    MenuScene,
    DialogScene
  ],
  scale: {
    mode: Phaser.Scale.FIT,
    autoCenter: Phaser.Scale.CENTER_BOTH,
    min: {
      width: 640,
      height: 360
    },
    max: {
      width: 1920,
      height: 1080
    }
  },
  render: {
    antialias: true,
    powerPreference: 'high-performance',
    mipmapFilter: 'LINEAR',
    roundPixels: false
  }
};
```

### 2.2 State Management Stack

```typescript
// Redux Store Structure
import { configureStore } from '@reduxjs/toolkit';
import { produce } from 'immer';

// Store slices
const store = configureStore({
  reducer: {
    gameState: gameStateReducer,      // Current game progress
    characters: charactersReducer,    // Party & all characters
    inventory: inventoryReducer,      // Items & equipment
    quests: questsReducer,           // Quest tracking
    combat: combatReducer,           // Battle state
    world: worldReducer,             // Map & location data
    ui: uiReducer,                   // UI state
    settings: settingsReducer        // User preferences
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        // Ignore Phaser objects in actions
        ignoredActions: ['combat/addSprite', 'ui/setScene'],
        ignoredPaths: ['combat.sprites', 'ui.currentScene']
      }
    })
});

// Type-safe hooks
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

### 2.3 Build Tool Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import path from 'path';

export default defineConfig({
  base: './', // Relative paths for GitHub Pages
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    rollupOptions: {
      output: {
        manualChunks: {
          phaser: ['phaser'],
          redux: ['@reduxjs/toolkit', 'immer'],
          ui: ['./src/ui']
        }
      }
    },
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true, // Remove console logs in production
        drop_debugger: true
      }
    }
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@assets': path.resolve(__dirname, './src/assets'),
      '@scenes': path.resolve(__dirname, './src/scenes'),
      '@systems': path.resolve(__dirname, './src/systems'),
      '@data': path.resolve(__dirname, './src/data')
    }
  },
  server: {
    port: 3000,
    open: true
  }
});
```

### 2.4 Development Dependencies

```json
{
  "name": "hybrid-rpg",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "vitest",
    "lint": "eslint src --ext ts,tsx",
    "format": "prettier --write src/**/*.ts"
  },
  "dependencies": {
    "phaser": "^3.80.1",
    "@reduxjs/toolkit": "^2.0.1",
    "immer": "^10.0.3",
    "idb": "^8.0.0",
    "easystarjs": "^0.4.4",
    "seedrandom": "^3.0.5"
  },
  "devDependencies": {
    "typescript": "^5.3.3",
    "vite": "^5.0.8",
    "vitest": "^1.1.0",
    "@types/node": "^20.10.6",
    "eslint": "^8.56.0",
    "prettier": "^3.1.1",
    "terser": "^5.26.0"
  }
}
```

---

## 3. Architecture Patterns

### 3.1 Scene-Based Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    BootScene                            │
│  (Asset generation, data loading, initialization)       │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────┐
│                  WorldMapScene                          │
│  • Metro-style map with nodes                           │
│  • City selection interface                             │
│  • Fast travel system                                   │
└─────────┬────────────────────┬──────────────────────────┘
          │                    │
          ▼                    ▼
┌─────────────────────┐  ┌────────────────────────────────┐
│ CityExplorationScene│  │       MenuScene                │
│ • Top-down movement │  │  • Character management        │
│ • NPC interaction   │  │  • Inventory & equipment       │
│ • Event triggers    │  │  • Quest log                   │
│ • Shop/facilities   │  │  • Save/Load                   │
└──────────┬──────────┘  └────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│                  CombatScene                            │
│  • Turn-based tactical combat                           │
│  • State machine driven                                 │
│  • Animation sequencing                                 │
│  • Reward selection (Hades-style)                       │
└─────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│                 DialogScene (Overlay)                   │
│  • Text display with typewriter effect                  │
│  • Choice selection (D&D skill checks)                  │
│  • Character portraits                                  │
│  • Dice roll animations                                 │
└─────────────────────────────────────────────────────────┘
```

### 3.2 Combat State Machine

```typescript
// Combat State Machine using XState pattern
type CombatState =
  | 'INITIALIZE'
  | 'DETERMINE_TURN_ORDER'
  | 'UNIT_TURN_START'
  | 'PLAYER_SELECT_ACTION'
  | 'PLAYER_SELECT_TARGET'
  | 'EXECUTE_ACTION'
  | 'RESOLVE_EFFECTS'
  | 'CHECK_VICTORY'
  | 'ENEMY_AI_TURN'
  | 'VICTORY'
  | 'DEFEAT';

class CombatStateMachine {
  private state: CombatState;
  private turnQueue: Combatant[];
  private currentActor: Combatant | null;
  private pendingAction: BattleAction | null;

  constructor(private scene: CombatScene) {
    this.state = 'INITIALIZE';
    this.turnQueue = [];
    this.currentActor = null;
    this.pendingAction = null;
  }

  transition(newState: CombatState, data?: any): void {
    console.log(`Combat: ${this.state} -> ${newState}`);
    this.exitState(this.state);
    this.state = newState;
    this.enterState(newState, data);
  }

  private enterState(state: CombatState, data?: any): void {
    switch(state) {
      case 'INITIALIZE':
        this.initializeCombat();
        break;

      case 'DETERMINE_TURN_ORDER':
        this.calculateTurnOrder();
        this.transition('UNIT_TURN_START');
        break;

      case 'UNIT_TURN_START':
        this.currentActor = this.turnQueue.shift() || null;
        if (!this.currentActor) {
          this.transition('DETERMINE_TURN_ORDER');
          return;
        }

        if (this.currentActor.isPlayer) {
          this.transition('PLAYER_SELECT_ACTION');
        } else {
          this.transition('ENEMY_AI_TURN');
        }
        break;

      case 'PLAYER_SELECT_ACTION':
        this.scene.showActionMenu(this.currentActor!);
        break;

      case 'PLAYER_SELECT_TARGET':
        this.scene.showTargetSelection(this.pendingAction!);
        break;

      case 'EXECUTE_ACTION':
        this.executeAction(data.action, data.targets);
        break;

      case 'RESOLVE_EFFECTS':
        this.resolveEffects();
        this.transition('CHECK_VICTORY');
        break;

      case 'CHECK_VICTORY':
        if (this.checkVictory()) {
          this.transition('VICTORY');
        } else if (this.checkDefeat()) {
          this.transition('DEFEAT');
        } else {
          this.transition('UNIT_TURN_START');
        }
        break;

      case 'ENEMY_AI_TURN':
        this.executeEnemyAI();
        break;

      case 'VICTORY':
        this.handleVictory();
        break;

      case 'DEFEAT':
        this.handleDefeat();
        break;
    }
  }

  private exitState(state: CombatState): void {
    switch(state) {
      case 'PLAYER_SELECT_ACTION':
        this.scene.hideActionMenu();
        break;
      case 'PLAYER_SELECT_TARGET':
        this.scene.hideTargetSelection();
        break;
    }
  }

  // Public methods for scene to call
  onActionSelected(action: BattleAction): void {
    this.pendingAction = action;
    if (action.requiresTarget) {
      this.transition('PLAYER_SELECT_TARGET');
    } else {
      this.transition('EXECUTE_ACTION', {
        action,
        targets: [this.currentActor]
      });
    }
  }

  onTargetSelected(targets: Combatant[]): void {
    this.transition('EXECUTE_ACTION', {
      action: this.pendingAction,
      targets
    });
  }

  private calculateTurnOrder(): void {
    // Speed-based turn order with randomization
    this.turnQueue = this.scene.getAllCombatants()
      .filter(c => c.hp > 0)
      .sort((a, b) => {
        const speedDiff = b.stats.speed - a.stats.speed;
        if (speedDiff !== 0) return speedDiff;
        // Tiebreaker: random
        return Math.random() - 0.5;
      });
  }

  private executeAction(action: BattleAction, targets: Combatant[]): void {
    // Queue animation sequence
    this.scene.animationController.queueSequence({
      type: 'action',
      actor: this.currentActor!,
      action,
      targets,
      onComplete: () => {
        this.transition('RESOLVE_EFFECTS');
      }
    });
  }

  private resolveEffects(): void {
    // Apply damage, status effects, etc.
    // This happens after animations complete
  }

  private executeEnemyAI(): void {
    const ai = new EnemyAI(this.currentActor!, this.scene);
    const decision = ai.makeDecision();

    // Slight delay to make AI feel deliberate
    this.scene.time.delayedCall(500, () => {
      this.transition('EXECUTE_ACTION', decision);
    });
  }

  private checkVictory(): boolean {
    return this.scene.getEnemies().every(e => e.hp <= 0);
  }

  private checkDefeat(): boolean {
    return this.scene.getAllies().every(a => a.hp <= 0);
  }

  private handleVictory(): void {
    this.scene.showVictoryScreen();
  }

  private handleDefeat(): void {
    this.scene.showDefeatScreen();
  }
}
```

### 3.3 Event System Architecture

```typescript
// Global event bus for cross-scene communication
class EventBus extends Phaser.Events.EventEmitter {
  private static instance: EventBus;

  static getInstance(): EventBus {
    if (!EventBus.instance) {
      EventBus.instance = new EventBus();
    }
    return EventBus.instance;
  }

  // Type-safe event definitions
  static readonly Events = {
    // State changes
    STATE_CHANGED: 'state:changed',
    SAVE_GAME: 'game:save',
    LOAD_GAME: 'game:load',

    // Character events
    CHARACTER_LEVEL_UP: 'character:levelUp',
    CHARACTER_HP_CHANGED: 'character:hpChanged',
    CHARACTER_DIED: 'character:died',

    // Combat events
    COMBAT_START: 'combat:start',
    COMBAT_END: 'combat:end',
    TURN_START: 'combat:turnStart',
    ACTION_EXECUTED: 'combat:actionExecuted',

    // Quest events
    QUEST_STARTED: 'quest:started',
    QUEST_COMPLETED: 'quest:completed',
    QUEST_FAILED: 'quest:failed',

    // UI events
    DIALOG_START: 'ui:dialogStart',
    DIALOG_END: 'ui:dialogEnd',
    MENU_OPEN: 'ui:menuOpen',
    MENU_CLOSE: 'ui:menuClose',

    // World events
    LOCATION_UNLOCKED: 'world:locationUnlocked',
    LOCATION_ENTERED: 'world:locationEntered',
    EVENT_TRIGGERED: 'world:eventTriggered'
  };
}

// Usage in scenes
class CombatScene extends Phaser.Scene {
  create() {
    const eventBus = EventBus.getInstance();

    // Listen to events
    eventBus.on(EventBus.Events.CHARACTER_DIED, this.onCharacterDied, this);

    // Emit events
    eventBus.emit(EventBus.Events.COMBAT_START, {
      enemies: this.enemies,
      location: this.locationId
    });
  }

  onCharacterDied(data: { character: Character }) {
    console.log(`${data.character.name} has died!`);
    // Handle death
  }

  shutdown() {
    // Clean up listeners
    const eventBus = EventBus.getInstance();
    eventBus.off(EventBus.Events.CHARACTER_DIED, this.onCharacterDied, this);
  }
}
```

### 3.4 Component-Based Entity System

```typescript
// Entity Component System for characters
interface Component {
  type: string;
  update?(deltaTime: number): void;
  destroy?(): void;
}

class Entity {
  private components: Map<string, Component> = new Map();

  constructor(public id: string) {}

  addComponent<T extends Component>(component: T): void {
    this.components.set(component.type, component);
  }

  getComponent<T extends Component>(type: string): T | undefined {
    return this.components.get(type) as T;
  }

  hasComponent(type: string): boolean {
    return this.components.has(type);
  }

  removeComponent(type: string): void {
    const component = this.components.get(type);
    if (component?.destroy) {
      component.destroy();
    }
    this.components.delete(type);
  }

  update(deltaTime: number): void {
    this.components.forEach(component => {
      component.update?.(deltaTime);
    });
  }
}

// Example components
class StatsComponent implements Component {
  type = 'stats';

  constructor(
    public hp: number,
    public maxHp: number,
    public mp: number,
    public maxMp: number,
    public strength: number,
    public dexterity: number,
    public constitution: number,
    public intelligence: number,
    public wisdom: number,
    public charisma: number,
    public speed: number
  ) {}

  getModifier(ability: 'strength' | 'dexterity' | 'constitution' |
                      'intelligence' | 'wisdom' | 'charisma'): number {
    return Math.floor((this[ability] - 10) / 2);
  }
}

class SpriteComponent implements Component {
  type = 'sprite';

  constructor(
    public sprite: Phaser.GameObjects.Sprite,
    public scene: Phaser.Scene
  ) {}

  playAnimation(key: string): void {
    this.sprite.play(key);
  }

  destroy(): void {
    this.sprite.destroy();
  }
}

class StatusEffectComponent implements Component {
  type = 'statusEffects';
  private effects: Map<string, StatusEffect> = new Map();

  addEffect(effect: StatusEffect): void {
    this.effects.set(effect.id, effect);
  }

  removeEffect(id: string): void {
    this.effects.delete(id);
  }

  update(deltaTime: number): void {
    this.effects.forEach((effect, id) => {
      effect.duration -= deltaTime;
      if (effect.duration <= 0) {
        this.removeEffect(id);
      }
    });
  }
}

// Character factory
class CharacterFactory {
  static createCharacter(data: CharacterData, scene: Phaser.Scene): Entity {
    const entity = new Entity(data.id);

    // Add stats component
    entity.addComponent(new StatsComponent(
      data.hp, data.maxHp,
      data.mp, data.maxMp,
      data.strength, data.dexterity, data.constitution,
      data.intelligence, data.wisdom, data.charisma,
      data.speed
    ));

    // Add sprite component
    const sprite = scene.add.sprite(data.x, data.y, data.spriteKey);
    entity.addComponent(new SpriteComponent(sprite, scene));

    // Add status effects component
    entity.addComponent(new StatusEffectComponent());

    return entity;
  }
}
```

---

## 4. Data Architecture

### 4.1 Game State Schema

```typescript
// Root game state structure
interface GameState {
  meta: {
    version: string;
    saveDate: number;
    playTime: number;
    lastScene: string;
  };

  player: PlayerState;
  characters: CharactersState;
  inventory: InventoryState;
  quests: QuestsState;
  world: WorldState;
  combat: CombatState | null;
  flags: Record<string, boolean>;
  variables: Record<string, any>;
}

// Player state
interface PlayerState {
  name: string;
  gold: number;
  reputation: Record<string, number>;
  currentLocation: string;
  partyIds: string[]; // Active party member IDs
}

// Characters state (all recruited characters)
interface CharactersState {
  byId: Record<string, Character>;
  allIds: string[];
  activeParty: string[]; // Max 4-6 characters
  reserve: string[];
}

interface Character {
  id: string;
  name: string;
  class: CharacterClass;
  level: number;
  xp: number;

  // D&D-style ability scores
  abilities: {
    strength: number;
    dexterity: number;
    constitution: number;
    intelligence: number;
    wisdom: number;
    charisma: number;
  };

  // Skills (D&D-style)
  skills: {
    [key in Skill]: {
      proficient: boolean;
      bonus: number;
    }
  };

  // Combat stats
  hp: number;
  maxHp: number;
  mp: number;
  maxMp: number;
  ac: number; // Armor class
  speed: number;
  initiative: number;

  // Equipment
  equipment: {
    weapon: string | null;
    armor: string | null;
    accessory1: string | null;
    accessory2: string | null;
  };

  // Abilities & spells
  abilities: string[];
  spellSlots: {
    level1: { current: number; max: number };
    level2: { current: number; max: number };
    level3: { current: number; max: number };
    level4: { current: number; max: number };
    level5: { current: number; max: number };
  };

  // Support/relationship levels
  supports: Record<string, number>; // characterId -> support level (0-3)

  // Progression
  abilityPoints: number;
  unlockedAbilities: string[];
}

// Inventory state
interface InventoryState {
  items: {
    [itemId: string]: {
      quantity: number;
      equipped: boolean;
    }
  };
  capacity: number;
  currentWeight: number;
}

// Quest state
interface QuestsState {
  active: Quest[];
  completed: string[];
  failed: string[];
}

interface Quest {
  id: string;
  title: string;
  description: string;
  objectives: QuestObjective[];
  rewards: QuestReward[];
  status: 'active' | 'completed' | 'failed';
  giver: string;
  location: string;
}

// World state
interface WorldState {
  currentMap: string;
  unlockedLocations: string[];
  visitedLocations: string[];
  locationStates: Record<string, LocationState>;
  time: {
    day: number;
    hour: number;
  };
}

interface LocationState {
  id: string;
  unlocked: boolean;
  visited: boolean;
  npcs: Record<string, NPCState>;
  events: Record<string, EventState>;
  shops: Record<string, ShopState>;
}

// Combat state (ephemeral, not saved)
interface CombatState {
  location: string;
  enemies: Combatant[];
  allies: Combatant[];
  turnOrder: string[];
  currentTurn: number;
  roundNumber: number;
  temporaryBuffs: Record<string, TemporaryBuff[]>;
}
```

### 4.2 Save System Design

```typescript
// IndexedDB save system
import { openDB, DBSchema, IDBPDatabase } from 'idb';

interface SaveDB extends DBSchema {
  saves: {
    key: string;
    value: SaveData;
    indexes: {
      'by-date': number;
    };
  };
  settings: {
    key: string;
    value: any;
  };
}

class SaveManager {
  private db: IDBPDatabase<SaveDB> | null = null;
  private readonly DB_NAME = 'hybrid-rpg-saves';
  private readonly DB_VERSION = 1;

  async initialize(): Promise<void> {
    this.db = await openDB<SaveDB>(this.DB_NAME, this.DB_VERSION, {
      upgrade(db) {
        // Create saves store
        const saveStore = db.createObjectStore('saves', {
          keyPath: 'id'
        });
        saveStore.createIndex('by-date', 'timestamp');

        // Create settings store
        db.createObjectStore('settings');
      }
    });
  }

  async saveGame(slot: string, gameState: GameState): Promise<void> {
    if (!this.db) throw new Error('Database not initialized');

    const saveData: SaveData = {
      id: slot,
      timestamp: Date.now(),
      gameState: this.serializeGameState(gameState),
      screenshot: await this.captureScreenshot()
    };

    await this.db.put('saves', saveData);

    // Also save to localStorage as backup (compressed)
    try {
      const compressed = this.compress(saveData);
      localStorage.setItem(`save-${slot}`, compressed);
    } catch (e) {
      console.warn('localStorage save failed:', e);
    }
  }

  async loadGame(slot: string): Promise<GameState | null> {
    if (!this.db) throw new Error('Database not initialized');

    try {
      const saveData = await this.db.get('saves', slot);
      if (!saveData) return null;

      return this.deserializeGameState(saveData.gameState);
    } catch (e) {
      // Fallback to localStorage
      console.warn('IndexedDB load failed, trying localStorage:', e);
      const compressed = localStorage.getItem(`save-${slot}`);
      if (!compressed) return null;

      const saveData = this.decompress(compressed);
      return this.deserializeGameState(saveData.gameState);
    }
  }

  async listSaves(): Promise<SaveInfo[]> {
    if (!this.db) throw new Error('Database not initialized');

    const saves = await this.db.getAllFromIndex('saves', 'by-date');
    return saves.map(save => ({
      id: save.id,
      timestamp: save.timestamp,
      screenshot: save.screenshot,
      playerName: save.gameState.player.name,
      playTime: save.gameState.meta.playTime,
      location: save.gameState.player.currentLocation
    }));
  }

  async deleteSave(slot: string): Promise<void> {
    if (!this.db) throw new Error('Database not initialized');

    await this.db.delete('saves', slot);
    localStorage.removeItem(`save-${slot}`);
  }

  async autoSave(gameState: GameState): Promise<void> {
    await this.saveGame('autosave', gameState);
  }

  private serializeGameState(state: GameState): string {
    // Use Immer's produce to ensure clean serialization
    const cleanState = produce(state, draft => {
      // Remove non-serializable data
      draft.combat = null;
    });

    return JSON.stringify(cleanState);
  }

  private deserializeGameState(json: string): GameState {
    return JSON.parse(json);
  }

  private async captureScreenshot(): Promise<string> {
    // Capture current game canvas as base64 PNG
    const game = (window as any).game as Phaser.Game;
    const canvas = game.canvas;
    return canvas.toDataURL('image/png');
  }

  private compress(data: SaveData): string {
    // Simple LZ-string compression for localStorage
    const json = JSON.stringify(data);
    // In production, use: import LZString from 'lz-string';
    // return LZString.compress(json);
    return json; // Placeholder
  }

  private decompress(compressed: string): SaveData {
    // return JSON.parse(LZString.decompress(compressed));
    return JSON.parse(compressed); // Placeholder
  }
}

// Cloud save integration (optional)
class CloudSaveManager {
  constructor(private backend: 'firebase' | 'supabase') {}

  async syncSave(slot: string, saveData: SaveData): Promise<void> {
    // Upload to cloud backend
    // Only sync when user is authenticated
  }

  async downloadSave(slot: string): Promise<SaveData | null> {
    // Download from cloud backend
    return null;
  }

  async listCloudSaves(): Promise<SaveInfo[]> {
    // List saves from cloud
    return [];
  }
}
```

### 4.3 Data-Driven Content Definition

```typescript
// All game content defined in JSON files
// src/data/characters.json
{
  "fighter_01": {
    "id": "fighter_01",
    "name": "Aria Stormwind",
    "class": "fighter",
    "baseStats": {
      "strength": 16,
      "dexterity": 12,
      "constitution": 14,
      "intelligence": 10,
      "wisdom": 11,
      "charisma": 8
    },
    "startingLevel": 1,
    "startingEquipment": ["longsword", "chainmail", "shield"],
    "proficiencies": {
      "skills": ["athletics", "intimidation"],
      "saves": ["strength", "constitution"],
      "weapons": ["simple", "martial"],
      "armor": ["light", "medium", "heavy", "shields"]
    },
    "abilities": ["second_wind", "action_surge"],
    "visualData": {
      "procedural": {
        "bodyType": "athletic",
        "hairColor": "#3d2817",
        "skinTone": "#d4a57a",
        "height": 1.75,
        "equipment": true
      }
    }
  }
}

// src/data/abilities.json
{
  "second_wind": {
    "id": "second_wind",
    "name": "Second Wind",
    "description": "Regain HP equal to 1d10 + Fighter level",
    "type": "bonus_action",
    "cost": { "type": "uses_per_rest", "uses": 1, "restType": "short" },
    "effects": [
      {
        "type": "heal",
        "formula": "1d10 + @level",
        "target": "self"
      }
    ],
    "animation": "heal_glow"
  },
  "power_attack": {
    "id": "power_attack",
    "name": "Power Attack",
    "description": "A powerful melee attack dealing extra damage",
    "type": "action",
    "cost": { "type": "stamina", "amount": 2 },
    "targetType": "single_enemy",
    "range": 1,
    "effects": [
      {
        "type": "damage",
        "damageType": "physical",
        "formula": "(@weaponDamage * 1.5) + @strengthMod",
        "target": "selected"
      }
    ],
    "animation": "sword_slash_heavy",
    "hitEffects": ["screen_shake", "impact_flash"]
  }
}

// src/data/enemies.json
{
  "goblin_warrior": {
    "id": "goblin_warrior",
    "name": "Goblin Warrior",
    "type": "humanoid",
    "stats": {
      "hp": 15,
      "ac": 13,
      "strength": 8,
      "dexterity": 14,
      "constitution": 10,
      "intelligence": 8,
      "wisdom": 8,
      "charisma": 6,
      "speed": 30
    },
    "actions": ["shortsword_attack", "shortbow_attack"],
    "ai": {
      "type": "aggressive",
      "preferredRange": "melee",
      "fleeThreshold": 0.25
    },
    "rewards": {
      "xp": 50,
      "gold": [5, 15],
      "loot": [
        { "item": "shortsword", "chance": 0.3 },
        { "item": "leather_armor", "chance": 0.2 }
      ]
    },
    "visualData": {
      "procedural": {
        "species": "goblin",
        "skinTone": "#7a9b65",
        "height": 1.0,
        "equipment": ["leather_scraps", "rusty_blade"]
      }
    }
  }
}

// src/data/quests.json
{
  "main_001": {
    "id": "main_001",
    "type": "main",
    "title": "The Calling",
    "description": "Investigate strange energy readings in the Solar District",
    "giver": "npc_council_elder",
    "location": "hub_city",
    "prerequisites": [],
    "objectives": [
      {
        "id": "obj_001",
        "type": "talk_to_npc",
        "target": "npc_scientist_maya",
        "description": "Speak with Dr. Maya in the Research Lab"
      },
      {
        "id": "obj_002",
        "type": "explore_location",
        "target": "solar_district_ruins",
        "description": "Explore the Solar District ruins"
      },
      {
        "id": "obj_003",
        "type": "defeat_enemies",
        "target": "corrupted_construct",
        "count": 3,
        "description": "Defeat 3 Corrupted Constructs"
      }
    ],
    "rewards": {
      "xp": 500,
      "gold": 200,
      "items": ["solar_charm"],
      "unlocks": ["location_solar_district"]
    },
    "branches": [
      {
        "condition": { "skill_check": "investigation", "dc": 15 },
        "objectiveId": "obj_002",
        "alternatePath": "obj_002_alternate",
        "description": "Find hidden entrance",
        "bonusRewards": { "xp": 100, "items": ["ancient_key"] }
      }
    ]
  }
}
```

---

## 5. Zero-Touch Development Workflow

### 5.1 Procedural Asset Generation

```typescript
// Procedural sprite generator using Canvas API
class ProceduralSpriteGenerator {
  private canvas: HTMLCanvasElement;
  private ctx: CanvasRenderingContext2D;

  constructor(width: number = 64, height: number = 64) {
    this.canvas = document.createElement('canvas');
    this.canvas.width = width;
    this.canvas.height = height;
    this.ctx = this.canvas.getContext('2d')!;
  }

  /**
   * Generate a character sprite from definition
   * Uses deterministic random seed for consistency
   */
  generateCharacterSprite(data: CharacterVisualData, seed: string): HTMLCanvasElement {
    const rng = seedrandom(seed);
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

    // Draw character layers
    this.drawBody(data, rng);
    this.drawEquipment(data, rng);
    this.drawFace(data, rng);

    return this.canvas;
  }

  private drawBody(data: CharacterVisualData, rng: () => number): void {
    const { bodyType, skinTone, height } = data.procedural;

    // Body (simplified for demonstration)
    this.ctx.fillStyle = skinTone;
    this.ctx.beginPath();
    this.ctx.ellipse(32, 40, 10, 15 * height, 0, 0, Math.PI * 2);
    this.ctx.fill();

    // Head
    this.ctx.beginPath();
    this.ctx.arc(32, 20, 8, 0, Math.PI * 2);
    this.ctx.fill();
  }

  private drawEquipment(data: CharacterVisualData, rng: () => number): void {
    if (!data.procedural.equipment) return;

    // Draw armor/clothing based on class
    this.ctx.fillStyle = this.getEquipmentColor(data);
    this.ctx.fillRect(27, 35, 10, 12); // Simple torso armor
  }

  private drawFace(data: CharacterVisualData, rng: () => number): void {
    // Eyes
    this.ctx.fillStyle = '#000000';
    this.ctx.fillRect(29, 19, 2, 2);
    this.ctx.fillRect(35, 19, 2, 2);
  }

  private getEquipmentColor(data: CharacterVisualData): string {
    // Color coding by class/role
    const colorMap: Record<string, string> = {
      fighter: '#8B4513',
      mage: '#4169E1',
      cleric: '#FFD700',
      rogue: '#2F4F4F'
    };
    return colorMap[data.class] || '#808080';
  }

  /**
   * Generate particle texture for effects
   */
  generateParticleTexture(type: 'fire' | 'ice' | 'lightning' | 'heal'): HTMLCanvasElement {
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

    const gradient = this.ctx.createRadialGradient(32, 32, 0, 32, 32, 32);

    switch(type) {
      case 'fire':
        gradient.addColorStop(0, '#FF4500');
        gradient.addColorStop(0.5, '#FF8C00');
        gradient.addColorStop(1, 'rgba(255, 69, 0, 0)');
        break;
      case 'ice':
        gradient.addColorStop(0, '#00FFFF');
        gradient.addColorStop(0.5, '#4682B4');
        gradient.addColorStop(1, 'rgba(70, 130, 180, 0)');
        break;
      case 'lightning':
        gradient.addColorStop(0, '#FFFFFF');
        gradient.addColorStop(0.5, '#FFD700');
        gradient.addColorStop(1, 'rgba(255, 215, 0, 0)');
        break;
      case 'heal':
        gradient.addColorStop(0, '#00FF00');
        gradient.addColorStop(0.5, '#32CD32');
        gradient.addColorStop(1, 'rgba(50, 205, 50, 0)');
        break;
    }

    this.ctx.fillStyle = gradient;
    this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);

    return this.canvas;
  }

  /**
   * Generate UI element (button, panel, etc.)
   */
  generateUIElement(type: 'button' | 'panel', width: number, height: number): HTMLCanvasElement {
    const canvas = document.createElement('canvas');
    canvas.width = width;
    canvas.height = height;
    const ctx = canvas.getContext('2d')!;

    if (type === 'button') {
      // Gradient button
      const gradient = ctx.createLinearGradient(0, 0, 0, height);
      gradient.addColorStop(0, '#4a5568');
      gradient.addColorStop(1, '#2d3748');
      ctx.fillStyle = gradient;

      this.roundRect(ctx, 0, 0, width, height, 8);
      ctx.fill();

      // Border
      ctx.strokeStyle = '#718096';
      ctx.lineWidth = 2;
      ctx.stroke();
    } else if (type === 'panel') {
      // Semi-transparent panel
      ctx.fillStyle = 'rgba(26, 32, 44, 0.9)';
      this.roundRect(ctx, 0, 0, width, height, 12);
      ctx.fill();

      // Border
      ctx.strokeStyle = '#4a5568';
      ctx.lineWidth = 2;
      ctx.stroke();
    }

    return canvas;
  }

  private roundRect(
    ctx: CanvasRenderingContext2D,
    x: number,
    y: number,
    width: number,
    height: number,
    radius: number
  ): void {
    ctx.beginPath();
    ctx.moveTo(x + radius, y);
    ctx.lineTo(x + width - radius, y);
    ctx.quadraticCurveTo(x + width, y, x + width, y + radius);
    ctx.lineTo(x + width, y + height - radius);
    ctx.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
    ctx.lineTo(x + radius, y + height);
    ctx.quadraticCurveTo(x, y + height, x, y + height - radius);
    ctx.lineTo(x, y + radius);
    ctx.quadraticCurveTo(x, y, x + radius, y);
    ctx.closePath();
  }

  /**
   * Convert canvas to Phaser texture
   */
  toPhaserTexture(scene: Phaser.Scene, key: string, canvas: HTMLCanvasElement): void {
    if (scene.textures.exists(key)) {
      scene.textures.remove(key);
    }
    scene.textures.addCanvas(key, canvas);
  }
}

// Asset generation during boot
class BootScene extends Phaser.Scene {
  constructor() {
    super({ key: 'BootScene' });
  }

  preload(): void {
    // Show loading bar
    this.createLoadingBar();
  }

  create(): void {
    this.generateAssets();
  }

  private generateAssets(): void {
    const generator = new ProceduralSpriteGenerator(64, 64);

    // Load character data
    const characters = this.cache.json.get('characters');

    // Generate sprites for all characters
    Object.values(characters).forEach((char: any) => {
      const canvas = generator.generateCharacterSprite(
        char.visualData,
        char.id // Use ID as seed for consistency
      );
      generator.toPhaserTexture(this, `char_${char.id}`, canvas);

      // Create animation frames (idle, walk, attack, etc.)
      this.generateAnimationFrames(generator, char);
    });

    // Generate particle textures
    ['fire', 'ice', 'lightning', 'heal'].forEach(type => {
      const canvas = generator.generateParticleTexture(type as any);
      generator.toPhaserTexture(this, `particle_${type}`, canvas);
    });

    // Generate UI elements
    const buttonCanvas = generator.generateUIElement('button', 200, 50);
    generator.toPhaserTexture(this, 'ui_button', buttonCanvas);

    const panelCanvas = generator.generateUIElement('panel', 400, 300);
    generator.toPhaserTexture(this, 'ui_panel', panelCanvas);

    // Proceed to main menu
    this.scene.start('WorldMapScene');
  }

  private generateAnimationFrames(generator: ProceduralSpriteGenerator, char: any): void {
    // Generate multiple frames for animations
    // This is a simplified version - full implementation would generate
    // multiple poses for walk cycles, attacks, etc.

    const frames = ['idle', 'walk_0', 'walk_1', 'attack_0', 'attack_1', 'hurt', 'dead'];
    frames.forEach((frame, index) => {
      // Modify character data slightly for each frame
      const frameData = { ...char.visualData };
      // Add frame-specific modifications here

      const canvas = generator.generateCharacterSprite(
        frameData,
        `${char.id}_${frame}`
      );
      generator.toPhaserTexture(this, `char_${char.id}_${frame}`, canvas);
    });
  }

  private createLoadingBar(): void {
    const width = this.cameras.main.width;
    const height = this.cameras.main.height;

    const progressBar = this.add.graphics();
    const progressBox = this.add.graphics();
    progressBox.fillStyle(0x222222, 0.8);
    progressBox.fillRect(width / 2 - 160, height / 2 - 25, 320, 50);

    const loadingText = this.add.text(width / 2, height / 2 - 50, 'Generating Assets...', {
      fontSize: '20px',
      color: '#ffffff'
    }).setOrigin(0.5);

    this.load.on('progress', (value: number) => {
      progressBar.clear();
      progressBar.fillStyle(0xffffff, 1);
      progressBar.fillRect(width / 2 - 150, height / 2 - 15, 300 * value, 30);
    });

    this.load.on('complete', () => {
      progressBar.destroy();
      progressBox.destroy();
      loadingText.destroy();
    });
  }
}
```

### 5.2 AI API Integration (Optional Enhancement)

```typescript
// AI-powered asset generation using external APIs
class AIAssetGenerator {
  private apiKey: string;
  private provider: 'openai' | 'stability' | 'replicate';

  constructor(provider: 'openai' | 'stability' | 'replicate', apiKey: string) {
    this.provider = provider;
    this.apiKey = apiKey;
  }

  /**
   * Generate character portrait using AI
   * Fallback to procedural if API unavailable
   */
  async generateCharacterPortrait(character: Character): Promise<HTMLImageElement> {
    if (!this.apiKey) {
      return this.fallbackGeneration(character);
    }

    try {
      const prompt = this.buildCharacterPrompt(character);
      const imageUrl = await this.requestAIImage(prompt);

      return new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = () => resolve(img);
        img.onerror = () => reject();
        img.src = imageUrl;
      });
    } catch (error) {
      console.warn('AI generation failed, using procedural fallback:', error);
      return this.fallbackGeneration(character);
    }
  }

  private buildCharacterPrompt(character: Character): string {
    // Build detailed prompt from character data
    return `
      A portrait of a ${character.class} character named ${character.name}.
      Style: pixel art, 64x64, RPG character portrait.
      Appearance: ${this.describeAppearance(character)}.
      Equipment: ${this.describeEquipment(character)}.
      Mood: determined, heroic.
      Background: transparent.
    `.trim();
  }

  private describeAppearance(character: Character): string {
    // Convert stats/data to descriptive text
    const traits = [];

    if (character.abilities.strength > 15) traits.push('muscular build');
    if (character.abilities.dexterity > 15) traits.push('agile posture');
    if (character.abilities.intelligence > 15) traits.push('scholarly appearance');

    return traits.join(', ') || 'balanced appearance';
  }

  private describeEquipment(character: Character): string {
    const equipment = [];

    if (character.equipment.weapon) equipment.push(character.equipment.weapon);
    if (character.equipment.armor) equipment.push(character.equipment.armor);

    return equipment.join(', ') || 'basic adventurer gear';
  }

  private async requestAIImage(prompt: string): Promise<string> {
    // API call to generate image
    // Implementation depends on provider

    switch(this.provider) {
      case 'openai':
        return this.requestOpenAI(prompt);
      case 'stability':
        return this.requestStabilityAI(prompt);
      case 'replicate':
        return this.requestReplicate(prompt);
      default:
        throw new Error('Unknown provider');
    }
  }

  private async requestOpenAI(prompt: string): Promise<string> {
    const response = await fetch('https://api.openai.com/v1/images/generations', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.apiKey}`
      },
      body: JSON.stringify({
        model: 'dall-e-3',
        prompt: prompt,
        n: 1,
        size: '1024x1024',
        style: 'vivid'
      })
    });

    const data = await response.json();
    return data.data[0].url;
  }

  private fallbackGeneration(character: Character): HTMLImageElement {
    // Use procedural generation as fallback
    const generator = new ProceduralSpriteGenerator();
    const canvas = generator.generateCharacterSprite(
      character.visualData,
      character.id
    );

    const img = new Image();
    img.src = canvas.toDataURL();
    return img;
  }

  /**
   * Generate effect descriptions using AI
   */
  async generateAbilityDescription(ability: Ability): Promise<string> {
    if (!this.apiKey) {
      return ability.description;
    }

    try {
      // Use GPT to generate flavorful description
      const response = await fetch('https://api.openai.com/v1/chat/completions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.apiKey}`
        },
        body: JSON.stringify({
          model: 'gpt-4',
          messages: [
            {
              role: 'system',
              content: 'You are a creative writer for RPG ability descriptions. Write concise, flavorful descriptions (max 50 words).'
            },
            {
              role: 'user',
              content: `Describe this RPG ability:\nName: ${ability.name}\nMechanics: ${ability.effects.map(e => e.type).join(', ')}`
            }
          ],
          max_tokens: 100
        })
      });

      const data = await response.json();
      return data.choices[0].message.content;
    } catch (error) {
      return ability.description;
    }
  }
}
```

### 5.3 Build Automation

```yaml
# .github/workflows/build-and-deploy.yml
name: Build and Deploy

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Build project
        run: npm run build
        env:
          NODE_ENV: production

      - name: Generate game data
        run: npm run generate:data

      - name: Optimize assets
        run: npm run optimize:assets

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
          retention-days: 7

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
          cname: yourgame.com  # Optional custom domain

      - name: Create release (on tag)
        if: startsWith(github.ref, 'refs/tags/')
        uses: softprops/action-gh-release@v1
        with:
          files: dist/**
          generate_release_notes: true

  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json

      - name: Run E2E tests
        run: npm run test:e2e
```

### 5.4 Procedural Data Generation Scripts

```typescript
// scripts/generate-game-data.ts
// Run during build: npm run generate:data

import * as fs from 'fs';
import * as path from 'path';
import seedrandom from 'seedrandom';

// Generate balanced character stats
function generateCharacterStats(className: string, seed: string): CharacterStats {
  const rng = seedrandom(seed);

  // Base stats by class
  const baseStats: Record<string, Partial<CharacterStats>> = {
    fighter: {
      strength: 16,
      dexterity: 12,
      constitution: 14,
      intelligence: 10,
      wisdom: 11,
      charisma: 8
    },
    mage: {
      strength: 8,
      dexterity: 12,
      constitution: 10,
      intelligence: 16,
      wisdom: 14,
      charisma: 11
    },
    cleric: {
      strength: 12,
      dexterity: 10,
      constitution: 14,
      intelligence: 11,
      wisdom: 16,
      charisma: 12
    },
    rogue: {
      strength: 10,
      dexterity: 16,
      constitution: 12,
      intelligence: 12,
      wisdom: 11,
      charisma: 14
    }
  };

  const base = baseStats[className] || {};

  // Add minor random variation (±2)
  return {
    strength: (base.strength || 10) + Math.floor(rng() * 5) - 2,
    dexterity: (base.dexterity || 10) + Math.floor(rng() * 5) - 2,
    constitution: (base.constitution || 10) + Math.floor(rng() * 5) - 2,
    intelligence: (base.intelligence || 10) + Math.floor(rng() * 5) - 2,
    wisdom: (base.wisdom || 10) + Math.floor(rng() * 5) - 2,
    charisma: (base.charisma || 10) + Math.floor(rng() * 5) - 2
  };
}

// Generate loot tables
function generateLootTable(enemyType: string, difficulty: number): LootTable {
  const rng = seedrandom(`${enemyType}_${difficulty}`);

  const baseGold = difficulty * 10;
  const goldRange = [
    Math.floor(baseGold * 0.5),
    Math.floor(baseGold * 1.5)
  ];

  return {
    gold: goldRange,
    items: [
      {
        item: 'health_potion',
        chance: 0.3 + (difficulty * 0.05)
      },
      {
        item: `${enemyType}_trophy`,
        chance: 0.1
      }
    ]
  };
}

// Generate encounter data
function generateEncounters(count: number = 50): void {
  const encounters = [];

  for (let i = 0; i < count; i++) {
    const difficulty = Math.floor(i / 10) + 1; // Gradually increase difficulty
    const enemyCount = Math.min(2 + Math.floor(difficulty / 2), 6);

    encounters.push({
      id: `encounter_${i.toString().padStart(3, '0')}`,
      difficulty,
      enemies: generateEnemyComposition(enemyCount, difficulty),
      environment: selectRandomEnvironment(),
      rewards: {
        xp: difficulty * 100,
        gold: [difficulty * 50, difficulty * 150],
        loot: generateLootTable('mixed', difficulty)
      }
    });
  }

  // Write to file
  const outputPath = path.join(__dirname, '../src/data/encounters.json');
  fs.writeFileSync(outputPath, JSON.stringify(encounters, null, 2));
  console.log(`Generated ${count} encounters -> ${outputPath}`);
}

function generateEnemyComposition(count: number, difficulty: number): Array<{type: string, level: number}> {
  const enemyTypes = ['goblin', 'orc', 'skeleton', 'wolf', 'bandit'];
  const composition = [];

  for (let i = 0; i < count; i++) {
    composition.push({
      type: enemyTypes[Math.floor(Math.random() * enemyTypes.length)],
      level: difficulty
    });
  }

  return composition;
}

function selectRandomEnvironment(): string {
  const environments = ['forest', 'cave', 'ruins', 'plains', 'dungeon'];
  return environments[Math.floor(Math.random() * environments.length)];
}

// Main generation function
function generateAllData(): void {
  console.log('Generating game data...');

  generateEncounters(50);

  // Generate more data types as needed
  // generateQuests(20);
  // generateItems(100);
  // generateNPCs(30);

  console.log('Game data generation complete!');
}

// Run if called directly
if (require.main === module) {
  generateAllData();
}
```

---

## 6. System Implementations

### 6.1 World Map Navigation System

```typescript
// Metro-style world map with node-based navigation
class WorldMapScene extends Phaser.Scene {
  private locations: LocationNode[] = [];
  private connections: Connection[] = [];
  private playerPosition: string = 'hub_city';

  constructor() {
    super({ key: 'WorldMapScene' });
  }

  create(): void {
    this.add.text(640, 50, 'World Map', {
      fontSize: '32px',
      color: '#ffffff'
    }).setOrigin(0.5);

    this.loadWorldData();
    this.renderMap();
    this.setupInput();
  }

  private loadWorldData(): void {
    const worldData = this.cache.json.get('world');

    this.locations = worldData.locations.map((loc: any) => ({
      id: loc.id,
      name: loc.name,
      x: loc.x,
      y: loc.y,
      unlocked: this.isLocationUnlocked(loc.id),
      type: loc.type,
      description: loc.description
    }));

    this.connections = worldData.connections;
  }

  private isLocationUnlocked(locationId: string): boolean {
    const gameState = store.getState();
    return gameState.world.unlockedLocations.includes(locationId);
  }

  private renderMap(): void {
    const graphics = this.add.graphics();

    // Draw connections (metro lines)
    this.connections.forEach(conn => {
      const from = this.locations.find(l => l.id === conn.from)!;
      const to = this.locations.find(l => l.id === conn.to)!;

      // Only draw if both locations are unlocked
      if (!from.unlocked || !to.unlocked) return;

      graphics.lineStyle(4, 0x4a5568, 1);
      graphics.beginPath();
      graphics.moveTo(from.x, from.y);
      graphics.lineTo(to.x, to.y);
      graphics.strokePath();
    });

    // Draw location nodes
    this.locations.forEach(location => {
      this.renderLocationNode(location);
    });

    // Draw player indicator
    this.renderPlayerPosition();
  }

  private renderLocationNode(location: LocationNode): void {
    const color = location.unlocked ? 0x48bb78 : 0x4a5568;
    const size = location.id === this.playerPosition ? 25 : 20;

    // Node circle
    const circle = this.add.circle(location.x, location.y, size, color);
    circle.setStrokeStyle(3, 0xffffff);

    if (location.unlocked) {
      circle.setInteractive({ useHandCursor: true });
      circle.on('pointerover', () => this.showLocationInfo(location));
      circle.on('pointerout', () => this.hideLocationInfo());
      circle.on('pointerdown', () => this.selectLocation(location));
    }

    // Location name
    const label = this.add.text(
      location.x,
      location.y + 35,
      location.name,
      {
        fontSize: '14px',
        color: location.unlocked ? '#ffffff' : '#718096',
        backgroundColor: 'rgba(0, 0, 0, 0.7)',
        padding: { x: 6, y: 4 }
      }
    ).setOrigin(0.5);
  }

  private renderPlayerPosition(): void {
    const currentLoc = this.locations.find(l => l.id === this.playerPosition);
    if (!currentLoc) return;

    // Pulsing indicator
    const indicator = this.add.circle(currentLoc.x, currentLoc.y, 30, 0xffd700, 0.3);

    this.tweens.add({
      targets: indicator,
      scale: { from: 1, to: 1.3 },
      alpha: { from: 0.3, to: 0 },
      duration: 1000,
      repeat: -1,
      ease: 'Sine.easeInOut'
    });
  }

  private showLocationInfo(location: LocationNode): void {
    // Show info panel
    const panel = this.add.rectangle(
      location.x,
      location.y - 80,
      250,
      80,
      0x1a202c,
      0.95
    );
    panel.setStrokeStyle(2, 0x4a5568);
    panel.setName('infoPanel');

    const title = this.add.text(
      location.x,
      location.y - 95,
      location.name,
      { fontSize: '18px', color: '#f7fafc', fontStyle: 'bold' }
    ).setOrigin(0.5);
    title.setName('infoTitle');

    const desc = this.add.text(
      location.x,
      location.y - 65,
      location.description,
      { fontSize: '12px', color: '#cbd5e0', wordWrap: { width: 230 } }
    ).setOrigin(0.5, 0);
    desc.setName('infoDesc');
  }

  private hideLocationInfo(): void {
    this.children.getByName('infoPanel')?.destroy();
    this.children.getByName('infoTitle')?.destroy();
    this.children.getByName('infoDesc')?.destroy();
  }

  private selectLocation(location: LocationNode): void {
    if (!location.unlocked) return;
    if (location.id === this.playerPosition) {
      // Already here, enter location
      this.enterLocation(location);
      return;
    }

    // Check if location is accessible from current position
    if (!this.isAccessibleFrom(this.playerPosition, location.id)) {
      this.showMessage('Cannot reach that location from here!');
      return;
    }

    // Travel to location
    this.travelToLocation(location);
  }

  private isAccessibleFrom(from: string, to: string): boolean {
    return this.connections.some(conn =>
      (conn.from === from && conn.to === to) ||
      (conn.to === from && conn.from === to)
    );
  }

  private travelToLocation(location: LocationNode): void {
    // Show travel animation/transition
    this.cameras.main.fadeOut(500, 0, 0, 0);

    this.time.delayedCall(500, () => {
      this.playerPosition = location.id;
      store.dispatch(updatePlayerLocation(location.id));

      this.cameras.main.fadeIn(500, 0, 0, 0);
      this.scene.restart(); // Refresh map
    });
  }

  private enterLocation(location: LocationNode): void {
    // Transition to exploration scene
    this.cameras.main.fadeOut(300, 0, 0, 0);

    this.time.delayedCall(300, () => {
      if (location.type === 'city') {
        this.scene.start('CityExplorationScene', { locationId: location.id });
      } else if (location.type === 'combat') {
        this.scene.start('CombatScene', { locationId: location.id });
      }
    });
  }

  private setupInput(): void {
    // ESC to open menu
    this.input.keyboard?.on('keydown-ESC', () => {
      this.scene.launch('MenuScene');
      this.scene.pause();
    });
  }

  private showMessage(text: string): void {
    const message = this.add.text(640, 600, text, {
      fontSize: '16px',
      color: '#fc8181',
      backgroundColor: 'rgba(0, 0, 0, 0.8)',
      padding: { x: 12, y: 8 }
    }).setOrigin(0.5);

    this.tweens.add({
      targets: message,
      alpha: 0,
      y: 550,
      duration: 2000,
      ease: 'Power2',
      onComplete: () => message.destroy()
    });
  }
}

interface LocationNode {
  id: string;
  name: string;
  x: number;
  y: number;
  unlocked: boolean;
  type: 'city' | 'combat' | 'event';
  description: string;
}

interface Connection {
  from: string;
  to: string;
  travelTime: number;
}
```

### 6.2 City Exploration Engine

```typescript
// Top-down exploration with Fire Emblem-style hub mechanics
class CityExplorationScene extends Phaser.Scene {
  private player!: Phaser.Physics.Arcade.Sprite;
  private map!: Phaser.Tilemaps.Tilemap;
  private cursors!: Phaser.Types.Input.Keyboard.CursorKeys;
  private interactionKey!: Phaser.Input.Keyboard.Key;
  private nearbyNPC: NPC | null = null;
  private activityPoints: number = 5;

  constructor() {
    super({ key: 'CityExplorationScene' });
  }

  init(data: { locationId: string }): void {
    this.locationId = data.locationId;
  }

  create(): void {
    this.createMap();
    this.createPlayer();
    this.createNPCs();
    this.createFacilities();
    this.setupInput();
    this.createUI();
  }

  private createMap(): void {
    // Generate procedural city layout
    const generator = new CityLayoutGenerator(this, 40, 30);
    this.map = generator.generate(this.locationId);

    // Set collision
    const collisionLayer = this.map.getLayer('Collision');
    if (collisionLayer) {
      collisionLayer.tilemapLayer.setCollisionByProperty({ collides: true });
    }
  }

  private createPlayer(): void {
    const spawnPoint = this.map.findObject('Objects', obj => obj.name === 'PlayerSpawn');

    this.player = this.physics.add.sprite(
      spawnPoint?.x || 400,
      spawnPoint?.y || 300,
      'player'
    );

    this.player.setCollideWorldBounds(true);

    // Camera follow player
    this.cameras.main.startFollow(this.player, true, 0.1, 0.1);
    this.cameras.main.setZoom(1.5);

    // Collision with walls
    const collisionLayer = this.map.getLayer('Collision')?.tilemapLayer;
    if (collisionLayer) {
      this.physics.add.collider(this.player, collisionLayer);
    }

    // Create player animations
    this.createPlayerAnimations();
  }

  private createPlayerAnimations(): void {
    const directions = ['down', 'up', 'left', 'right'];

    directions.forEach(dir => {
      this.anims.create({
        key: `walk_${dir}`,
        frames: this.anims.generateFrameNumbers('player', {
          start: 0,
          end: 3
        }),
        frameRate: 10,
        repeat: -1
      });
    });
  }

  private createNPCs(): void {
    const locationState = store.getState().world.locationStates[this.locationId];

    Object.values(locationState.npcs).forEach(npcData => {
      const npc = new NPC(this, npcData);

      // Overlap detection for interaction
      this.physics.add.overlap(
        this.player,
        npc.sprite,
        () => this.onNearNPC(npc),
        undefined,
        this
      );
    });
  }

  private createFacilities(): void {
    const facilities = this.map.filterObjects('Facilities', obj => obj.type === 'facility');

    facilities.forEach(facilityObj => {
      const zone = this.add.zone(
        facilityObj.x!,
        facilityObj.y!,
        facilityObj.width,
        facilityObj.height
      );

      this.physics.add.existing(zone);

      // Icon above facility
      const icon = this.add.image(
        facilityObj.x!,
        facilityObj.y! - 40,
        `icon_${facilityObj.name}`
      );

      this.physics.add.overlap(
        this.player,
        zone,
        () => this.onNearFacility(facilityObj.name!),
        undefined,
        this
      );
    });
  }

  update(time: number, delta: number): void {
    this.handlePlayerMovement();
    this.updateUI();
  }

  private handlePlayerMovement(): void {
    const speed = 160;
    let moving = false;

    this.player.setVelocity(0);

    if (this.cursors.left.isDown) {
      this.player.setVelocityX(-speed);
      this.player.play('walk_left', true);
      moving = true;
    } else if (this.cursors.right.isDown) {
      this.player.setVelocityX(speed);
      this.player.play('walk_right', true);
      moving = true;
    }

    if (this.cursors.up.isDown) {
      this.player.setVelocityY(-speed);
      this.player.play('walk_up', true);
      moving = true;
    } else if (this.cursors.down.isDown) {
      this.player.setVelocityY(speed);
      this.player.play('walk_down', true);
      moving = true;
    }

    if (!moving) {
      this.player.anims.stop();
    }
  }

  private setupInput(): void {
    this.cursors = this.input.keyboard!.createCursorKeys();
    this.interactionKey = this.input.keyboard!.addKey(Phaser.Input.Keyboard.KeyCodes.SPACE);

    this.interactionKey.on('down', () => {
      if (this.nearbyNPC) {
        this.interactWithNPC(this.nearbyNPC);
      }
    });

    // Menu
    this.input.keyboard!.on('keydown-ESC', () => {
      this.scene.launch('MenuScene');
      this.scene.pause();
    });
  }

  private onNearNPC(npc: NPC): void {
    this.nearbyNPC = npc;
    this.showInteractionPrompt('Press SPACE to talk');
  }

  private onNearFacility(facilityName: string): void {
    this.showInteractionPrompt(`Press SPACE to enter ${facilityName}`);
  }

  private interactWithNPC(npc: NPC): void {
    if (this.activityPoints <= 0) {
      this.showMessage('No activity points remaining!');
      return;
    }

    // Start dialog
    this.scene.launch('DialogScene', {
      npcId: npc.id,
      onComplete: (result: DialogResult) => {
        this.handleDialogResult(result);
      }
    });
    this.scene.pause();
  }

  private handleDialogResult(result: DialogResult): void {
    if (result.questStarted) {
      store.dispatch(startQuest(result.questId));
      this.activityPoints -= 1;
    }

    if (result.supportIncreased) {
      store.dispatch(increaseSupportLevel(result.characterId));
    }

    this.scene.resume();
  }

  private createUI(): void {
    // Activity Points display
    const apText = this.add.text(16, 16, `Activity Points: ${this.activityPoints}`, {
      fontSize: '18px',
      color: '#ffffff',
      backgroundColor: 'rgba(0, 0, 0, 0.7)',
      padding: { x: 10, y: 6 }
    }).setScrollFactor(0);
    apText.setName('apText');

    // Location name
    const locationName = this.getLocationName(this.locationId);
    this.add.text(640, 16, locationName, {
      fontSize: '24px',
      color: '#ffffff',
      backgroundColor: 'rgba(0, 0, 0, 0.7)',
      padding: { x: 12, y: 8 }
    }).setOrigin(0.5, 0).setScrollFactor(0);

    // Minimap (optional)
    this.createMinimap();
  }

  private updateUI(): void {
    const apText = this.children.getByName('apText') as Phaser.GameObjects.Text;
    if (apText) {
      apText.setText(`Activity Points: ${this.activityPoints}`);
    }
  }

  private createMinimap(): void {
    // Simplified minimap in corner
    const minimapWidth = 200;
    const minimapHeight = 150;

    const minimapCamera = this.cameras.add(
      this.cameras.main.width - minimapWidth - 16,
      16,
      minimapWidth,
      minimapHeight
    );

    minimapCamera.setZoom(0.2);
    minimapCamera.startFollow(this.player);
    minimapCamera.setBackgroundColor(0x000000);
    minimapCamera.ignore([this.children.getByName('apText')]);
  }

  private showInteractionPrompt(text: string): void {
    // Implementation
  }

  private showMessage(text: string): void {
    // Implementation similar to WorldMapScene
  }

  private getLocationName(locationId: string): string {
    const worldData = this.cache.json.get('world');
    const location = worldData.locations.find((l: any) => l.id === locationId);
    return location?.name || 'Unknown Location';
  }
}

// NPC class
class NPC {
  public sprite: Phaser.Physics.Arcade.Sprite;
  public id: string;

  constructor(scene: Phaser.Scene, data: NPCData) {
    this.id = data.id;
    this.sprite = scene.physics.add.sprite(data.x, data.y, `npc_${data.type}`);
    this.sprite.setImmovable(true);

    // Name label
    scene.add.text(
      data.x,
      data.y - 40,
      data.name,
      { fontSize: '12px', color: '#ffffff', backgroundColor: 'rgba(0, 0, 0, 0.7)', padding: { x: 4, y: 2 } }
    ).setOrigin(0.5);

    // Quest indicator if available
    if (data.hasQuest) {
      const indicator = scene.add.text(
        data.x,
        data.y - 55,
        '!',
        { fontSize: '24px', color: '#ffd700', fontStyle: 'bold' }
      ).setOrigin(0.5);

      scene.tweens.add({
        targets: indicator,
        y: indicator.y - 10,
        yoyo: true,
        duration: 500,
        repeat: -1
      });
    }
  }
}

// Procedural city layout generator
class CityLayoutGenerator {
  constructor(
    private scene: Phaser.Scene,
    private width: number,
    private height: number
  ) {}

  generate(seed: string): Phaser.Tilemaps.Tilemap {
    const rng = seedrandom(seed);

    // Create tilemap
    const map = this.scene.make.tilemap({
      tileWidth: 32,
      tileHeight: 32,
      width: this.width,
      height: this.height
    });

    // Generate layers
    this.generateGround(map, rng);
    this.generateWalls(map, rng);
    this.generateBuildings(map, rng);
    this.placeObjects(map, rng);

    return map;
  }

  private generateGround(map: Phaser.Tilemaps.Tilemap, rng: () => number): void {
    // Simple ground layer with variation
    const groundLayer = map.createBlankLayer('Ground', 'tiles')!;

    for (let y = 0; y < this.height; y++) {
      for (let x = 0; x < this.width; x++) {
        // Mix of ground tiles for variety
        const tileIndex = rng() > 0.9 ? 2 : 1;
        groundLayer.putTileAt(tileIndex, x, y);
      }
    }
  }

  private generateWalls(map: Phaser.Tilemaps.Tilemap, rng: () => number): void {
    const wallLayer = map.createBlankLayer('Walls', 'tiles')!;

    // Perimeter walls
    for (let x = 0; x < this.width; x++) {
      wallLayer.putTileAt(10, x, 0);
      wallLayer.putTileAt(10, x, this.height - 1);
    }

    for (let y = 0; y < this.height; y++) {
      wallLayer.putTileAt(10, 0, y);
      wallLayer.putTileAt(10, this.width - 1, y);
    }
  }

  private generateBuildings(map: Phaser.Tilemaps.Tilemap, rng: () => number): void {
    // Place rectangular buildings
    // Implementation depends on desired city layout
  }

  private placeObjects(map: Phaser.Tilemaps.Tilemap, rng: () => number): void {
    // Add object layer for NPCs, facilities, etc.
    const objectLayer = map.createBlankLayer('Objects', 'tiles')!;

    // Place player spawn
    objectLayer.putTileAt(100, Math.floor(this.width / 2), Math.floor(this.height / 2));
  }
}
```

### 6.3 Combat Engine Architecture

```typescript
// Turn-based tactical combat with D&D mechanics
class CombatScene extends Phaser.Scene {
  private stateMachine!: CombatStateMachine;
  private animationController!: BattleAnimationController;
  private allies: Combatant[] = [];
  private enemies: Combatant[] = [];
  private selectedAction: BattleAction | null = null;

  constructor() {
    super({ key: 'CombatScene' });
  }

  create(): void {
    this.setupBattlefield();
    this.initializeCombatants();
    this.createUI();

    this.stateMachine = new CombatStateMachine(this);
    this.animationController = new BattleAnimationController(this);

    this.stateMachine.transition('INITIALIZE');
  }

  private setupBattlefield(): void {
    // Background
    this.add.rectangle(640, 360, 1280, 720, 0x1a202c);

    // Battle grid (optional for tactical positioning)
    this.drawGrid();

    // Position indicators
    this.drawFormationZones();
  }

  private drawGrid(): void {
    const graphics = this.add.graphics();
    graphics.lineStyle(1, 0x4a5568, 0.3);

    const gridSize = 64;
    const rows = 11;
    const cols = 20;
    const startX = (1280 - cols * gridSize) / 2;
    const startY = (720 - rows * gridSize) / 2;

    // Vertical lines
    for (let i = 0; i <= cols; i++) {
      graphics.lineBetween(
        startX + i * gridSize,
        startY,
        startX + i * gridSize,
        startY + rows * gridSize
      );
    }

    // Horizontal lines
    for (let i = 0; i <= rows; i++) {
      graphics.lineBetween(
        startX,
        startY + i * gridSize,
        startX + cols * gridSize,
        startY + i * gridSize
      );
    }
  }

  private drawFormationZones(): void {
    const graphics = this.add.graphics();

    // Allied zone (left, blue tint)
    graphics.fillStyle(0x3182ce, 0.1);
    graphics.fillRect(50, 150, 400, 420);
    graphics.lineStyle(2, 0x3182ce, 0.5);
    graphics.strokeRect(50, 150, 400, 420);

    // Enemy zone (right, red tint)
    graphics.fillStyle(0xe53e3e, 0.1);
    graphics.fillRect(830, 150, 400, 420);
    graphics.lineStyle(2, 0xe53e3e, 0.5);
    graphics.strokeRect(830, 150, 400, 420);
  }

  private initializeCombatants(): void {
    const gameState = store.getState();

    // Load party members
    gameState.characters.activeParty.forEach((charId, index) => {
      const character = gameState.characters.byId[charId];
      const combatant = this.createCombatant(character, index, true);
      this.allies.push(combatant);
    });

    // Load enemies (from encounter data)
    const encounter = this.cache.json.get('encounters')[0]; // Example
    encounter.enemies.forEach((enemyData: any, index: number) => {
      const combatant = this.createEnemyCombatant(enemyData, index);
      this.enemies.push(combatant);
    });
  }

  private createCombatant(character: Character, index: number, isAlly: boolean): Combatant {
    const formation = new FormationManager().getFormation('STANDARD');
    const position = formation.positions[index];

    const sprite = this.add.sprite(
      isAlly ? 200 + position.x : 1080 - position.x,
      200 + position.y,
      `char_${character.id}`
    );

    sprite.setScale(2);

    // HP bar above sprite
    const hpBar = new HPBar(this, sprite.x, sprite.y - 50, character.hp, character.maxHp);

    // Status effect container
    const statusIcons: Phaser.GameObjects.Image[] = [];

    const combatant: Combatant = {
      id: character.id,
      name: character.name,
      isPlayer: isAlly,
      stats: {
        hp: character.hp,
        maxHp: character.maxHp,
        mp: character.mp,
        maxMp: character.maxMp,
        ac: character.ac,
        speed: character.speed,
        abilities: character.abilities
      },
      position: { x: sprite.x, y: sprite.y },
      sprite,
      hpBar,
      statusIcons,
      statusEffects: [],
      character
    };

    return combatant;
  }

  private createEnemyCombatant(enemyData: any, index: number): Combatant {
    // Similar to createCombatant but for enemies
    const enemyDef = this.cache.json.get('enemies')[enemyData.type];

    const sprite = this.add.sprite(
      950 + (index % 2) * 100,
      200 + Math.floor(index / 2) * 120,
      `enemy_${enemyData.type}`
    );

    sprite.setScale(2);
    sprite.setFlipX(true); // Face left

    const hpBar = new HPBar(this, sprite.x, sprite.y - 50, enemyDef.stats.hp, enemyDef.stats.hp);

    return {
      id: `enemy_${index}`,
      name: enemyDef.name,
      isPlayer: false,
      stats: {
        hp: enemyDef.stats.hp,
        maxHp: enemyDef.stats.hp,
        mp: 0,
        maxMp: 0,
        ac: enemyDef.stats.ac,
        speed: enemyDef.stats.speed,
        abilities: enemyDef.abilities
      },
      position: { x: sprite.x, y: sprite.y },
      sprite,
      hpBar,
      statusIcons: [],
      statusEffects: [],
      enemyDef
    };
  }

  private createUI(): void {
    // Turn order display (top)
    this.createTurnOrderDisplay();

    // Action menu (bottom)
    this.createActionMenu();

    // Combat log (left side, scrollable)
    this.createCombatLog();
  }

  private createTurnOrderDisplay(): void {
    const container = this.add.container(640, 50);
    container.setName('turnOrderDisplay');

    const bg = this.add.rectangle(0, 0, 800, 60, 0x1a202c, 0.9);
    bg.setStrokeStyle(2, 0x4a5568);
    container.add(bg);

    const title = this.add.text(-380, 0, 'Turn Order:', {
      fontSize: '16px',
      color: '#cbd5e0'
    }).setOrigin(0, 0.5);
    container.add(title);
  }

  private createActionMenu(): void {
    const menu = this.add.container(640, 600);
    menu.setName('actionMenu');
    menu.setVisible(false);

    const bg = this.add.rectangle(0, 0, 600, 100, 0x1a202c, 0.95);
    bg.setStrokeStyle(2, 0x4a5568);
    menu.add(bg);

    const actions = ['Attack', 'Skill', 'Item', 'Defend'];
    actions.forEach((action, index) => {
      const button = this.createActionButton(
        -250 + index * 170,
        0,
        action,
        () => this.onActionSelected(action)
      );
      menu.add(button);
    });
  }

  private createActionButton(
    x: number,
    y: number,
    text: string,
    callback: () => void
  ): Phaser.GameObjects.Container {
    const container = this.add.container(x, y);

    const bg = this.add.rectangle(0, 0, 150, 60, 0x2d3748);
    bg.setStrokeStyle(2, 0x4a5568);
    bg.setInteractive({ useHandCursor: true });

    const label = this.add.text(0, 0, text, {
      fontSize: '16px',
      color: '#f7fafc'
    }).setOrigin(0.5);

    container.add([bg, label]);

    bg.on('pointerover', () => {
      bg.setFillStyle(0x4a5568);
    });

    bg.on('pointerout', () => {
      bg.setFillStyle(0x2d3748);
    });

    bg.on('pointerdown', callback);

    return container;
  }

  private createCombatLog(): void {
    const log = this.add.container(50, 200);
    log.setName('combatLog');

    const bg = this.add.rectangle(0, 0, 280, 400, 0x1a202c, 0.8);
    bg.setStrokeStyle(2, 0x4a5568);
    bg.setOrigin(0);
    log.add(bg);

    const title = this.add.text(10, 10, 'Combat Log', {
      fontSize: '16px',
      color: '#cbd5e0',
      fontStyle: 'bold'
    });
    log.add(title);
  }

  // Public methods called by state machine
  showActionMenu(actor: Combatant): void {
    const menu = this.children.getByName('actionMenu') as Phaser.GameObjects.Container;
    if (menu) {
      menu.setVisible(true);
    }
  }

  hideActionMenu(): void {
    const menu = this.children.getByName('actionMenu') as Phaser.GameObjects.Container;
    if (menu) {
      menu.setVisible(false);
    }
  }

  showTargetSelection(action: BattleAction): void {
    // Highlight valid targets
    const targets = action.targetType === 'enemy' ? this.enemies : this.allies;

    targets.forEach(target => {
      if (target.stats.hp > 0) {
        const highlight = this.add.circle(
          target.sprite.x,
          target.sprite.y,
          40,
          0xffd700,
          0.3
        );
        highlight.setStrokeStyle(3, 0xffd700);
        highlight.setName(`highlight_${target.id}`);

        // Make clickable
        highlight.setInteractive({ useHandCursor: true });
        highlight.on('pointerdown', () => {
          this.onTargetSelected([target]);
        });
      }
    });
  }

  hideTargetSelection(): void {
    this.children.each(child => {
      if (child.name?.startsWith('highlight_')) {
        child.destroy();
      }
    });
  }

  onActionSelected(actionName: string): void {
    // Look up action from current actor's abilities
    const action = this.getCurrentActorActions().find(a => a.name === actionName);
    if (action) {
      this.stateMachine.onActionSelected(action);
    }
  }

  onTargetSelected(targets: Combatant[]): void {
    this.hideTargetSelection();
    this.stateMachine.onTargetSelected(targets);
  }

  getAllCombatants(): Combatant[] {
    return [...this.allies, ...this.enemies];
  }

  getEnemies(): Combatant[] {
    return this.enemies;
  }

  getAllies(): Combatant[] {
    return this.allies;
  }

  getCurrentActorActions(): BattleAction[] {
    // Return available actions for current actor
    return [];
  }

  showVictoryScreen(): void {
    // Show victory UI, rewards, etc.
    this.cameras.main.fadeOut(1000);
    this.time.delayedCall(1000, () => {
      this.scene.start('RewardScene', { victory: true });
    });
  }

  showDefeatScreen(): void {
    // Show defeat UI
    this.cameras.main.fadeOut(1000);
    this.time.delayedCall(1000, () => {
      this.scene.start('WorldMapScene');
    });
  }
}

// HP Bar component
class HPBar {
  private container: Phaser.GameObjects.Container;
  private bar: Phaser.GameObjects.Graphics;
  private current: number;
  private max: number;

  constructor(
    scene: Phaser.Scene,
    x: number,
    y: number,
    current: number,
    max: number
  ) {
    this.current = current;
    this.max = max;

    this.container = scene.add.container(x, y);

    // Background
    const bg = scene.add.graphics();
    bg.fillStyle(0x000000, 0.5);
    bg.fillRect(-25, 0, 50, 6);
    this.container.add(bg);

    // HP bar
    this.bar = scene.add.graphics();
    this.container.add(this.bar);

    this.update(current);
  }

  update(newHP: number, animate: boolean = true): void {
    this.current = Phaser.Math.Clamp(newHP, 0, this.max);

    const percentage = this.current / this.max;
    const barWidth = 48 * percentage;

    // Color based on HP percentage
    let color = 0x48bb78; // Green
    if (percentage < 0.5) color = 0xecc94b; // Yellow
    if (percentage < 0.25) color = 0xf56565; // Red

    this.bar.clear();
    this.bar.fillStyle(color);
    this.bar.fillRect(-24, 1, barWidth, 4);
  }

  destroy(): void {
    this.container.destroy();
  }
}

// Combat animation controller
class BattleAnimationController {
  private scene: Phaser.Scene;
  private queue: AnimationSequence[] = [];
  private isPlaying: boolean = false;

  constructor(scene: Phaser.Scene) {
    this.scene = scene;
  }

  queueSequence(sequence: AnimationSequence): void {
    this.queue.push(sequence);

    if (!this.isPlaying) {
      this.playNext();
    }
  }

  private playNext(): void {
    if (this.queue.length === 0) {
      this.isPlaying = false;
      return;
    }

    this.isPlaying = true;
    const sequence = this.queue.shift()!;

    switch(sequence.type) {
      case 'action':
        this.playActionSequence(sequence);
        break;
      case 'damage':
        this.playDamageEffect(sequence);
        break;
      case 'heal':
        this.playHealEffect(sequence);
        break;
    }
  }

  private playActionSequence(sequence: AnimationSequence): void {
    const { actor, action, targets } = sequence;

    // Build timeline
    const timeline = this.scene.tweens.createTimeline();

    // Step 1: Actor approaches (if melee)
    if (action.range === 'melee') {
      timeline.add({
        targets: actor.sprite,
        x: actor.sprite.x + (actor.isPlayer ? 150 : -150),
        duration: 200,
        ease: 'Power2'
      });
    }

    // Step 2: Play attack animation
    timeline.add({
      targets: actor.sprite,
      duration: 300,
      onStart: () => {
        actor.sprite.play(`${action.animation}_anim`);
      }
    });

    // Step 3: Apply effects to targets
    targets.forEach((target, index) => {
      timeline.add({
        targets: target.sprite,
        duration: 100,
        delay: index * 50,
        onStart: () => {
          this.showHitEffect(target, action);
          this.calculateAndApplyDamage(actor, target, action);
        }
      });
    });

    // Step 4: Actor returns
    if (action.range === 'melee') {
      timeline.add({
        targets: actor.sprite,
        x: actor.position.x,
        duration: 200,
        ease: 'Power2'
      });
    }

    timeline.on('complete', () => {
      sequence.onComplete?.();
      this.playNext();
    });

    timeline.play();
  }

  private showHitEffect(target: Combatant, action: BattleAction): void {
    // Screen shake
    if (action.hitEffects?.includes('screen_shake')) {
      this.scene.cameras.main.shake(100, 0.005);
    }

    // Flash
    if (action.hitEffects?.includes('impact_flash')) {
      target.sprite.setTint(0xffffff);
      this.scene.time.delayedCall(100, () => {
        target.sprite.clearTint();
      });
    }

    // Particles
    this.createHitParticles(target, action.damageType || 'physical');
  }

  private createHitParticles(target: Combatant, damageType: string): void {
    const particles = this.scene.add.particles(target.sprite.x, target.sprite.y, `particle_${damageType}`, {
      speed: { min: 50, max: 150 },
      angle: { min: 0, max: 360 },
      scale: { start: 1, end: 0 },
      blendMode: 'ADD',
      lifespan: 500,
      quantity: 15
    });

    this.scene.time.delayedCall(500, () => particles.destroy());
  }

  private calculateAndApplyDamage(actor: Combatant, target: Combatant, action: BattleAction): void {
    // D&D-style damage calculation
    const attackRoll = this.rollD20() + this.getAttackModifier(actor, action);
    const targetAC = target.stats.ac;

    if (attackRoll >= targetAC) {
      // Hit!
      const damage = this.rollDamage(action, actor);
      target.stats.hp -= damage;
      target.hpBar.update(target.stats.hp);

      // Show damage number
      this.showDamageNumber(target, damage, attackRoll === 20);

      if (target.stats.hp <= 0) {
        this.playDeathAnimation(target);
      }
    } else {
      // Miss!
      this.showMissText(target);
    }
  }

  private rollD20(): number {
    return Math.floor(Math.random() * 20) + 1;
  }

  private getAttackModifier(actor: Combatant, action: BattleAction): number {
    // Simplified - use character's relevant ability modifier
    const char = actor.character;
    if (!char) return 0;

    const abilityMod = action.abilityUsed === 'strength'
      ? Math.floor((char.abilities.strength - 10) / 2)
      : Math.floor((char.abilities.dexterity - 10) / 2);

    const profBonus = Math.floor((char.level - 1) / 4) + 2;

    return abilityMod + profBonus;
  }

  private rollDamage(action: BattleAction, actor: Combatant): number {
    // Parse damage formula (e.g., "1d8+3")
    // Simplified implementation
    return Math.floor(Math.random() * 8) + 1 + 3;
  }

  private showDamageNumber(target: Combatant, damage: number, critical: boolean): void {
    const color = critical ? '#ffd700' : '#ff6b6b';
    const fontSize = critical ? '32px' : '24px';

    const text = this.scene.add.text(
      target.sprite.x,
      target.sprite.y - 30,
      damage.toString(),
      {
        fontSize,
        color,
        fontStyle: 'bold',
        stroke: '#000000',
        strokeThickness: 4
      }
    ).setOrigin(0.5);

    this.scene.tweens.add({
      targets: text,
      y: text.y - 60,
      alpha: 0,
      duration: 1000,
      ease: 'Power2',
      onComplete: () => text.destroy()
    });
  }

  private showMissText(target: Combatant): void {
    const text = this.scene.add.text(
      target.sprite.x,
      target.sprite.y - 30,
      'MISS',
      {
        fontSize: '20px',
        color: '#a0aec0',
        fontStyle: 'italic'
      }
    ).setOrigin(0.5);

    this.scene.tweens.add({
      targets: text,
      y: text.y - 40,
      alpha: 0,
      duration: 800,
      onComplete: () => text.destroy()
    });
  }

  private playDeathAnimation(target: Combatant): void {
    this.scene.tweens.add({
      targets: target.sprite,
      alpha: 0,
      y: target.sprite.y + 20,
      duration: 500,
      ease: 'Power2'
    });
  }

  private playDamageEffect(sequence: AnimationSequence): void {
    // Implementation
    this.playNext();
  }

  private playHealEffect(sequence: AnimationSequence): void {
    // Implementation
    this.playNext();
  }
}

// Enemy AI system
class EnemyAI {
  constructor(
    private enemy: Combatant,
    private scene: CombatScene
  ) {}

  makeDecision(): { action: BattleAction, targets: Combatant[] } {
    const allies = this.scene.getAllies().filter(a => a.stats.hp > 0);

    // Simple AI: attack lowest HP ally
    const target = allies.reduce((lowest, current) =>
      current.stats.hp < lowest.stats.hp ? current : lowest
    );

    // Choose action based on enemy type
    const action = this.selectAction();

    return { action, targets: [target] };
  }

  private selectAction(): BattleAction {
    // Get available actions from enemy definition
    const actions = this.enemy.enemyDef.actions;

    // Simple selection: random from available actions
    return actions[Math.floor(Math.random() * actions.length)];
  }
}
```

### 6.4 Character Progression System

```typescript
// XP and leveling system with D&D-style progression
class ProgressionManager {
  private static readonly XP_CURVE: Record<number, number> = {
    1: 0,
    2: 300,
    3: 900,
    4: 2700,
    5: 6500,
    6: 14000,
    7: 23000,
    8: 34000,
    9: 48000,
    10: 64000,
    11: 85000,
    12: 100000,
    13: 120000,
    14: 140000,
    15: 165000,
    16: 195000,
    17: 225000,
    18: 265000,
    19: 305000,
    20: 355000
  };

  /**
   * Award XP to character and check for level up
   */
  static awardXP(character: Character, amount: number): LevelUpResult | null {
    character.xp += amount;

    // Check if leveled up
    const newLevel = this.calculateLevel(character.xp);

    if (newLevel > character.level) {
      return this.levelUp(character, newLevel);
    }

    return null;
  }

  private static calculateLevel(xp: number): number {
    for (let level = 20; level >= 1; level--) {
      if (xp >= this.XP_CURVE[level]) {
        return level;
      }
    }
    return 1;
  }

  private static levelUp(character: Character, newLevel: number): LevelUpResult {
    const oldLevel = character.level;
    character.level = newLevel;

    const result: LevelUpResult = {
      oldLevel,
      newLevel,
      statIncreases: {},
      newAbilities: [],
      abilityPointsGained: 0
    };

    // Calculate stat increases
    for (let level = oldLevel + 1; level <= newLevel; level++) {
      const increases = this.calculateStatIncreases(character, level);
      Object.entries(increases).forEach(([stat, value]) => {
        result.statIncreases[stat] = (result.statIncreases[stat] || 0) + value;
      });

      // HP increase
      const hpIncrease = this.calculateHPIncrease(character.class, level);
      character.maxHp += hpIncrease;
      character.hp += hpIncrease;

      // MP increase (if applicable)
      if (this.isMagicUser(character.class)) {
        const mpIncrease = this.calculateMPIncrease(character.class, level);
        character.maxMp += mpIncrease;
        character.mp += mpIncrease;
      }

      // Check for ability unlocks
      const unlockedAbilities = this.getAbilitiesAtLevel(character.class, level);
      result.newAbilities.push(...unlockedAbilities);
      character.abilities.push(...unlockedAbilities);

      // Ability score improvements (every 4 levels)
      if (level % 4 === 0) {
        result.abilityPointsGained += 2;
        character.abilityPoints += 2;
      }
    }

    // Apply stat increases
    Object.entries(result.statIncreases).forEach(([stat, value]) => {
      (character as any)[stat] += value;
    });

    // Update spell slots if spellcaster
    if (this.isSpellcaster(character.class)) {
      this.updateSpellSlots(character);
    }

    return result;
  }

  private static calculateStatIncreases(character: Character, level: number): Record<string, number> {
    // Class-based stat growth
    const growthRates: Record<string, Record<string, number>> = {
      fighter: {
        strength: 0.7,
        constitution: 0.5,
        dexterity: 0.3
      },
      mage: {
        intelligence: 0.7,
        wisdom: 0.4,
        dexterity: 0.2
      },
      cleric: {
        wisdom: 0.7,
        constitution: 0.4,
        charisma: 0.3
      },
      rogue: {
        dexterity: 0.7,
        charisma: 0.4,
        intelligence: 0.3
      }
    };

    const rates = growthRates[character.class] || {};
    const increases: Record<string, number> = {};

    Object.entries(rates).forEach(([stat, rate]) => {
      // Roll for stat increase (probability-based)
      if (Math.random() < rate) {
        increases[stat] = 1;
      }
    });

    return increases;
  }

  private static calculateHPIncrease(characterClass: string, level: number): number {
    // D&D-style hit die per class
    const hitDice: Record<string, number> = {
      fighter: 10,
      cleric: 8,
      rogue: 8,
      mage: 6
    };

    const die = hitDice[characterClass] || 8;

    // Roll hit die or take average (we'll take average for consistency)
    return Math.floor(die / 2) + 1;
  }

  private static calculateMPIncrease(characterClass: string, level: number): number {
    // MP increase for spellcasters
    const increases: Record<string, number> = {
      mage: 6,
      cleric: 4,
      fighter: 0,
      rogue: 0
    };

    return increases[characterClass] || 0;
  }

  private static isMagicUser(characterClass: string): boolean {
    return ['mage', 'cleric'].includes(characterClass);
  }

  private static isSpellcaster(characterClass: string): boolean {
    return this.isMagicUser(characterClass);
  }

  private static getAbilitiesAtLevel(characterClass: string, level: number): string[] {
    // Look up class progression table
    const classData = this.getClassData(characterClass);
    return classData.abilitiesByLevel[level] || [];
  }

  private static updateSpellSlots(character: Character): void {
    // D&D 5e spell slot progression
    const spellSlotTable: Record<number, Record<string, number>> = {
      1: { level1: 2 },
      2: { level1: 3 },
      3: { level1: 4, level2: 2 },
      4: { level1: 4, level2: 3 },
      5: { level1: 4, level2: 3, level3: 2 },
      // ... full table up to level 20
    };

    const slots = spellSlotTable[character.level] || {};
    character.spellSlots = {
      level1: { current: slots.level1 || 0, max: slots.level1 || 0 },
      level2: { current: slots.level2 || 0, max: slots.level2 || 0 },
      level3: { current: slots.level3 || 0, max: slots.level3 || 0 },
      level4: { current: slots.level4 || 0, max: slots.level4 || 0 },
      level5: { current: slots.level5 || 0, max: slots.level5 || 0 }
    };
  }

  private static getClassData(characterClass: string): any {
    // Load from data files
    return {};
  }

  /**
   * Support system (Fire Emblem-style)
   */
  static increaseSupportLevel(char1: Character, char2: Character): SupportResult | null {
    const supportKey = [char1.id, char2.id].sort().join('_');
    const currentLevel = char1.supports[char2.id] || 0;

    if (currentLevel >= 3) {
      return null; // Max support level (S-rank)
    }

    char1.supports[char2.id] = currentLevel + 1;
    char2.supports[char1.id] = currentLevel + 1;

    return {
      characters: [char1.id, char2.id],
      newLevel: currentLevel + 1,
      bonuses: this.getSupportBonuses(currentLevel + 1)
    };
  }

  private static getSupportBonuses(level: number): SupportBonuses {
    const bonuses: Record<number, SupportBonuses> = {
      1: { hit: 5, avoid: 5, crit: 2 },      // C-rank
      2: { hit: 10, avoid: 10, crit: 5 },    // B-rank
      3: { hit: 15, avoid: 15, crit: 10 }    // A-rank
    };

    return bonuses[level] || { hit: 0, avoid: 0, crit: 0 };
  }
}

interface LevelUpResult {
  oldLevel: number;
  newLevel: number;
  statIncreases: Record<string, number>;
  newAbilities: string[];
  abilityPointsGained: number;
}

interface SupportResult {
  characters: string[];
  newLevel: number;
  bonuses: SupportBonuses;
}

interface SupportBonuses {
  hit: number;
  avoid: number;
  crit: number;
}
```

### 6.5 D&D Mechanics Implementation

```typescript
// D&D-style dice rolling and skill checks
class DiceRoller {
  /**
   * Roll dice with D&D notation (e.g., "2d6+3", "1d20")
   */
  static roll(notation: string): DiceResult {
    const match = notation.match(/(\d+)d(\d+)([+-]\d+)?/);
    if (!match) throw new Error(`Invalid dice notation: ${notation}`);

    const count = parseInt(match[1]);
    const sides = parseInt(match[2]);
    const modifier = match[3] ? parseInt(match[3]) : 0;

    const rolls: number[] = [];
    let total = 0;

    for (let i = 0; i < count; i++) {
      const roll = Math.floor(Math.random() * sides) + 1;
      rolls.push(roll);
      total += roll;
    }

    total += modifier;

    return {
      notation,
      rolls,
      modifier,
      total,
      isCritical: rolls.length === 1 && sides === 20 && rolls[0] === 20,
      isCriticalFailure: rolls.length === 1 && sides === 20 && rolls[0] === 1
    };
  }

  /**
   * Roll with advantage (roll twice, take higher)
   */
  static rollWithAdvantage(notation: string): DiceResult {
    const roll1 = this.roll(notation);
    const roll2 = this.roll(notation);

    return roll1.total >= roll2.total ? roll1 : roll2;
  }

  /**
   * Roll with disadvantage (roll twice, take lower)
   */
  static rollWithDisadvantage(notation: string): DiceResult {
    const roll1 = this.roll(notation);
    const roll2 = this.roll(notation);

    return roll1.total <= roll2.total ? roll1 : roll2;
  }
}

interface DiceResult {
  notation: string;
  rolls: number[];
  modifier: number;
  total: number;
  isCritical: boolean;
  isCriticalFailure: boolean;
}

// Skill check system
class SkillCheckSystem {
  /**
   * Perform a skill check
   */
  static performCheck(
    character: Character,
    skill: Skill,
    dc: number,
    advantage: boolean = false,
    disadvantage: boolean = false
  ): SkillCheckResult {
    // Get ability modifier
    const ability = this.getSkillAbility(skill);
    const abilityMod = Math.floor((character.abilities[ability] - 10) / 2);

    // Get proficiency bonus
    const proficiencyBonus = character.skills[skill].proficient
      ? this.getProficiencyBonus(character.level)
      : 0;

    // Additional bonuses from equipment, buffs, etc.
    const additionalBonus = character.skills[skill].bonus || 0;

    const totalModifier = abilityMod + proficiencyBonus + additionalBonus;

    // Roll d20
    let diceResult: DiceResult;
    if (advantage) {
      diceResult = DiceRoller.rollWithAdvantage('1d20');
    } else if (disadvantage) {
      diceResult = DiceRoller.rollWithDisadvantage('1d20');
    } else {
      diceResult = DiceRoller.roll('1d20');
    }

    const finalResult = diceResult.total + totalModifier;
    const success = finalResult >= dc || diceResult.isCritical;
    const criticalSuccess = diceResult.isCritical;
    const criticalFailure = diceResult.isCriticalFailure;

    return {
      skill,
      dc,
      roll: diceResult.total,
      modifier: totalModifier,
      total: finalResult,
      success,
      criticalSuccess,
      criticalFailure,
      breakdown: {
        roll: diceResult.total,
        abilityModifier: abilityMod,
        proficiencyBonus,
        additionalBonus
      }
    };
  }

  private static getSkillAbility(skill: Skill): keyof Character['abilities'] {
    const skillAbilities: Record<Skill, keyof Character['abilities']> = {
      athletics: 'strength',
      acrobatics: 'dexterity',
      sleightOfHand: 'dexterity',
      stealth: 'dexterity',
      arcana: 'intelligence',
      history: 'intelligence',
      investigation: 'intelligence',
      nature: 'intelligence',
      religion: 'intelligence',
      animalHandling: 'wisdom',
      insight: 'wisdom',
      medicine: 'wisdom',
      perception: 'wisdom',
      survival: 'wisdom',
      deception: 'charisma',
      intimidation: 'charisma',
      performance: 'charisma',
      persuasion: 'charisma'
    };

    return skillAbilities[skill];
  }

  private static getProficiencyBonus(level: number): number {
    return Math.floor((level - 1) / 4) + 2;
  }
}

type Skill =
  | 'athletics'
  | 'acrobatics'
  | 'sleightOfHand'
  | 'stealth'
  | 'arcana'
  | 'history'
  | 'investigation'
  | 'nature'
  | 'religion'
  | 'animalHandling'
  | 'insight'
  | 'medicine'
  | 'perception'
  | 'survival'
  | 'deception'
  | 'intimidation'
  | 'performance'
  | 'persuasion';

interface SkillCheckResult {
  skill: Skill;
  dc: number;
  roll: number;
  modifier: number;
  total: number;
  success: boolean;
  criticalSuccess: boolean;
  criticalFailure: boolean;
  breakdown: {
    roll: number;
    abilityModifier: number;
    proficiencyBonus: number;
    additionalBonus: number;
  };
}

// Dialog scene with skill checks
class DialogScene extends Phaser.Scene {
  private currentDialog: Dialog | null = null;
  private currentNode: number = 0;

  constructor() {
    super({ key: 'DialogScene' });
  }

  init(data: { npcId: string; onComplete: (result: DialogResult) => void }): void {
    this.npcId = data.npcId;
    this.onComplete = data.onComplete;
  }

  create(): void {
    // Semi-transparent background
    this.add.rectangle(640, 360, 1280, 720, 0x000000, 0.7);

    // Dialog box
    this.createDialogBox();

    // Load dialog data
    this.loadDialog();
  }

  private createDialogBox(): void {
    const box = this.add.rectangle(640, 550, 1100, 250, 0x1a202c, 0.95);
    box.setStrokeStyle(3, 0x4a5568);

    this.dialogText = this.add.text(160, 460, '', {
      fontSize: '18px',
      color: '#f7fafc',
      wordWrap: { width: 950 }
    });

    this.speakerText = this.add.text(160, 430, '', {
      fontSize: '20px',
      color: '#ffd700',
      fontStyle: 'bold'
    });

    // Continue indicator
    this.continueIndicator = this.add.text(1040, 650, '▼', {
      fontSize: '24px',
      color: '#cbd5e0'
    });

    this.tweens.add({
      targets: this.continueIndicator,
      y: this.continueIndicator.y + 10,
      yoyo: true,
      duration: 500,
      repeat: -1
    });
  }

  private loadDialog(): void {
    const npcData = this.cache.json.get('npcs')[this.npcId];
    this.currentDialog = npcData.dialog;
    this.showNode(0);
  }

  private showNode(nodeIndex: number): void {
    const node = this.currentDialog!.nodes[nodeIndex];

    this.speakerText.setText(node.speaker);
    this.typewriterText(node.text);

    // Wait for text to finish, then show choices
    this.time.delayedCall(node.text.length * 30, () => {
      if (node.choices && node.choices.length > 0) {
        this.showChoices(node.choices);
      } else if (node.next !== undefined) {
        // Auto-continue
        this.input.once('pointerdown', () => {
          this.showNode(node.next!);
        });
      } else {
        // End of dialog
        this.input.once('pointerdown', () => {
          this.endDialog();
        });
      }
    });
  }

  private typewriterText(text: string): void {
    let currentText = '';
    let index = 0;

    const interval = setInterval(() => {
      if (index < text.length) {
        currentText += text[index];
        this.dialogText.setText(currentText);
        index++;
      } else {
        clearInterval(interval);
      }
    }, 30);
  }

  private showChoices(choices: DialogChoice[]): void {
    this.continueIndicator.setVisible(false);

    choices.forEach((choice, index) => {
      const y = 480 + index * 50;

      // Check if skill check is required
      let displayText = choice.text;
      let enabled = true;

      if (choice.skillCheck) {
        const { skill, dc } = choice.skillCheck;
        displayText = `[${skill.toUpperCase()} DC ${dc}] ${choice.text}`;

        // Could show chance of success here
        // enabled = this.canAttemptCheck(skill);
      }

      const choiceButton = this.createChoiceButton(160, y, displayText, enabled, () => {
        this.onChoiceSelected(choice);
      });
    });
  }

  private createChoiceButton(
    x: number,
    y: number,
    text: string,
    enabled: boolean,
    callback: () => void
  ): Phaser.GameObjects.Container {
    const container = this.add.container(x, y);

    const bg = this.add.rectangle(0, 0, 950, 40, enabled ? 0x2d3748 : 0x1a202c);
    bg.setOrigin(0, 0.5);
    bg.setStrokeStyle(2, enabled ? 0x4a5568 : 0x2d3748);

    if (enabled) {
      bg.setInteractive({ useHandCursor: true });

      bg.on('pointerover', () => {
        bg.setFillStyle(0x4a5568);
      });

      bg.on('pointerout', () => {
        bg.setFillStyle(0x2d3748);
      });

      bg.on('pointerdown', callback);
    }

    const label = this.add.text(10, 0, text, {
      fontSize: '16px',
      color: enabled ? '#f7fafc' : '#4a5568'
    }).setOrigin(0, 0.5);

    container.add([bg, label]);

    return container;
  }

  private onChoiceSelected(choice: DialogChoice): void {
    // Clear choices
    this.children.each(child => {
      if (child.type === 'Container' && child.y >= 480) {
        child.destroy();
      }
    });

    this.continueIndicator.setVisible(true);

    if (choice.skillCheck) {
      this.performSkillCheck(choice);
    } else {
      this.proceedToNext(choice.next);
    }
  }

  private performSkillCheck(choice: DialogChoice): void {
    const { skill, dc } = choice.skillCheck!;
    const player = store.getState().characters.byId['player'];

    // Show dice rolling animation
    this.showDiceRoll(skill, dc, () => {
      const result = SkillCheckSystem.performCheck(player, skill, dc);

      // Show result
      this.showCheckResult(result);

      // Proceed based on result
      if (result.success) {
        this.proceedToNext(choice.successNode!);
      } else {
        this.proceedToNext(choice.failureNode!);
      }
    });
  }

  private showDiceRoll(skill: Skill, dc: number, onComplete: () => void): void {
    // Show animated d20 rolling
    const dice = this.add.image(640, 360, 'd20');
    dice.setScale(2);

    this.tweens.add({
      targets: dice,
      angle: 720,
      scale: 3,
      duration: 1000,
      ease: 'Cubic.easeOut',
      onComplete: () => {
        dice.destroy();
        onComplete();
      }
    });
  }

  private showCheckResult(result: SkillCheckResult): void {
    const text = result.success ? 'SUCCESS!' : 'FAILURE...';
    const color = result.success ? '#48bb78' : '#f56565';

    const resultText = this.add.text(640, 360, text, {
      fontSize: '48px',
      color,
      fontStyle: 'bold',
      stroke: '#000000',
      strokeThickness: 6
    }).setOrigin(0.5);

    const breakdown = this.add.text(
      640,
      410,
      `Roll: ${result.roll} + ${result.modifier} = ${result.total} vs DC ${result.dc}`,
      { fontSize: '20px', color: '#cbd5e0' }
    ).setOrigin(0.5);

    this.time.delayedCall(2000, () => {
      resultText.destroy();
      breakdown.destroy();
    });
  }

  private proceedToNext(next: number): void {
    this.time.delayedCall(2000, () => {
      if (next !== undefined) {
        this.showNode(next);
      } else {
        this.endDialog();
      }
    });
  }

  private endDialog(): void {
    const result: DialogResult = {
      completed: true
    };

    this.onComplete(result);
    this.scene.stop();
  }
}

interface Dialog {
  nodes: DialogNode[];
}

interface DialogNode {
  speaker: string;
  text: string;
  choices?: DialogChoice[];
  next?: number;
}

interface DialogChoice {
  text: string;
  next: number;
  skillCheck?: {
    skill: Skill;
    dc: number;
  };
  successNode?: number;
  failureNode?: number;
}

interface DialogResult {
  completed: boolean;
  questStarted?: boolean;
  questId?: string;
  supportIncreased?: boolean;
  characterId?: string;
}
```

---

## 7. Performance Strategy

### 7.1 Asset Loading Optimization

```typescript
// Asynchronous asset loading with progress tracking
class AssetLoader {
  private scene: Phaser.Scene;
  private totalAssets: number = 0;
  private loadedAssets: number = 0;

  constructor(scene: Phaser.Scene) {
    this.scene = scene;
  }

  /**
   * Load all game assets asynchronously
   */
  async loadAllAssets(onProgress?: (progress: number) => void): Promise<void> {
    // Load JSON data first
    await this.loadDataFiles();

    // Generate procedural assets
    await this.generateAssets(onProgress);

    // Load any external assets (if API keys provided)
    await this.loadExternalAssets(onProgress);
  }

  private async loadDataFiles(): Promise<void> {
    const dataFiles = [
      'characters',
      'abilities',
      'enemies',
      'items',
      'quests',
      'world',
      'npcs',
      'encounters'
    ];

    const promises = dataFiles.map(file =>
      fetch(`./data/${file}.json`).then(res => res.json())
    );

    const results = await Promise.all(promises);

    results.forEach((data, index) => {
      this.scene.cache.json.add(dataFiles[index], data);
    });
  }

  private async generateAssets(onProgress?: (progress: number) => void): Promise<void> {
    const generator = new ProceduralSpriteGenerator();

    // Get all entities that need sprites
    const characters = this.scene.cache.json.get('characters');
    const enemies = this.scene.cache.json.get('enemies');

    this.totalAssets = Object.keys(characters).length + Object.keys(enemies).length;

    // Generate character sprites
    let generated = 0;
    for (const [id, char] of Object.entries(characters as Record<string, any>)) {
      const canvas = generator.generateCharacterSprite(char.visualData, id);
      generator.toPhaserTexture(this.scene, `char_${id}`, canvas);

      generated++;
      onProgress?.(generated / this.totalAssets);

      // Yield to prevent blocking
      await this.wait(10);
    }

    // Generate enemy sprites
    for (const [id, enemy] of Object.entries(enemies as Record<string, any>)) {
      const canvas = generator.generateCharacterSprite(enemy.visualData, id);
      generator.toPhaserTexture(this.scene, `enemy_${id}`, canvas);

      generated++;
      onProgress?.(generated / this.totalAssets);

      await this.wait(10);
    }

    // Generate particle textures
    const particleTypes = ['fire', 'ice', 'lightning', 'heal', 'physical'];
    for (const type of particleTypes) {
      const canvas = generator.generateParticleTexture(type as any);
      generator.toPhaserTexture(this.scene, `particle_${type}`, canvas);
    }

    // Generate UI elements
    const buttonCanvas = generator.generateUIElement('button', 200, 50);
    generator.toPhaserTexture(this.scene, 'ui_button', buttonCanvas);

    const panelCanvas = generator.generateUIElement('panel', 400, 300);
    generator.toPhaserTexture(this.scene, 'ui_panel', panelCanvas);
  }

  private async loadExternalAssets(onProgress?: (progress: number) => void): Promise<void> {
    // Placeholder for AI-generated assets
    // Only load if API keys are configured
    if (!process.env.OPENAI_API_KEY) {
      return;
    }

    // Implementation would load AI-generated portraits, etc.
  }

  private wait(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

### 7.2 Memory Management

```typescript
// Object pooling for frequently created/destroyed objects
class ObjectPool<T> {
  private available: T[] = [];
  private inUse: Set<T> = new Set();

  constructor(
    private factory: () => T,
    private reset: (obj: T) => void,
    initialSize: number = 10
  ) {
    // Pre-populate pool
    for (let i = 0; i < initialSize; i++) {
      this.available.push(this.factory());
    }
  }

  acquire(): T {
    let obj: T;

    if (this.available.length > 0) {
      obj = this.available.pop()!;
    } else {
      obj = this.factory();
    }

    this.inUse.add(obj);
    return obj;
  }

  release(obj: T): void {
    if (this.inUse.has(obj)) {
      this.inUse.delete(obj);
      this.reset(obj);
      this.available.push(obj);
    }
  }

  releaseAll(): void {
    this.inUse.forEach(obj => {
      this.reset(obj);
      this.available.push(obj);
    });
    this.inUse.clear();
  }

  get size(): number {
    return this.available.length + this.inUse.size;
  }
}

// Example usage: Particle pool
class ParticleManager {
  private particlePool: ObjectPool<Phaser.GameObjects.Sprite>;

  constructor(private scene: Phaser.Scene) {
    this.particlePool = new ObjectPool(
      () => scene.add.sprite(0, 0, 'particle'),
      (sprite) => {
        sprite.setVisible(false);
        sprite.setActive(false);
        sprite.setScale(1);
        sprite.setAlpha(1);
      },
      50
    );
  }

  emitParticle(x: number, y: number, texture: string): void {
    const particle = this.particlePool.acquire();
    particle.setTexture(texture);
    particle.setPosition(x, y);
    particle.setVisible(true);
    particle.setActive(true);

    // Animate particle
    this.scene.tweens.add({
      targets: particle,
      alpha: 0,
      y: y - 50,
      duration: 1000,
      onComplete: () => {
        this.particlePool.release(particle);
      }
    });
  }

  destroy(): void {
    this.particlePool.releaseAll();
  }
}

// Texture management
class TextureManager {
  private scene: Phaser.Scene;
  private textureUsage: Map<string, number> = new Map();

  constructor(scene: Phaser.Scene) {
    this.scene = scene;
  }

  /**
   * Load texture and track usage
   */
  loadTexture(key: string, url: string): void {
    if (!this.scene.textures.exists(key)) {
      this.scene.load.image(key, url);
      this.textureUsage.set(key, 0);
    }
  }

  /**
   * Increment usage counter
   */
  useTexture(key: string): void {
    const count = this.textureUsage.get(key) || 0;
    this.textureUsage.set(key, count + 1);
  }

  /**
   * Decrement usage counter and unload if unused
   */
  releaseTexture(key: string): void {
    const count = this.textureUsage.get(key) || 0;
    if (count > 0) {
      this.textureUsage.set(key, count - 1);

      if (count - 1 === 0) {
        // Texture no longer in use, can unload
        this.unloadTexture(key);
      }
    }
  }

  /**
   * Unload texture from memory
   */
  unloadTexture(key: string): void {
    if (this.scene.textures.exists(key)) {
      this.scene.textures.remove(key);
      this.textureUsage.delete(key);
    }
  }

  /**
   * Clean up unused textures
   */
  cleanupUnused(): void {
    this.textureUsage.forEach((count, key) => {
      if (count === 0) {
        this.unloadTexture(key);
      }
    });
  }
}
```

### 7.3 Mobile Optimization

```typescript
// Mobile-specific optimizations
class MobileOptimizer {
  private scene: Phaser.Scene;
  private isMobile: boolean;

  constructor(scene: Phaser.Scene) {
    this.scene = scene;
    this.isMobile = this.detectMobile();
  }

  private detectMobile(): boolean {
    return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
      navigator.userAgent
    );
  }

  /**
   * Apply mobile-specific optimizations
   */
  optimize(): void {
    if (!this.isMobile) return;

    // Reduce particle count
    this.optimizeParticles();

    // Disable expensive visual effects
    this.disableExpensiveEffects();

    // Simplify physics
    this.simplifyPhysics();

    // Add touch controls
    this.addTouchControls();

    // Adjust UI for touch
    this.optimizeUIForTouch();
  }

  private optimizeParticles(): void {
    // Reduce max particles
    const particleConfig = {
      maxParticles: 30, // Down from 100
      emitZone: {
        quantity: 5 // Down from 15
      }
    };
  }

  private disableExpensiveEffects(): void {
    // Disable screen shake on mobile
    this.scene.cameras.main.setShake = () => {};

    // Disable glow effects
    // Disable complex filters
  }

  private simplifyPhysics(): void {
    if (this.scene.physics.world) {
      // Reduce physics FPS
      this.scene.physics.world.setFPS(30); // Down from 60
    }
  }

  private addTouchControls(): void {
    // Virtual joystick for movement
    this.createVirtualJoystick();

    // Touch buttons for actions
    this.createTouchButtons();
  }

  private createVirtualJoystick(): void {
    const joystick = this.scene.add.circle(100, 620, 50, 0xffffff, 0.3);
    joystick.setStrokeStyle(2, 0xffffff, 0.8);
    joystick.setScrollFactor(0);

    const stick = this.scene.add.circle(100, 620, 20, 0xffffff, 0.6);
    stick.setScrollFactor(0);

    joystick.setInteractive();
    this.scene.input.on('pointermove', (pointer: Phaser.Input.Pointer) => {
      if (pointer.isDown) {
        const distance = Phaser.Math.Distance.Between(
          joystick.x,
          joystick.y,
          pointer.x,
          pointer.y
        );

        if (distance < 50) {
          stick.setPosition(pointer.x, pointer.y);
        } else {
          const angle = Phaser.Math.Angle.Between(
            joystick.x,
            joystick.y,
            pointer.x,
            pointer.y
          );
          stick.setPosition(
            joystick.x + Math.cos(angle) * 50,
            joystick.y + Math.sin(angle) * 50
          );
        }

        // Emit movement event
        this.scene.events.emit('joystick', {
          x: stick.x - joystick.x,
          y: stick.y - joystick.y
        });
      }
    });

    this.scene.input.on('pointerup', () => {
      stick.setPosition(joystick.x, joystick.y);
    });
  }

  private createTouchButtons(): void {
    const buttons = [
      { x: 1180, y: 620, text: 'A', action: 'confirm' },
      { x: 1100, y: 580, text: 'B', action: 'cancel' }
    ];

    buttons.forEach(btn => {
      const button = this.scene.add.circle(btn.x, btn.y, 30, 0xffffff, 0.3);
      button.setStrokeStyle(2, 0xffffff, 0.8);
      button.setScrollFactor(0);
      button.setInteractive();

      const label = this.scene.add.text(btn.x, btn.y, btn.text, {
        fontSize: '20px',
        color: '#ffffff',
        fontStyle: 'bold'
      }).setOrigin(0.5).setScrollFactor(0);

      button.on('pointerdown', () => {
        this.scene.events.emit(btn.action);
        button.setScale(0.9);
      });

      button.on('pointerup', () => {
        button.setScale(1);
      });
    });
  }

  private optimizeUIForTouch(): void {
    // Increase button sizes for touch
    // Add more spacing between interactive elements
    // Show touch feedback (highlights on tap)
  }
}
```

### 7.4 Performance Monitoring

```typescript
// Performance monitoring and profiling
class PerformanceMonitor {
  private scene: Phaser.Scene;
  private fpsText!: Phaser.GameObjects.Text;
  private memoryText!: Phaser.GameObjects.Text;
  private enabled: boolean = false;

  constructor(scene: Phaser.Scene) {
    this.scene = scene;
  }

  enable(): void {
    this.enabled = true;

    // FPS counter
    this.fpsText = this.scene.add.text(10, 10, 'FPS: 60', {
      fontSize: '16px',
      color: '#00ff00',
      backgroundColor: 'rgba(0, 0, 0, 0.7)',
      padding: { x: 8, y: 4 }
    }).setScrollFactor(0).setDepth(10000);

    // Memory usage (if available)
    if ((performance as any).memory) {
      this.memoryText = this.scene.add.text(10, 35, 'Memory: 0 MB', {
        fontSize: '16px',
        color: '#00ff00',
        backgroundColor: 'rgba(0, 0, 0, 0.7)',
        padding: { x: 8, y: 4 }
      }).setScrollFactor(0).setDepth(10000);
    }
  }

  disable(): void {
    this.enabled = false;
    this.fpsText?.destroy();
    this.memoryText?.destroy();
  }

  update(): void {
    if (!this.enabled) return;

    // Update FPS
    const fps = Math.round(this.scene.game.loop.actualFps);
    this.fpsText.setText(`FPS: ${fps}`);

    // Color code FPS
    if (fps >= 50) {
      this.fpsText.setColor('#00ff00');
    } else if (fps >= 30) {
      this.fpsText.setColor('#ffff00');
    } else {
      this.fpsText.setColor('#ff0000');
    }

    // Update memory (Chrome only)
    if (this.memoryText && (performance as any).memory) {
      const memory = (performance as any).memory;
      const usedMB = Math.round(memory.usedJSHeapSize / 1048576);
      const totalMB = Math.round(memory.jsHeapSizeLimit / 1048576);
      this.memoryText.setText(`Memory: ${usedMB}/${totalMB} MB`);

      // Warn if memory usage is high
      if (usedMB / totalMB > 0.8) {
        this.memoryText.setColor('#ff0000');
      } else {
        this.memoryText.setColor('#00ff00');
      }
    }
  }

  /**
   * Log performance metrics
   */
  logMetrics(): void {
    const metrics = {
      fps: Math.round(this.scene.game.loop.actualFps),
      delta: this.scene.game.loop.delta,
      frame: this.scene.game.loop.frame,
      textures: this.scene.textures.list.length,
      gameObjects: this.scene.children.length
    };

    if ((performance as any).memory) {
      const memory = (performance as any).memory;
      Object.assign(metrics, {
        usedMemoryMB: Math.round(memory.usedJSHeapSize / 1048576),
        totalMemoryMB: Math.round(memory.jsHeapSizeLimit / 1048576)
      });
    }

    console.table(metrics);
  }
}
```

---

## 8. Development Setup

### 8.1 Project Structure

```
hybrid-rpg/
├── .github/
│   └── workflows/
│       └── build-and-deploy.yml      # CI/CD pipeline
│
├── public/
│   ├── index.html
│   └── favicon.ico
│
├── src/
│   ├── main.ts                       # Entry point
│   │
│   ├── config/
│   │   ├── game.config.ts            # Phaser configuration
│   │   └── constants.ts              # Game constants
│   │
│   ├── scenes/
│   │   ├── BootScene.ts              # Asset loading
│   │   ├── WorldMapScene.ts          # World navigation
│   │   ├── CityExplorationScene.ts   # City hub
│   │   ├── CombatScene.ts            # Battle system
│   │   ├── MenuScene.ts              # Menus & management
│   │   └── DialogScene.ts            # Dialog & skill checks
│   │
│   ├── systems/
│   │   ├── CombatStateMachine.ts     # Combat flow
│   │   ├── ProgressionManager.ts     # XP & leveling
│   │   ├── SkillCheckSystem.ts       # D&D mechanics
│   │   ├── SaveManager.ts            # Save/load
│   │   └── EventBus.ts               # Global events
│   │
│   ├── components/
│   │   ├── Entity.ts                 # ECS base
│   │   ├── StatsComponent.ts         # Character stats
│   │   ├── SpriteComponent.ts        # Visual representation
│   │   └── StatusEffectComponent.ts  # Buffs/debuffs
│   │
│   ├── generators/
│   │   ├── ProceduralSpriteGenerator.ts  # Asset generation
│   │   ├── CityLayoutGenerator.ts         # Map generation
│   │   └── AIAssetGenerator.ts            # AI integration
│   │
│   ├── ui/
│   │   ├── HPBar.ts                  # HP display
│   │   ├── ActionMenu.ts             # Combat menu
│   │   ├── DialogBox.ts              # Text display
│   │   └── CharacterSheet.ts         # Stats UI
│   │
│   ├── data/
│   │   ├── characters.json
│   │   ├── abilities.json
│   │   ├── enemies.json
│   │   ├── items.json
│   │   ├── quests.json
│   │   ├── world.json
│   │   └── npcs.json
│   │
│   ├── state/
│   │   ├── store.ts                  # Redux store
│   │   └── slices/
│   │       ├── gameStateSlice.ts
│   │       ├── charactersSlice.ts
│   │       ├── inventorySlice.ts
│   │       ├── questsSlice.ts
│   │       └── worldSlice.ts
│   │
│   ├── types/
│   │   ├── character.types.ts
│   │   ├── combat.types.ts
│   │   ├── dialog.types.ts
│   │   └── game.types.ts
│   │
│   └── utils/
│       ├── dice.ts                   # Dice rolling
│       ├── math.ts                   # Math helpers
│       └── random.ts                 # Seeded random
│
├── scripts/
│   └── generate-game-data.ts         # Data generation
│
├── tests/
│   ├── unit/
│   │   ├── dice.test.ts
│   │   ├── progression.test.ts
│   │   └── combat.test.ts
│   │
│   └── e2e/
│       ├── combat.spec.ts
│       └── exploration.spec.ts
│
├── .gitignore
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

### 8.2 Installation Instructions

```bash
# Clone repository
git clone https://github.com/yourusername/hybrid-rpg.git
cd hybrid-rpg

# Install dependencies
npm install

# Generate game data
npm run generate:data

# Start development server
npm run dev

# Open browser to http://localhost:3000
```

### 8.3 npm Scripts

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:e2e": "playwright test",
    "lint": "eslint src --ext ts,tsx",
    "format": "prettier --write src/**/*.ts",
    "generate:data": "ts-node scripts/generate-game-data.ts",
    "optimize:assets": "ts-node scripts/optimize-assets.ts",
    "deploy": "npm run build && gh-pages -d dist"
  }
}
```

### 8.4 Environment Configuration

```typescript
// .env.example
# Optional: AI API keys for enhanced asset generation
VITE_OPENAI_API_KEY=your_key_here
VITE_STABILITY_API_KEY=your_key_here

# Optional: Analytics
VITE_GA_TRACKING_ID=UA-XXXXX-X

# Development
VITE_DEBUG_MODE=true
VITE_SHOW_PERFORMANCE=true
```

---

## 9. Deployment Pipeline

### 9.1 GitHub Actions Workflow

The complete workflow is defined in `.github/workflows/build-and-deploy.yml` (shown in section 5.3).

**Deployment Steps:**

1. **On Push to `main`:**
   - Run linter
   - Run unit tests
   - Build production bundle
   - Generate game data
   - Optimize assets
   - Deploy to GitHub Pages

2. **On Pull Request:**
   - Run all tests
   - Build preview
   - Upload artifacts for review

3. **On Tag Push (releases):**
   - Create GitHub release
   - Attach build artifacts

### 9.2 Deployment Targets

**Primary: GitHub Pages**
- Free hosting
- Automatic HTTPS
- Custom domain support
- Version history

**Alternative: Netlify/Vercel**
```yaml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

**Alternative: Itch.io**
```bash
# Upload to itch.io
npm run build
zip -r game.zip dist/
# Upload game.zip to itch.io project
```

### 9.3 Versioning Strategy

```typescript
// src/config/version.ts
export const VERSION = {
  major: 1,
  minor: 0,
  patch: 0,
  build: process.env.VITE_BUILD_NUMBER || 'dev'
};

export const VERSION_STRING = `v${VERSION.major}.${VERSION.minor}.${VERSION.patch}`;

// Show in-game
console.log(`Game Version: ${VERSION_STRING}-${VERSION.build}`);
```

---

## 10. Testing Strategy

### 10.1 Unit Tests

```typescript
// tests/unit/dice.test.ts
import { describe, it, expect } from 'vitest';
import { DiceRoller } from '@/systems/SkillCheckSystem';

describe('DiceRoller', () => {
  it('should roll 1d20 correctly', () => {
    const result = DiceRoller.roll('1d20');
    expect(result.rolls).toHaveLength(1);
    expect(result.rolls[0]).toBeGreaterThanOrEqual(1);
    expect(result.rolls[0]).toBeLessThanOrEqual(20);
    expect(result.total).toBe(result.rolls[0]);
  });

  it('should apply modifiers correctly', () => {
    const result = DiceRoller.roll('1d20+5');
    expect(result.modifier).toBe(5);
    expect(result.total).toBe(result.rolls[0] + 5);
  });

  it('should handle multiple dice', () => {
    const result = DiceRoller.roll('3d6');
    expect(result.rolls).toHaveLength(3);
    result.rolls.forEach(roll => {
      expect(roll).toBeGreaterThanOrEqual(1);
      expect(roll).toBeLessThanOrEqual(6);
    });
  });

  it('should detect critical hits', () => {
    // Mock random to always roll 20
    const originalRandom = Math.random;
    Math.random = () => 0.99;

    const result = DiceRoller.roll('1d20');
    expect(result.rolls[0]).toBe(20);
    expect(result.isCritical).toBe(true);

    Math.random = originalRandom;
  });
});
```

```typescript
// tests/unit/progression.test.ts
import { describe, it, expect } from 'vitest';
import { ProgressionManager } from '@/systems/ProgressionManager';

describe('ProgressionManager', () => {
  it('should award XP correctly', () => {
    const character: Character = {
      id: 'test',
      level: 1,
      xp: 0,
      // ... other properties
    };

    const result = ProgressionManager.awardXP(character, 300);

    expect(character.xp).toBe(300);
    expect(result).not.toBeNull();
    expect(result!.newLevel).toBe(2);
  });

  it('should increase stats on level up', () => {
    const character: Character = {
      id: 'test',
      level: 1,
      xp: 0,
      class: 'fighter',
      maxHp: 12,
      // ... other properties
    };

    const oldMaxHp = character.maxHp;
    const result = ProgressionManager.awardXP(character, 300);

    expect(character.maxHp).toBeGreaterThan(oldMaxHp);
  });
});
```

### 10.2 Integration Tests

```typescript
// tests/integration/combat.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import Phaser from 'phaser';
import { CombatScene } from '@/scenes/CombatScene';

describe('CombatScene', () => {
  let game: Phaser.Game;
  let scene: CombatScene;

  beforeEach(() => {
    game = new Phaser.Game({
      type: Phaser.HEADLESS,
      scene: CombatScene
    });

    scene = game.scene.getScene('CombatScene') as CombatScene;
  });

  it('should initialize combat correctly', () => {
    expect(scene).toBeDefined();
    expect(scene.getAllies().length).toBeGreaterThan(0);
    expect(scene.getEnemies().length).toBeGreaterThan(0);
  });

  it('should handle turn order correctly', () => {
    const combatants = scene.getAllCombatants();
    const speeds = combatants.map(c => c.stats.speed);

    // Verify turn order is sorted by speed
    for (let i = 0; i < speeds.length - 1; i++) {
      expect(speeds[i]).toBeGreaterThanOrEqual(speeds[i + 1]);
    }
  });

  afterEach(() => {
    game.destroy(true);
  });
});
```

### 10.3 E2E Tests (Playwright)

```typescript
// tests/e2e/combat.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Combat System', () => {
  test('should complete a full combat encounter', async ({ page }) => {
    await page.goto('http://localhost:3000');

    // Wait for game to load
    await page.waitForSelector('#game-container');

    // Start combat encounter
    await page.click('[data-testid="start-combat"]');

    // Wait for combat scene
    await page.waitForSelector('[data-testid="combat-scene"]');

    // Verify UI elements
    const actionMenu = page.locator('[data-testid="action-menu"]');
    await expect(actionMenu).toBeVisible();

    // Select attack action
    await page.click('[data-testid="action-attack"]');

    // Select target
    await page.click('[data-testid="enemy-0"]');

    // Wait for animation
    await page.waitForTimeout(2000);

    // Verify damage was dealt
    const enemyHP = page.locator('[data-testid="enemy-0-hp"]');
    await expect(enemyHP).not.toContainText('100%');
  });

  test('should display skill check dialog', async ({ page }) => {
    await page.goto('http://localhost:3000');

    // Navigate to city
    await page.click('[data-testid="enter-city"]');

    // Talk to NPC
    await page.click('[data-testid="npc-scientist"]');

    // Wait for dialog
    await page.waitForSelector('[data-testid="dialog-box"]');

    // Select skill check option
    await page.click('[data-testid="choice-persuasion"]');

    // Wait for dice roll animation
    await page.waitForSelector('[data-testid="dice-roll"]');

    // Verify result shown
    const result = page.locator('[data-testid="check-result"]');
    await expect(result).toBeVisible();
  });
});
```

### 10.4 Performance Testing

```typescript
// tests/performance/load-test.ts
import { test, expect } from '@playwright/test';

test.describe('Performance', () => {
  test('should load under 3 seconds', async ({ page }) => {
    const startTime = Date.now();

    await page.goto('http://localhost:3000');
    await page.waitForSelector('#game-container');

    const loadTime = Date.now() - startTime;
    expect(loadTime).toBeLessThan(3000);
  });

  test('should maintain 30+ FPS during combat', async ({ page }) => {
    await page.goto('http://localhost:3000');

    // Enable performance monitor
    await page.evaluate(() => {
      (window as any).game.enablePerformanceMonitor();
    });

    // Start combat
    await page.click('[data-testid="start-combat"]');

    // Let combat run for 10 seconds
    await page.waitForTimeout(10000);

    // Check average FPS
    const avgFPS = await page.evaluate(() => {
      return (window as any).game.getAverageFPS();
    });

    expect(avgFPS).toBeGreaterThan(30);
  });
});
```

---

## Appendix A: Type Definitions

```typescript
// Complete type definitions for reference
interface Character {
  id: string;
  name: string;
  class: CharacterClass;
  level: number;
  xp: number;

  abilities: {
    strength: number;
    dexterity: number;
    constitution: number;
    intelligence: number;
    wisdom: number;
    charisma: number;
  };

  skills: {
    [key in Skill]: {
      proficient: boolean;
      bonus: number;
    }
  };

  hp: number;
  maxHp: number;
  mp: number;
  maxMp: number;
  ac: number;
  speed: number;
  initiative: number;

  equipment: {
    weapon: string | null;
    armor: string | null;
    accessory1: string | null;
    accessory2: string | null;
  };

  abilities: string[];
  spellSlots: {
    level1: { current: number; max: number };
    level2: { current: number; max: number };
    level3: { current: number; max: number };
    level4: { current: number; max: number };
    level5: { current: number; max: number };
  };

  supports: Record<string, number>;
  abilityPoints: number;
  unlockedAbilities: string[];

  visualData: CharacterVisualData;
}

type CharacterClass = 'fighter' | 'mage' | 'cleric' | 'rogue';

interface CharacterVisualData {
  procedural: {
    bodyType: string;
    hairColor: string;
    skinTone: string;
    height: number;
    equipment: boolean;
  };
  class: CharacterClass;
}

interface Combatant {
  id: string;
  name: string;
  isPlayer: boolean;
  stats: {
    hp: number;
    maxHp: number;
    mp: number;
    maxMp: number;
    ac: number;
    speed: number;
    abilities: Record<string, number>;
  };
  position: { x: number; y: number };
  sprite: Phaser.GameObjects.Sprite;
  hpBar: HPBar;
  statusIcons: Phaser.GameObjects.Image[];
  statusEffects: StatusEffect[];
  character?: Character;
  enemyDef?: any;
}

interface BattleAction {
  id: string;
  name: string;
  type: 'action' | 'bonus_action' | 'reaction';
  range: 'melee' | 'ranged' | number;
  targetType: 'single_enemy' | 'single_ally' | 'all_enemies' | 'all_allies' | 'self';
  requiresTarget: boolean;
  cost?: {
    type: 'mp' | 'stamina' | 'spell_slot' | 'uses_per_rest';
    amount?: number;
    level?: number;
  };
  effects: ActionEffect[];
  animation: string;
  hitEffects?: string[];
  damageType?: 'physical' | 'magical' | 'fire' | 'ice' | 'lightning';
  abilityUsed?: 'strength' | 'dexterity' | 'intelligence' | 'wisdom';
}

interface ActionEffect {
  type: 'damage' | 'heal' | 'buff' | 'debuff' | 'status';
  formula?: string;
  target: 'self' | 'selected' | 'all';
  duration?: number;
  statusEffect?: string;
}

interface StatusEffect {
  id: string;
  name: string;
  type: 'buff' | 'debuff';
  duration: number;
  effects: Record<string, number>;
  icon: string;
}

interface AnimationSequence {
  type: 'action' | 'damage' | 'heal';
  actor: Combatant;
  action: BattleAction;
  targets: Combatant[];
  onComplete?: () => void;
}
```

---

## Appendix B: Configuration Examples

```typescript
// Complete game configuration
export const GAME_CONFIG = {
  // Display
  WIDTH: 1280,
  HEIGHT: 720,
  TITLE: 'Hybrid RPG',

  // Gameplay
  MAX_PARTY_SIZE: 4,
  MAX_LEVEL: 20,
  STARTING_GOLD: 100,
  AUTOSAVE_INTERVAL: 60000, // 1 minute

  // Combat
  COMBAT: {
    TURN_TIME_LIMIT: 60, // seconds
    MAX_COMBATANTS: 10,
    CRIT_MULTIPLIER: 2.0,
    BASE_HIT_CHANCE: 0.85
  },

  // Progression
  XP_SCALING: 'hybrid', // 'linear' | 'exponential' | 'hybrid'
  STAT_GROWTH_VARIANCE: 0.2, // ±20%

  // UI
  UI: {
    ANIMATION_SPEED: 1.0,
    TEXT_SPEED: 30, // ms per character
    SHOW_DAMAGE_NUMBERS: true,
    SHOW_HIT_CHANCES: true
  },

  // Performance
  PERFORMANCE: {
    MAX_PARTICLES: 100,
    ENABLE_SHADOWS: true,
    ENABLE_POST_PROCESSING: true,
    TARGET_FPS: 60
  },

  // Debug
  DEBUG: {
    SHOW_FPS: false,
    SHOW_HITBOXES: false,
    GOD_MODE: false,
    UNLIMITED_RESOURCES: false
  }
};
```

---

## Appendix C: Data File Examples

```json
// Example: src/data/characters.json
{
  "player": {
    "id": "player",
    "name": "Hero",
    "class": "fighter",
    "baseStats": {
      "strength": 16,
      "dexterity": 12,
      "constitution": 14,
      "intelligence": 10,
      "wisdom": 11,
      "charisma": 13
    },
    "startingLevel": 1,
    "startingEquipment": ["longsword", "chainmail", "shield"],
    "proficiencies": {
      "skills": ["athletics", "intimidation"],
      "saves": ["strength", "constitution"]
    }
  }
}
```

```json
// Example: src/data/quests.json
{
  "main_001": {
    "id": "main_001",
    "type": "main",
    "title": "The Awakening",
    "description": "Strange energy readings have been detected in the Solar District.",
    "objectives": [
      {
        "id": "talk_to_scientist",
        "type": "talk_to_npc",
        "target": "npc_scientist",
        "description": "Speak with Dr. Maya at the Research Lab"
      }
    ],
    "rewards": {
      "xp": 500,
      "gold": 200,
      "items": ["solar_charm"]
    }
  }
}
```

---

## Conclusion

This Technical Architecture Blueprint provides a complete foundation for building a web-based hybrid RPG with zero-touch development workflow. The architecture is:

- **Modular**: Each system can be developed and tested independently
- **Scalable**: Easy to add new cities, characters, and features
- **Maintainable**: Type-safe TypeScript, clear separation of concerns
- **Performant**: Optimized for both desktop and mobile browsers
- **Automated**: CI/CD pipeline handles all deployment steps

### Next Steps

1. **Initialize project**: Set up repository and install dependencies
2. **Implement core systems**: Start with state management and scene architecture
3. **Build procedural generators**: Create asset generation pipeline
4. **Develop game scenes**: Implement world map, exploration, combat
5. **Test thoroughly**: Unit tests, integration tests, E2E tests
6. **Deploy**: Push to GitHub and let CI/CD handle deployment

### Key Success Factors

- **Zero-touch workflow**: All assets generated via code
- **Data-driven design**: Easy to modify game content without code changes
- **Performance-first**: Optimized for 60 FPS on desktop, 30 FPS on mobile
- **Player experience**: Smooth gameplay, clear feedback, engaging mechanics

**Total Implementation Estimate:** 3-6 months for MVP with core features

---

*Document Version: 1.0*
*Last Updated: November 1, 2025*
*Total Word Count: ~30,000 words*
