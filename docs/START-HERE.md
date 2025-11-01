# START HERE: Developer Quick-Start Guide

**Welcome to Solar Punk Chronicles!** This guide gets you coding in 5 minutes.

---

## 1. Project Overview (What You're Building)

**Solar Punk Chronicles** is a hybrid turn-based RPG combining:
- **JRPG combat** (Final Fantasy-style tactical battles)
- **WRPG exploration** (Fire Emblem city hubs + D&D skill checks)
- **Roguelike progression** (Hades-style randomized upgrades)
- **Solarpunk storytelling** (hopeful sci-fi with character depth)

**Tech Stack:** Phaser 3 + TypeScript + Vite + JSON data files

**Timeline:** 15 weeks for Phase 1 vertical slice (one playable city, complete game loop)

---

## 2. Documentation Map (Read These In Order)

### Essential Reading (Start Here)

1. **docs/START-HERE.md** ← You are here (5 min)
   - Quick orientation and first steps

2. **docs/EXECUTIVE-SUMMARY.md** (15 min)
   - Complete project vision, systems overview, timeline
   - Read before writing any code

3. **docs/research-and-preparation/development-roadmap/PRAGMATIC-PHASE-1-APPROACH.md** (20 min)
   - Week 1-5 implementation strategy
   - Essential infrastructure (data loading, file structure)
   - Critical: explains our minimal approach

### Technical Reference (Use When Needed)

4. **docs/research-and-preparation/technical-architecture/MINIMAL-INFRASTRUCTURE-GUIDE.md**
   - Zero-touch workflow details
   - Code examples for data loading
   - Pattern library

5. **docs/research-and-preparation/development-roadmap/PHASE-1-ROADMAP.md**
   - Week-by-week breakdown
   - Milestone definitions
   - Detailed task lists

### Design Reference (When Building Features)

6. **docs/research-and-preparation/MASTER-GAME-DESIGN.md**
   - All game mechanics defined
   - Character classes, abilities, combat rules
   - Quest design patterns

---

## 3. Getting Started (5 Steps to First Commit)

### Step 1: Read the Executive Summary
```bash
# Open and read (15 minutes)
cat docs/EXECUTIVE-SUMMARY.md
```
**Why:** Understand what you're building before touching code

### Step 2: Set Up Your Environment
```bash
# Install dependencies
node --version  # Need v18+
npm --version

# Create project (if not already done)
npm create vite@latest solar-punk-rpg -- --template vanilla-ts
cd solar-punk-rpg
npm install
npm install phaser

# Start dev server
npm run dev
```
**Verify:** Browser opens to http://localhost:5173

### Step 3: Read the Pragmatic Approach
```bash
# Open and read (20 minutes)
cat docs/research-and-preparation/development-roadmap/PRAGMATIC-PHASE-1-APPROACH.md
```
**Why:** Understand our minimal infrastructure philosophy

### Step 4: Build the Data Loader (Day 1-2 Task)
```bash
# Create the data loader
mkdir -p src/utils
touch src/utils/DataLoader.ts

# Copy the implementation from PRAGMATIC-PHASE-1-APPROACH.md
# Lines 186-254 (the DataLoader class)
```

**Verify:** DataLoader compiles without errors

### Step 5: Create File Structure (Day 3 Task)
```bash
# Create data directories
mkdir -p public/data/characters
mkdir -p public/data/abilities
mkdir -p public/data/enemies

# Create index files and starter data
# Follow examples in PRAGMATIC-PHASE-1-APPROACH.md lines 286-528
```

**Done!** You're ready to start Week 1 tasks

---

## 4. Week 1 Checklist (Your First 5 Days)

### Day 1: Data Loader Pattern
- [ ] Create `src/utils/DataLoader.ts`
- [ ] Implement `load()` and `loadOne()` methods
- [ ] Add caching functionality
- [ ] Test basic JSON loading

**Deliverable:** Working DataLoader class (~100 lines)

**Validation:** Can load a JSON file and cache it

### Day 2: File Structure + Documentation
- [ ] Create `/public/data/` directory structure
- [ ] Create example JSON files (characters, abilities, enemies)
- [ ] Write `data/README.md` with conventions
- [ ] Integrate DataLoader into BootScene

**Deliverable:** Complete file structure with example data

**Validation:** BootScene successfully loads all data files

### Day 3: TypeScript Types
- [ ] Define core types (Character, Ability, Enemy)
- [ ] Define placeholder types (Item, Quest, Location)
- [ ] Add JSDoc comments
- [ ] Update example JSON to match types

**Deliverable:** `game-data.types.ts` with comprehensive types

**Validation:** No TypeScript errors, auto-complete works

### Day 4: Integration + Examples
- [ ] Create comprehensive example data
- [ ] Write helper functions (getCharacter, getAbility)
- [ ] Test data access patterns
- [ ] Create DevTools helper for data inspection

**Deliverable:** Rich example dataset + documentation

**Validation:** Can inspect data in browser console

### Day 5: Testing + Cleanup
- [ ] End-to-end test of data loading
- [ ] Performance check (all data loads in <1 second)
- [ ] Error scenario testing (missing files, malformed JSON)
- [ ] Code review and cleanup

**Deliverable:** Production-ready data loading system

**Validation:** All tests pass, ready for Week 2 (combat system)

---

## 5. Quick Reference Links

### Game Design
📖 **Full Game Design:** `/docs/research-and-preparation/MASTER-GAME-DESIGN.md`
- Character classes, abilities, combat mechanics
- Quest design patterns
- D&D skill system integration

### Technical Architecture
🏗️ **Minimal Infrastructure:** `/docs/research-and-preparation/technical-architecture/MINIMAL-INFRASTRUCTURE-GUIDE.md`
- Data loading patterns
- Convention over configuration
- Zero-touch workflow

⚙️ **Data-Driven Architecture:** `/docs/research-and-preparation/technical-architecture/DATA-DRIVEN-ARCHITECTURE-SUMMARY.md`
- JSON schemas (when you need them)
- Validation system (add later)
- Hot reload (Phase 2)

### Development Roadmap
📅 **Phase 1 Roadmap:** `/docs/research-and-preparation/development-roadmap/PHASE-1-ROADMAP.md`
- 15-week milestone breakdown
- Week-by-week task lists
- Asset creation plan

🎯 **Pragmatic Approach:** `/docs/research-and-preparation/development-roadmap/PRAGMATIC-PHASE-1-APPROACH.md`
- Essential infrastructure first
- What to build now vs. later
- Day-by-day Week 1 plan

### Code Examples
💻 **Implementation Guide:** `/docs/research-and-preparation/technical-architecture/web-rpg-implementation-guide.md`
- Phaser 3 patterns
- Combat system implementation
- State management

🎨 **Game Juice Guide:** `/docs/research-and-preparation/visual-design/rpg-combat-game-juice-guide.md`
- Screen shake, particles, VFX
- Making combat feel satisfying
- Animation patterns

### Research & Inspiration
🔬 **Mechanics Research:** `/docs/research-and-preparation/game-mechanics/hybrid-rpg-mechanics-research.md`
- Fire Emblem + FF + Hades + D&D
- System inspirations

🎭 **Solarpunk Design:** `/docs/research-and-preparation/story-and-characters/solarpunk-design-guide.md`
- Aesthetic guidelines
- Narrative tone
- Character design principles

---

## 6. Critical Decisions Made (Know These)

### Pragmatic Approach
✅ **Minimal infrastructure** (3-5 days, not 2 weeks)
- Build patterns, not complete implementations
- Add validation when it hurts
- Hot-reload in Phase 2

### JSON Data Files
✅ **Convention over configuration**
- No database, just JSON files in `/public/data/`
- Define schemas when we need them (not upfront)
- Expand structure as patterns emerge

### Phaser 3 + TypeScript
✅ **Modern web game engine**
- Phaser 3.80+ for rendering and physics
- TypeScript for type safety
- Vite for fast development

### Zero-Touch Workflow
✅ **No manual asset creation**
- Procedural graphics (code-generated)
- AI-assisted assets (optional)
- Fully automated deployment

### 15-Week Timeline
✅ **Realistic scope**
- Week 1-2: Foundation + data loading
- Week 3-4: Combat system
- Week 5-6: Character systems
- Week 7-8: Exploration
- Week 9-10: World map
- Week 11-12: Game juice
- Week 13-14: Content
- Week 15: Testing & launch

---

## 7. Key Principles (Live By These)

### Convention Over Configuration
**Don't write schemas.** Follow examples. Add structure when patterns emerge.

```json
// Good: Follow convention (id, name, stats)
{
  "fighter_01": {
    "id": "fighter_01",
    "name": "Aria",
    "stats": { "hp": 100, "attack": 15 }
  }
}
```

### Build Games, Not Frameworks
**Don't abstract until you have 3 examples** (Rule of Three)

```javascript
// Week 1: Just load it
const heroes = await loader.load('characters/heroes');

// Week 2: Copy the pattern
const weapons = await loader.load('items/weapons');

// Week 3: NOW create ContentManager base class
```

### Add Complexity When It Hurts
**Wait for the pain before building infrastructure**

- Typos causing bugs? → Add validation
- F5 getting slow? → Add hot-reload
- Data structure changing? → That's OK, refactor it

### Ship Features, Iterate Infrastructure
**Focus on gameplay, not tooling**

- Week 1-10: Build gameplay features
- Week 11-15: Polish and infrastructure
- Phase 2: Advanced tooling if needed

---

## 8. Who to Ask (Where to Look)

### Game Design Questions
❓ "How does combat work?"
→ Read `/docs/research-and-preparation/MASTER-GAME-DESIGN.md`

❓ "What are the character classes?"
→ Read `/docs/research-and-preparation/MASTER-GAME-DESIGN.md` (Character section)

❓ "How do skill checks work?"
→ Read `/docs/research-and-preparation/game-mechanics/dnd-video-game-mechanics-guide.md`

### Technical Questions
❓ "How do I load data?"
→ Read `/docs/research-and-preparation/development-roadmap/PRAGMATIC-PHASE-1-APPROACH.md` (Days 1-5)

❓ "How do I structure files?"
→ Read `/docs/research-and-preparation/technical-architecture/MINIMAL-INFRASTRUCTURE-GUIDE.md`

❓ "How do I implement combat?"
→ Read `/docs/research-and-preparation/technical-architecture/web-rpg-implementation-guide.md`

❓ "How do I add visual effects?"
→ Read `/docs/research-and-preparation/visual-design/rpg-combat-game-juice-guide.md`

### Process Questions
❓ "What should I work on today?"
→ Check `/docs/research-and-preparation/development-roadmap/PHASE-1-ROADMAP.md` (current week)

❓ "Am I on track?"
→ Check milestone deliverables in PHASE-1-ROADMAP.md

❓ "Should I add this feature?"
→ Ask: "Is it essential for the vertical slice?" (If no, defer to Phase 2)

---

## 9. Your Next 15 Minutes

### Immediate Actions (Right Now)

1. **Open the Executive Summary**
   ```bash
   cat docs/EXECUTIVE-SUMMARY.md
   ```
   Skim for 5 minutes to understand the project vision

2. **Check Your Environment**
   ```bash
   node --version  # v18+?
   npm --version
   git --version
   ```

3. **Open the Pragmatic Approach**
   ```bash
   cat docs/research-and-preparation/development-roadmap/PRAGMATIC-PHASE-1-APPROACH.md
   ```
   Read Day 1-2 section (Data Loader implementation)

4. **Start Coding**
   - If project exists: `npm install && npm run dev`
   - If new: Follow Step 2 in "Getting Started" section above

5. **Join the Community** (Optional)
   - Phaser Discord: https://discord.gg/phaser
   - r/gamedev for questions
   - Share progress on Screenshot Saturday

---

## 10. Success Criteria (How to Know You're On Track)

### Week 1 Success ✅
- DataLoader works and loads JSON files
- File structure is established (`/public/data/`)
- Example data files exist and load successfully
- No errors in console
- You understand the pragmatic approach

### Month 1 Success ✅
- Combat system works (can fight 1v1 battles)
- 3 character classes implemented
- Data-driven design (characters/abilities in JSON)
- No major refactoring needed

### Phase 1 Success ✅
- Complete playable loop (world map → city → combat → progression)
- 2-3 hours of gameplay content
- 60 FPS performance
- Deployed and playable in browser
- No critical bugs

---

## 11. Common Pitfalls (Avoid These)

### ❌ DON'T: Build Complete Schemas Upfront
**Problem:** You don't know what you need yet

**Instead:** Use conventions, add schemas when you have 10+ content files

### ❌ DON'T: Abstract Too Early
**Problem:** Premature abstraction adds complexity

**Instead:** Wait for Rule of Three (3 examples before abstracting)

### ❌ DON'T: Add Hot Reload Week 1
**Problem:** Adds complexity before you need it

**Instead:** F5 to refresh is fine. Add hot-reload in Week 11 if it hurts

### ❌ DON'T: Build "A System" Without Use Cases
**Problem:** Over-engineering blocks progress

**Instead:** Build specific features, extract systems when patterns emerge

### ❌ DON'T: Skip Reading the Docs
**Problem:** You'll build the wrong thing

**Instead:** Spend 1 hour reading (saves 10+ hours later)

---

## 12. Emergency Help (When You're Stuck)

### Stuck on Week 1 Setup?
1. Re-read PRAGMATIC-PHASE-1-APPROACH.md (Days 1-5)
2. Copy the DataLoader example exactly (lines 186-254)
3. Check you have Node 18+, npm, and TypeScript installed

### Stuck on What to Build Next?
1. Check PHASE-1-ROADMAP.md for current week tasks
2. Look at milestone deliverables
3. Follow checklist in that milestone section

### Stuck on How to Implement Something?
1. Check MINIMAL-INFRASTRUCTURE-GUIDE.md for patterns
2. Check web-rpg-implementation-guide.md for code examples
3. Search Phaser 3 examples: https://phaser.io/examples

### Stuck on Design Decisions?
1. Read MASTER-GAME-DESIGN.md for game rules
2. Follow existing examples in the docs
3. When in doubt, keep it simple (YAGNI)

---

## 13. Motivation & Mindset

### You Have Everything You Need
- 80,000+ words of research and planning
- Complete game design document
- Week-by-week roadmap
- Code examples and patterns
- Clear scope and timeline

### Start Small, Build Incrementally
**Week 1:** Just load JSON files
**Week 4:** Basic combat works
**Week 8:** First quest completable
**Week 15:** Shipped vertical slice

### It's OK to Feel Overwhelmed
**Feeling:** "This is a huge project, where do I start?"
**Reality:** Start with Week 1, Day 1. Just build the DataLoader.

**Feeling:** "I'm not building infrastructure, am I falling behind?"
**Reality:** That's the pragmatic approach. Build features, not frameworks.

**Feeling:** "Should I add validation/hot-reload/schemas now?"
**Reality:** Not in Week 1. Add complexity when it hurts.

---

## 14. Celebrate Early Wins 🎉

### End of Day 1
✨ You have a DataLoader that loads JSON files

### End of Day 5
✨ You have a complete data loading system with examples

### End of Week 4
✨ You can fight a battle in your game

### End of Week 8
✨ You can complete a quest

### End of Week 15
✨ You shipped a playable game

---

## The Path Forward

```
Today      → Read docs (1 hour)
This Week  → Build data loader (Week 1 tasks)
This Month → Working combat system
Month 2    → Exploration and quests
Month 3    → Polish and integration
Month 4    → Ship vertical slice
```

---

## Ready? Let's Go! 🚀

**Your next action:** Read the Executive Summary (15 minutes)

```bash
cat docs/EXECUTIVE-SUMMARY.md
```

Then return here and start Week 1, Day 1.

**Remember:**
- Build games, not frameworks
- Convention over configuration
- Add complexity when it hurts
- Ship features, iterate infrastructure

**The world needs more hopeful futures. Let's build one.**

🌱⚡🎮

---

*Last Updated: 2025-11-01*
*Project: Solar Punk Chronicles*
*Status: Ready for Development*
