# Memory Palace

> A framework-free, offline-first spatial notes app where ideas live as objects in a 2D "memory palace" of rooms—pan/zoom, drag-drop, and glow-based search for fast recall.

## 🎯 Overview

Memory Palace is an innovative spatial note-taking application that transforms how you organize and retrieve information. Inspired by the ancient mnemonic technique of the "Method of Loci," this app allows you to place notes as objects in a 2D infinite canvas, creating visual connections and spatial memory cues.

## ✨ Key Features

### 🗺️ Spatial Engine
- **Infinite Canvas:** Navigate through an unlimited 2D space with smooth pan and zoom
- **Cursor-Centric Zooming:** Zoom in/out while keeping the point under your cursor stationary
- **Visual Grid System:** CSS-based grid that scales with camera movement for depth perception

### 🧠 Smart Data Management
- **Quadtree Optimization:** Efficiently handle thousands of objects without performance drops
- **Viewport Culling:** Only render visible elements for optimal performance
- **Offline-First:** All data stored locally in IndexedDB for complete offline functionality

### 📝 Spatial Entities
- **Note Cards:** Lightweight, draggable objects that represent your ideas
- **Rooms:** Organize notes into visual "blob" containers with containment detection
- **Real-time Updates:** Drag-and-drop system that updates spatial data structures instantly

### 🔍 Retrieval & Recall
- **Command Palette:** `Ctrl+K` search with visual beacon system
- **Camera Sequencing:** Smooth camera transitions to focus on specific notes
- **Visual Connections:** Canvas-based link visualization between related objects

### 🎨 Level of Detail (LOD)
- **Adaptive Rendering:** Content detail adjusts based on zoom level
  - **Zoom > 1.5:** Full Markdown content
  - **0.5 < Zoom < 1.5:** Title and Tags only
  - **Zoom < 0.5:** Abstract dot view
- **Mini-map:** Overview of your entire memory palace

## 🛠️ Technology Stack

- **HTML5** - Semantic structure
- **CSS3** - Variables, animations, and modern styling
- **Vanilla JavaScript (ES6+)** - No framework dependencies
- **IndexedDB** - Client-side persistence

## 🚀 Getting Started

### Prerequisites

- A modern web browser with IndexedDB support
- No build tools or dependencies required!

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/MemoryPalace.git

# Navigate to the project directory
cd MemoryPalace

# Open index.html in your browser
# Or use a local server:
python -m http.server 8000
# Then visit http://localhost:8000
```

## 📖 Usage

### Basic Navigation
- **Pan:** Click and drag to move around the canvas
- **Zoom:** Use mouse wheel or pinch gesture
- **Create Note:** Click on empty space (coming soon)
- **Search:** Press `Ctrl+K` to open command palette

### Organizing Notes
- **Drag Notes:** Click and drag notes to reposition them
- **Create Rooms:** Group related notes in visual containers
- **Spatial Memory:** Use spatial relationships to remember connections

## 🗺️ Development Roadmap

This project is being developed in phases. See the [Development Roadmap](Docs/DEVELOPMENT_ROADMAP.md) for detailed information about:

- **Phase 1:** Spatial Engine (Foundation)
- **Phase 2:** Data Structure & Quadtree
- **Phase 3:** Spatial Entities (Rooms & Objects)
- **Phase 4:** Retrieval & Recall
- **Phase 5:** Refinement & LOD

## 🏗️ Architecture Highlights

### Modular Architecture
The project follows a clean, modular architecture for maintainability and scalability:

```
assets/js/
├── core/          # Spatial engine (Camera, CoordinateSystem, Grid)
├── data/          # Data structures (Quadtree, IndexedDB, Culling)
├── entities/      # Spatial entities (Object, Room, Draggable)
├── ui/            # User interface (CommandPalette, MiniMap, LOD)
└── utils/         # Utilities (Math helpers, Animation, Debounce)
```

Each module has a single responsibility and communicates via events or callbacks. See the [Development Roadmap](Docs/DEVELOPMENT_ROADMAP.md) for detailed module breakdown.

### Performance Optimizations
- GPU-accelerated transforms using `will-change: transform`
- Pointer Events API for natural touch/stylus support
- Container-based scaling to avoid individual element transforms
- Debounced IndexedDB writes to prevent performance bottlenecks

### Design Principles
- **Framework-Free:** Pure vanilla JavaScript for maximum performance and minimal overhead
- **Modular:** ES6 modules with clear separation of concerns
- **Offline-First:** Complete functionality without internet connection
- **Spatial Memory:** Leverage human spatial cognition for better information recall

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

See [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

Inspired by the ancient mnemonic technique of the "Method of Loci" (Memory Palace technique), where information is associated with spatial locations to improve recall.

---

**Status:** 🚧 In Active Development
