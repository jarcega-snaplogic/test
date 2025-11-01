# Minimal Infrastructure Guide for Agile Game Development

**Philosophy: Build Games, Not Frameworks**

Start simple. Add complexity only when pain demands it. Ship features, not architecture.

---

## Table of Contents

1. [Core Philosophy](#core-philosophy)
2. [Minimal Data Loading System](#minimal-data-loading-system)
3. [Convention-Based File Structure](#convention-based-file-structure)
4. [Essential Patterns](#essential-patterns)
5. [Expand-As-You-Go Strategy](#expand-as-you-go-strategy)
6. [Development Workflow](#development-workflow)
7. [When to Add Infrastructure](#when-to-add-infrastructure)
8. [Anti-Patterns (YAGNI Violations)](#anti-patterns-yagni-violations)
9. [Migration Path](#migration-path)

---

## Core Philosophy

### Start Simple, Expand as Needed

```
Week 1-2:  Simple JSON loader + Manual testing
           ↓
Week 3-4:  Add validation when typos hurt productivity
           ↓
Month 2:   Add hot-reload when F5 gets annoying
           ↓
Phase 2:   Add asset manager when non-programmers join
```

### Guiding Principles

1. **Convention over Configuration** - Agree on structure, skip the metadata
2. **Rule of Three** - Wait until you have 3 examples before abstracting
3. **YAGNI** - You Aren't Gonna Need It (until you do)
4. **Pain-Driven Development** - Add tooling when manual work hurts
5. **Ship Features** - Users don't see your architecture

---

## Minimal Data Loading System

### Complete Working Implementation (~100 lines)

```javascript
/**
 * ContentLoader - Minimal JSON loader with graceful fallbacks
 *
 * Features:
 * - Async loading with fetch
 * - Basic error handling (log and continue)
 * - Simple caching
 * - No validation (add later when needed)
 *
 * Usage:
 *   const loader = new ContentLoader();
 *   const chars = await loader.load('characters/heroes');
 *   const char = await loader.loadOne('characters/heroes/warrior');
 */

class ContentLoader {
  constructor(basePath = '/data') {
    this.basePath = basePath;
    this.cache = new Map();
    this.loadPromises = new Map(); // Prevent duplicate loads
  }

  /**
   * Load a single JSON file
   * @param {string} path - Path relative to basePath (without .json)
   * @returns {Promise<Object|null>}
   */
  async loadOne(path) {
    // Check cache first
    const cacheKey = path;
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }

    // Check if already loading
    if (this.loadPromises.has(cacheKey)) {
      return this.loadPromises.get(cacheKey);
    }

    // Start loading
    const loadPromise = this._fetchJson(path);
    this.loadPromises.set(cacheKey, loadPromise);

    try {
      const data = await loadPromise;
      this.cache.set(cacheKey, data);
      return data;
    } catch (error) {
      console.error(`Failed to load ${path}:`, error);
      return null; // Graceful fallback
    } finally {
      this.loadPromises.delete(cacheKey);
    }
  }

  /**
   * Load all JSON files in a directory
   * @param {string} dirPath - Directory path relative to basePath
   * @returns {Promise<Object>} Map of filename (without .json) to data
   */
  async load(dirPath) {
    const cacheKey = `dir:${dirPath}`;
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }

    try {
      // Load index file that lists all files in directory
      // Alternative: Use manifest.json or hardcode file list
      const indexPath = `${dirPath}/index.json`;
      const index = await this._fetchJson(indexPath);

      if (!index || !index.files) {
        console.warn(`No index found at ${indexPath}, trying manifest`);
        return await this._loadFromManifest(dirPath);
      }

      // Load all files in parallel
      const results = await Promise.allSettled(
        index.files.map(file => this.loadOne(`${dirPath}/${file}`))
      );

      // Build map of results (skip failures)
      const data = {};
      results.forEach((result, idx) => {
        if (result.status === 'fulfilled' && result.value) {
          const filename = index.files[idx].replace('.json', '');
          data[filename] = result.value;
        }
      });

      this.cache.set(cacheKey, data);
      return data;

    } catch (error) {
      console.error(`Failed to load directory ${dirPath}:`, error);
      return {}; // Return empty object on failure
    }
  }

  /**
   * Load using manifest.json (fallback method)
   */
  async _loadFromManifest(dirPath) {
    try {
      const manifest = await this._fetchJson('manifest.json');
      const files = manifest[dirPath] || [];

      const results = await Promise.allSettled(
        files.map(file => this.loadOne(`${dirPath}/${file}`))
      );

      const data = {};
      results.forEach((result, idx) => {
        if (result.status === 'fulfilled' && result.value) {
          const filename = files[idx].replace('.json', '');
          data[filename] = result.value;
        }
      });

      return data;
    } catch (error) {
      console.error(`Manifest fallback failed for ${dirPath}:`, error);
      return {};
    }
  }

  /**
   * Internal fetch wrapper
   */
  async _fetchJson(path) {
    const url = `${this.basePath}/${path}.json`;
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return response.json();
  }

  /**
   * Clear cache (useful for development)
   */
  clearCache() {
    this.cache.clear();
    console.log('Cache cleared');
  }

  /**
   * Preload content (optional optimization)
   */
  async preload(paths) {
    const promises = paths.map(path => this.loadOne(path));
    await Promise.allSettled(promises);
  }
}

// Export singleton instance
const contentLoader = new ContentLoader();
export default contentLoader;
```

### Usage Example

```javascript
// main.js - Game initialization

import contentLoader from './ContentLoader.js';

async function initGame() {
  console.log('Loading game content...');

  // Load all heroes
  const heroes = await contentLoader.load('characters/heroes');
  console.log('Loaded heroes:', Object.keys(heroes));

  // Load specific character
  const warrior = await contentLoader.loadOne('characters/heroes/warrior');
  console.log('Warrior stats:', warrior);

  // If load fails, we get null - handle gracefully
  if (!warrior) {
    console.warn('Using default warrior stats');
    // Use hardcoded defaults
  }

  // Start game with loaded content
  startGame(heroes);
}

initGame().catch(error => {
  console.error('Failed to initialize game:', error);
  // Show user-friendly error message
  document.body.innerHTML = '<h1>Failed to load game. Please refresh.</h1>';
});
```

---

## Convention-Based File Structure

### Directory Layout

```
/data
  /characters
    /heroes
      index.json          # Lists: ["warrior.json", "mage.json", ...]
      warrior.json
      mage.json
      rogue.json
    /enemies
      index.json
      goblin.json
      orc.json
  /items
    /weapons
      index.json
      sword.json
      staff.json
    /armor
      index.json
      leather.json
      plate.json
  /quests
    index.json
    tutorial.json
    main-quest-1.json
  manifest.json           # Top-level manifest (optional)
```

### Convention Rules

1. **Naming**: `kebab-case` for files, `camelCase` for properties
2. **IDs**: Use filename as ID (e.g., `warrior.json` → id: "warrior")
3. **References**: Use string IDs (e.g., `"weaponId": "iron-sword"`)
4. **Index Files**: List all files in directory for batch loading
5. **No Deep Nesting**: Max 2-3 levels (easier to navigate)

### Example: Character Data

```json
// /data/characters/heroes/warrior.json
{
  "id": "warrior",
  "name": "Warrior",
  "description": "A brave melee fighter",
  "stats": {
    "hp": 120,
    "strength": 18,
    "dexterity": 12,
    "intelligence": 8,
    "defense": 15
  },
  "startingEquipment": [
    "iron-sword",
    "leather-armor"
  ],
  "abilities": [
    "power-strike",
    "shield-bash"
  ],
  "sprite": "warrior.png"
}
```

```json
// /data/characters/heroes/index.json
{
  "files": [
    "warrior",
    "mage",
    "rogue",
    "cleric"
  ]
}
```

### Why This Works

- **No schema validation needed** - Just load and use
- **Easy to add new content** - Copy file, edit, add to index
- **Git-friendly** - One file per entity, easy diffs
- **Designer-friendly** - JSON is readable, editable
- **ID-based refs** - No complex foreign keys, just strings
- **Validate later** - Add validation when typos become painful

---

## Essential Patterns

### Pattern 1: ContentManager Base Class

Simple base class for managing content of a specific type.

```javascript
/**
 * Base class for content managers
 * Extend this for each content type
 */
class ContentManager {
  constructor(contentPath) {
    this.contentPath = contentPath;
    this.items = new Map();
    this.loaded = false;
  }

  async load() {
    if (this.loaded) return;

    const data = await contentLoader.load(this.contentPath);

    // Store in Map for fast lookup
    Object.entries(data).forEach(([id, itemData]) => {
      this.items.set(id, itemData);
    });

    this.loaded = true;
    console.log(`Loaded ${this.items.size} items from ${this.contentPath}`);
  }

  get(id) {
    const item = this.items.get(id);
    if (!item) {
      console.warn(`Item not found: ${id}`);
    }
    return item;
  }

  getAll() {
    return Array.from(this.items.values());
  }

  has(id) {
    return this.items.has(id);
  }

  // Override in subclass for validation
  validate(itemData) {
    return true; // No validation yet
  }
}

// Example: Character Manager
class CharacterManager extends ContentManager {
  constructor() {
    super('characters/heroes');
  }

  // Add convenience methods
  getByClass(className) {
    return this.getAll().filter(char => char.class === className);
  }

  getStartingCharacters() {
    return this.getAll().filter(char => char.startingCharacter);
  }

  // Add validation when needed (later)
  validate(char) {
    if (!char.stats || !char.stats.hp) {
      console.error('Character missing HP:', char.id);
      return false;
    }
    return true;
  }
}

// Global managers
export const characterManager = new CharacterManager();
```

### Pattern 2: Simple Factory Pattern

Create game entities from content data.

```javascript
/**
 * CharacterFactory - Create character instances from data
 */
class CharacterFactory {
  static create(characterId, overrides = {}) {
    const data = characterManager.get(characterId);

    if (!data) {
      console.error(`Cannot create character: ${characterId}`);
      return null;
    }

    // Create character instance with data
    return new Character({
      ...data,
      ...overrides,
      // Add runtime state
      currentHp: data.stats.hp,
      position: { x: 0, y: 0 },
      status: 'idle'
    });
  }

  static createFromTemplate(templateId, customizations) {
    const template = characterManager.get(templateId);

    // Merge template with customizations
    const data = {
      ...template,
      name: customizations.name || template.name,
      stats: { ...template.stats, ...customizations.stats }
    };

    return this.create(data.id, data);
  }
}

// Usage
const player = CharacterFactory.create('warrior', {
  name: 'Thorin',
  position: { x: 100, y: 100 }
});
```

### Pattern 3: Event Bus for Loose Coupling

Simple pub/sub for decoupled communication.

```javascript
/**
 * EventBus - Minimal pub/sub system
 */
class EventBus {
  constructor() {
    this.listeners = new Map();
  }

  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);

    // Return unsubscribe function
    return () => this.off(event, callback);
  }

  off(event, callback) {
    if (!this.listeners.has(event)) return;

    const callbacks = this.listeners.get(event);
    const index = callbacks.indexOf(callback);
    if (index > -1) {
      callbacks.splice(index, 1);
    }
  }

  emit(event, data) {
    if (!this.listeners.has(event)) return;

    this.listeners.get(event).forEach(callback => {
      try {
        callback(data);
      } catch (error) {
        console.error(`Error in ${event} listener:`, error);
      }
    });
  }

  once(event, callback) {
    const unsubscribe = this.on(event, (data) => {
      unsubscribe();
      callback(data);
    });
    return unsubscribe;
  }
}

// Global event bus
export const eventBus = new EventBus();

// Usage example
eventBus.on('character:damaged', ({ character, damage }) => {
  console.log(`${character.name} took ${damage} damage!`);
  // Update UI, play sound, etc.
});

eventBus.emit('character:damaged', { character: player, damage: 15 });
```

---

## Expand-As-You-Go Strategy

### Start with One Content Type

**Week 1: Characters Only**

```javascript
// Simple, hardcoded approach
const heroes = await contentLoader.load('characters/heroes');
const warrior = heroes.warrior;

// Use data directly
player.stats = warrior.stats;
player.sprite = warrior.sprite;
```

**Don't build:**
- Complex validation system
- Generic content manager
- Asset preloading
- Hot reload

**Do build:**
- Simple loader
- Manual testing
- Console logs for debugging

### Copy Pattern for New Types

**Week 2: Add Items**

Copy the character loading pattern:

```javascript
// If this works for characters...
const heroes = await contentLoader.load('characters/heroes');

// ...just copy it for items
const weapons = await contentLoader.load('items/weapons');
const armor = await contentLoader.load('items/armor');

// Same pattern, no abstraction yet
```

### Rule of Three: Refactor After 3 Examples

**Week 3: Three Content Types? Time to Refactor**

```javascript
// NOW create ContentManager base class
class ContentManager { /* ... */ }

// Extend for each type
class CharacterManager extends ContentManager { /* ... */ }
class ItemManager extends ContentManager { /* ... */ }
class QuestManager extends ContentManager { /* ... */ }
```

**Why wait?**
- You now understand the common pattern
- You know what actually varies between types
- You avoid premature abstraction

### Add Validation When It Hurts

**Signals you need validation:**
- Lost 30 minutes to a typo in JSON
- Loaded invalid data caused runtime crash
- Designer asked "how do I know if my JSON is valid?"

**Before:**
```javascript
// Just load and hope
const warrior = await contentLoader.loadOne('characters/heroes/warrior');
player.stats = warrior.stats; // Hope stats exists!
```

**After (minimal validation):**
```javascript
function validateCharacter(data) {
  const required = ['id', 'name', 'stats'];
  const missing = required.filter(field => !data[field]);

  if (missing.length > 0) {
    console.error(`Character missing fields: ${missing.join(', ')}`);
    return false;
  }

  if (!data.stats.hp || data.stats.hp <= 0) {
    console.error('Character has invalid HP');
    return false;
  }

  return true;
}

// Use it
const warrior = await contentLoader.loadOne('characters/heroes/warrior');
if (!validateCharacter(warrior)) {
  // Use default or show error
}
```

**When to add full schema validation?**
- You have 10+ content types
- Multiple people editing content
- You've had 3+ bugs from invalid data
- See: [VALIDATION-SYSTEM-IMPLEMENTATION.md](./VALIDATION-SYSTEM-IMPLEMENTATION.md)

---

## Development Workflow

### Day-to-Day Process (Week 1-2)

```
1. Edit JSON file in /data
   ↓
2. Save file
   ↓
3. Refresh browser (F5)
   ↓
4. Check console for errors
   ↓
5. Test in game
```

**Tools:**
- Text editor with JSON syntax highlighting
- Browser DevTools Console
- Your eyes

**No need for:**
- Hot reload
- Auto-validation
- Build process
- Asset pipeline

### Debugging with Console Logs

**Liberally add logs during development:**

```javascript
async load(dirPath) {
  console.log(`📂 Loading directory: ${dirPath}`);

  const data = await contentLoader.load(dirPath);
  console.log(`✅ Loaded ${Object.keys(data).length} items:`, Object.keys(data));

  Object.entries(data).forEach(([id, itemData]) => {
    console.log(`  - ${id}:`, itemData);
    this.items.set(id, itemData);
  });

  console.log(`📊 Manager now has ${this.items.size} items`);
}
```

**Remove or disable logs in production:**

```javascript
const DEBUG = true; // Set to false for production

function log(...args) {
  if (DEBUG) console.log(...args);
}
```

### Manual Testing Checklist

Create a simple checklist for testing:

```markdown
## Content Testing Checklist

- [ ] Load game, check console for errors
- [ ] Verify all characters loaded (check count)
- [ ] Test creating character from each template
- [ ] Verify stats are correct
- [ ] Test invalid ID (should log warning)
- [ ] Test missing file (should gracefully fail)
```

Run through this after making changes.

**When to automate?**
- After you've run manual tests 10+ times
- When you have multiple content types
- When manual testing takes >5 minutes

---

## When to Add Infrastructure

### Decision Tree

```
Is manual process painful?
├─ No → Keep it manual
└─ Yes → Will automation save time?
    ├─ No → Keep it manual
    └─ Yes → Build minimal automation
```

### Specific Triggers

#### Add Hot Reload When:
- [ ] Refreshing browser feels slow (3+ seconds)
- [ ] You refresh 20+ times per hour
- [ ] Team asks "can we have hot reload?"
- [ ] You're in Week 3-4 of development

**Time investment:** 2-4 hours
**Time saved:** 5-10 minutes per hour
**ROI:** Break even after ~20-40 hours of development
**See:** [HOT-RELOAD-IMPLEMENTATION.md](./HOT-RELOAD-IMPLEMENTATION.md)

#### Add Asset Manager When:
- [ ] Non-programmer needs to add content
- [ ] Managing 50+ image/sound files manually
- [ ] File paths are hardcoded everywhere
- [ ] You're in Phase 2 (content expansion)

**Time investment:** 4-8 hours
**Time saved:** Enables non-technical contributors
**ROI:** High if you have content creators
**See:** [ASSET-MANAGER-SPECIFICATION.md](./ASSET-MANAGER-SPECIFICATION.md)

#### Add Schema Validation When:
- [ ] Lost 2+ hours to typos/invalid data
- [ ] Have 5+ content types
- [ ] Multiple people editing content
- [ ] Designer asks "how do I validate this?"

**Time investment:** 3-6 hours
**Time saved:** Prevents bugs, faster debugging
**ROI:** High with multiple content editors
**See:** [VALIDATION-SYSTEM-IMPLEMENTATION.md](./VALIDATION-SYSTEM-IMPLEMENTATION.md)

#### Add Build Process When:
- [ ] Need to minify/optimize for production
- [ ] Using TypeScript or other transpilation
- [ ] Bundle size matters (mobile/slow connections)
- [ ] Ready to deploy

**Time investment:** 4-8 hours
**Time saved:** Better production performance
**ROI:** Essential for production, skip in development

### Complexity Budget

Track complexity as you add features:

```
Week 1-2:   ██░░░░░░░░  (2/10) Minimal loader
Week 3-4:   ████░░░░░░  (4/10) + Validation
Month 2:    ██████░░░░  (6/10) + Hot reload
Phase 2:    ████████░░  (8/10) + Asset manager
Production: ██████████  (10/10) + Full system
```

**Rule:** Don't exceed 4/10 complexity in first month.

---

## Anti-Patterns (YAGNI Violations)

### Don't Build These Too Early

#### ❌ Complex Schema System (Week 1)

**Too early:**
```javascript
// 300 lines of JSON Schema validation
// Custom validation DSL
// Automatic schema generation
// Schema versioning system
```

**Right-sized:**
```javascript
// Simple required field check
function validate(data) {
  return data.id && data.name && data.stats;
}
```

#### ❌ Generic Content Pipeline (Week 1)

**Too early:**
```javascript
// Abstract factory factories
// Plugin-based loader system
// Configurable content transformers
// Dependency injection container
```

**Right-sized:**
```javascript
// One ContentLoader class
// Extend when you need specific behavior
class ContentLoader { /* 100 lines */ }
```

#### ❌ Advanced Caching Strategy (Week 1)

**Too early:**
```javascript
// LRU cache with size limits
// Cache invalidation strategies
// Persistent cache with IndexedDB
// Cache warming system
```

**Right-sized:**
```javascript
// Simple Map-based cache
this.cache = new Map();
```

#### ❌ Premature Optimization

**Too early:**
```javascript
// Object pooling for all entities
// Custom serialization format
// Web Workers for loading
// Aggressive code splitting
```

**Right-sized:**
```javascript
// Load everything on startup
// Use JSON.parse
// Single thread is fine
// One bundle is fine
```

### Warning Signs

You're over-engineering if:

- [ ] You're building "a system" not "a feature"
- [ ] Your abstraction has no concrete use cases yet
- [ ] You're solving hypothetical future problems
- [ ] Your code has more comments than logic
- [ ] You've spent more time on tools than game features
- [ ] You can't remember why you're building this
- [ ] You're building for "flexibility" without specific needs

### The YAGNI Mantra

```
"I might need this later"  → Probably YAGNI
"This hurts right now"     → Build it
"We'll need this in Phase 2" → Wait until Phase 2
"This will save us time"   → Measure the savings
```

---

## Migration Path

### From Minimal to Full Infrastructure

#### Phase 1: Minimal (Week 1-4)
**What you have:**
- ContentLoader class (~100 lines)
- Manual browser refresh
- Console.log debugging
- Simple file structure

**Good for:**
- 1-2 developers
- <10 content types
- Early prototyping
- Rapid iteration

#### Phase 2: Enhanced (Month 2-3)
**Add when painful:**
- Basic validation (required fields)
- Hot reload for JSON files
- ContentManager base class
- Simple error reporting

**Good for:**
- 2-3 developers
- 10-20 content types
- Playtesting phase
- Content iteration

**Migration:**
```javascript
// Add to existing ContentLoader
async loadOne(path) {
  const data = await this._fetchJson(path);

  // Add validation
  if (this.validator) {
    const errors = this.validator.validate(data);
    if (errors.length > 0) {
      console.error('Validation errors:', errors);
    }
  }

  return data;
}
```

#### Phase 3: Production-Ready (Month 4+)
**Add for production:**
- Full schema validation
- Asset manager with optimization
- Build process for minification
- Error tracking
- Performance monitoring

**Good for:**
- Team of 3+
- 20+ content types
- Production deployment
- Multiple contributors

**Migration:** See individual guides
- [VALIDATION-SYSTEM-IMPLEMENTATION.md](./VALIDATION-SYSTEM-IMPLEMENTATION.md)
- [ASSET-MANAGER-SPECIFICATION.md](./ASSET-MANAGER-SPECIFICATION.md)
- [HOT-RELOAD-IMPLEMENTATION.md](./HOT-RELOAD-IMPLEMENTATION.md)

### Backward Compatibility

When upgrading, keep things working:

```javascript
// Old code still works
const heroes = await contentLoader.load('characters/heroes');

// New code uses managers
await characterManager.load();
const hero = characterManager.get('warrior');

// Both can coexist during migration
```

### Incremental Migration Strategy

1. **Keep old system running**
2. **Add new system alongside**
3. **Migrate one content type at a time**
4. **Remove old system when migration complete**

```javascript
// Week 1: Both systems
const heroesOld = await contentLoader.load('characters/heroes');
await characterManager.load();

// Week 2: Migrate characters to new system
// const heroesOld = await contentLoader.load('characters/heroes'); // Remove
const hero = characterManager.get('warrior'); // Use this

// Week 3: Old system still available for items
const items = await contentLoader.load('items/weapons');

// Week 4: Migrate items
await itemManager.load();
```

---

## Complete Minimal Example

### Full Working Implementation

Here's everything you need to get started:

#### File: `ContentLoader.js`

```javascript
/**
 * Minimal ContentLoader - Complete implementation
 */
class ContentLoader {
  constructor(basePath = '/data') {
    this.basePath = basePath;
    this.cache = new Map();
  }

  async loadOne(path) {
    if (this.cache.has(path)) {
      return this.cache.get(path);
    }

    try {
      const url = `${this.basePath}/${path}.json`;
      const response = await fetch(url);

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      const data = await response.json();
      this.cache.set(path, data);
      return data;
    } catch (error) {
      console.error(`Failed to load ${path}:`, error);
      return null;
    }
  }

  async load(dirPath) {
    try {
      const index = await this.loadOne(`${dirPath}/index`);

      if (!index || !index.files) {
        return {};
      }

      const results = await Promise.allSettled(
        index.files.map(file => this.loadOne(`${dirPath}/${file}`))
      );

      const data = {};
      results.forEach((result, idx) => {
        if (result.status === 'fulfilled' && result.value) {
          const filename = index.files[idx].replace('.json', '');
          data[filename] = result.value;
        }
      });

      return data;
    } catch (error) {
      console.error(`Failed to load directory ${dirPath}:`, error);
      return {};
    }
  }

  clearCache() {
    this.cache.clear();
  }
}

export default new ContentLoader();
```

#### File: `main.js`

```javascript
/**
 * Minimal game initialization
 */
import contentLoader from './ContentLoader.js';

// Simple game state
const game = {
  heroes: {},
  player: null,
  loaded: false
};

async function initGame() {
  console.log('🎮 Loading game...');

  // Load heroes
  game.heroes = await contentLoader.load('characters/heroes');
  console.log('✅ Loaded heroes:', Object.keys(game.heroes));

  // Create player character
  const warriorData = game.heroes.warrior;
  if (warriorData) {
    game.player = createCharacter(warriorData);
    console.log('👤 Player created:', game.player.name);
  }

  game.loaded = true;
  console.log('🎮 Game ready!');

  // Start game loop
  startGame();
}

function createCharacter(data) {
  return {
    ...data,
    currentHp: data.stats.hp,
    position: { x: 0, y: 0 },
    inventory: []
  };
}

function startGame() {
  console.log('Game started with player:', game.player);
  // Your game loop here
}

// Initialize
initGame().catch(error => {
  console.error('❌ Failed to initialize:', error);
});
```

#### File: `/data/characters/heroes/index.json`

```json
{
  "files": ["warrior", "mage", "rogue"]
}
```

#### File: `/data/characters/heroes/warrior.json`

```json
{
  "id": "warrior",
  "name": "Warrior",
  "class": "Fighter",
  "stats": {
    "hp": 120,
    "strength": 18,
    "dexterity": 12,
    "intelligence": 8
  },
  "sprite": "warrior.png",
  "startingEquipment": ["iron-sword", "leather-armor"]
}
```

### HTML to Run It

```html
<!DOCTYPE html>
<html>
<head>
  <title>Minimal Game</title>
</head>
<body>
  <h1>Check Console</h1>
  <script type="module" src="main.js"></script>
</body>
</html>
```

### Testing It

```bash
# Serve with any static file server
python -m http.server 8000

# Or use Node
npx http-server

# Open browser
# http://localhost:8000

# Check console - you should see:
# 🎮 Loading game...
# ✅ Loaded heroes: warrior, mage, rogue
# 👤 Player created: Warrior
# 🎮 Game ready!
```

---

## Summary

### The Minimal Approach

1. **Start with 100 lines** - One ContentLoader class
2. **Use conventions** - File structure over configuration
3. **Load and pray** - No validation initially
4. **Console.log everything** - Manual debugging
5. **F5 to refresh** - No hot reload yet

### When to Grow

- **Week 3-4:** Add basic validation
- **Month 2:** Add hot reload
- **Phase 2:** Add asset manager
- **Production:** Add full infrastructure

### Key Mantras

- **"Build games, not frameworks"**
- **"Wait for the pain"**
- **"Rule of Three"**
- **"Ship features, not architecture"**

### Next Steps

1. **Copy ContentLoader.js** - Use as-is
2. **Create /data structure** - Follow conventions
3. **Build one feature** - Test the pattern
4. **Add second feature** - Copy the pattern
5. **Add third feature** - NOW refactor if needed

---

## Additional Resources

- [DATA-DRIVEN-ARCHITECTURE-SUMMARY.md](./DATA-DRIVEN-ARCHITECTURE-SUMMARY.md) - Full architecture overview
- [VALIDATION-SYSTEM-IMPLEMENTATION.md](./VALIDATION-SYSTEM-IMPLEMENTATION.md) - Add validation when needed
- [HOT-RELOAD-IMPLEMENTATION.md](./HOT-RELOAD-IMPLEMENTATION.md) - Add hot reload when F5 hurts
- [ASSET-MANAGER-SPECIFICATION.md](./ASSET-MANAGER-SPECIFICATION.md) - Add asset management in Phase 2

---

**Remember:** This guide is about building games quickly. Start minimal. Add complexity only when you feel the pain. Your users care about gameplay, not your architecture.

Now go build something!
