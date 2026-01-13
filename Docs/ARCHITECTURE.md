# Memory Palace - Architecture Documentation

## Table of Contents
1. [System Overview](#system-overview)
2. [Architecture Layers](#architecture-layers)
3. [Core Modules](#core-modules)
4. [Data Flow](#data-flow)
5. [Coordinate System](#coordinate-system)
6. [Rendering Pipeline](#rendering-pipeline)
7. [Performance Architecture](#performance-architecture)
8. [Event System](#event-system)
9. [State Management](#state-management)
10. [Module Interfaces](#module-interfaces)

---

## System Overview

Memory Palace is built as a **modular, event-driven architecture** that separates concerns into distinct layers:

```
┌─────────────────────────────────────────────────────────┐
│                    Application Layer                     │
│                  (app.js - Orchestrator)                 │
└─────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌───────▼────────┐  ┌───────▼────────┐
│   Core Layer   │  │   Data Layer   │  │  Entity Layer  │
│  (Spatial)     │  │  (Storage)     │  │  (Objects)     │
└────────────────┘  └────────────────┘  └────────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
┌───────────────────────────▼───────────────────────────┐
│                    UI Layer                            │
│         (CommandPalette, MiniMap, LOD, etc.)          │
└────────────────────────────────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────┐
│                  Utility Layer                        │
│            (Math, Animation, Debounce)                │
└────────────────────────────────────────────────────────┘
```

---

## Architecture Layers

### 1. Core Layer (`core/`)
**Responsibility:** Spatial transformations and camera control

- **CoordinateSystem.js**: Pure transformation functions (world ↔ screen)
- **Camera.js**: Camera state, pan/zoom logic, event handling
- **Grid.js**: Background grid rendering and scaling

**Key Characteristics:**
- Stateless transformation functions where possible
- Camera maintains position (x, y) and zoom (scale) state
- All transforms use matrix math for efficiency

### 2. Data Layer (`data/`)
**Responsibility:** Spatial indexing and persistence

- **Quadtree.js**: Spatial partitioning for efficient queries
- **CullingEngine.js**: Viewport-based filtering
- **IndexedDB.js**: Persistence wrapper with debouncing

**Key Characteristics:**
- Quadtree handles O(log n) spatial queries
- Culling reduces render load by 90%+ for large datasets
- IndexedDB writes are debounced (500ms) to prevent I/O blocking

### 3. Entity Layer (`entities/`)
**Responsibility:** Spatial objects and interactions

- **Object.js**: Note card factory and state machine
- **Room.js**: Container entity with blob shapes
- **Draggable.js**: Drag handler with real-time Quadtree updates

**Key Characteristics:**
- Objects have three states: `Idle`, `Dragged`, `Focused`
- Rooms use CSS `border-radius` for organic shapes
- Dragging triggers Quadtree `update()` on every frame

### 4. UI Layer (`ui/`)
**Responsibility:** User interface components

- **CommandPalette.js**: Search interface with beacon system
- **MiniMap.js**: Overview canvas rendering
- **LOD.js**: Level-of-detail manager
- **CanvasOverlay.js**: Link visualization layer

**Key Characteristics:**
- CommandPalette uses keyboard shortcuts (Ctrl+K)
- MiniMap renders simplified Quadtree structure
- LOD switches rendering detail based on zoom level
- CanvasOverlay avoids DOM bloat for connections

### 5. Utility Layer (`utils/`)
**Responsibility:** Shared helper functions

- **Math.js**: Lerp, distance, collision detection
- **Animation.js**: Easing functions, interpolation
- **Debounce.js**: Throttle/debounce utilities

**Key Characteristics:**
- Pure functions for testability
- No side effects
- Reusable across modules

---

## Core Modules

### CoordinateSystem.js

**Purpose:** Transform coordinates between world space and screen space.

**API:**
```javascript
class CoordinateSystem {
  // World to Screen transformation
  static worldToScreen(worldX, worldY, camera) {
    // Returns: { x, y } in screen coordinates
  }
  
  // Screen to World transformation
  static screenToWorld(screenX, screenY, camera) {
    // Returns: { x, y } in world coordinates
  }
  
  // Get transformation matrix
  static getTransformMatrix(camera) {
    // Returns: { scale, translateX, translateY }
  }
}
```

**Implementation Notes:**
- Uses matrix multiplication: `screen = (world - camera.pos) * camera.scale`
- Inverse transform: `world = screen / camera.scale + camera.pos`
- All calculations are pure functions (no state)

### Camera.js

**Purpose:** Manage camera state and handle user input for pan/zoom.

**API:**
```javascript
class Camera {
  constructor(container) {
    this.x = 0;           // World X position
    this.y = 0;           // World Y position
    this.scale = 1.0;     // Zoom level
    this.container = container;
  }
  
  pan(deltaX, deltaY) { }
  zoom(factor, screenX, screenY) { }  // Cursor-centric zoom
  setPosition(x, y) { }
  setZoom(scale) { }
  lerpTo(x, y, scale, duration) { }   // Smooth transitions
}
```

**Implementation Notes:**
- Uses `requestAnimationFrame` for smooth panning
- Cursor-centric zoom: calculates offset so point under cursor stays fixed
- Lerp uses easing function for smooth camera sequencing
- Emits `camera:changed` event on state updates

### Quadtree.js

**Purpose:** Spatial partitioning for efficient object queries.

**API:**
```javascript
class Quadtree {
  constructor(bounds, maxObjects = 10, maxLevels = 5) {
    this.bounds = bounds;  // { x, y, width, height }
    this.maxObjects = maxObjects;
    this.maxLevels = maxLevels;
  }
  
  insert(object) { }
  update(object, oldBounds) { }
  query(bounds) { }  // Returns all objects in bounds
  clear() { }
}
```

**Implementation Notes:**
- Subdivides space when `maxObjects` exceeded
- `update()` removes from old position, inserts at new position
- `query()` used by CullingEngine to get viewport objects
- Typical performance: O(log n) for insert/query

### IndexedDB.js

**Purpose:** Persist spatial data to browser storage.

**API:**
```javascript
class IndexedDB {
  constructor(dbName = 'memoryPalace') {
    this.dbName = dbName;
    this.storeName = 'objects';
  }
  
  async save(object) { }
  async load(id) { }
  async loadAll() { }
  async delete(id) { }
}
```

**Implementation Notes:**
- Uses debounced save (500ms) to prevent excessive writes
- Stores: `{ id, x, y, content, metadata }`
- All operations are async (returns Promise)
- Handles database versioning and migration

---

## Data Flow

### Object Creation Flow
```
User Action → app.js → Object.create() → Quadtree.insert()
                                      → IndexedDB.save() (debounced)
                                      → DOM.render()
```

### Drag Flow
```
Pointer Down → Draggable.start() → Object.setState('Dragged')
Pointer Move → Draggable.update() → Object.setPosition()
                                 → Quadtree.update()
                                 → Camera.pan() (if edge dragging)
Pointer Up   → Draggable.end() → Object.setState('Idle')
                              → IndexedDB.save() (debounced)
```

### Search Flow
```
Ctrl+K → CommandPalette.open() → User types query
                              → Filter objects
                              → Camera.lerpTo() (smooth transition)
                              → CanvasOverlay.drawPath() (visual connection)
```

### Render Flow
```
requestAnimationFrame → CullingEngine.getVisibleObjects()
                    → LOD.determineDetailLevel()
                    → Object.render() / Object.renderAbstract()
                    → CanvasOverlay.drawLinks()
```

---

## Coordinate System

### World Space
- **Origin:** Center of infinite canvas (0, 0)
- **Units:** Pixels (logical)
- **Range:** Unbounded (infinite 2D space)

### Screen Space
- **Origin:** Top-left of viewport (0, 0)
- **Units:** CSS pixels
- **Range:** Viewport dimensions (window.innerWidth, window.innerHeight)

### Transformation
```
Screen X = (World X - Camera X) * Camera Scale + Viewport Center X
Screen Y = (World Y - Camera Y) * Camera Scale + Viewport Center Y

World X = (Screen X - Viewport Center X) / Camera Scale + Camera X
World Y = (Screen Y - Viewport Center Y) / Camera Scale + Camera Y
```

### Cursor-Centric Zoom
When zooming at point (screenX, screenY):
1. Convert screen point to world: `worldPoint = screenToWorld(screenX, screenY)`
2. Apply zoom: `newScale = oldScale * zoomFactor`
3. Convert world point back to screen: `newScreenPoint = worldToScreen(worldPoint)`
4. Calculate offset: `offset = screenPoint - newScreenPoint`
5. Adjust camera: `camera.x += offset.x / newScale`, `camera.y += offset.y / newScale`

---

## Rendering Pipeline

### Frame Rendering Steps

1. **Culling Phase**
   - Get viewport bounds in world space
   - Query Quadtree for visible objects
   - Filter out objects outside viewport

2. **LOD Phase**
   - Determine zoom level
   - Assign detail level to each object:
     - `Zoom > 1.5`: Full detail (markdown content)
     - `0.5 < Zoom < 1.5`: Medium detail (title + tags)
     - `Zoom < 0.5`: Abstract (colored dot)

3. **DOM Update Phase**
   - Update existing DOM elements (reuse when possible)
   - Create new elements for newly visible objects
   - Remove elements for objects that left viewport
   - Apply transforms: `transform: translate(x, y) scale(scale)`

4. **Canvas Overlay Phase**
   - Clear canvas
   - Draw connection lines between related objects
   - Draw path hints for search results
   - Draw glow effects for focused objects

5. **Grid Update Phase**
   - Update CSS background-position based on camera
   - Scale grid pattern based on zoom level

### Performance Optimizations
- **Element Pooling:** Reuse DOM elements instead of creating/destroying
- **Batch DOM Updates:** Use DocumentFragment for multiple inserts
- **GPU Acceleration:** `will-change: transform` on animated elements
- **RequestAnimationFrame:** All rendering happens in RAF loop

---

## Performance Architecture

### Optimization Strategies

#### 1. Spatial Indexing (Quadtree)
- **Problem:** Checking collision with 10,000 objects = 10,000 checks
- **Solution:** Quadtree reduces to ~10-20 checks per query
- **Impact:** 100-1000x faster spatial queries

#### 2. Viewport Culling
- **Problem:** Rendering 10,000 objects when only 50 are visible
- **Solution:** Only render objects in viewport
- **Impact:** 99% reduction in render operations

#### 3. Level of Detail (LOD)
- **Problem:** Rendering full markdown for tiny objects
- **Solution:** Render simplified version at low zoom
- **Impact:** 80-90% reduction in DOM complexity

#### 4. Debounced Persistence
- **Problem:** Writing to IndexedDB on every drag frame (60fps = 60 writes/sec)
- **Solution:** Debounce saves to 500ms after last change
- **Impact:** 30x reduction in I/O operations

#### 5. GPU Acceleration
- **Problem:** CPU-based transforms cause jank
- **Solution:** `will-change: transform` promotes to GPU layer
- **Impact:** 60fps smooth animations

#### 6. Container Scaling
- **Problem:** Scaling 1000 individual elements
- **Solution:** Scale parent container, use `transform-origin: 0 0`
- **Impact:** Single transform operation vs 1000

---

## Event System

### Custom Events

Memory Palace uses a custom event system for module communication:

```javascript
// Event emitter pattern
class EventEmitter {
  on(event, callback) { }
  off(event, callback) { }
  emit(event, data) { }
}
```

### Event Types

#### Camera Events
- `camera:pan` - Camera position changed
- `camera:zoom` - Camera zoom changed
- `camera:changed` - Any camera state change

#### Object Events
- `object:created` - New object created
- `object:moved` - Object position changed
- `object:selected` - Object focused/selected
- `object:deleted` - Object removed

#### Room Events
- `room:containment` - Object entered/left room
- `room:created` - New room created

#### UI Events
- `search:opened` - Command palette opened
- `search:result` - Search result selected
- `lod:changed` - Detail level changed

### Event Flow Example

```
User drags object
  → Draggable emits 'object:moved'
  → Quadtree listens, updates spatial index
  → Camera listens, checks if edge-dragging needed
  → IndexedDB listens, schedules debounced save
  → LOD listens, updates detail if zoom changed
```

---

## State Management

### Application State

```javascript
const AppState = {
  camera: {
    x: 0,
    y: 0,
    scale: 1.0
  },
  objects: Map<id, Object>,  // All objects
  rooms: Map<id, Room>,      // All rooms
  selectedObject: null,       // Currently focused
  searchQuery: '',           // Current search
  viewport: {
    width: window.innerWidth,
    height: window.innerHeight
  }
};
```

### State Updates

- **Immutable Updates:** Create new state objects, don't mutate
- **Single Source of Truth:** State lives in `app.js`
- **Reactive Updates:** Modules subscribe to state changes
- **Debounced Writes:** State changes trigger debounced IndexedDB saves

### State Machine (Object States)

```
[Idle] ──click──> [Focused]
  │                  │
  │                  │ drag
  │                  ▼
  │              [Dragged]
  │                  │
  │                  │ release
  └──────────────────┘
```

---

## Module Interfaces

### Object.js Interface

```javascript
class Object {
  constructor(id, x, y, content) {
    this.id = id;
    this.x = x;
    this.y = y;
    this.content = content;
    this.state = 'Idle';
    this.element = null;  // DOM element
  }
  
  setState(state) { }      // 'Idle' | 'Dragged' | 'Focused'
  setPosition(x, y) { }
  render(detailLevel) { }  // 'full' | 'medium' | 'abstract'
  destroy() { }
}
```

### Room.js Interface

```javascript
class Room {
  constructor(id, bounds) {
    this.id = id;
    this.bounds = bounds;  // { x, y, width, height, path }
    this.objects = Set<Object>;
  }
  
  contains(object) { }     // Check if object center is in bounds
  addObject(object) { }
  removeObject(object) { }
  render() { }
}
```

### Draggable.js Interface

```javascript
class Draggable {
  constructor(object, camera, quadtree) {
    this.object = object;
    this.camera = camera;
    this.quadtree = quadtree;
  }
  
  start(event) { }         // Pointer down
  update(event) { }        // Pointer move
  end(event) { }           // Pointer up
}
```

### LOD.js Interface

```javascript
class LOD {
  static getDetailLevel(zoom) {
    if (zoom > 1.5) return 'full';
    if (zoom > 0.5) return 'medium';
    return 'abstract';
  }
  
  static shouldRender(object, viewport, camera) {
    // Check if object should be rendered at all
  }
}
```

---

## Best Practices

### Module Development

1. **Single Responsibility:** Each module does one thing well
2. **Pure Functions:** Prefer pure functions over stateful classes
3. **Dependency Injection:** Pass dependencies explicitly
4. **Event-Driven:** Use events for cross-module communication
5. **Error Handling:** Always handle async errors (IndexedDB, etc.)

### Performance Guidelines

1. **Avoid Layout Thrashing:** Batch DOM reads/writes
2. **Use RAF:** All animations in `requestAnimationFrame`
3. **Debounce I/O:** Never write to IndexedDB on every frame
4. **Reuse Elements:** Pool DOM elements when possible
5. **Monitor FPS:** Use performance API to track frame times

### Testing Strategy

1. **Unit Tests:** Test pure functions (CoordinateSystem, Math utils)
2. **Integration Tests:** Test module interactions (Camera + Quadtree)
3. **E2E Tests:** Test user workflows (drag, search, zoom)
4. **Performance Tests:** Measure frame times with 1000+ objects

---

## Future Considerations

### Scalability
- **Web Workers:** Move Quadtree queries to worker thread
- **Virtual Scrolling:** Only render visible portion of large lists
- **Incremental Loading:** Load objects as user pans around

### Extensibility
- **Plugin System:** Allow custom object types
- **Theme System:** CSS variables for easy theming
- **Export/Import:** JSON export for backup/sharing

### Accessibility
- **Keyboard Navigation:** Full keyboard support
- **Screen Readers:** ARIA labels and roles
- **High Contrast:** Theme support for accessibility

---

**Last Updated:** See [Development Roadmap](DEVELOPMENT_ROADMAP.md) for implementation timeline.
