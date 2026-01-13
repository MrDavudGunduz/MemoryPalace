# Project "Memory Palace" – Development Roadmap

## Core Stack
- **HTML5**
- **CSS3** (Variables & Animations)
- **Vanilla JavaScript** (ES6+)
- **IndexedDB**

---

## Project Structure (Modular Architecture)

A clean, modular architecture ensures maintainability, testability, and scalability.

```
MemoryPalace/
├── index.html                 # Entry point
├── assets/
│   ├── css/
│   │   ├── main.css          # Global styles, CSS variables
│   │   ├── grid.css          # Grid system styles
│   │   └── animations.css    # Keyframe animations
│   └── js/
│       ├── core/             # Core engine modules
│       │   ├── Camera.js     # Camera controller (pan/zoom)
│       │   ├── CoordinateSystem.js  # World-to-Screen transforms
│       │   └── Grid.js       # Grid rendering system
│       ├── data/             # Data management
│       │   ├── Quadtree.js   # Spatial partitioning
│       │   ├── IndexedDB.js  # Persistence wrapper
│       │   └── CullingEngine.js  # Viewport culling logic
│       ├── entities/         # Spatial entities
│       │   ├── Object.js     # Note card component
│       │   ├── Room.js       # Room entity with containment
│       │   └── Draggable.js # Drag-and-drop system
│       ├── ui/               # User interface modules
│       │   ├── CommandPalette.js  # Ctrl+K search
│       │   ├── MiniMap.js    # Overview map
│       │   ├── LOD.js        # Level of detail manager
│       │   └── CanvasOverlay.js  # Link visualization
│       ├── utils/            # Utility modules
│       │   ├── Math.js       # Lerp, distance, collision helpers
│       │   ├── Animation.js  # Animation utilities
│       │   └── Debounce.js   # Debounce/throttle helpers
│       └── app.js            # Main application orchestrator
└── Docs/
│   ├──DEVELOPMENT_ROADMAP.md
│   ├── ARCHITECTURE.md
└── README.md
```

### Module Design Principles
- **Single Responsibility:** Each module has one clear purpose
- **ES6 Modules:** Use `import/export` for dependency management
- **Event-Driven:** Modules communicate via custom events or callbacks
- **Dependency Injection:** Pass dependencies explicitly, avoid globals
- **Pure Functions:** Where possible, use pure functions for testability

---

## Phase 1: The Spatial Engine (The Foundation)

**Focus:** Establishing the 2D coordinate system and the Infinite Canvas.

**Modules:** `core/CoordinateSystem.js`, `core/Camera.js`, `core/Grid.js`

### Development Tasks

#### Coordinate Mapping
- Implement a "World-to-Screen" and "Screen-to-World" transformation matrix.
- **Module:** `core/CoordinateSystem.js` - Pure transformation functions

#### The Camera Controller
- **Pan:** Implement `pointermove` logic with `requestAnimationFrame` for buttery smooth movement.
- **Zoom:** Implement Cursor-Centric Zooming (calculating the offset so the point under the mouse stays stationary during scaling).
- **Module:** `core/Camera.js` - Camera state management and event handling

#### The Grid System
- A CSS-based repeating background grid that scales with the camera to provide a sense of depth and movement.
- **Module:** `core/Grid.js` - Grid rendering and scaling logic

### Testing (Developed Alongside)

- **Unit Tests:** Write tests for `CoordinateSystem` transformations before/while implementing
  - Test world-to-screen conversions with various camera positions
  - Test screen-to-world conversions (inverse transforms)
  - Test edge cases (negative coordinates, zero scale, etc.)
- **Unit Tests:** Test `Camera` pan/zoom calculations
  - Test cursor-centric zoom maintains point under cursor
  - Test pan calculations with different delta values
- **Integration Tests:** Test Camera + CoordinateSystem interaction
  - Verify transforms work correctly with camera state changes
- **E2E Tests:** Manual/automated testing of smooth panning and zooming

---

## Phase 2: Data Structure & Quadtree (The Brain)

**Focus:** Managing thousands of objects without dropping frames.

**Modules:** `data/Quadtree.js`, `data/CullingEngine.js`, `data/IndexedDB.js`

### Development Tasks

#### Quadtree Implementation
- Develop a Quadtree class to partition the 2D space.
- Implement `insert()`, `update()`, and `query(bounds)` methods.
- **Module:** `data/Quadtree.js` - Spatial data structure

#### Culling Engine
- Create a logic that only renders elements currently within the ViewportRect.
- **Module:** `data/CullingEngine.js` - Viewport-based filtering

#### Persistence Layer
- Set up IndexedDB (using a wrapper or raw) to store object coordinates and content locally, ensuring the "Palace" is persistent.
- **Module:** `data/IndexedDB.js` - Database wrapper with debounced save
- **Migration Support:** Implement version management and migration logic
  - Handle database version upgrades gracefully
  - Migrate existing data when schema changes (e.g., adding new fields)
  - Preserve all existing notes during version updates
  - Provide default values for new fields in migrated data

### Testing (Developed Alongside)

- **Unit Tests:** Write tests for Quadtree operations as you implement them
  - Test `insert()` with objects at various positions
  - Test `update()` when objects move
  - Test `query()` returns correct objects in bounds
  - Test subdivision logic when maxObjects exceeded
- **Unit Tests:** Test CullingEngine viewport filtering
  - Verify only visible objects are returned
  - Test edge cases (objects partially visible, etc.)
- **Unit Tests:** Test IndexedDB wrapper operations
  - Test save/load/delete operations
  - Test debouncing behavior (verify saves don't happen too frequently)
- **Migration Tests:** Test IndexedDB version upgrades and data migration
  - Create test data with old schema version
  - Upgrade to new schema version (e.g., add new field like `tags`, `color`, `metadata`)
  - Verify all existing notes are preserved and migrated correctly
  - Verify new fields have default values for old data
  - Test multiple version jumps (v1 → v2 → v3)
  - Ensure no data loss during migration
  - Test rollback scenarios if migration fails
- **Integration Tests:** Test Quadtree + CullingEngine together
  - Verify performance with large datasets
- **Performance Tests:** Measure query time with 10,000 objects
  - Ensure O(log n) performance characteristics

---

## Phase 3: Spatial Entities (Rooms & Objects)

**Focus:** Creating the "Movable Furniture" of the mind.

**Modules:** `entities/Object.js`, `entities/Room.js`, `entities/Draggable.js`

### Development Tasks

#### The "Object" Component
- A lightweight DOM factory for Note Cards.
- **States:** Idle, Dragged, Focused.
- **Module:** `entities/Object.js` - Note card factory and state management

#### The "Room" Entity
- Implement "Blob" shapes using CSS `border-radius` and `z-index` layering.
- Logic for "Containment": Detecting when an Object's center is within a Room's bounds.
- **Module:** `entities/Room.js` - Room rendering and containment detection

#### Drag-and-Drop System
- Implement a custom Draggable system that updates the Quadtree position in real-time.
- **Module:** `entities/Draggable.js` - Drag handler with Quadtree integration

### Testing (Developed Alongside)

- **Unit Tests:** Test Object state machine as you implement states
  - Test state transitions: Idle → Focused → Dragged → Idle
  - Test invalid state transitions are prevented
  - Test DOM element creation and updates
- **Unit Tests:** Test Room containment detection
  - Test `contains()` method with objects inside/outside bounds
  - Test edge cases (object on boundary, etc.)
- **Integration Tests:** Test Draggable + Quadtree interaction
  - Verify Quadtree updates during drag
  - Test real-time position updates don't cause performance issues
- **E2E Tests:** Test complete drag-and-drop workflow
  - Create object → Drag → Release → Verify position saved
- **Performance Tests:** Test drag smoothness with 1000 objects
  - Ensure 60fps during dragging

---

## Phase 4: Retrieval & Recall 

**Focus:** Visualizing search and connections.

**Modules:** `ui/CommandPalette.js`, `utils/Math.js`, `ui/CanvasOverlay.js`

### Development Tasks

#### The Search (Command Palette)
- `Ctrl+K` implementation that triggers the "Beacon" system.
- **Module:** `ui/CommandPalette.js` - Search UI and beacon triggering

#### Camera Sequencing
- A helper function to lerp (Linear Interpolation) the camera position and zoom level to focus on specific coordinates.
- **Module:** `utils/Math.js` - Lerp and animation utilities

#### Glow & Path Effects
- **CSS Pulsing:** Using `box-shadow` animations for the Glow Ring.
- **SVG/Canvas Overlay:** Use a transparent `<canvas>` over the DOM to draw the "Links" (lines) and "Path Hints" between objects to avoid DOM bloat.
- **Module:** `ui/CanvasOverlay.js` - Canvas-based link visualization

### Testing (Developed Alongside)

- **Unit Tests:** Test CommandPalette search filtering as you implement
  - Test search query matching logic
  - Test keyboard shortcut handling (Ctrl+K)
  - Test beacon triggering
- **Unit Tests:** Test lerp calculations in Math utils
  - Test linear interpolation accuracy
  - Test easing functions
- **Integration Tests:** Test complete search flow
  - Test Search → Camera sequencing → Canvas overlay rendering
  - Verify smooth camera transitions
- **E2E Tests:** Test complete search and navigation workflow
  - Open palette → Type query → Select result → Verify camera moves

---

## Phase 5: Refinement & LOD (Level of Detail)

**Focus:** Optimization and professional polish.

**Modules:** `ui/LOD.js`, `utils/Math.js`, `ui/MiniMap.js`

### Development Tasks

#### LOD Implementation
- **Zoom > 1.5:** Show full Markdown content.
- **0.5 < Zoom < 1.5:** Show Title and Tags only.
- **Zoom < 0.5:** Show only a colored dot/pixel (Abstract view).
- **Module:** `ui/LOD.js` - Level of detail manager with zoom-based rendering

#### Collision Handling
- Simple "Repulsion" logic to prevent notes from stacking directly on top of each other.
- **Module:** `utils/Math.js` - Collision detection and repulsion algorithms

#### Mini-map
- A small fixed-position canvas that renders a simplified version of the Quadtree nodes.
- **Module:** `ui/MiniMap.js` - Overview map rendering

### Testing (Developed Alongside)

- **Unit Tests:** Test LOD detail level calculations
  - Test correct detail level returned for each zoom range
  - Test boundary conditions (zoom = 1.5, zoom = 0.5)
- **Unit Tests:** Test collision detection algorithms
  - Test repulsion calculations
  - Test collision detection accuracy
- **Integration Tests:** Test LOD + Rendering pipeline
  - Verify correct detail level rendered at each zoom
  - Test smooth transitions between detail levels
- **Performance Tests:** Measure frame times at different zoom levels
  - Ensure 60fps at all zoom levels
  - Verify LOD reduces DOM complexity as expected
- **E2E Tests:** Test zoom in/out with LOD transitions
  - Verify smooth visual transitions
  - Verify no rendering glitches

---

## Testing Strategy

**Approach: Test-Driven Development (TDD) / Test-While-Developing**

Tests are written **alongside** or **before** implementation, not after. This ensures:
- ✅ Code quality from the start
- ✅ Better design (testable code is better designed)
- ✅ Immediate feedback on correctness
- ✅ Confidence when refactoring
- ✅ Living documentation of expected behavior

See [Architecture Documentation](ARCHITECTURE.md) for detailed testing guidelines.

### Test Types

1. **Unit Tests:** Test pure functions and isolated modules
   - Write tests **before or during** implementation
   - Focus on isolated functions with predictable inputs/outputs
   - Test edge cases and boundary conditions
   - Target modules: `core/CoordinateSystem.js`, `utils/Math.js`, `utils/Animation.js`

2. **Integration Tests:** Test module interactions
   - Write tests **after** modules are implemented
   - Verify modules work together correctly
   - Test event communication between modules
   - Target interactions: Camera ↔ CoordinateSystem, Quadtree ↔ CullingEngine, Draggable ↔ Quadtree

3. **E2E Tests:** Test user workflows
   - Write tests **after** features are complete
   - Simulate real user interactions
   - Test complete feature flows
   - Scenarios: Create note → Drag → Save, Search → Navigate → Select, Pan → Zoom → Pan

4. **Performance Tests:** Measure frame times and optimization
   - Write tests **during** optimization phases
   - Ensure 60fps with large datasets
   - Test Quadtree query performance
   - Measure IndexedDB write/read latency
   - Benchmark rendering pipeline with various zoom levels

### Development Workflow

For each feature/module:
1. **Write test first** (TDD approach) or **alongside** implementation
2. **Implement feature** to make test pass
3. **Refactor** if needed (tests ensure nothing breaks)
4. **Run tests** frequently during development
5. **Commit** only when tests pass

### Testing Tools & Setup

- **Test Framework:** Choose based on project needs (Jest, Vitest, or Mocha)
- **E2E Framework:** Playwright or Cypress for browser automation
- **Performance:** Chrome DevTools Performance API, `performance.now()`
- **Coverage:** Aim for 80%+ code coverage on critical modules

### Continuous Testing

- Run unit tests on every commit
- Run integration tests before merging PRs
- Run E2E tests on staging environment
- Run performance tests weekly or before releases

---


