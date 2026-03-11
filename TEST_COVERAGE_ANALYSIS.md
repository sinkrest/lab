# Test Coverage Analysis

## Current State

**Test coverage: 0%** — The codebase has no test files, no testing framework, no test configuration, and no CI/CD pipeline. All source code lives in 10 standalone HTML files with embedded JavaScript (~1,500-2,000 lines of JS total).

## Testable JavaScript by File

### 1. `shop.html` — Shopping Cart Logic (HIGH PRIORITY)

**~120 lines of JS** containing pure business logic ideal for unit testing:

| Function | What it Does | Testability |
|----------|-------------|-------------|
| `addToCart(id)` | Finds product by ID, increments qty or adds new item | High — pure state mutation |
| `removeFromCart(id)` | Filters cart array by ID | High — pure array operation |
| `changeQty(id, delta)` | Adjusts quantity, removes if <= 0 | High — edge case rich |
| `updateCart()` | Calculates total count and price | High — arithmetic |
| `checkout()` | Clears cart if non-empty | High — state reset |

**Suggested tests:**
- Add item to empty cart, verify cart length and item qty
- Add same item twice, verify qty increments (not duplicate entry)
- Remove item, verify it's gone
- `changeQty` with negative delta reducing to 0 triggers removal
- `updateCart` total price calculation with multiple items at different quantities
- `checkout` on empty cart is a no-op
- `checkout` clears all items and resets total to 0

---

### 2. `snake.html` — Game Engine Logic (HIGH PRIORITY)

**~280 lines of JS** with complex state management and game mechanics:

| Function | What it Does | Testability |
|----------|-------------|-------------|
| `placeFood()` | Random placement avoiding snake and obstacles | Medium — needs seeded random |
| `spawnObstacle()` | Creates obstacle avoiding snake/food, caps at 6 | High — boundary logic |
| `tick()` | Core game loop: movement, collision, scoring, stage advancement | High — critical logic |
| `changeDir(x, y)` | Validates direction change (no 180-degree turns) | High — pure logic |
| `gameOver(reason)` | Ends game, calculates final stats | High — state transition |
| Stage advancement logic | Advances stage every 3 milestones | High — counter logic |
| Speed scaling | Decreases interval as score increases, floors at 60ms | High — boundary logic |
| Health calculation | Maps snake length to 3-tier health | High — threshold logic |

**Suggested tests:**
- `changeDir` rejects reverse direction (e.g., moving right, can't go left)
- `changeDir` on stopped game starts a new game
- Wall collision at each boundary triggers game over
- Obstacle collision triggers game over
- Food collection increments score and advances stage after 3 pickups
- Speed never drops below 60ms
- Health tiers: length < 8 = 3, < 15 = 2, else 1
- `spawnObstacle` respects max of 6 obstacles
- Obstacle decay: obstacles removed when life reaches 0

---

### 3. `command-center.html` — Scenario Simulator (MEDIUM PRIORITY)

**~300 lines of JS** with data visualization and simulation math:

| Function | What it Does | Testability |
|----------|-------------|-------------|
| `runScenario()` | Calculates efficiency, throughput, cost impact, risk score | High — pure math |
| `addLogEntry(msg, type)` | Prepends log, caps at 30 entries | High — DOM + cap logic |
| `resizeCanvas()` | Maps node positions to canvas dimensions | Medium — coordinate math |
| Particle system | Spawns/moves particles along edges | Low — animation |

**`runScenario()` formulas are the best testing target:**
- `efficiency = min(99, 70 + auto * 0.2 - risk * 2 + (work - 100) * 0.05)`
- `throughput = round(1000 * (demand/100) * (efficiency/100) * (work/100))`
- `costImpact = (demand/100 - 1) * 15 + risk * 3 - (auto - 50) * 0.1`
- `riskScore = min(10, risk + (demand > 150 ? 2 : 0) + (lead > 30 ? 1 : 0) - auto * 0.02)`

**Suggested tests:**
- Default scenario values produce expected output
- Efficiency caps at 99%
- Risk score caps at 10
- High demand (> 150) adds +2 to risk
- High lead time (> 30) adds +1 to risk
- Automation reduces risk score
- Log feed respects 30-entry maximum

---

### 4. `toolkit.html` — PD Decision Tools (MEDIUM PRIORITY)

**~180 lines of JS** with four distinct calculators:

| Calculator | Key Logic | Testability |
|-----------|-----------|-------------|
| Stage-Gate Checker | Checkbox count / total = readiness % | High — percentage calculation |
| FMEA Calculator | RPN = Severity x Occurrence x Detection | High — pure multiplication |
| Make vs Buy | Sum of slider values / max possible = recommendation % | High — scoring |
| Cycle Time Estimator | Sum weeks, find longest phase, calculate % | High — arithmetic |

**Suggested tests:**
- **Gate readiness**: 0 checked = 0%, all checked = 100%, color thresholds (80%+ = green, 50%+ = amber, below = red)
- **FMEA RPN**: 1x1x1 = 1, 10x10x10 = 1000, critical threshold at >= 200
- **FMEA summary**: correct count of critical items, correct max/avg
- **Make vs Buy**: all sliders at 1 = strong Buy, all at 10 = strong Make, mixed = Hybrid
- **Cycle Time**: total weeks calculation, longest phase identification, months conversion (weeks / 4.33)

---

### 5. `os.html` — Desktop Simulation (LOW PRIORITY)

Primarily DOM/UI manipulation (window dragging, z-index management). Limited pure logic to test.

### 6. `dashboard.html`, `kreativ.html`, `newsletter.html`, `dog.html`, `index.html` (LOW PRIORITY)

Minimal JavaScript — mostly static content, simple event handlers, or CSS-driven interactions.

---

## Recommended Testing Strategy

### Phase 1: Set Up Infrastructure
1. Initialize `package.json` with `npm init`
2. Install a testing framework — **Vitest** or **Jest** with jsdom environment
3. Add a `tests/` directory
4. Extract testable JavaScript from HTML files into importable ES modules under a `src/` directory

### Phase 2: Extract and Test Pure Logic (Highest ROI)
Refactor embedded `<script>` blocks into importable modules:

```
src/
  cart.js          # from shop.html — cart state management
  game-engine.js   # from snake.html — game loop, collision, scoring
  simulator.js     # from command-center.html — scenario calculations
  toolkit.js       # from toolkit.html — gate, FMEA, make-vs-buy, cycle time
tests/
  cart.test.js
  game-engine.test.js
  simulator.test.js
  toolkit.test.js
```

### Phase 3: Add CI/CD
Create `.github/workflows/test.yml` to run tests on every push and PR.

---

## Priority Ranking

| Priority | File | Reason |
|----------|------|--------|
| 1 | `shop.html` (cart) | Real e-commerce logic; state management bugs directly affect UX |
| 2 | `snake.html` (game) | Complex state machine with many edge cases; collision/scoring bugs break gameplay |
| 3 | `command-center.html` (simulator) | Math-heavy formulas that are easy to get wrong; silent calculation errors |
| 4 | `toolkit.html` (calculators) | Four independent tools with testable math; used for real PD decisions |
| 5 | Everything else | Mostly static or presentational — low risk of logic bugs |

## Estimated Impact

Extracting and testing the top 4 files would cover approximately **80-90% of all JavaScript logic** in the codebase. The remaining files contain primarily DOM rendering and event wiring with little testable business logic.
