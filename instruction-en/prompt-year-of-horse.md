Firecracker String Ignition & Year of the Horse Mascot 3D Demo
-------------------------------

(CC-BY-NC-SA 4.0 by karminski-牙医)

Please use three.js to implement a realistic 3D demo of a "firecracker string being set off". All code (including HTML, CSS, JavaScript) must be encapsulated in a single standalone HTML file. The scene shows the full process of a string of traditional Chinese firecrackers hanging in mid-air being lit and going off one by one, with a Year of the Horse mascot finally dropping out of the octagonal box at the top.

---

## 1. Tech Stack Requirements

*   **Core library:** Three.js (r150+ via CDN)
*   **Interaction:** OrbitControls (with smooth damping)
*   **Animation:** Use `requestAnimationFrame` together with `THREE.Clock`
*   **Post-processing:** `EffectComposer`, `RenderPass`, `UnrealBloomPass`
*   **SVG loading:** `SVGLoader` (used to parse the SVG outline of the Year of the Horse mascot)
*   **Modeling approach:** Pure procedural code generation (firecrackers, octagonal box, etc.); the Year of the Horse mascot is created by loading the external SVG file `horse-4-svgrepo-com.svg` (in the same directory as the HTML file), parsing its outline, and extruding it into 3D geometry

---

## 2. Scene & Environment Setup

*   **Background:**
    *   Create a radial gradient background using CanvasTexture.
    *   Center color: warm deep red (`#4A0000`).
    *   Edge color: near-black dark red (`#1A0000`).
    *   Evoke a festive, lively Chinese New Year atmosphere.

*   **Ground:**
    *   Create a horizontal plane sized 80×80.
    *   Material: `MeshStandardMaterial` with `{color: 0x555555, roughness: 0.85, metalness: 0.1}`.
    *   Receives shadows: `receiveShadow = true`.

*   **Lighting system:**
    *   **Ambient light (AmbientLight):** intensity 0.5, warm white color (`#FFF5E6`), ensuring no pure-black areas in the scene.
    *   **Key light (DirectionalLight):** positioned at a 45-degree angle to the upper right, intensity 1.0, warm color (`#FFEEDD`), shadows enabled with `castShadow = true`, shadow map resolution 1024.
    *   **Fill light (DirectionalLight):** positioned at the back left, intensity 0.3, cool tone (`#AABBFF`), does not cast shadows.
    *   **Performance constraint: do not create dynamic point lights during the animation. All fire-glow effects are achieved with emissive materials plus Bloom post-processing, not by adding new light sources.**

*   **Tone mapping:** Use `THREE.ACESFilmicToneMapping` with exposure `renderer.toneMappingExposure = 1.0`.

---

## 3. Firecracker String Model — Detailed Specification

Overall structure: one vertical main fuse, with 50 firecrackers arranged alternately on each of the left and right sides (100 total), an octagonal box hanging at the top, and a blank segment of about 2 units left at the bottom of the main fuse for ignition. The whole assembly hangs slightly above the center of the scene, with its bottom about 8 units above the ground.

### 3.1 Main Fuse

*   **Appearance:** A vertical reddish-brown cord, total length about 30 units. Built from multiple segments of `TubeGeometry` along a straight path, radius 0.06.
*   **Material:** `{color: 0x8B4513, roughness: 0.9, metalness: 0.0}`.
*   **Logical segmentation:** The main fuse is logically divided into 100 segments, each corresponding to one firecracker's attachment point. Left-side firecrackers attach to odd segments and right-side firecrackers to even segments, forming an alternating arrangement.
*   **Bottom blank:** The very bottom of the main fuse leaves about 2 units of length with no firecrackers attached, serving as the ignition zone.
*   **Burning behavior:** The main fuse burns gradually from bottom to top. Burned segments first briefly turn dark gray-black `{color: 0x222222, emissive: 0x331100}`, then fade their `opacity` to 0 within 0.3 seconds and are hidden (`visible = false`). **Burned main-fuse segments must disappear from view — they must not linger in the scene.** The burning front has an orange-red glowing point (a small sphere with `emissive: 0xFF4400, emissiveIntensity: 3.0`), relying on Bloom to produce the fire-glow effect. The main fuse material must set `transparent: true` to support the opacity fade.

### 3.2 Individual Firecracker

*   **Body:** Red cylinder, height 0.8, diameter 0.18.
    *   Material: `{color: 0xCC0000, roughness: 0.5, metalness: 0.15}`.
*   **Clay seals:** A small earthen-yellow cylinder at both the top and bottom of the body, height 0.04.
    *   Material: `{color: 0xDAA520, roughness: 0.8}`.
*   **Fuse (branch fuse):** A thin blue-green cylinder extending from the firecracker's top seal, length 0.4, diameter 0.03, connecting to the main fuse.
    *   Material: `{color: 0x20B2AA, roughness: 0.4}`.
*   **Arrangement:** Firecrackers on the left side of the main fuse have their fuses extending from their right side to connect to the main fuse, and vice versa on the right. Each firecracker is spaced about 0.3 units from the main fuse. Vertical spacing between adjacent firecrackers is about 0.25 units.
*   **Physics properties:** Each firecracker has mass, position, velocity, and angular velocity attributes for the physics simulation after it detaches.

### 3.3 Octagonal Box

*   **Position:** Hangs at the very top of the main fuse.
*   **Shape:** A regular octagonal prism, about 3 units wide and about 2.5 units tall. Built with `CylinderGeometry(radius, radius, height, 8)` to form the octagon.
*   **Body material:** Bright red, `{color: 0xBB0000, roughness: 0.4, metalness: 0.3}`.
*   **Golden ornamental patterns:**
    *   A ring of golden trim along both the top and bottom edges (implemented as slightly larger thin octagonal slices layered on top), `{color: 0xFFD700, roughness: 0.2, metalness: 0.9}`.
    *   Each side face of the box has a golden diamond or circular decorative motif (implemented with flat geometry fitted to the surface).
*   **Bottom slit:** The bottom of the box has a visible thin slit around its rim (a visual gap is sufficient), used later for the fireworks-jet effect.
*   **Hanging cord:** A red cord extends upward from the top of the box, its upper end fixed to an invisible suspension point above the scene.

---

## 4. Animation Flow

### Phase Zero: Static Display

*   After the page loads, show a static view of the complete firecracker string hanging in mid-air.
*   Centered at the bottom of the page, display a "Start Ignition" button with red background and gold text; the button styling should have a Chinese New Year feel (rounded corners, golden border).
*   The camera views the whole firecracker string from a 30-degree side angle, and can be rotated by mouse drag.

### Phase One: Ignition & Main Fuse Burning (approx. 0-2 s)

1.  After the button is clicked, the button disappears.
2.  A glowing spark point appears at the bottom of the main fuse and begins burning upward at constant speed.
3.  **Burn speed:** The main fuse must burn at **3 times** the burn speed of an individual firecracker's branch fuse. A branch fuse is 0.4 units long and burns in 0.3 seconds (i.e. 1.33 units/second), so the main fuse burn rate is **4 units/second**; with a total main-fuse length of about 30 units, the total burn time is about **7.5 seconds** (from the bottom up to the octagonal box at the top). This fast pace makes the firecrackers drop off densely and rapidly, producing a lively "crackling and scattering" effect.
4.  The spark point uses a high-emissive sphere plus Bloom for its glow; no dynamic point lights.
5.  Burned portions of the main fuse turn gray and black segment by segment, then fade their opacity to 0 within 0.3 seconds and disappear (consistent with the description in Section 3.1).

### Phase Two: Firecrackers Detaching & Exploding in Sequence (approx. 2-10 s)

When the main fuse burns to a firecracker's attachment point:

1.  **Fuse ignition:** That firecracker's branch fuse ignites; a glowing spark point appears at the top of the fuse, and the fuse burns down (shortens) from the attachment end toward the firecracker end within 0.3 seconds.
2.  **Detachment:** When the fuse burns through at the attachment point, the firecracker breaks off from the main fuse and enters free fall.
    *   On detachment, give the firecracker a small random horizontal initial velocity (±0.5 units/second) so it does not fall perfectly straight down.
    *   The firecracker tumbles and rotates randomly while falling.
3.  **Elastic collisions:** Firecrackers bounce elastically when they hit the ground.
    *   Restitution coefficient 0.3-0.5 (not perfectly elastic).
    *   Bounce height decays with each successive bounce.
    *   There is also simple collision detection between firecrackers, and explosion shockwaves can push nearby unexploded firecrackers away.
4.  **Explosion:** After a firecracker's fuse burns out completely (a random delay of about 0.8-1.2 seconds after detachment), the firecracker explodes:
    *   The firecracker model disappears instantly.
    *   An emissive, highly bright sphere rapidly expands and then vanishes (lasting 0.15 seconds), simulating the explosion flash.
    *   **Debris system (performance-optimized):** Use a **single `InstancedMesh`** to manage all debris. Each explosion produces **8-12** small red fragments (far fewer than the old prompt's 50-80). Fragments fall under gravity, and after landing they fade to transparent within 3 seconds and are recycled back into the instance pool.
    *   **Smoke:** 3-5 gray-white semi-transparent spheres spawn at the explosion position, slowly rising and enlarging, fading their opacity to 0 within 2 seconds before being recycled. Use `InstancedMesh` or an object pool.
5.  **Impact effect:** The explosion produces an impact force with a 1.5-unit radius, knocking away other landed but unexploded firecrackers within range.
6.  **Rhythm:** Left and right firecrackers ignite alternately, producing an overall dense "crackling" rhythm. The interval between adjacent firecracker ignitions is about 0.075 seconds (matching the main fuse's 3x burn speed, so all 100 firecrackers ignite within about 7.5 seconds).

### Phase Three: Octagonal Box Fireworks (approx. 10-17 s)

When the main fuse burns up to the bottom of the octagonal box:

1.  **Fire jets from the box bottom:** Firelight starts pouring from the slit at the bottom of the octagonal box.
    *   Use a downward-jetting particle system (InstancedMesh); the particles are small golden spheres `{color: 0xFFD700, emissive: 0xFFAA00, emissiveIntensity: 2.0}`.
    *   Particles initially spray downward, then spread along parabolic trajectories under gravity, simulating golden-spark fireworks.
    *   The jet lasts about 5 seconds, with particle intensity first increasing then decreasing.
2.  **Golden star spray:**
    *   Particles ejected from the bottom of the box form a fan-shaped scatter.
    *   3-5 particles are generated per frame, and particles have a trailing effect (achieved by lowering the brightness of older particles frame by frame).
    *   Particles fade out after 1-2 seconds of flight and are recycled.
3.  **Slight box shake:** During the jet, the box vibrates subtly and randomly (displacement amplitude ±0.05 units), simulating jet recoil.

### Phase Four: Year of the Horse Mascot Drops Out (approx. 17-22 s)

After the fireworks jet ends:

1.  **Box bottom opens:** A panel at the bottom of the box slowly swings open downward (about 0.5 seconds).
2.  **Mascot drops out:** A horse-shaped mascot falls out of the box.
    *   **Mascot modeling (SVG extrusion based):**
        *   Use `SVGLoader` to load the `horse-4-svgrepo-com.svg` file from the same directory (fetch the SVG text, then pass it to SVGLoader.parse for parsing).
        *   Convert the parsed paths into an array of `THREE.Shape` via `SVGLoader.createShapes()`.
        *   For each Shape, use `ExtrudeGeometry` to extrude along the Z axis with some thickness (depth), turning it into a volumetric 3D "relief"-style horse figure.
        *   Enable bevel so the edges are rounded and smooth, giving it the feel of a metal pendant or ceramic ornament.
        *   **Materials:** Use a two-material array (`ExtrudeGeometry` supports material index). The front and back faces use festive red `{color: 0xCC0000, roughness: 0.3, metalness: 0.4}`; the side faces (extrusion walls and bevel faces) use gold `{color: 0xFFD700, roughness: 0.2, metalness: 0.8}`, producing a festive red-body-with-gold-trim effect.
        *   **Note the SVG coordinate-system conversion:** SVG's Y axis points down while Three.js's Y axis points up, so after loading, flip the whole model 180 degrees around the X axis (`rotation.x = Math.PI`) to correct the orientation.
        *   **Scaling & centering:** The original SVG size is 512×512. After loading, compute the bounding box, center the model at the origin, and uniformly scale it so its overall height is about 1.5 units to match the scene scale.
    *   **Hanging cord:** The top of the mascot connects via a thin red cord (about 4 units long) to a fixed point inside the box.
3.  **Pendulum motion:** After dropping out, the mascot swings like a pendulum about the cord's attachment point.
    *   Initial angle about 60 degrees (from the initial momentum gained when dropping out).
    *   Damping coefficient 0.98; swing amplitude gradually decreases.
    *   Use the simple harmonic motion formula: `angle = maxAngle * sin(time * frequency) * damping^time`.
    *   While swinging back and forth, the mascot always faces the direction of its swing (achieved via rotation).
4.  **Final settle:** The mascot eventually slows to a stop near the lowest point, gently swaying.

---

## 5. Performance Optimization Requirements (Critical)

Because this scene has many particles and dynamic objects, performance must be strictly controlled:

1.  **Unified debris & particle management:**
    *   All explosion debris uses **one** `InstancedMesh` (max instance count 200). When debris disappears, its instance slot is recycled and reused.
    *   All smoke particles use **one** `InstancedMesh` (max instance count 50).
    *   Fireworks golden stars use **one** `InstancedMesh` (max instance count 100).
2.  **Dynamic lights forbidden:**
    *   **Do not create or destroy `PointLight` during the animation.** All fire-glow and explosion-flash effects must be achieved via the material's `emissive` property combined with `UnrealBloomPass` post-processing.
    *   This is the core strategy for fixing the stutter issues of the previous version.
3.  **Bloom settings:**
    *   Threshold: 0.8.
    *   Strength: 0.6.
    *   Radius: 0.4.
    *   Control objects' `emissiveIntensity` (2.0-5.0) so the fire glow "crosses" the threshold and produces the bloom effect.
4.  **Geometry optimization:**
    *   Firecracker cylinder segment count kept at `radialSegments: 8`.
    *   Do not use excessive subdivision for the ground.
    *   Use as few faces as possible for the octagonal box decorations.
5.  **Object Pool:**
    *   After a firecracker explodes, its Mesh is not `dispose()`d and rebuilt; instead it is hidden and returned to an object pool, avoiding GC jitter.
    *   Smoke and debris all use the InstancedMesh instance-pool pattern.
6.  **Shadow optimization:**
    *   Only the main directional light casts shadows.
    *   Shadow map resolution 1024 (no higher).
    *   Firecracker debris and smoke particles do not cast shadows.

---

## 6. Camera & Controls

*   **Initial view:** The camera is positioned to the front-side of the firecracker string, tilted slightly upward at about 15 degrees from horizontal, about 25 units away from the string, with the entire firecracker string and the ground fully visible.
*   **OrbitControls:** Smooth damping enabled; mouse rotation, zoom, and pan allowed.
*   **During the animation:** The camera does not move automatically; it remains under user control.

---

## 7. Interaction Controls

*   **"Start Ignition" button:** Centered at the bottom of the page, red background with gold text, golden border, rounded corners. Hidden after being clicked.
*   **"Reset Scene" button:** Appears in the bottom-right corner of the page after the animation starts; a small button that, when clicked, fully resets the scene to its initial state.
*   **Stats display:** The top-left corner of the page shows:
    *   Number of firecrackers set off / total count
    *   Current FPS

---

## 8. Code Structure Requirements

*   Create a `Firecracker` class managing an individual firecracker's state (unlit, fuse lit, detaching/falling, exploded) and physics attributes.
*   Create a `FirecrackerString` class managing the arrangement of the whole string, the main-fuse burning logic, and the sequential ignition scheduling.
*   Create an `OctagonalBox` class managing the octagonal box's model, fireworks jet, and mascot release.
*   Create a `ParticleManager` class to uniformly manage all InstancedMesh particle systems (debris, smoke, fireworks stars).
*   Create a `HorseMascot` class managing the mascot's SVG loading, ExtrudeGeometry extrusion modeling, material setup, coordinate-system correction, scaling/centering, and pendulum-motion physics.
*   All physics calculations use simplified Euler integration; do not bring in an external physics engine.
*   Add brief English comments in the code explaining the key logic.

Please generate the complete code, ensuring the animation is smooth, the visuals are festive, and performance is excellent.
