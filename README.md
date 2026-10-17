# Cubix Word Search 3D

An interactive 3D word search puzzle where words wrap seamlessly across cube edges, grids rotate in real-time, and finding words clears the die.

The entire game is self-contained in a single file: **`standalone.html`**. There are no build steps, no package managers, and no runtime dependencies.

---

## Testing the HTML-in-Canvas WICG API

This application serves as an interactive demonstration of the **experimental WICG HTML-in-Canvas specification**.

The HTML-in-Canvas API bridges standard HTML/CSS DOM elements with high-performance canvas rendering through three core primitives:

1. **`layoutsubtree` & `content="drawable"` Attribute**:
   Six `<canvas layoutsubtree content="drawable">` elements live in the document. Inside each canvas, the child container `<div class="face-dom-grid" drawable>` carries the required `drawable` attribute. The browser computes full CSS styles (grid layout, typography, borders, shadows, `:hover` pseudo-classes, and state classes) without compositing them onto the standard web page.

2. **The `paint` / `onpaint` Event Listener**:
   Each `<canvas>` listens for the `paint` event:
   ```javascript
   canvas.addEventListener('paint', () => {
     const ctx = canvas.getContext('2d');
     if (typeof ctx.drawElementImage === 'function') {
       ctx.clearRect(0, 0, canvas.width, canvas.height);
       ctx.drawElementImage(domFaceGrid, 0, 0);
       texture.needsUpdate = true;
     }
   });
   ```
   Whenever a tile's letter or state (e.g. `.selected`, `.hovered`, or `.blank`) changes in the DOM tree, the browser's layout engine fires this event to keep the canvas in sync.

3. **Context-Specific Drawing Method (`drawElementImage`)**:
   In the 2D canvas context, `ctx.drawElementImage(domFaceGrid, 0, 0)` rasterizes the styled HTML subtree marked with `drawable` directly onto the canvas, which is then mapped onto the 3D cube mesh via Three.js.

### Testing with the Experimental Flag
- In **Chrome Canary** or **Chromium**:
  1. Navigate to `chrome://flags/#enable-experimental-web-platform-features`
  2. Set the flag to **Enabled** and relaunch.
  3. Open `standalone.html`. The top bar badge will display **`HTML-in-Canvas: Native Active`**.
- In standard browsers without the flag enabled, the app automatically switches to **`HTML-in-Canvas: Ready (Compat Mode)`**, falling back gracefully so the game remains fully playable anywhere.

---

## How It Works

### 1. HTML5 Canvas & WebGL Integration
- **6 Offscreen Canvases with Subtrees**: Six 1024×1024 HTML `<canvas>` elements represent the faces of the cube (Front, Back, Top, Bottom, Right, Left), populated with real HTML `<div class="dom-tile">` elements containing `<span class="tile-letter">`.
- **CSS On-Hover Interactive Scaling**: Moving the mouse over the 3D cube uses raycasting to determine the hovered tile, triggering both the CSS `:hover` pseudo-class and synthetic pointer events. The CSS rule `.dom-tile:hover .tile-letter` applies `transform: scale(1.3)` with warm amber luminescence, making the hovered character smoothly grow in size in real-time. In standard compatibility mode, the 2D canvas fallback automatically scales the font size up by 30% to match.
- **Dynamic Texture Pipeline**: Each 2D canvas is passed to a `THREE.CanvasTexture` mapped onto a plane mesh on the 3D cube. When the player selects letters, triggers a hint, hovers over tiles, or solves a word, the DOM tree updates and `drawElementImage` renders the new layout.
- **UV Raycasting**: When the player hovers or clicks on the 3D cube, a `THREE.Raycaster` identifies which face mesh was struck and returns the intersection's UV coordinates `(u, v)`. The game translates these coordinates directly into the exact `(row, col)` tile on that face's 2D canvas.

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
| **Rotate Cube** | Click/drag with mouse or touch/swipe on screen |
| **Select Letters** | Click or tap tiles in sequence to form a word |
| **Submit Word** | Click **Submit**, press **Enter**, or right-click anywhere |
| **Hint** | Click **Hint** or press **H** (press again for next letter) |
| **Clear Selection** | Click **Clear** or press **Escape** |
| **Snap Camera** | Click face buttons (`3D`, `Front`, `Right`, `Back`, `Left`, `Top`, `Bottom`) |
| **Change Grid Size** | Click `3×3`, `4×4`, `5×5`, or `6×6` |

---

## License

This project is licensed under the **MIT License** — a royalty-free, permissive open-source license. The full license text is included directly in the header of `standalone.html`.
