# Cubix Word Search 3D

An interactive 3D word search puzzle where words wrap seamlessly across cube edges, grids rotate in real-time, and finding words clears the die.

The entire game is self-contained in a single file: **`standalone.html`**. There are no build steps, no package managers, and no runtime dependencies.

---

## Why This App Was Built

This project was built as an example application to experiment with **HTML in Canvas**—specifically:

1. **HTML5 2D Canvas as 3D Textures**: Using offscreen 2D HTML `<canvas>` elements as dynamic, real-time texture sources mapped directly onto 3D WebGL geometry in Three.js.
2. **Raycasting UV-to-Grid Coordinate Translation**: Bridging 3D user interactions (clicking and rotating in a WebGL scene) back into exact 2D pixel/grid cells on the canvas surfaces.
3. **Pure Single-File Portability**: Demonstrating that a rich 3D game—complete with ~17,500+ English words, procedural board generation, topological pathfinding, procedural Web Audio, and confetti effects—can live entirely inside a single `standalone.html` file that runs anywhere.

---

## How It Works

### 1. HTML5 Canvas & WebGL Integration
- **6 Offscreen Canvases**: Six 1024×1024 HTML `<canvas>` elements are created in memory, one for each face of the cube (Front, Back, Top, Bottom, Right, Left).
- **2D Canvas Rendering**: Standard HTML5 Canvas 2D context methods (`fillRect`, `strokeRect`, `fillText`) render the letter tiles, grid lines, rounded borders, letter typography, active selection paths, and cleared tile states.
- **Dynamic Texture Pipeline**: Each 2D canvas is passed to a `THREE.CanvasTexture` mapped onto a plane mesh on the 3D cube. When the player selects letters, triggers a hint, or solves a word, `updateTextures()` redraws the canvas surfaces and marks `tex.needsUpdate = true`.
- **UV Raycasting**: When the player clicks on the 3D cube, a `THREE.Raycaster` identifies which face mesh was struck and returns the intersection's UV coordinates `(u, v)`. The game translates these coordinates directly into the exact `(row, col)` tile on that face's 2D canvas.

### 2. 3D Polyhedral Edge Topology
Unlike traditional flat word searches, words in Cubix wrap across adjacent faces in 3D space:
- When a path reaches the edge of a face, `getStep(face, row, col, dir, N)` translates movement across the cube's 90-degree fold onto the correct face, row, and column.
- Words can snake across 2, 3, or more faces of the die in any 8-way directional step (orthogonal and diagonal).

### 3. Procedural Board Generation (Big Words First)
- **Built-in Vocabulary**: Includes ~17,500+ curated common English words (lengths 4 to 12) embedded directly in `standalone.html`.
- **Descending Big-to-Small Packing**: The generator prioritizes packing the longest possible dictionary words first (10–12 letters on large grids, 8–9 on medium grids). As space fills, it progressively steps down to smaller words (down to 4 letters).
- **Dead-End Avoidance (Warnsdorff's Heuristic)**: Pathfinding prioritizes neighbor tiles with fewer free degrees to prevent isolating lone tiles.
- **Bidirectional Absorption**: Any small leftover pockets are absorbed into the ends of adjacent words and mapped to exact dictionary entries, guaranteeing 100% full coverage with zero empty or blank tiles.

### 4. Interactive Hint System & Web Audio
- **Step-by-Step Hints**: Clicking **Hint** (or pressing **H**) clears any active letters and reveals the starting letter of an unsolved word, automatically snapping the 3D camera to face it. Pressing Hint again reveals the next letter along that word's path.
- **Synthesized Audio**: Sound effects (selection steps, victory chords, error thuds) are synthesized procedurally via the browser's built-in Web Audio API without requiring any external audio files.

---

## How to Play

### Running the Game
Since **`standalone.html`** is completely self-contained, no web server or installation is needed:
- **Double-click `standalone.html`** in your file manager, or
- **Drag and drop `standalone.html`** into any web browser (Chrome, Firefox, Safari, Edge), or
- Open via command line:
  ```bash
  # macOS
  open standalone.html

  # Linux
  xdg-open standalone.html

  # Windows
  start standalone.html
  ```

You can also host `standalone.html` on GitHub Pages or any static file host.

---

## Controls

| Action | Control |
| :--- | :--- |
| **Rotate Cube** | Click and drag on the background |
| **Select Letters** | Click tiles in sequence to form a word |
| **Submit Word** | Click **Submit**, press **Enter**, or right-click anywhere |
| **Hint** | Click **Hint** or press **H** (press again for next letter) |
| **Clear Selection** | Click **Clear** or press **Escape** |
| **Snap Camera** | Click face buttons (`Front`, `Back`, `Top`, `Bottom`, `Right`, `Left`) |
| **Change Grid Size** | Click `3×3`, `4×4`, `5×5`, or `6×6` at the top |

---

## License

This project is licensed under the **MIT License** — a royalty-free, permissive open-source license. The full license text is included directly in the header of `standalone.html`.
