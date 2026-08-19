Floppy Disk Exploded View
----------
(CC-BY-NC-SA 4.0 by karminski-牙医)


You are now a senior WebGL / Three.js expert with 10 years of experience, proficient in building complex 3D scenes and data visualizations purely in code.

I need you to use native Three.js (no React Three Fiber, just pure HTML/JS) to recreate an engineering blueprint design of a "3.5-inch Floppy Disk Exploded View".

Please strictly follow the design specifications, visual style, and component structure below, and write complete, runnable code (contained in a single HTML file, with Three.js, OrbitControls, and CSS2DRenderer imported via CDN).

### 1. Visual Style and Scene Setup (Art Direction)
*   **Background**: Light grayish-white background (e.g. `#F5F7FA`), with a Dot Grid pattern drawn via Canvas as the background.
*   **Camera**: Must use an **Orthographic Camera (OrthographicCamera)**, with position set to `(100, 80, 100)` and `lookAt(0,0,0)`, producing an Isometric Perspective.
*   **Material Style (Blueprint Wireframe)**: 
    *   Except for the Magnetic Disk, all other geometry **must not** have solid fill colors.
    *   **Exception — SHUTTER (metal sliding cover, mandatory)**: The shutter body must be rendered as an **opaque metal plate** (visually with no see-through, no blueprint ghost translucency); you may use an opaque `MeshBasicMaterial` (e.g. dark blue-gray `#4a5568` or slightly lighter than the wireframe blue) + `EdgesGeometry` outlines, or express it with opaque faces + hole geometry. It is **forbidden** to make the shutter look like translucent glass, or to render it as the same low-opacity thin sheet as the "exploded-state translucent fill".
    *   Use `EdgesGeometry` with `LineSegments` to draw edge lines, with moderate line width.
    *   All lines uniformly use the color `#4169E1` (engineering blue).
    *   Semi-transparent parts use `MeshBasicMaterial`; the **target after the explosion has fully expanded** is `opacity: 0.08`, `transparent: true`, with a blue-white color, overlaid with blue edge lines (**does not apply to the shutter body**; the shutter may remain opaque in the exploded state, and only needs to move along with its Group).
*   **Assembled vs. Exploded Opacity (Mandatory)**:
    *   **Assembled state (`assembled`, `collapsing` returning to assembled, and when `exploding` progress is 0)**: All Meshes whose opacity can be set (including the semi-transparent fills above, **including the MAGNETIC DISK's solid material**) must appear visually **opaque**: set the corresponding `material.opacity` to **1** (`transparent` may remain `true` to avoid shader recompilation on toggle, or be set to `false` for performance until labels need to fade). Lines from `LineSegments` / `Line` / `EdgesGeometry` keep their original visibility and do not need to be hidden by the opacity rules.
    *   **After the explosion has fully expanded (`exploded`, and near the end of `exploding`, or once `collapsing` has left the assembled state)**: restore the translucency parameters from the blueprint spec (fills at about **0.08**, magnetic disk at about **0.5**, etc.), consistent with the rest of this document.
    *   **Recommended implementation**: store `{ assembledOpacity, explodedOpacity }` on `parts` or on each part's Group (the disk and fills have different targets), and inside `updateAnimation` **lerp** using the **same progress** as the displacement (or a progress delayed by 0–150ms), so the "fading to transparent" is synchronized with the layered fly-out; the collapse animation interpolates back to 1 in reverse.
*   **Interaction — Camera (OrbitControls, mandatory)**:
    *   Use `OrbitControls` and **explicitly** set the mouse button behavior to be consistent with common DCC conventions:
        *   **Left mouse button**: rotate the view (`THREE.MOUSE.ROTATE`).
        *   **Mouse wheel**: zoom / dolly the camera (`controls.enableZoom = true`; the wheel zooms by default; do not make the middle button the only way to zoom).
        *   **Right mouse button**: pan the scene (`THREE.MOUSE.PAN`).
    *   Example (set after creating `OrbitControls`):
        *   `controls.mouseButtons.LEFT = THREE.MOUSE.ROTATE;`
        *   `controls.mouseButtons.MIDDLE = THREE.MOUSE.DOLLY;` (the middle button may still zoom; it can coexist with or replace the wheel, but the wheel must work)
        *   `controls.mouseButtons.RIGHT = THREE.MOUSE.PAN;`
    *   Listen for `contextmenu` on `renderer.domElement` and call `preventDefault()`, so the browser's right-click menu does not interfere with panning.
    *   `controls.enablePan = true`, `controls.enableRotate = true`, and limit the zoom and pan ranges: if using an **OrthographicCamera**, prefer `controls.minZoom` / `controls.maxZoom` (with a sensible initial `zoom`) to constrain wheel zoom; if you switch to a perspective camera later, use `minDistance` / `maxDistance` instead. Pan range can be constrained by limiting the movement radius of `controls.target` or clamping via a custom `change` callback, so the floppy disk cannot be dragged too far out of frame.
    *   Near the bottom hint UI, add a small line of text explaining the controls, e.g.: `LMB: ORBIT · WHEEL: ZOOM · RMB: PAN` (Courier New, same color, semi-transparent), placed so it does not obscure the main hint.

### 2. Geometry Construction Guidelines (Important!)
**Each Shell is not a single Box, but a composite structure assembled from multiple sub-geometries via a Group.** 
Please combine all sub-parts with `THREE.Group()`, and generate EdgesGeometry separately for each sub-part.
For "recessed groove" effects: overlay a thin box or cylinder at a slightly lower Y value on top of the main body and outline it (no real boolean operations needed).
For "cut-out" effects: use `Shape` + `holes` + `ExtrudeGeometry` to create real holes.

### 3. Geometry Component Hierarchy (Dual-State Position Definitions)
**Every part must define two Y-coordinate states**:
- `assembledY`: the Y value in the assembled state (all parts nearly flush, total thickness about 4 units, simulating the look of a real assembled floppy disk).
- `explodedY`: the Y value in the exploded state (spaced about 15 units apart from top to bottom).

Reference values (assembled → exploded):
| Part              | assembledY | explodedY                                  |
| ----------------- | ---------- | ------------------------------------------ |
| TOP SHELL         | 1.5        | 45                                         |
| DUST LINER (upper) | 1.0        | 30                                         |
| MAGNETIC DISK     | 0.5        | 15                                         |
| HUB               | 0.7        | 18                                         |
| DUST LINER (lower) | 0.0        | 0                                          |
| BOTTOM SHELL      | -1.5       | -15                                        |
| SHUTTER           | -1.5       | -15 (with an X/Z offset to 30 units beyond the front-left of the bottom shell) |
| WRITE PROTECT TAB | -1.0       | -15 (with an X/Z offset to outside the rear-right of the bottom shell)       |

**On initial render, all parts must be at `assembledY`** (i.e. it should look like a complete floppy disk).
In the assembled state, the SHUTTER and WRITE PROTECT TAB must be seated in their corresponding slots on the bottom shell (X/Z displacement also participates in the animation interpolation).

#### 3.1 TOP SHELL — Composite Structure
Base dimensions approximately 60 x 2 x 60. Use `THREE.Shape` to draw a polygonal outer contour with chamfered corners (3-unit chamfers on all four corners), then extrude to a thickness of 2 with ExtrudeGeometry.
**Must include the following sub-parts (all outlined with EdgesGeometry overlaid on the main body's top surface)**:
- **(a) HD NOTCH**: a 4x4 square cut-out in the top-left corner (implemented as a real hole using the Shape's holes).
- **(b) Label recess**: a large rectangle (about 36 x 22) centered in the upper half, appearing as a sunken shallow recess — overlay a thin Box slightly below the main body's surface.
- **(c) Inner recess**: a smaller rectangle (about 20 x 10) below the label recess, representing the depressed area where the metal shutter slides.
- **(d) Shutter opening slot (aligned with the shutter window, mandatory)**: cut a **rectangular through-hole** in the main body's **front edge** (the side facing the drive head when inserted into the drive); its size and placement must, in the assembled state, be **collinear and of the same width and height across all three locations** — the rectangular opening on the **3.7 SHUTTER** and the **3.6 (g) bottom-shell opening** (a single head channel); it may be slightly offset to one side to match the reference engineering drawing, but it **must not** be drawn with independently arbitrary dimensions misaligned from the shutter hole. The cut-out must be a real hole made with `Shape` + `holes` or an equivalent method, not merely a wireframe rectangle stuck on the surface.
- **(e) Arrow marker**: draw a small triangular arrow with a few Lines near the shutter opening to indicate direction.

#### 3.2 DUST LINER (Upper Dust Liner)
A C-shape (a ring with a notch), drawn using `Shape` with an outer circle + inner circle holes, and with a rectangular notch cut somewhere to simulate the head window; extrude extremely thin with ExtrudeGeometry.

#### 3.3 MAGNETIC DISK
**The only part with a solid fill.** CylinderGeometry (radius about 22, height 0.3), material is semi-transparent blue `#7b9df8` (opacity 0.5), overlaid with dark-blue EdgesGeometry. You may draw two or three short diagonal Lines on the surface to suggest data track reflections.

#### 3.4 HUB (Center Metal Disc)
A small cylinder (radius about 5, height 0.4), located at the center of the magnetic disk. Overlay a smaller circular hole at its center, then a square hole beside it (these two are the drive spindle engagement slots).

#### 3.5 DUST LINER (Lower Dust Liner)
Same shape as the upper dust liner.

#### 3.6 BOTTOM SHELL — Composite Structure
Base shape identical to the Top Shell (chamfered rectangle, ExtrudeGeometry).
**Must include the following sub-parts**:
- **(a) Large central circular recess**: a thin cylinder about 44 in diameter overlaid on the top surface, outlined — representing the disk containment area.
- **(b) Central circular hole**: at the exact center of the large circular recess, a cut-out hole about 14 in diameter (implemented with Shape holes), representing the hole the spindle passes through.
- **(c) Four corner locating posts**: place one tiny cylinder near each of the four corners (radius 1.5, height 1), representing screw bosses / locating pins.
- **(d) LIFTER**: a small rectangular protrusion in the lower-middle area (about 8 x 4 x 1).
- **(e) SHUTTER SPRING area**: a small rectangular recess (about 10 x 4) at the middle of the front edge.
- **(f) WRITE PROTECT NOTCH**: a small square cut-out (about 3 x 3) at the rear-right corner.
- **(g) Shutter opening slot (aligned with the top shell and shutter, mandatory)**: a **rectangular through-hole** in the front edge at the same position and size as **3.1 (d)**, mirror-coaxial with the Top Shell; in the assembled state it must align with the rectangular window on the shutter, forming a continuous head channel.

#### 3.7 Auxiliary Small Parts (Located Outside the Bottom Shell, Not on the Vertical Stacking Axis)
*   **SHUTTER (Metal Sliding Cover) — Geometry and Material (must be strictly followed)**  
    *   **Position**: When exploded, located about 30 units to the front-left of the bottom shell; in the assembled state, seated in the sliding slot on the bottom shell's front edge (consistent with `assembledY` / XZ in the table).  
    *   **Opaque**: see §1 "Exception — SHUTTER"; the body has a solid metal-plate look, with **no** semi-transparent fill.  
    *   **Hole shape and placement**: cut a **rectangular through-hole** in the shutter plate face (using `Shape` + `holes` + `ExtrudeGeometry` or a boolean equivalent to make a real cut-out).  
        *   **Off-center position**: the rectangular hole sits on **one side along the sliding-rail length direction** of the shutter plate face; referencing a real 3.5" floppy disk, it is **toward the left** (viewed from the disk toward the front edge, the hole area is biased toward the observer's left-hand side / that side of the disk body); it is **forbidden** to make it a large centered symmetric window, unless the top and bottom shell holes are simultaneously centered and this still matches the reference drawing.  
        *   **Orientation constraint (consistent with the reference drawing)**: let the longer pair of opposite edges of the shutter plate face be the "shutter long edges" (generally along the disk insertion/sliding direction or parallel to the front edge), and the shorter pair be the "shutter short edges". The **short edges of the rectangular hole must be parallel to the shutter's long edges**, and its long edges parallel to the shutter's short edges (i.e. the hole appears as a "crosswise strip" relative to the elongated plate face; or equivalently: **the long axis of the hole rectangle ⟂ the shutter's long axis**).  
    *   **Fit with the shells**: in the assembled state, the hole must **align and coincide** with the rectangular cut-outs of **3.1 (d)** and **3.6 (g)** (same width, height, and positional relationship along the front edge), so as to illustrate "with the shutter slid open, the head aligns with the magnetic disk". For implementation, it is recommended to first fix the "head channel" rectangle's dimensions and its local coordinates along the front edge, then reuse the same set of parameters to generate the holes at all three locations (Top/Bottom/Shutter).  
    *   **Dimension reference (may be fine-tuned, but must be consistent across all three locations)**: the shutter as a whole about **22 × 1 × 16** (thickness about 1); with a rectangular window of about **14 × 6**, the relationship must satisfy "6 is the edge parallel to the shutter's long edges, 14 is the edge parallel to the shutter's short edges" (if the overall proportions are adjusted, this parallelism relationship and the leftward-offset position must be preserved). A U-shaped wrap-around edge may be kept if it matches the reference, but **the hole must remain a single rectangle**; do not replace it with a large rounded hole.
*   **WRITE PROTECT TAB**: located outside the rear-right of the bottom shell. A tiny block (about 3 x 2 x 3).

### 4. Connector Lines and Annotations
*   **Vertical dashed lines**: use `LineDashedMaterial` to draw 4 vertical dashed lines, each passing through one of the body's four chamfered corner positions, spanning Top Shell → Bottom Shell (be sure to call `computeLineDistances()`). Connect the Shutter and Write Protect Tab to their corresponding positions on the bottom shell with diagonal dashed lines.
*   **Text annotations**: 
    *   Use `CSS2DRenderer` and `CSS2DObject`.
    *   Monospace font (Courier New), font size 12px, color `#4169E1`.
    *   Each label is connected to its part with a short Line as a leader line.
    *   Labels: HD NOTCH, TOP SHELL, DUST LINER (×2), MAGNETIC DISK, HUB, BOTTOM SHELL, LIFTER, SHUTTER SPRING, WRITE PROTECT NOTCH, SHUTTER, WRITE PROTECT TAB.
    *   On the left side of the screen, use absolutely positioned HTML to typeset "FIG_001" vertically; on the right side, typeset "[ 3.5\" FLOPPY DISK ]" vertically.

### 5. Output Requirements
Please output the complete `<!DOCTYPE html>` code directly, with a clear structure and thorough comments. Encapsulate the composite construction process of each Shell in a dedicated function (e.g. `createTopShell()` / `createBottomShell()`) that returns a Group. The focus is on recreating the high-tech, wireframe, isometric blueprint aesthetic, and the detail recesses/cut-outs on the Shells must be visible.

**Shutter and hole self-check (after implementation, confirm each item in comments or a README)**:
1. The SHUTTER body is **opaque** with a metallic look, not a semi-transparent ghost surface.  
2. The SHUTTER has a **real rectangular cut-out**, **offset to the left** (relative to the front edge / reference drawing), and **the hole's short edges are parallel to the shutter's long edges**.  
3. The rectangular through-holes of TOP SHELL (d) and BOTTOM SHELL (g) use **the same set of dimensions and local coordinates** as the shutter hole, aligned in the assembled state.

### 6. Animation and Interaction Sequencing (Animation Sequence)

#### 6.1 State Machine
Define a global state variable `animState` that can take the following values:
- `'assembled'`: initial state, parts assembled, no annotations.
- `'exploding'`: explosion animation in progress.
- `'exploded'`: explosion complete, annotations fading in or already shown.
- `'collapsing'`: reverse animation (optional, pressing space again collapses).

#### 6.2 Trigger
- Listen for the `keydown` event; pressing the **Space bar (Space)** toggles the animation direction.
- First press: `assembled` → `exploding` → `exploded`
- Press again: `exploded` → `collapsing` → `assembled` (annotations fade out first)
- Pressing space again while an animation is in progress should be ignored, to avoid state confusion.

#### 6.3 Explosion Animation Details
- **Total duration**: about 3 seconds.
- **Easing function**: use easeInOutCubic easing (please implement it by hand, without depending on external libraries):
- **Staggered start (Stagger)**: parts should not all start at once; delay each by 80ms in top-to-bottom order, so the TOP SHELL flies out first and the BOTTOM SHELL sinks last, creating a layered effect.
- **Interpolation**: in each `requestAnimationFrame` frame, compute progress from the `elapsed` time and lerp each part Group's `position.y` (and `position.x` / `position.z` when necessary).
- **Opacity synchronization**: consistent with the "Assembled vs. Exploded Opacity" section above, lerp each Mesh material's `opacity` inside the same `updateAnimation` using the same progress (optionally with slight per-part stagger via delay): **the assembled end is 1, the expanded end is each part's target opacity**; the collapse animation interpolates in reverse.
- Recommended architecture: maintain a `parts` array where each entry contains `{ group, fromPos, toPos, delay, fromOpacity, toOpacity }` (or an array of material references), advanced uniformly in the animate loop.

#### 6.4 Annotation and Dashed Line Display Sequencing
- **Initial state**: all CSS2DObject labels, leader Lines, and vertical dashed lines (LineDashedMaterial) are set to `visible = false`, with their corresponding DOM elements' opacity set to 0 and CSS `transition: opacity 0.4s ease`.
- **After the explosion completes** (state becomes `exploded`):
  - After a 200ms delay, first set the 4 vertical dashed lines to `visible = true` and tween material.opacity from 0 to 1 (using the same easeInOutCubic).
  - After another 200ms delay, reveal the labels one by one in top-to-bottom order, at 60ms intervals, setting each DOM element's opacity to 1 (the CSS transition handles the fade-in automatically).
  - Leader Lines fade in synchronously with their corresponding labels.
- **On collapse**: first set all labels and dashed lines back to opacity 0 (completing in about 0.3s), then start the collapse animation.

#### 6.5 Hint UI
- Place an HTML element centered at the bottom of the screen with the text "PRESS [ SPACE ] TO EXPLODE", using Courier New, color `#4169E1`, opacity 0.6, with a slow breathing/blinking animation (CSS `@keyframes`).
- When `animState !== 'assembled'`, this hint fades out and hides.

#### 6.6 Suggested Code Structure
Please encapsulate the animation-related logic as:
- `setupParts()`: collect all movable parts and their start/end positions.
- `setupControls()`: create `OrbitControls`, configure `mouseButtons` (left rotate, right pan, middle/wheel zoom), `minZoom`/`maxZoom` (orthographic camera), and pan/rotate toggles.
- `triggerExplode()` / `triggerCollapse()`: start the forward/reverse animation.
- `updateAnimation(deltaTime)`: called in the render loop, advancing interpolation for all parts (**including position and material opacity**).
- `showLabels()` / `hideLabels()`: control the fade-in/out of the annotation group.

And mark animation-related code sections in comments, e.g. `// === ANIMATION: explode trigger ===`.
