# Comprehensive Guide to Web-Based RPG Development with Zero-Touch Workflow

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Recommended Tech Stack](#recommended-tech-stack)
3. [Web-Based RPG Frameworks & Engines](#web-based-rpg-frameworks--engines)
4. [Turn-Based Combat Implementation](#turn-based-combat-implementation)
5. [Map & Exploration Systems](#map--exploration-systems)
6. [Save Systems & State Management](#save-systems--state-management)
7. [Performance Optimization](#performance-optimization)
8. [Zero-Touch Development Workflow](#zero-touch-development-workflow)
9. [Architecture Patterns](#architecture-patterns)
10. [Step-by-Step Implementation Guide](#step-by-step-implementation-guide)

---

## Executive Summary

This guide provides comprehensive technical documentation for building web-based RPG games with a zero-touch development workflow, emphasizing code-only asset generation, procedural content creation, and automated deployment.

**Key Findings:**
- **Phaser 3** is the most mature and well-documented framework for web-based RPG development
- **WebGL** offers superior performance for complex RPGs but requires more expertise than Canvas 2D
- **IndexedDB** is essential for complex RPG save systems (localStorage limited to ~5MB)
- **Zero-touch workflows** are achievable through procedural generation and API-based asset creation
- **State machines** are the recommended pattern for battle systems and game flow
- **GitHub Actions** provides robust CI/CD for automated builds and deployment

---

## Recommended Tech Stack

### Core Game Engine
```javascript
// Primary: Phaser 3
{
  engine: "Phaser 3.80+",
  renderer: "WebGL (with Canvas 2D fallback)",
  language: "JavaScript/TypeScript",
  bundler: "Webpack 5 or Vite",
  stateManagement: "Redux or MobX"
}
```

### Asset Pipeline
```javascript
{
  maps: "Tiled Map Editor (JSON export)",
  sprites: "Procedural generation + AI tools",
  audio: "Web Audio API",
  assetLoading: "Asynchronous with preloading"
}
```

### Data & Persistence
```javascript
{
  localSaves: "IndexedDB",
  cloudSaves: "Firebase or Supabase",
  stateFormat: "JSON with custom serialization"
}
```

### Development Tools
```javascript
{
  ci_cd: "GitHub Actions",
  deployment: "GitHub Pages / Netlify / Vercel",
  testing: "Jest + Puppeteer",
  assetGeneration: "Procedural tools + AI APIs"
}
```

---

## Web-Based RPG Frameworks & Engines

### Phaser 3 - Recommended Choice

**Strengths:**
- Mature ecosystem with 97% of browsers supporting WebGL
- Built-in animation system with timing controls
- Extensive plugin system for RPG-specific features
- Strong community with numerous RPG tutorials
- Device scaling and mobile browser support out of the box

**Key Features for RPGs:**
```javascript
// Scene Management for RPG structure
class BattleScene extends Phaser.Scene {
  create() {
    // Battle initialization
    this.setupCombatSystem();
    this.loadEnemies();
    this.initializeParty();
  }

  update() {
    // Game loop for battle logic
    this.battleStateMachine.update();
  }
}

// Animation with timing control
this.anims.create({
  key: 'attack',
  frames: this.anims.generateFrameNumbers('hero', { start: 0, end: 3 }),
  frameRate: 10,
  repeat: 0
});

// Event-based sequencing
this.events.on('animationcomplete-attack', () => {
  this.dealDamage();
});
```

**RPG-Specific Plugins:**
- Pathfinding plugins (EasyStar.js integration)
- Dialog system plugins
- Inventory management extensions
- Turn-based battle plugins

**Best Practices:**
1. Use **scene-based architecture** (separate scenes for world map, battles, menus)
2. Implement **event-driven communication** between scenes
3. Leverage **Phaser's built-in state management** for game objects
4. Use **Tiled Map Editor** integration for level design

### RPG Maker MV/MZ Web Export

**Capabilities:**
- Built on HTML5/JavaScript foundation
- All exports (Windows, macOS, mobile) are HTML5-wrapped
- Web export creates a `www` folder with index.html
- Save games stored in HTML5 storage (not cookies)

**Pros:**
- Rapid prototyping for traditional JRPGs
- Built-in battle systems and event handling
- No coding required for basic functionality

**Cons:**
- Less flexible than Phaser 3 for custom mechanics
- Larger bundle sizes
- Limited control over rendering pipeline
- Not ideal for zero-touch workflows (requires GUI editor)

**Recommendation:** Use for prototyping or if you need traditional JRPG mechanics without custom code. For zero-touch workflows, Phaser 3 is superior.

### Canvas 2D vs WebGL

**Performance Analysis (2025):**

| Aspect | Canvas 2D | WebGL |
|--------|-----------|-------|
| Browser Support | 99%+ | 97%+ |
| Performance (simple) | Good | Overhead may reduce performance |
| Performance (complex) | Poor at scale | Excellent |
| GPU Acceleration | Limited | Full |
| Learning Curve | Easy | Moderate-Hard |
| Matrix Transforms | CPU-bound | GPU-accelerated |

**Canvas 2D Best For:**
- Simple RPGs with <50 sprites on screen
- Turn-based games with minimal animation
- Pixel-perfect rendering needs
- Projects prioritizing development speed

**WebGL Best For:**
- Complex particle effects (spell animations)
- Large numbers of sprites (>100 simultaneous)
- Real-time lighting/shadows
- Advanced post-processing effects

**Recommendation for RPGs:**
```javascript
// Start with Canvas 2D, implement WebGL backend later
const config = {
  type: Phaser.AUTO, // Auto-selects WebGL with Canvas fallback
  width: 800,
  height: 600,
  pixelArt: true, // For retro RPGs
  render: {
    antiAlias: false,
    pixelArt: true
  }
};
```

**Performance Impact:**
- Optimization techniques can increase framerate by **50%**
- Loading time reductions of **72%** with proper asset management
- WebGL can reduce triangle count by **69%** and overdraw by **30%**

---

## Turn-Based Combat Implementation

### Battle System Architecture Patterns

#### 1. State Machine Pattern (Recommended)

```javascript
class BattleStateMachine {
  constructor() {
    this.states = {
      INITIALIZE: 'initialize',
      PLAYER_TURN: 'playerTurn',
      SELECT_ACTION: 'selectAction',
      SELECT_TARGET: 'selectTarget',
      EXECUTE_ACTION: 'executeAction',
      ENEMY_TURN: 'enemyTurn',
      VICTORY: 'victory',
      DEFEAT: 'defeat'
    };

    this.currentState = this.states.INITIALIZE;
    this.actorQueue = [];
  }

  transition(newState) {
    console.log(`Battle: ${this.currentState} -> ${newState}`);
    this.exitState(this.currentState);
    this.currentState = newState;
    this.enterState(newState);
  }

  enterState(state) {
    switch(state) {
      case this.states.PLAYER_TURN:
        this.enablePlayerInput();
        break;
      case this.states.EXECUTE_ACTION:
        this.performAction();
        break;
      case this.states.ENEMY_TURN:
        this.performEnemyAI();
        break;
    }
  }

  exitState(state) {
    switch(state) {
      case this.states.PLAYER_TURN:
        this.disablePlayerInput();
        break;
    }
  }

  update() {
    // State-specific update logic
    this.handleStateLogic(this.currentState);
  }
}
```

**Libraries:**
- **javascript-state-machine** - Finite state machine library
- **XState** - Visual state machine library with excellent tooling
- **Machina.js** - Hierarchical state machines

#### 2. Command Pattern for Actions

```javascript
class BattleCommand {
  constructor(actor, action, targets) {
    this.actor = actor;
    this.action = action;
    this.targets = targets;
  }

  execute() {
    return this.action.perform(this.actor, this.targets);
  }

  canExecute() {
    return this.actor.canUseAction(this.action);
  }
}

class AttackAction {
  perform(actor, targets) {
    const damage = this.calculateDamage(actor, targets[0]);
    targets[0].takeDamage(damage);
    return {
      type: 'damage',
      amount: damage,
      target: targets[0]
    };
  }

  calculateDamage(attacker, defender) {
    const baseDamage = attacker.stats.attack - defender.stats.defense;
    const variance = Math.random() * 0.2 + 0.9; // ±10%
    return Math.max(1, Math.floor(baseDamage * variance));
  }
}
```

#### 3. Turn Order System (ATB/Speed-Based)

```javascript
class TurnOrderSystem {
  constructor() {
    this.actors = [];
    this.turnQueue = [];
  }

  // Active Time Battle style
  updateATB(deltaTime) {
    this.actors.forEach(actor => {
      actor.energy += actor.stats.speed * deltaTime;

      if (actor.energy >= 100 && !this.turnQueue.includes(actor)) {
        this.turnQueue.push(actor);
        actor.energy = 0;
      }
    });
  }

  // Traditional speed-based
  calculateTurnOrder() {
    return this.actors
      .sort((a, b) => b.stats.speed - a.stats.speed)
      .map(actor => actor.id);
  }

  getNextActor() {
    return this.turnQueue.shift();
  }
}
```

### Animation and Timing in Combat

#### Phaser 3 Animation Sequencing

```javascript
class BattleAnimationController {
  constructor(scene) {
    this.scene = scene;
    this.sequenceQueue = [];
    this.isPlaying = false;
  }

  // Queue animation sequences
  queueSequence(sequence) {
    this.sequenceQueue.push(sequence);
    if (!this.isPlaying) {
      this.playNext();
    }
  }

  playNext() {
    if (this.sequenceQueue.length === 0) {
      this.isPlaying = false;
      return;
    }

    this.isPlaying = true;
    const sequence = this.sequenceQueue.shift();
    this.playSequence(sequence);
  }

  // Example: Attack sequence
  playAttackSequence(attacker, target, damage) {
    const timeline = this.scene.tweens.createTimeline();

    // Step 1: Attacker moves forward
    timeline.add({
      targets: attacker.sprite,
      x: attacker.sprite.x + 50,
      duration: 200,
      ease: 'Power2'
    });

    // Step 2: Play attack animation
    timeline.add({
      targets: attacker.sprite,
      duration: 300,
      onStart: () => {
        attacker.sprite.play('attack');
      }
    });

    // Step 3: Show damage and play hit effect
    timeline.add({
      targets: target.sprite,
      duration: 100,
      onStart: () => {
        this.showDamageNumber(target, damage);
        target.sprite.play('hurt');
        this.scene.cameras.main.shake(100, 0.005);
      }
    });

    // Step 4: Attacker returns to position
    timeline.add({
      targets: attacker.sprite,
      x: attacker.sprite.x,
      duration: 200,
      ease: 'Power2'
    });

    timeline.on('complete', () => {
      this.playNext();
    });

    timeline.play();
  }

  showDamageNumber(target, damage) {
    const damageText = this.scene.add.text(
      target.sprite.x,
      target.sprite.y - 30,
      damage.toString(),
      { fontSize: '32px', color: '#ff0000', fontStyle: 'bold' }
    );

    this.scene.tweens.add({
      targets: damageText,
      y: damageText.y - 50,
      alpha: 0,
      duration: 1000,
      ease: 'Power2',
      onComplete: () => damageText.destroy()
    });
  }
}
```

**Timing Best Practices:**
- Use **300ms delay** between action selection and execution
- **200-300ms** for movement animations
- **100-200ms** for hit effects
- **500-1000ms** for spell/skill animations
- Always wait for `ANIMATION_COMPLETE` event before next action

### Party Management and Character Switching

#### Party Data Structure

```javascript
class PartyManager {
  constructor() {
    this.activeParty = []; // Characters in battle (max 4)
    this.reserveParty = []; // Bench characters
    this.allCharacters = new Map(); // All recruited characters
  }

  // Add character to party
  addCharacter(character) {
    this.allCharacters.set(character.id, character);

    if (this.activeParty.length < 4) {
      this.activeParty.push(character);
    } else {
      this.reserveParty.push(character);
    }
  }

  // Swap party members
  swapCharacters(activeIndex, reserveIndex) {
    const temp = this.activeParty[activeIndex];
    this.activeParty[activeIndex] = this.reserveParty[reserveIndex];
    this.reserveParty[reserveIndex] = temp;

    // Trigger UI update event
    this.emit('partyChanged', this.activeParty);
  }

  // Mid-battle switching
  switchCharacter(outIndex, inCharacter) {
    const outCharacter = this.activeParty[outIndex];
    this.activeParty[outIndex] = inCharacter;

    // Move old character to reserve
    const reserveIndex = this.reserveParty.indexOf(inCharacter);
    if (reserveIndex !== -1) {
      this.reserveParty[reserveIndex] = outCharacter;
    }

    return {
      out: outCharacter,
      in: inCharacter
    };
  }

  // Get character at position
  getCharacter(position) {
    return this.activeParty[position];
  }

  // Check if party can continue
  isDefeated() {
    return this.activeParty.every(char => char.hp <= 0);
  }

  // Serialize for save
  serialize() {
    return {
      active: this.activeParty.map(c => c.id),
      reserve: this.reserveParty.map(c => c.id),
      characters: Array.from(this.allCharacters.values()).map(c => c.serialize())
    };
  }
}
```

#### Party Formation System

```javascript
class FormationManager {
  constructor() {
    this.formations = {
      STANDARD: {
        positions: [
          { x: 100, y: 150, row: 'front' },
          { x: 100, y: 200, row: 'front' },
          { x: 50, y: 175, row: 'back' },
          { x: 50, y: 225, row: 'back' }
        ]
      },
      DEFENSIVE: {
        positions: [
          { x: 75, y: 150, row: 'back' },
          { x: 75, y: 200, row: 'back' },
          { x: 75, y: 250, row: 'back' },
          { x: 75, y: 300, row: 'back' }
        ]
      },
      AGGRESSIVE: {
        positions: [
          { x: 150, y: 150, row: 'front' },
          { x: 150, y: 200, row: 'front' },
          { x: 150, y: 250, row: 'front' },
          { x: 100, y: 225, row: 'back' }
        ]
      }
    };

    this.currentFormation = 'STANDARD';
  }

  changeFormation(formationName) {
    this.currentFormation = formationName;
    return this.formations[formationName];
  }

  getPosition(characterIndex) {
    return this.formations[this.currentFormation].positions[characterIndex];
  }
}
```

### Effect Sequencing and Visual Feedback

```javascript
class EffectManager {
  constructor(scene) {
    this.scene = scene;
    this.effectPool = new Map();
  }

  // Particle effect for spell
  createSpellEffect(target, spellType) {
    const particles = this.scene.add.particles('particle');

    const emitter = particles.createEmitter({
      x: target.x,
      y: target.y,
      speed: { min: -100, max: 100 },
      angle: { min: 0, max: 360 },
      scale: { start: 1, end: 0 },
      blendMode: 'ADD',
      lifespan: 600,
      gravityY: 200
    });

    this.scene.time.delayedCall(1000, () => {
      particles.destroy();
    });
  }

  // Status effect indicators
  showStatusEffect(character, status) {
    const icon = this.scene.add.sprite(
      character.sprite.x + 20,
      character.sprite.y - 40,
      `status_${status}`
    );

    this.scene.tweens.add({
      targets: icon,
      y: icon.y - 5,
      yoyo: true,
      repeat: -1,
      duration: 500
    });

    character.statusIcons = character.statusIcons || [];
    character.statusIcons.push(icon);
  }

  // Damage type feedback (physical vs magical)
  showDamageEffect(target, damageType, amount) {
    const color = damageType === 'physical' ? '#ff6600' : '#66ccff';
    const text = this.scene.add.text(
      target.sprite.x,
      target.sprite.y - 30,
      amount.toString(),
      {
        fontSize: '28px',
        color: color,
        stroke: '#000000',
        strokeThickness: 4,
        fontStyle: 'bold'
      }
    );

    this.scene.tweens.add({
      targets: text,
      y: text.y - 60,
      alpha: 0,
      duration: 1200,
      ease: 'Cubic.easeOut',
      onComplete: () => text.destroy()
    });

    // Screen shake for critical hits
    if (damageType === 'critical') {
      this.scene.cameras.main.shake(150, 0.01);
    }
  }
}
```

---

## Map & Exploration Systems

### Tile-Based Map Rendering

#### Tiled Map Editor Integration with Phaser 3

**Workflow:**
1. Create maps in Tiled (.tmx format)
2. Export as JSON (uncompressed)
3. Load in Phaser 3

```javascript
class WorldScene extends Phaser.Scene {
  preload() {
    // Load tileset image
    this.load.image('tiles', 'assets/tilesets/dungeon.png');

    // Load map JSON
    this.load.tilemapTiledJSON('map1', 'assets/maps/dungeon_floor1.json');
  }

  create() {
    // Create tilemap
    const map = this.make.tilemap({ key: 'map1' });

    // Add tileset
    const tileset = map.addTilesetImage('dungeon', 'tiles');

    // Create layers
    this.groundLayer = map.createLayer('Ground', tileset, 0, 0);
    this.wallsLayer = map.createLayer('Walls', tileset, 0, 0);
    this.decorLayer = map.createLayer('Decorations', tileset, 0, 0);

    // Set collision
    this.wallsLayer.setCollisionByProperty({ collides: true });

    // Enable collision with player
    this.physics.add.collider(this.player, this.wallsLayer);
  }
}
```

**Tiled Export Settings (Critical):**
- Tile layer format: **CSV or Base64 (uncompressed)** - NOT Base64 (zlib compressed)
- Map format: **JSON**
- Tileset: Check **"Embed in Map"** option
- Export location: `assets/maps/`

#### Metro Map Style / City Exploration

```javascript
class MetroMapScene extends Phaser.Scene {
  create() {
    // Create node-based map instead of tile-based
    this.locations = [
      { id: 'downtown', x: 400, y: 300, name: 'Downtown', unlocked: true },
      { id: 'suburbs', x: 600, y: 400, name: 'Suburbs', unlocked: true },
      { id: 'industrial', x: 200, y: 450, name: 'Industrial Zone', unlocked: false }
    ];

    this.connections = [
      { from: 'downtown', to: 'suburbs', travelTime: 5 },
      { from: 'downtown', to: 'industrial', travelTime: 10 }
    ];

    this.drawMetroMap();
  }

  drawMetroMap() {
    // Draw connections (metro lines)
    const graphics = this.add.graphics();
    graphics.lineStyle(4, 0x666666);

    this.connections.forEach(conn => {
      const from = this.locations.find(l => l.id === conn.from);
      const to = this.locations.find(l => l.id === conn.to);

      graphics.beginPath();
      graphics.moveTo(from.x, from.y);
      graphics.lineTo(to.x, to.y);
      graphics.strokePath();
    });

    // Draw location nodes
    this.locations.forEach(loc => {
      const node = this.add.circle(
        loc.x, loc.y, 20,
        loc.unlocked ? 0x00ff00 : 0x888888
      );

      node.setInteractive();
      node.on('pointerdown', () => this.selectLocation(loc));

      const label = this.add.text(
        loc.x, loc.y + 30, loc.name,
        { fontSize: '16px', color: '#ffffff' }
      ).setOrigin(0.5);
    });
  }

  selectLocation(location) {
    if (!location.unlocked) return;

    this.scene.start('ExplorationScene', { location: location.id });
  }
}
```

### Collision Detection and Pathfinding

#### Tile-Based Collision

```javascript
class CollisionManager {
  constructor(map, collisionLayer) {
    this.map = map;
    this.collisionLayer = collisionLayer;
    this.tileSize = map.tileWidth;
  }

  // Convert world position to tile coordinates
  worldToTile(x, y) {
    return {
      x: Math.floor(x / this.tileSize),
      y: Math.floor(y / this.tileSize)
    };
  }

  // Check if tile is walkable
  isWalkable(tileX, tileY) {
    const tile = this.collisionLayer.getTileAt(tileX, tileY);
    return !tile || !tile.properties.collides;
  }

  // Get walkable neighbors for pathfinding
  getNeighbors(tileX, tileY) {
    const neighbors = [];
    const directions = [
      { x: 0, y: -1 }, // North
      { x: 1, y: 0 },  // East
      { x: 0, y: 1 },  // South
      { x: -1, y: 0 }  // West
    ];

    directions.forEach(dir => {
      const nx = tileX + dir.x;
      const ny = tileY + dir.y;

      if (this.isWalkable(nx, ny)) {
        neighbors.push({ x: nx, y: ny });
      }
    });

    return neighbors;
  }
}
```

#### A* Pathfinding Implementation

```javascript
// Using EasyStar.js library (recommended)
class PathfindingManager {
  constructor(scene, collisionLayer) {
    this.scene = scene;
    this.easystar = new EasyStar.js();

    // Create grid from collision layer
    this.setupGrid(collisionLayer);
  }

  setupGrid(collisionLayer) {
    const grid = [];
    for (let y = 0; y < collisionLayer.height; y++) {
      const row = [];
      for (let x = 0; x < collisionLayer.width; x++) {
        const tile = collisionLayer.getTileAt(x, y);
        row.push(tile && tile.properties.collides ? 1 : 0);
      }
      grid.push(row);
    }

    this.easystar.setGrid(grid);
    this.easystar.setAcceptableTiles([0]); // 0 = walkable
    this.easystar.enableDiagonals();
    this.easystar.enableCornerCutting(false);
  }

  findPath(startX, startY, endX, endY, callback) {
    this.easystar.findPath(startX, startY, endX, endY, (path) => {
      if (path === null) {
        console.log('No path found');
        callback(null);
      } else {
        callback(path);
      }
    });

    this.easystar.calculate();
  }

  // Move character along path
  followPath(character, path) {
    if (!path || path.length === 0) return;

    const timeline = this.scene.tweens.createTimeline();

    path.forEach((point, index) => {
      if (index === 0) return; // Skip starting position

      timeline.add({
        targets: character,
        x: point.x * 32, // Tile size
        y: point.y * 32,
        duration: 200,
        ease: 'Linear'
      });
    });

    timeline.play();
  }
}
```

**Performance Note:**
- EasyStar.js is asynchronous and won't block game loop
- For large maps (>100x100), consider caching paths
- Use A* for player movement, simpler algorithms for NPCs

### Event Triggering Systems

```javascript
class EventTriggerManager {
  constructor(scene) {
    this.scene = scene;
    this.triggers = new Map();
    this.activatedTriggers = new Set();
  }

  // Create trigger from Tiled object layer
  loadFromTiledMap(map) {
    const objectLayer = map.getObjectLayer('Events');

    objectLayer.objects.forEach(obj => {
      this.addTrigger({
        id: obj.name,
        x: obj.x,
        y: obj.y,
        width: obj.width,
        height: obj.height,
        type: obj.type,
        properties: obj.properties,
        once: obj.properties.once || false,
        condition: obj.properties.condition || null
      });
    });
  }

  addTrigger(config) {
    const trigger = this.scene.add.zone(
      config.x,
      config.y,
      config.width,
      config.height
    );

    this.scene.physics.add.existing(trigger);
    trigger.body.setAllowGravity(false);

    this.triggers.set(config.id, {
      zone: trigger,
      config: config
    });
  }

  checkTriggers(player) {
    this.triggers.forEach((trigger, id) => {
      // Skip if already activated and set to once
      if (trigger.config.once && this.activatedTriggers.has(id)) {
        return;
      }

      // Check if player overlaps trigger
      if (this.scene.physics.overlap(player, trigger.zone)) {
        // Check condition if specified
        if (trigger.config.condition) {
          if (!this.evaluateCondition(trigger.config.condition)) {
            return;
          }
        }

        this.activateTrigger(id, trigger.config);
      }
    });
  }

  activateTrigger(id, config) {
    console.log(`Trigger activated: ${id}`);

    switch(config.type) {
      case 'dialog':
        this.scene.events.emit('showDialog', config.properties.dialogId);
        break;
      case 'battle':
        this.scene.scene.start('BattleScene', {
          enemies: config.properties.enemies
        });
        break;
      case 'item':
        this.scene.events.emit('giveItem', config.properties.itemId);
        break;
      case 'teleport':
        this.scene.events.emit('teleport', {
          map: config.properties.targetMap,
          x: config.properties.targetX,
          y: config.properties.targetY
        });
        break;
    }

    if (config.once) {
      this.activatedTriggers.add(id);
    }
  }

  evaluateCondition(condition) {
    // Simple condition evaluation
    // Format: "flag:FLAG_NAME" or "item:ITEM_ID" or "level:5"
    const [type, value] = condition.split(':');

    switch(type) {
      case 'flag':
        return this.scene.gameState.hasFlag(value);
      case 'item':
        return this.scene.gameState.hasItem(value);
      case 'level':
        return this.scene.gameState.playerLevel >= parseInt(value);
      default:
        return true;
    }
  }
}
```

### Dialog and Cutscene Systems

#### Dialog System with Yarn/JSON

```javascript
class DialogSystem {
  constructor(scene) {
    this.scene = scene;
    this.dialogBox = null;
    this.isActive = false;
    this.currentDialog = null;
    this.currentNode = 0;
  }

  // Load dialog from JSON
  loadDialog(dialogId) {
    // Dialog JSON format:
    // {
    //   "id": "intro_001",
    //   "nodes": [
    //     {
    //       "speaker": "Elder",
    //       "text": "Welcome, traveler.",
    //       "portrait": "elder_neutral"
    //     }
    //   ]
    // }

    return this.scene.cache.json.get(`dialog_${dialogId}`);
  }

  startDialog(dialogId) {
    this.currentDialog = this.loadDialog(dialogId);
    this.currentNode = 0;
    this.isActive = true;

    this.createDialogUI();
    this.showNode(0);
  }

  createDialogUI() {
    const { width, height } = this.scene.cameras.main;

    // Dialog box background
    this.dialogBox = this.scene.add.container(0, height - 150);

    const bg = this.scene.add.rectangle(
      0, 0, width, 150, 0x000000, 0.8
    ).setOrigin(0);

    // Speaker name
    this.speakerText = this.scene.add.text(
      20, 10, '',
      { fontSize: '20px', color: '#ffff00', fontStyle: 'bold' }
    );

    // Dialog text
    this.dialogText = this.scene.add.text(
      20, 40, '',
      { fontSize: '18px', color: '#ffffff', wordWrap: { width: width - 40 } }
    );

    // Continue indicator
    this.continueIcon = this.scene.add.text(
      width - 40, 120, '▼',
      { fontSize: '24px', color: '#ffffff' }
    );

    this.scene.tweens.add({
      targets: this.continueIcon,
      alpha: 0.3,
      yoyo: true,
      repeat: -1,
      duration: 500
    });

    this.dialogBox.add([bg, this.speakerText, this.dialogText, this.continueIcon]);
    this.dialogBox.setDepth(100);
  }

  showNode(nodeIndex) {
    const node = this.currentDialog.nodes[nodeIndex];

    this.speakerText.setText(node.speaker);

    // Typewriter effect
    this.dialogText.setText('');
    let charIndex = 0;

    this.typewriterTimer = this.scene.time.addEvent({
      delay: 50,
      callback: () => {
        this.dialogText.setText(node.text.substring(0, charIndex + 1));
        charIndex++;

        if (charIndex >= node.text.length) {
          this.typewriterTimer.destroy();
          this.enableContinue();
        }
      },
      loop: true
    });

    // Show portrait if specified
    if (node.portrait) {
      // Add portrait sprite
    }
  }

  enableContinue() {
    this.scene.input.once('pointerdown', () => {
      this.nextNode();
    });
  }

  nextNode() {
    this.currentNode++;

    if (this.currentNode >= this.currentDialog.nodes.length) {
      this.endDialog();
    } else {
      this.showNode(this.currentNode);
    }
  }

  endDialog() {
    this.isActive = false;
    this.dialogBox.destroy();
    this.scene.events.emit('dialogComplete', this.currentDialog.id);
  }
}
```

#### Cutscene System

```javascript
class CutsceneManager {
  constructor(scene) {
    this.scene = scene;
    this.isPlaying = false;
    this.commands = [];
    this.currentCommand = 0;
  }

  // Load cutscene from XML/JSON
  loadCutscene(cutsceneId) {
    // Cutscene format:
    // {
    //   "commands": [
    //     { "type": "move", "actor": "hero", "x": 100, "y": 200, "duration": 1000 },
    //     { "type": "dialog", "dialogId": "intro_001" },
    //     { "type": "fadeOut", "duration": 500 }
    //   ]
    // }

    const data = this.scene.cache.json.get(`cutscene_${cutsceneId}`);
    return data.commands;
  }

  playCutscene(cutsceneId) {
    this.commands = this.loadCutscene(cutsceneId);
    this.currentCommand = 0;
    this.isPlaying = true;

    // Disable player control
    this.scene.player.inputEnabled = false;

    this.executeNextCommand();
  }

  executeNextCommand() {
    if (this.currentCommand >= this.commands.length) {
      this.endCutscene();
      return;
    }

    const command = this.commands[this.currentCommand];
    this.currentCommand++;

    switch(command.type) {
      case 'move':
        this.executeMoveCommand(command);
        break;
      case 'dialog':
        this.executeDialogCommand(command);
        break;
      case 'fadeOut':
        this.executeFadeCommand(command, 0);
        break;
      case 'fadeIn':
        this.executeFadeCommand(command, 1);
        break;
      case 'wait':
        this.executeWaitCommand(command);
        break;
      case 'playSound':
        this.executePlaySoundCommand(command);
        break;
    }
  }

  executeMoveCommand(command) {
    const actor = this.scene[command.actor];

    this.scene.tweens.add({
      targets: actor,
      x: command.x,
      y: command.y,
      duration: command.duration,
      onComplete: () => this.executeNextCommand()
    });
  }

  executeDialogCommand(command) {
    this.scene.events.once('dialogComplete', () => {
      this.executeNextCommand();
    });

    this.scene.dialogSystem.startDialog(command.dialogId);
  }

  executeFadeCommand(command, targetAlpha) {
    this.scene.cameras.main.fade(command.duration, 0, 0, 0);

    this.scene.time.delayedCall(command.duration, () => {
      this.executeNextCommand();
    });
  }

  executeWaitCommand(command) {
    this.scene.time.delayedCall(command.duration, () => {
      this.executeNextCommand();
    });
  }

  executePlaySoundCommand(command) {
    this.scene.sound.play(command.soundKey);
    this.executeNextCommand(); // Don't wait for sound to finish
  }

  endCutscene() {
    this.isPlaying = false;
    this.scene.player.inputEnabled = true;
    this.scene.events.emit('cutsceneComplete');
  }
}
```

---

## Save Systems & State Management

### LocalStorage vs IndexedDB Comparison

| Feature | LocalStorage | IndexedDB |
|---------|--------------|-----------|
| **Capacity** | ~5-10 MB | Unlimited (quota-based) |
| **API Type** | Synchronous | Asynchronous |
| **Data Types** | Strings only | Native JavaScript objects |
| **Performance** | Blocks execution | Non-blocking |
| **Querying** | Key-value only | Advanced queries, indexes |
| **Best For** | Simple preferences | Complex RPG save data |

**Recommendation:** Use **IndexedDB** for RPG save systems due to:
- Larger save files (party, inventory, quest flags, maps)
- No JSON serialization overhead
- Asynchronous = no frame drops when saving
- Support for multiple save slots easily

### IndexedDB Implementation for RPG

```javascript
class SaveSystem {
  constructor() {
    this.dbName = 'RPGSaveData';
    this.version = 1;
    this.db = null;
  }

  async initialize() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve(this.db);
      };

      request.onupgradeneeded = (event) => {
        const db = event.target.result;

        // Create object stores
        if (!db.objectStoreNames.contains('saves')) {
          const saveStore = db.createObjectStore('saves', { keyPath: 'slot' });
          saveStore.createIndex('timestamp', 'timestamp', { unique: false });
        }

        if (!db.objectStoreNames.contains('globalData')) {
          db.createObjectStore('globalData', { keyPath: 'key' });
        }
      };
    });
  }

  async saveGame(slot, gameState) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(['saves'], 'readwrite');
      const store = transaction.objectStore('saves');

      const saveData = {
        slot: slot,
        timestamp: Date.now(),
        playtime: gameState.playtime,
        location: gameState.currentMap,
        playerLevel: gameState.player.level,
        // Full game state
        state: this.serializeGameState(gameState)
      };

      const request = store.put(saveData);

      request.onsuccess = () => resolve(saveData);
      request.onerror = () => reject(request.error);
    });
  }

  async loadGame(slot) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(['saves'], 'readonly');
      const store = transaction.objectStore('saves');
      const request = store.get(slot);

      request.onsuccess = () => {
        if (request.result) {
          const gameState = this.deserializeGameState(request.result.state);
          resolve(gameState);
        } else {
          reject(new Error(`No save found in slot ${slot}`));
        }
      };

      request.onerror = () => reject(request.error);
    });
  }

  async listSaves() {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(['saves'], 'readonly');
      const store = transaction.objectStore('saves');
      const request = store.getAll();

      request.onsuccess = () => {
        // Return save metadata only
        const saves = request.result.map(save => ({
          slot: save.slot,
          timestamp: save.timestamp,
          playtime: save.playtime,
          location: save.location,
          playerLevel: save.playerLevel
        }));
        resolve(saves);
      };

      request.onerror = () => reject(request.error);
    });
  }

  async deleteSave(slot) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(['saves'], 'readwrite');
      const store = transaction.objectStore('saves');
      const request = store.delete(slot);

      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }

  serializeGameState(gameState) {
    // Custom serialization for complex objects
    return {
      player: gameState.player.serialize(),
      party: gameState.party.serialize(),
      inventory: gameState.inventory.serialize(),
      flags: Array.from(gameState.flags),
      quests: gameState.quests.serialize(),
      map: gameState.currentMap,
      position: { x: gameState.player.x, y: gameState.player.y }
    };
  }

  deserializeGameState(data) {
    // Reconstruct game state from saved data
    const gameState = new GameState();
    gameState.player = Player.deserialize(data.player);
    gameState.party = PartyManager.deserialize(data.party);
    gameState.inventory = Inventory.deserialize(data.inventory);
    gameState.flags = new Set(data.flags);
    gameState.quests = QuestManager.deserialize(data.quests);
    gameState.currentMap = data.map;
    gameState.player.setPosition(data.position.x, data.position.y);

    return gameState;
  }
}
```

### Cloud Save Implementation

#### Firebase Realtime Database

```javascript
import { initializeApp } from 'firebase/app';
import { getDatabase, ref, set, get, child } from 'firebase/database';

class FirebaseSaveSystem {
  constructor(config) {
    this.app = initializeApp(config);
    this.db = getDatabase(this.app);
  }

  async saveToCloud(userId, slot, gameState) {
    const saveRef = ref(this.db, `users/${userId}/saves/${slot}`);

    const saveData = {
      timestamp: Date.now(),
      state: gameState.serialize()
    };

    try {
      await set(saveRef, saveData);
      console.log('Save uploaded to cloud');
      return true;
    } catch (error) {
      console.error('Cloud save failed:', error);
      return false;
    }
  }

  async loadFromCloud(userId, slot) {
    const dbRef = ref(this.db);
    const snapshot = await get(child(dbRef, `users/${userId}/saves/${slot}`));

    if (snapshot.exists()) {
      return snapshot.val().state;
    } else {
      throw new Error('No cloud save found');
    }
  }

  async listCloudSaves(userId) {
    const dbRef = ref(this.db);
    const snapshot = await get(child(dbRef, `users/${userId}/saves`));

    if (snapshot.exists()) {
      return Object.entries(snapshot.val()).map(([slot, data]) => ({
        slot,
        timestamp: data.timestamp
      }));
    }

    return [];
  }
}
```

#### Supabase PostgreSQL

```javascript
import { createClient } from '@supabase/supabase-js';

class SupabaseSaveSystem {
  constructor(supabaseUrl, supabaseKey) {
    this.supabase = createClient(supabaseUrl, supabaseKey);
  }

  async saveToCloud(userId, slot, gameState) {
    const { data, error } = await this.supabase
      .from('saves')
      .upsert({
        user_id: userId,
        slot: slot,
        save_data: gameState.serialize(),
        updated_at: new Date().toISOString()
      });

    if (error) {
      console.error('Supabase save error:', error);
      return false;
    }

    return true;
  }

  async loadFromCloud(userId, slot) {
    const { data, error } = await this.supabase
      .from('saves')
      .select('save_data')
      .eq('user_id', userId)
      .eq('slot', slot)
      .single();

    if (error) {
      throw new Error('No cloud save found');
    }

    return data.save_data;
  }

  // Real-time sync
  subscribeToSaveChanges(userId, callback) {
    return this.supabase
      .channel('save-changes')
      .on(
        'postgres_changes',
        {
          event: 'UPDATE',
          schema: 'public',
          table: 'saves',
          filter: `user_id=eq.${userId}`
        },
        (payload) => callback(payload.new)
      )
      .subscribe();
  }
}
```

### State Management Patterns

#### Redux for Complex RPG Systems

```javascript
// Store structure for RPG
const initialState = {
  player: {
    stats: { hp: 100, mp: 50, level: 1 },
    equipment: {},
    position: { x: 0, y: 0 }
  },
  party: {
    active: [],
    reserve: []
  },
  inventory: {
    items: [],
    gold: 0
  },
  quests: {
    active: [],
    completed: []
  },
  flags: {},
  currentMap: 'town_start'
};

// Actions
const actions = {
  updatePlayerStats: (stats) => ({
    type: 'UPDATE_PLAYER_STATS',
    payload: stats
  }),

  addItemToInventory: (item) => ({
    type: 'ADD_ITEM',
    payload: item
  }),

  setFlag: (flagName, value) => ({
    type: 'SET_FLAG',
    payload: { flagName, value }
  })
};

// Reducer
function gameReducer(state = initialState, action) {
  switch (action.type) {
    case 'UPDATE_PLAYER_STATS':
      return {
        ...state,
        player: {
          ...state.player,
          stats: { ...state.player.stats, ...action.payload }
        }
      };

    case 'ADD_ITEM':
      return {
        ...state,
        inventory: {
          ...state.inventory,
          items: [...state.inventory.items, action.payload]
        }
      };

    case 'SET_FLAG':
      return {
        ...state,
        flags: {
          ...state.flags,
          [action.payload.flagName]: action.payload.value
        }
      };

    default:
      return state;
  }
}
```

#### MobX for Reactive State

```javascript
import { makeAutoObservable } from 'mobx';

class GameState {
  player = {
    hp: 100,
    mp: 50,
    level: 1
  };

  inventory = [];
  flags = new Map();

  constructor() {
    makeAutoObservable(this);
  }

  // Computed values
  get isPlayerAlive() {
    return this.player.hp > 0;
  }

  get inventoryWeight() {
    return this.inventory.reduce((sum, item) => sum + item.weight, 0);
  }

  // Actions
  takeDamage(amount) {
    this.player.hp = Math.max(0, this.player.hp - amount);
  }

  addItem(item) {
    this.inventory.push(item);
  }

  setFlag(name, value) {
    this.flags.set(name, value);
  }

  hasFlag(name) {
    return this.flags.get(name) === true;
  }
}

// Usage in Phaser
class GameScene extends Phaser.Scene {
  create() {
    this.gameState = new GameState();

    // React to state changes
    autorun(() => {
      if (!this.gameState.isPlayerAlive) {
        this.scene.start('GameOverScene');
      }
    });
  }
}
```

---

## Performance Optimization

### Asset Loading Strategies

#### Preloading and Lazy Loading

```javascript
class AssetManager {
  constructor(scene) {
    this.scene = scene;
    this.loadedAssets = new Set();
  }

  // Preload critical assets
  preloadCriticalAssets() {
    // UI and essential game assets
    this.scene.load.image('ui_frame', 'assets/ui/frame.png');
    this.scene.load.atlas(
      'ui_atlas',
      'assets/ui/ui_spritesheet.png',
      'assets/ui/ui_spritesheet.json'
    );

    // Player character
    this.scene.load.spritesheet(
      'player',
      'assets/characters/hero.png',
      { frameWidth: 32, frameHeight: 32 }
    );
  }

  // Lazy load level assets
  async loadLevelAssets(levelId) {
    return new Promise((resolve) => {
      // Check if already loaded
      if (this.loadedAssets.has(levelId)) {
        resolve();
        return;
      }

      const loader = new Phaser.Loader.LoaderPlugin(this.scene);

      // Load level-specific assets
      loader.tilemapTiledJSON(`map_${levelId}`, `assets/maps/${levelId}.json`);
      loader.image(`tiles_${levelId}`, `assets/tilesets/${levelId}.png`);

      loader.once('complete', () => {
        this.loadedAssets.add(levelId);
        resolve();
      });

      loader.start();
    });
  }

  // Unload unused assets to free memory
  unloadLevelAssets(levelId) {
    this.scene.textures.remove(`map_${levelId}`);
    this.scene.textures.remove(`tiles_${levelId}`);
    this.loadedAssets.delete(levelId);
  }
}
```

#### Progressive Loading with Loading Screen

```javascript
class LoadingScene extends Phaser.Scene {
  preload() {
    // Create progress bar
    const width = this.cameras.main.width;
    const height = this.cameras.main.height;

    const progressBar = this.add.graphics();
    const progressBox = this.add.graphics();
    progressBox.fillStyle(0x222222, 0.8);
    progressBox.fillRect(width / 2 - 160, height / 2 - 25, 320, 50);

    const loadingText = this.add.text(width / 2, height / 2 - 50, 'Loading...', {
      fontSize: '20px',
      color: '#ffffff'
    }).setOrigin(0.5);

    const percentText = this.add.text(width / 2, height / 2, '0%', {
      fontSize: '18px',
      color: '#ffffff'
    }).setOrigin(0.5);

    const assetText = this.add.text(width / 2, height / 2 + 50, '', {
      fontSize: '16px',
      color: '#ffffff'
    }).setOrigin(0.5);

    // Progress events
    this.load.on('progress', (value) => {
      progressBar.clear();
      progressBar.fillStyle(0xffffff, 1);
      progressBar.fillRect(width / 2 - 150, height / 2 - 15, 300 * value, 30);
      percentText.setText(Math.floor(value * 100) + '%');
    });

    this.load.on('fileprogress', (file) => {
      assetText.setText('Loading: ' + file.key);
    });

    this.load.on('complete', () => {
      progressBar.destroy();
      progressBox.destroy();
      loadingText.destroy();
      percentText.destroy();
      assetText.destroy();
    });

    // Load all game assets
    this.loadGameAssets();
  }

  loadGameAssets() {
    // Organize by priority

    // Priority 1: UI and core systems
    this.load.setPath('assets/');
    this.load.atlas('ui', 'ui/ui.png', 'ui/ui.json');

    // Priority 2: Common game assets
    this.load.spritesheet('characters', 'sprites/characters.png', {
      frameWidth: 32,
      frameHeight: 32
    });

    // Priority 3: Audio (can fail gracefully)
    this.load.audio('bgm_battle', 'audio/battle.ogg');
    this.load.audio('sfx_attack', 'audio/attack.ogg');

    // Priority 4: Large assets
    this.load.tilemapTiledJSON('worldmap', 'maps/world.json');
  }

  create() {
    this.scene.start('MainMenuScene');
  }
}
```

### Sprite Sheet Optimization

**Best Practices:**

1. **Use Power-of-2 Dimensions:** 512x512, 1024x1024, 2048x2048
2. **Texture Atlas Tools:** TexturePacker, Shoebox, Free Texture Packer
3. **Compression:** Use WebP for modern browsers, PNG fallback
4. **Trim Transparency:** Remove empty pixels around sprites

```javascript
// Example sprite sheet configuration
{
  "frames": {
    "hero_idle_01": {
      "frame": { "x": 0, "y": 0, "w": 32, "h": 32 },
      "rotated": false,
      "trimmed": true,
      "spriteSourceSize": { "x": 2, "y": 4, "w": 32, "h": 32 },
      "sourceSize": { "w": 36, "h": 36 }
    }
  },
  "meta": {
    "image": "characters.png",
    "format": "RGBA8888",
    "size": { "w": 1024, "h": 1024 },
    "scale": "1"
  }
}
```

**Performance Impact:**
- Sprite sheets reduce HTTP requests by 90%+
- Texture atlas can reduce draw calls by 69%
- Trimming reduces memory usage by 20-30%

### Memory Management for Long Sessions

#### Object Pooling

```javascript
class ObjectPool {
  constructor(scene, classType, initialSize = 10) {
    this.scene = scene;
    this.classType = classType;
    this.pool = [];

    // Pre-create objects
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createNew());
    }
  }

  createNew() {
    const obj = new this.classType(this.scene);
    obj.setActive(false);
    obj.setVisible(false);
    return obj;
  }

  spawn(x, y, config) {
    let obj = this.pool.find(p => !p.active);

    if (!obj) {
      obj = this.createNew();
      this.pool.push(obj);
    }

    obj.setActive(true);
    obj.setVisible(true);
    obj.setPosition(x, y);

    if (config) {
      Object.assign(obj, config);
    }

    return obj;
  }

  despawn(obj) {
    obj.setActive(false);
    obj.setVisible(false);
  }

  clear() {
    this.pool.forEach(obj => obj.destroy());
    this.pool = [];
  }
}

// Usage for damage numbers
class DamageNumberPool extends ObjectPool {
  constructor(scene) {
    super(scene, Phaser.GameObjects.Text, 20);
  }

  showDamage(x, y, damage) {
    const text = this.spawn(x, y, {
      text: damage.toString(),
      style: { fontSize: '24px', color: '#ff0000' }
    });

    this.scene.tweens.add({
      targets: text,
      y: y - 50,
      alpha: 0,
      duration: 1000,
      onComplete: () => this.despawn(text)
    });
  }
}
```

#### Garbage Collection Management

```javascript
class MemoryManager {
  constructor(scene) {
    this.scene = scene;
    this.memoryCheckInterval = 60000; // 1 minute
    this.setupMemoryMonitoring();
  }

  setupMemoryMonitoring() {
    // Check memory usage periodically
    this.scene.time.addEvent({
      delay: this.memoryCheckInterval,
      callback: () => this.checkMemory(),
      loop: true
    });
  }

  checkMemory() {
    if (performance.memory) {
      const usedMemory = performance.memory.usedJSHeapSize;
      const totalMemory = performance.memory.totalJSHeapSize;
      const memoryPercent = (usedMemory / totalMemory) * 100;

      console.log(`Memory usage: ${memoryPercent.toFixed(2)}%`);

      if (memoryPercent > 80) {
        this.cleanupMemory();
      }
    }
  }

  cleanupMemory() {
    console.log('Performing memory cleanup...');

    // Clear cached textures not used recently
    this.clearUnusedTextures();

    // Clear old particles
    this.clearParticles();

    // Trim audio buffers
    this.scene.sound.removeAll();
  }

  clearUnusedTextures() {
    // Remove textures not accessed in last 5 minutes
    const textureManager = this.scene.textures;
    const now = Date.now();

    textureManager.each((texture, key) => {
      if (texture.lastAccessed && (now - texture.lastAccessed) > 300000) {
        textureManager.remove(key);
      }
    });
  }

  clearParticles() {
    // Destroy completed particle emitters
    this.scene.children.list
      .filter(child => child instanceof Phaser.GameObjects.Particles.ParticleEmitterManager)
      .forEach(emitter => {
        if (!emitter.on) {
          emitter.destroy();
        }
      });
  }
}
```

**Anti-Patterns to Avoid:**
- Creating new objects in `update()` loop
- Not destroying tweens after completion
- Loading all maps at game start
- Creating new functions inside loops
- Not clearing event listeners

### Mobile Browser Compatibility

#### Responsive Design

```javascript
class ResponsiveGame {
  constructor() {
    this.config = {
      type: Phaser.AUTO,
      parent: 'game-container',
      width: 1280,
      height: 720,
      scale: {
        mode: Phaser.Scale.FIT,
        autoCenter: Phaser.Scale.CENTER_BOTH,
        orientation: Phaser.Scale.Orientation.LANDSCAPE
      },
      dom: {
        createContainer: true
      }
    };

    this.game = new Phaser.Game(this.config);
  }
}
```

#### Touch Controls

```javascript
class TouchControls {
  constructor(scene) {
    this.scene = scene;
    this.setupVirtualControls();
  }

  setupVirtualControls() {
    // Only show on mobile
    if (!this.scene.sys.game.device.os.desktop) {
      this.createDPad();
      this.createActionButtons();
    }
  }

  createDPad() {
    const x = 100;
    const y = this.scene.cameras.main.height - 100;

    // D-pad background
    const dpad = this.scene.add.circle(x, y, 60, 0x000000, 0.3);

    // Direction buttons
    const directions = [
      { angle: 0, key: 'right' },
      { angle: 90, key: 'down' },
      { angle: 180, key: 'left' },
      { angle: 270, key: 'up' }
    ];

    directions.forEach(dir => {
      const rad = Phaser.Math.DegToRad(dir.angle);
      const button = this.scene.add.circle(
        x + Math.cos(rad) * 40,
        y + Math.sin(rad) * 40,
        20,
        0xffffff,
        0.5
      );

      button.setInteractive()
        .on('pointerdown', () => this.scene.cursors[dir.key].isDown = true)
        .on('pointerup', () => this.scene.cursors[dir.key].isDown = false);
    });
  }

  createActionButtons() {
    const x = this.scene.cameras.main.width - 100;
    const y = this.scene.cameras.main.height - 100;

    // A button (confirm/attack)
    const aButton = this.scene.add.circle(x, y, 30, 0xff0000, 0.5);
    const aText = this.scene.add.text(x, y, 'A', {
      fontSize: '24px',
      color: '#ffffff'
    }).setOrigin(0.5);

    aButton.setInteractive()
      .on('pointerdown', () => this.scene.events.emit('actionA'));

    // B button (cancel/menu)
    const bButton = this.scene.add.circle(x - 60, y - 20, 30, 0x0000ff, 0.5);
    const bText = this.scene.add.text(x - 60, y - 20, 'B', {
      fontSize: '24px',
      color: '#ffffff'
    }).setOrigin(0.5);

    bButton.setInteractive()
      .on('pointerdown', () => this.scene.events.emit('actionB'));
  }
}
```

#### Mobile-Specific Optimizations

```javascript
class MobileOptimizer {
  static optimize(game) {
    const isMobile = !game.device.os.desktop;

    if (isMobile) {
      // Reduce particle counts
      game.config.physics.arcade.maxParticles = 50;

      // Lower texture quality on older devices
      if (this.isLowEndDevice()) {
        game.config.render.pixelArt = true;
        game.config.render.antiAlias = false;
      }

      // Use Canvas instead of WebGL on very old devices
      if (this.shouldUseCanvas()) {
        game.config.type = Phaser.CANVAS;
      }

      // Disable shadows and effects
      game.config.disableShadows = true;
    }
  }

  static isLowEndDevice() {
    // Check device memory (Chrome 63+)
    if (navigator.deviceMemory) {
      return navigator.deviceMemory < 4; // Less than 4GB RAM
    }

    // Fallback: check hardware concurrency
    if (navigator.hardwareConcurrency) {
      return navigator.hardwareConcurrency < 4; // Less than 4 cores
    }

    return false;
  }

  static shouldUseCanvas() {
    // Force Canvas on old Android devices
    const ua = navigator.userAgent;
    if (/Android [456]/.test(ua)) {
      return true;
    }

    return false;
  }
}
```

**Browser Compatibility Matrix (2025):**

| Browser | WebGL Support | IndexedDB | Web Audio API |
|---------|---------------|-----------|---------------|
| Chrome Mobile 90+ | ✅ Full | ✅ Full | ✅ Full |
| Safari iOS 14+ | ✅ Full | ✅ Full | ⚠️ Requires user interaction |
| Firefox Mobile 90+ | ✅ Full | ✅ Full | ✅ Full |
| Samsung Internet 14+ | ✅ Full | ✅ Full | ✅ Full |

**Key Considerations:**
- **Chrome** holds 66% mobile browser market share
- **WebGL 1.0** supported by 97% of browsers
- **Safari** requires special attention for WebGL features
- **Audio autoplay** requires user interaction on iOS

---

## Zero-Touch Development Workflow

### Procedural Asset Generation

#### Procedural Sprite Generation

```javascript
// Using HTML5 Canvas for runtime sprite generation
class ProceduralSpriteGenerator {
  constructor(width = 32, height = 32) {
    this.canvas = document.createElement('canvas');
    this.canvas.width = width;
    this.canvas.height = height;
    this.ctx = this.canvas.getContext('2d');
  }

  // Generate simple character sprite
  generateCharacter(seed, colorPalette) {
    // Seeded random for consistency
    const random = this.seededRandom(seed);

    // Clear canvas
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

    // Generate symmetric pattern
    const pattern = this.generateSymmetricPattern(random, 8, 12);

    // Render pattern with colors
    this.renderPattern(pattern, colorPalette, random);

    return this.canvas.toDataURL();
  }

  generateSymmetricPattern(random, width, height) {
    const pattern = [];
    const halfWidth = Math.floor(width / 2);

    for (let y = 0; y < height; y++) {
      const row = [];
      for (let x = 0; x < halfWidth; x++) {
        row.push(random() > 0.5 ? 1 : 0);
      }
      // Mirror for symmetry
      const fullRow = [...row, ...row.reverse()];
      pattern.push(fullRow);
    }

    return pattern;
  }

  renderPattern(pattern, colors, random) {
    const pixelSize = this.canvas.width / pattern[0].length;

    pattern.forEach((row, y) => {
      row.forEach((cell, x) => {
        if (cell) {
          const color = colors[Math.floor(random() * colors.length)];
          this.ctx.fillStyle = color;
          this.ctx.fillRect(
            x * pixelSize,
            y * pixelSize,
            pixelSize,
            pixelSize
          );
        }
      });
    });
  }

  seededRandom(seed) {
    let s = seed;
    return function() {
      s = Math.sin(s) * 10000;
      return s - Math.floor(s);
    };
  }

  // Generate tileset
  generateTileset(tileCount, tileSize = 32) {
    const tilesPerRow = Math.ceil(Math.sqrt(tileCount));
    const sheetCanvas = document.createElement('canvas');
    sheetCanvas.width = tilesPerRow * tileSize;
    sheetCanvas.height = tilesPerRow * tileSize;
    const ctx = sheetCanvas.getContext('2d');

    for (let i = 0; i < tileCount; i++) {
      const x = (i % tilesPerRow) * tileSize;
      const y = Math.floor(i / tilesPerRow) * tileSize;

      const tileData = this.generateTile(i, tileSize);
      const img = new Image();
      img.src = tileData;

      ctx.drawImage(img, x, y);
    }

    return sheetCanvas.toDataURL();
  }

  generateTile(type, size) {
    this.canvas.width = size;
    this.canvas.height = size;

    // Different tile types
    switch(type % 5) {
      case 0: // Grass
        this.ctx.fillStyle = '#2d6e2d';
        this.ctx.fillRect(0, 0, size, size);
        // Add noise
        for (let i = 0; i < size * 2; i++) {
          this.ctx.fillStyle = `rgba(${Math.random() * 50}, ${100 + Math.random() * 50}, ${Math.random() * 50}, 0.3)`;
          this.ctx.fillRect(Math.random() * size, Math.random() * size, 1, 1);
        }
        break;

      case 1: // Stone
        this.ctx.fillStyle = '#666666';
        this.ctx.fillRect(0, 0, size, size);
        break;

      case 2: // Water
        this.ctx.fillStyle = '#4444ff';
        this.ctx.fillRect(0, 0, size, size);
        break;

      // ... more types
    }

    return this.canvas.toDataURL();
  }
}

// Integration with Phaser
class GameScene extends Phaser.Scene {
  create() {
    const generator = new ProceduralSpriteGenerator(32, 32);

    // Generate and add to texture manager
    const spriteData = generator.generateCharacter(12345, ['#ff0000', '#00ff00', '#0000ff']);
    this.textures.addBase64('hero', spriteData);

    // Use the generated sprite
    const hero = this.add.sprite(100, 100, 'hero');
  }
}
```

**Libraries for Procedural Generation:**
- **perlin-noise** - Terrain generation
- **simplex-noise** - Smooth random patterns
- **Sprite-Generator** (Python) - Can be wrapped with Node.js

#### Procedural Map Generation

```javascript
class ProceduralMapGenerator {
  constructor(width, height, seed) {
    this.width = width;
    this.height = height;
    this.seed = seed;
    this.noise = new SimplexNoise(seed);
  }

  generateBiomeMap() {
    const map = [];

    for (let y = 0; y < this.height; y++) {
      const row = [];
      for (let x = 0; x < this.width; x++) {
        // Multiple octaves of noise
        const elevation = this.getElevation(x, y);
        const moisture = this.getMoisture(x, y);
        const temperature = this.getTemperature(x, y);

        const biome = this.determineBiome(elevation, moisture, temperature);
        row.push(biome);
      }
      map.push(row);
    }

    return map;
  }

  getElevation(x, y) {
    let value = 0;
    let amplitude = 1;
    let frequency = 0.01;

    // 4 octaves
    for (let i = 0; i < 4; i++) {
      value += this.noise.noise2D(x * frequency, y * frequency) * amplitude;
      amplitude *= 0.5;
      frequency *= 2;
    }

    return value;
  }

  getMoisture(x, y) {
    return this.noise.noise2D(x * 0.02, y * 0.02);
  }

  getTemperature(x, y) {
    // Temperature decreases with latitude
    const latitudeFactor = Math.abs(y - this.height / 2) / (this.height / 2);
    return 1 - latitudeFactor + this.noise.noise2D(x * 0.015, y * 0.015) * 0.3;
  }

  determineBiome(elevation, moisture, temperature) {
    // Water
    if (elevation < -0.3) return 'OCEAN';
    if (elevation < -0.1) return 'SHALLOW_WATER';

    // Land biomes based on moisture and temperature
    if (temperature < 0.2) {
      return moisture > 0 ? 'TUNDRA' : 'ICE';
    } else if (temperature < 0.4) {
      return moisture > 0.3 ? 'TAIGA' : 'GRASSLAND';
    } else if (temperature < 0.7) {
      if (moisture > 0.5) return 'FOREST';
      if (moisture > 0) return 'PLAINS';
      return 'DESERT';
    } else {
      return moisture > 0.3 ? 'JUNGLE' : 'SAVANNA';
    }
  }

  // Export to Tiled JSON format
  exportToTiledJSON(biomeToTileMap) {
    const layers = [{
      name: 'Ground',
      type: 'tilelayer',
      width: this.width,
      height: this.height,
      data: []
    }];

    const biomeMap = this.generateBiomeMap();

    biomeMap.forEach(row => {
      row.forEach(biome => {
        layers[0].data.push(biomeToTileMap[biome] || 0);
      });
    });

    return {
      width: this.width,
      height: this.height,
      tilewidth: 32,
      tileheight: 32,
      layers: layers,
      tilesets: [{
        name: 'terrain',
        tilewidth: 32,
        tileheight: 32,
        image: 'terrain.png'
      }]
    };
  }
}
```

### API-Based Asset Creation

#### AI Sprite Generation APIs

```javascript
class AIAssetGenerator {
  constructor(apiKey) {
    this.apiKey = apiKey;
  }

  // Generate sprite using Stable Diffusion API
  async generateSprite(prompt, style = 'pixel-art') {
    const response = await fetch('https://api.stability.ai/v1/generation/stable-diffusion-xl-1024-v1-0/text-to-image', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.apiKey}`
      },
      body: JSON.stringify({
        text_prompts: [{
          text: `${style}, ${prompt}, transparent background, sprite sheet`,
          weight: 1
        }],
        cfg_scale: 7,
        height: 512,
        width: 512,
        steps: 30,
        samples: 1
      })
    });

    const data = await response.json();
    return data.artifacts[0].base64;
  }

  // Generate multiple character variations
  async generateCharacterSet(characterDescription) {
    const variations = [
      'idle stance',
      'walking forward',
      'attacking',
      'hurt reaction',
      'victory pose'
    ];

    const sprites = {};

    for (const variation of variations) {
      const prompt = `${characterDescription}, ${variation}`;
      sprites[variation] = await this.generateSprite(prompt);

      // Rate limiting
      await this.sleep(1000);
    }

    return sprites;
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Alternative: Use local Stable Diffusion
class LocalSDGenerator {
  constructor(apiUrl = 'http://127.0.0.1:7860') {
    this.apiUrl = apiUrl;
  }

  async generateSprite(prompt) {
    const response = await fetch(`${this.apiUrl}/sdapi/v1/txt2img`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        prompt: prompt,
        negative_prompt: 'blurry, low quality, realistic',
        width: 512,
        height: 512,
        steps: 20,
        cfg_scale: 7
      })
    });

    const data = await response.json();
    return data.images[0]; // Base64 image
  }
}
```

#### Procedural Audio Generation

```javascript
// Using Web Audio API for sound effect generation
class ProceduralAudioGenerator {
  constructor(audioContext) {
    this.audioContext = audioContext || new AudioContext();
  }

  // Generate attack sound
  generateAttackSound() {
    const duration = 0.2;
    const buffer = this.audioContext.createBuffer(
      1,
      this.audioContext.sampleRate * duration,
      this.audioContext.sampleRate
    );

    const data = buffer.getChannelData(0);

    for (let i = 0; i < buffer.length; i++) {
      const t = i / this.audioContext.sampleRate;

      // Sharp attack with decay
      const envelope = Math.exp(-t * 10);

      // Frequency sweep
      const frequency = 200 - (t * 1000);

      data[i] = Math.sin(2 * Math.PI * frequency * t) * envelope * 0.3;
    }

    return buffer;
  }

  // Generate coin pickup sound
  generateCoinSound() {
    const duration = 0.3;
    const buffer = this.audioContext.createBuffer(
      1,
      this.audioContext.sampleRate * duration,
      this.audioContext.sampleRate
    );

    const data = buffer.getChannelData(0);

    for (let i = 0; i < buffer.length; i++) {
      const t = i / this.audioContext.sampleRate;
      const envelope = Math.exp(-t * 8);

      // Two frequency components for richer sound
      const freq1 = 800 + Math.sin(t * 50) * 200;
      const freq2 = 1600;

      data[i] = (
        Math.sin(2 * Math.PI * freq1 * t) * 0.5 +
        Math.sin(2 * Math.PI * freq2 * t) * 0.3
      ) * envelope * 0.2;
    }

    return buffer;
  }

  // Play generated sound
  playSound(buffer) {
    const source = this.audioContext.createBufferSource();
    source.buffer = buffer;
    source.connect(this.audioContext.destination);
    source.start();
  }

  // Export as WAV file
  exportAsWAV(buffer) {
    const length = buffer.length * buffer.numberOfChannels * 2;
    const arrayBuffer = new ArrayBuffer(44 + length);
    const view = new DataView(arrayBuffer);

    // WAV header
    this.writeString(view, 0, 'RIFF');
    view.setUint32(4, 36 + length, true);
    this.writeString(view, 8, 'WAVE');
    this.writeString(view, 12, 'fmt ');
    view.setUint32(16, 16, true);
    view.setUint16(20, 1, true);
    view.setUint16(22, buffer.numberOfChannels, true);
    view.setUint32(24, buffer.sampleRate, true);
    view.setUint32(28, buffer.sampleRate * 2 * buffer.numberOfChannels, true);
    view.setUint16(32, buffer.numberOfChannels * 2, true);
    view.setUint16(34, 16, true);
    this.writeString(view, 36, 'data');
    view.setUint32(40, length, true);

    // Write audio data
    const data = buffer.getChannelData(0);
    let offset = 44;
    for (let i = 0; i < data.length; i++) {
      const sample = Math.max(-1, Math.min(1, data[i]));
      view.setInt16(offset, sample < 0 ? sample * 0x8000 : sample * 0x7FFF, true);
      offset += 2;
    }

    return new Blob([arrayBuffer], { type: 'audio/wav' });
  }

  writeString(view, offset, string) {
    for (let i = 0; i < string.length; i++) {
      view.setUint8(offset + i, string.charCodeAt(i));
    }
  }
}
```

**AI Audio APIs:**
- **ElevenLabs** - Text-to-speech and sound effects
- **Soundraw** - AI music generation
- **Mubert API** - Generative music

### Build and Deployment Automation

#### GitHub Actions CI/CD Workflow

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy RPG Game

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Generate procedural assets
      run: npm run generate-assets

    - name: Run tests
      run: npm test

    - name: Build game
      run: npm run build
      env:
        NODE_ENV: production

    - name: Optimize assets
      run: |
        npm install -g imagemin-cli
        imagemin dist/assets/**/*.png --out-dir=dist/assets/optimized

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      if: github.ref == 'refs/heads/main'
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./dist

    - name: Deploy to Netlify
      uses: nwtgck/actions-netlify@v2
      with:
        publish-dir: './dist'
        production-deploy: true
      env:
        NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
        NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

#### Automated Asset Generation Script

```javascript
// scripts/generate-assets.js
const ProceduralSpriteGenerator = require('./lib/sprite-generator');
const ProceduralMapGenerator = require('./lib/map-generator');
const fs = require('fs');
const path = require('path');

async function generateAllAssets() {
  console.log('Generating procedural assets...');

  // Generate character sprites
  const spriteGen = new ProceduralSpriteGenerator(32, 32);
  const characters = [
    { id: 'hero', seed: 12345, palette: ['#ff6b6b', '#4ecdc4', '#45b7d1'] },
    { id: 'enemy1', seed: 54321, palette: ['#ff0000', '#990000', '#660000'] },
    { id: 'npc1', seed: 11111, palette: ['#feca57', '#48dbfb', '#ff9ff3'] }
  ];

  characters.forEach(char => {
    const spriteData = spriteGen.generateCharacter(char.seed, char.palette);
    const base64Data = spriteData.replace(/^data:image\/png;base64,/, '');
    fs.writeFileSync(
      path.join(__dirname, '../assets/sprites', `${char.id}.png`),
      base64Data,
      'base64'
    );
    console.log(`Generated ${char.id}.png`);
  });

  // Generate maps
  const mapGen = new ProceduralMapGenerator(50, 50, 9999);
  const mapData = mapGen.exportToTiledJSON({
    'OCEAN': 1,
    'SHALLOW_WATER': 2,
    'PLAINS': 3,
    'FOREST': 4,
    'DESERT': 5
  });

  fs.writeFileSync(
    path.join(__dirname, '../assets/maps/world.json'),
    JSON.stringify(mapData, null, 2)
  );
  console.log('Generated world.json');

  // Generate audio
  console.log('Audio generation requires Web Audio API (browser environment)');
  console.log('Skipping audio generation in build script');

  console.log('Asset generation complete!');
}

generateAllAssets().catch(console.error);
```

#### Package.json Scripts

```json
{
  "name": "web-rpg-game",
  "version": "1.0.0",
  "scripts": {
    "start": "webpack serve --mode development",
    "build": "webpack --mode production",
    "generate-assets": "node scripts/generate-assets.js",
    "test": "jest",
    "deploy": "npm run build && gh-pages -d dist",
    "prepare-release": "npm run generate-assets && npm run build && npm test"
  },
  "dependencies": {
    "phaser": "^3.80.0",
    "simplex-noise": "^4.0.0"
  },
  "devDependencies": {
    "webpack": "^5.88.0",
    "webpack-cli": "^5.1.4",
    "webpack-dev-server": "^4.15.1",
    "html-webpack-plugin": "^5.5.3",
    "copy-webpack-plugin": "^11.0.0",
    "gh-pages": "^5.0.0",
    "jest": "^29.6.0"
  }
}
```

---

## Architecture Patterns

### Overall Game Architecture

```
game/
├── src/
│   ├── main.js                  # Game entry point
│   ├── config.js                # Game configuration
│   ├── scenes/
│   │   ├── BootScene.js         # Initial loading
│   │   ├── PreloadScene.js      # Asset loading
│   │   ├── MainMenuScene.js     # Title screen
│   │   ├── WorldScene.js        # Exploration
│   │   ├── BattleScene.js       # Turn-based combat
│   │   ├── MenuScene.js         # Pause menu
│   │   └── UIScene.js           # Overlay UI
│   ├── entities/
│   │   ├── Player.js
│   │   ├── Enemy.js
│   │   └── NPC.js
│   ├── systems/
│   │   ├── BattleSystem.js
│   │   ├── DialogSystem.js
│   │   ├── QuestSystem.js
│   │   ├── SaveSystem.js
│   │   └── InventorySystem.js
│   ├── managers/
│   │   ├── PartyManager.js
│   │   ├── EventManager.js
│   │   ├── AssetManager.js
│   │   └── AudioManager.js
│   ├── utils/
│   │   ├── pathfinding.js
│   │   ├── collision.js
│   │   └── math.js
│   └── data/
│       ├── enemies.json
│       ├── items.json
│       ├── skills.json
│       └── quests.json
├── assets/
│   ├── sprites/
│   ├── tilesets/
│   ├── maps/
│   ├── audio/
│   └── ui/
├── scripts/
│   └── generate-assets.js
├── tests/
└── dist/
```

### Scene-Based Architecture

```javascript
// src/main.js
import Phaser from 'phaser';
import BootScene from './scenes/BootScene';
import PreloadScene from './scenes/PreloadScene';
import WorldScene from './scenes/WorldScene';
import BattleScene from './scenes/BattleScene';

const config = {
  type: Phaser.AUTO,
  width: 1280,
  height: 720,
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
    PreloadScene,
    WorldScene,
    BattleScene
  ]
};

const game = new Phaser.Game(config);
```

### Entity Component System (ECS) Pattern

```javascript
// Component-based entity system
class Component {
  constructor() {
    this.enabled = true;
  }

  update(delta) {}
  destroy() {}
}

class HealthComponent extends Component {
  constructor(maxHp) {
    super();
    this.maxHp = maxHp;
    this.currentHp = maxHp;
  }

  takeDamage(amount) {
    this.currentHp = Math.max(0, this.currentHp - amount);
    return this.currentHp <= 0; // Returns true if dead
  }

  heal(amount) {
    this.currentHp = Math.min(this.maxHp, this.currentHp + amount);
  }
}

class StatsComponent extends Component {
  constructor(stats) {
    super();
    this.attack = stats.attack || 10;
    this.defense = stats.defense || 5;
    this.speed = stats.speed || 10;
    this.magic = stats.magic || 5;
  }
}

class Entity {
  constructor(id) {
    this.id = id;
    this.components = new Map();
  }

  addComponent(name, component) {
    this.components.set(name, component);
    return this;
  }

  getComponent(name) {
    return this.components.get(name);
  }

  hasComponent(name) {
    return this.components.has(name);
  }

  update(delta) {
    this.components.forEach(component => {
      if (component.enabled) {
        component.update(delta);
      }
    });
  }

  destroy() {
    this.components.forEach(component => component.destroy());
    this.components.clear();
  }
}

// Usage
const hero = new Entity('hero')
  .addComponent('health', new HealthComponent(100))
  .addComponent('stats', new StatsComponent({ attack: 15, defense: 10 }));
```

### Event-Driven Architecture

```javascript
class EventBus {
  constructor() {
    this.events = new Map();
  }

  on(eventName, callback, context) {
    if (!this.events.has(eventName)) {
      this.events.set(eventName, []);
    }

    this.events.get(eventName).push({ callback, context });
  }

  emit(eventName, ...args) {
    if (!this.events.has(eventName)) return;

    this.events.get(eventName).forEach(({ callback, context }) => {
      callback.apply(context, args);
    });
  }

  off(eventName, callback) {
    if (!this.events.has(eventName)) return;

    const listeners = this.events.get(eventName);
    const index = listeners.findIndex(l => l.callback === callback);

    if (index !== -1) {
      listeners.splice(index, 1);
    }
  }
}

// Global event bus
window.GameEvents = new EventBus();

// Usage across systems
GameEvents.emit('player:levelUp', { newLevel: 5 });
GameEvents.on('enemy:defeated', (enemy) => {
  console.log(`Defeated ${enemy.name}`);
}, this);
```

---

## Step-by-Step Implementation Guide

### Phase 1: Project Setup (Week 1)

#### 1.1 Initialize Project

```bash
# Create project
mkdir web-rpg-game
cd web-rpg-game

# Initialize npm
npm init -y

# Install core dependencies
npm install phaser

# Install dev dependencies
npm install --save-dev webpack webpack-cli webpack-dev-server
npm install --save-dev html-webpack-plugin copy-webpack-plugin
npm install --save-dev @babel/core @babel/preset-env babel-loader

# Create directory structure
mkdir -p src/{scenes,entities,systems,managers,utils,data}
mkdir -p assets/{sprites,tilesets,maps,audio,ui}
mkdir -p scripts tests dist
```

#### 1.2 Configure Webpack

```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.[contenthash].js',
    clean: true
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: 'src/index.html',
      inject: 'body'
    }),
    new CopyWebpackPlugin({
      patterns: [
        { from: 'assets', to: 'assets' }
      ]
    })
  ],
  devServer: {
    static: './dist',
    hot: true,
    port: 8080
  }
};
```

#### 1.3 Create Basic Game Structure

```javascript
// src/main.js
import Phaser from 'phaser';
import BootScene from './scenes/BootScene';

const config = {
  type: Phaser.AUTO,
  width: 1280,
  height: 720,
  pixelArt: true,
  backgroundColor: '#000000',
  parent: 'game-container',
  physics: {
    default: 'arcade',
    arcade: {
      gravity: { y: 0 },
      debug: true
    }
  },
  scene: [BootScene]
};

const game = new Phaser.Game(config);

// Expose globally for debugging
window.game = game;
```

### Phase 2: Core Systems (Weeks 2-3)

#### 2.1 Asset Loading System

```javascript
// src/scenes/PreloadScene.js
export default class PreloadScene extends Phaser.Scene {
  constructor() {
    super({ key: 'PreloadScene' });
  }

  preload() {
    // Show loading bar
    this.createLoadingBar();

    // Load assets
    this.load.image('tiles', 'assets/tilesets/terrain.png');
    this.load.tilemapTiledJSON('map1', 'assets/maps/town.json');
    this.load.spritesheet('hero', 'assets/sprites/hero.png', {
      frameWidth: 32,
      frameHeight: 32
    });
  }

  create() {
    this.scene.start('WorldScene');
  }

  createLoadingBar() {
    const width = this.cameras.main.width;
    const height = this.cameras.main.height;

    const progressBar = this.add.graphics();
    const progressBox = this.add.graphics();
    progressBox.fillStyle(0x222222, 0.8);
    progressBox.fillRect(width / 2 - 160, height / 2 - 25, 320, 50);

    this.load.on('progress', (value) => {
      progressBar.clear();
      progressBar.fillStyle(0xffffff, 1);
      progressBar.fillRect(width / 2 - 150, height / 2 - 15, 300 * value, 30);
    });
  }
}
```

#### 2.2 Player Movement System

```javascript
// src/entities/Player.js
export default class Player extends Phaser.Physics.Arcade.Sprite {
  constructor(scene, x, y) {
    super(scene, x, y, 'hero');

    scene.add.existing(this);
    scene.physics.add.existing(this);

    this.setCollideWorldBounds(true);
    this.speed = 160;

    this.createAnimations();
  }

  createAnimations() {
    // Down
    this.scene.anims.create({
      key: 'walk-down',
      frames: this.anims.generateFrameNumbers('hero', { start: 0, end: 3 }),
      frameRate: 10,
      repeat: -1
    });

    // Up
    this.scene.anims.create({
      key: 'walk-up',
      frames: this.anims.generateFrameNumbers('hero', { start: 4, end: 7 }),
      frameRate: 10,
      repeat: -1
    });

    // Left/Right similar
  }

  update(cursors) {
    this.body.setVelocity(0);

    if (cursors.left.isDown) {
      this.body.setVelocityX(-this.speed);
      this.play('walk-left', true);
    } else if (cursors.right.isDown) {
      this.body.setVelocityX(this.speed);
      this.play('walk-right', true);
    } else if (cursors.up.isDown) {
      this.body.setVelocityY(-this.speed);
      this.play('walk-up', true);
    } else if (cursors.down.isDown) {
      this.body.setVelocityY(this.speed);
      this.play('walk-down', true);
    } else {
      this.anims.stop();
    }
  }
}
```

### Phase 3: Battle System (Weeks 4-5)

#### 3.1 Battle Scene Setup

```javascript
// src/scenes/BattleScene.js
export default class BattleScene extends Phaser.Scene {
  constructor() {
    super({ key: 'BattleScene' });
  }

  init(data) {
    this.enemies = data.enemies || ['slime', 'goblin'];
  }

  create() {
    // Background
    this.add.rectangle(0, 0, 1280, 720, 0x2d4a2e).setOrigin(0);

    // Initialize systems
    this.battleSystem = new BattleSystem(this);
    this.uiManager = new BattleUIManager(this);

    // Setup party
    this.setupParty();
    this.setupEnemies();

    // Start battle
    this.battleSystem.startBattle();
  }

  setupParty() {
    const party = this.registry.get('party');
    this.battleSystem.setParty(party);
  }

  setupEnemies() {
    this.battleSystem.setEnemies(this.enemies);
  }
}
```

### Phase 4: Save System (Week 6)

#### 4.1 Implement Save/Load

```javascript
// Integrate save system
class GameScene extends Phaser.Scene {
  create() {
    // Initialize save system
    this.saveSystem = new SaveSystem();
    this.saveSystem.initialize();

    // Auto-save every 5 minutes
    this.time.addEvent({
      delay: 300000,
      callback: () => this.autoSave(),
      loop: true
    });
  }

  async autoSave() {
    const gameState = this.collectGameState();
    await this.saveSystem.saveGame(0, gameState); // Auto-save slot
    console.log('Auto-saved');
  }

  collectGameState() {
    return {
      player: this.player.serialize(),
      party: this.partyManager.serialize(),
      inventory: this.inventoryManager.serialize(),
      currentMap: this.scene.key,
      playtime: this.registry.get('playtime') || 0,
      flags: this.registry.get('flags') || {}
    };
  }
}
```

### Phase 5: Polish & Optimization (Week 7)

#### 5.1 Performance Optimization

- Implement object pooling for particles
- Add sprite sheet optimization
- Configure proper garbage collection
- Test on mobile devices

#### 5.2 Testing

```javascript
// tests/battle-system.test.js
import BattleSystem from '../src/systems/BattleSystem';

describe('BattleSystem', () => {
  let battleSystem;

  beforeEach(() => {
    battleSystem = new BattleSystem();
  });

  test('should calculate damage correctly', () => {
    const attacker = { stats: { attack: 20 } };
    const defender = { stats: { defense: 10 } };

    const damage = battleSystem.calculateDamage(attacker, defender);
    expect(damage).toBeGreaterThan(0);
    expect(damage).toBeLessThanOrEqual(20);
  });
});
```

### Phase 6: Deployment (Week 8)

#### 6.1 Build for Production

```bash
# Build
npm run build

# Test production build locally
npx http-server dist

# Deploy
npm run deploy
```

---

## Resources & Tools

### Essential Libraries

- **Phaser 3** - Game framework
- **EasyStar.js** - A* pathfinding
- **SimplexNoise** - Procedural generation
- **Redux/MobX** - State management
- **Howler.js** - Audio management (alternative to Web Audio API)

### Development Tools

- **Tiled** - Map editor
- **TexturePacker** - Sprite sheet optimization
- **Visual Studio Code** - IDE
- **Chrome DevTools** - Debugging

### Asset Generation

- **Stable Diffusion** - AI sprite generation
- **ElevenLabs** - AI sound effects
- **Procedural Tileset Generator** - Code-based tilesets

### Testing & Deployment

- **Jest** - Unit testing
- **Puppeteer** - E2E testing
- **GitHub Actions** - CI/CD
- **Netlify/Vercel** - Hosting

---

## Conclusion

This guide provides a complete roadmap for building web-based RPG games with zero-touch development workflows. Key takeaways:

1. **Phaser 3** is the recommended framework for its maturity and features
2. **IndexedDB** is essential for complex save systems
3. **State machines** provide clean architecture for battle systems
4. **Procedural generation** enables zero-touch asset creation
5. **GitHub Actions** automates the entire build/deploy pipeline

By following these patterns and techniques, you can build scalable, performant RPG games that run in any modern browser without requiring manual asset creation or deployment processes.

---

**Document Version:** 1.0
**Last Updated:** November 2025
**Research Sources:** 20+ web searches across game development, frameworks, and tools
