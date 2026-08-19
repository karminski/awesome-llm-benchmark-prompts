Roche Limit Simulation Animation Prompt
--------------------------------

(CC-BY-NC-SA 4.0 by karminski-牙医)

Please use Three.js (imported via CDN importmap) to write a complete, web-based 3D demo simulating the astronomical disaster scenario of "the Moon crossing the Roche limit and crashing into the Earth". All code must be written in a single HTML file, with no dependency on any external image or resource files.

Build the complete scene according to the detailed "visual and logical description" below.

---

**1. Overall Visual Style and Environment:**

*   **Rendering style:** The scene should be very bright and clear, with ample lighting, ensuring the user can see all details. Please enable `THREE.ACESFilmicToneMapping` tone mapping, with `toneMappingExposure` set to around 1.2.
*   **Multi-light illumination:** Do not use just a single directional light. Set up at least: one ambient light (AmbientLight), one main directional light (simulating the sun), and one fill light (Fill Light, illuminating the dark side from the other direction). Ensure the scene is bright overall, with no completely black dead zones.
*   **Starfield background:** You must create a spherical starfield background composed of at least 2000–3000 randomly distributed small white points (`THREE.Points`), wrapping the entire scene. This must not be omitted.
*   **Reference line:** On the XZ plane around the Earth, draw a clearly visible red dashed circular ring (`THREE.LineDashedMaterial`) marking the "Roche limit" distance. Add a `THREE.Sprite` text label next to the ring displaying "ROCHE LIMIT". The ring's opacity should have a slow breathing pulse animation.

---

**2. Procedural Textures (Earth):**

Since no external images are loaded, the Earth texture must be procedurally generated using the Canvas 2D API. **Pay special attention to texture quality.**

*   **Earth texture (at least a 2048×1024 canvas):**
    *   Ocean base color: use a **vertical gradient** from deep blue (#0D47A1) to bright blue (#2196F3), not just a solid fill.
    *   Continents: draw 6–8 continental landmasses. Each continent's outline must be drawn using `quadraticCurveTo` or `bezierCurveTo` with **organic, irregular edge shapes**; they absolutely must not be just randomly scattered circles. Use greens of varying depth for the continents (#4CAF50, #66BB6A, #81C784, etc.).
    *   Terrain detail: overlay some smaller random color patches (dark green, bright green) on each continent to simulate the color variation of mountains and plains.
    *   Clouds: with `globalAlpha = 0.25` semi-transparent white, draw about 50–60 circles of varying sizes to simulate cloud clusters.
    *   Polar ice caps: draw white-to-transparent linear gradients at the top and bottom of the canvas to simulate polar ice and snow.

---

**3. Character Definitions:**

*   **Earth (radius set to 5 units):**
    *   Use `MeshStandardMaterial` with the procedural texture above, setting `roughness: 0.7`, `metalness: 0.1`.
    *   **Two-layer atmospheric glow (must be implemented):**
        *   **Inner layer (FrontSide):** a semi-transparent sphere slightly larger than the Earth, using a custom ShaderMaterial to implement a Fresnel rim-glow effect (rim = pow(1.0 - dot(viewDir, normal), 2.5)). Color is light blue, using `AdditiveBlending`.
        *   **Outer layer (BackSide):** a larger sphere, also using the Fresnel shader, but with side set to `THREE.BackSide` and a higher rim exponent (3.5), creating a broader outer halo.
    *   The Earth should have a slow self-rotation animation (rotation.y).

*   **Moon (Important! The Moon is not a single Mesh, but a spherical cluster of meteorite chunks):**
    *   **Core concept: the Moon is composed of about 70 independent large meteorite chunks (MoonChunk) tightly packed into a sphere.** Do not create any Moon Mesh or Moon texture. These meteorite chunks themselves are the Moon's skeletal structure.
    *   **Spherical arrangement:** use the Fibonacci sphere distribution algorithm to arrange the chunks evenly across three layers:
        *   Outer shell layer (~40 chunks, radius 0.78–1.0 × MOON_RADIUS, size 0.18–0.36)
        *   Middle layer (~20 chunks, radius 0.4–0.78, size 0.12–0.26)
        *   Core layer (~10 chunks, radius 0–0.45, size 0.14–0.32)
    *   Each chunk records its own `localOffset` (local offset vector) relative to the Moon's center.
    *   **Lunar maria color variation:** based on each chunk's angular position on the sphere, chunks in some regions use a darker gray (simulating the dark lunar maria), and chunks in other regions use a lighter gray (highlands), producing light-dark variation similar to the Moon's surface.
    *   **Chunk appearance (meteorite style; they must not be smooth geometry!):**
        *   Use `IcosahedronGeometry(size, detail)` as the base geometry; large chunks use detail=1 (42 vertices), small chunks use detail=0 (12 vertices).
        *   **Random stretch axes:** multiply each chunk by a random factor of 0.7–1.3 on each of the X/Y/Z axes, giving the chunks irregular base shapes such as elongated or flattened forms.
        *   **Multi-layer noise displacement:** for each vertex, apply three superimposed layers of radial noise displacement along its normal direction: large-scale bulges (amp = size×0.25), medium-scale bumps (amp = size×0.12), and small-scale roughness (amp = size×0.06). A 3D hash function may be used.
        *   **Crater indentations:** each chunk randomly generates 2 "impact crater" directions, and vertices near those directions are pushed inward.
        *   **flatShading: true** to emphasize the hard, angular faceted texture of rock.
    *   **Shell mini-meteorites (Shell Chunks, must be implemented, far more numerous than the large chunks):**
        *   On the outermost layer of all the meteorite chunks, cover them with about 5000 tiny meteorite Meshes (not particle points!), generated with `createMeteoriteGeometry(size, 0)`, size about 0.05–0.11, `flatShading: true`. Distribute them evenly on a sphere of MOON_RADIUS × (0.97–1.05).
        *   **Purpose:** the large chunks are few (~70), so the massive coverage of shell mini-meteorites (~5000) forms the Moon's complete surface appearance. This way the Moon looks densely and completely surfaced, while the large chunks provide the dramatic visual effect during breakup.
        *   **Approach phase:** the shell mini-meteorites move synchronously with moonCenter, together with the chunks forming the complete Moon appearance.
        *   **After entering the Roche limit:** all shell mini-meteorites are simultaneously released as independent physics bodies, inheriting the Moon's current velocity, plus an additional **tangential vortex velocity component** (computing the tangent direction perpendicular to the Earth-Moon axis via crossVectors), so the mini-meteorites are flung toward the Earth in a swirling vortex pattern, synchronized with the chunks' layered disintegration.
        *   **Performance optimization:** shell mini-meteorites do not get any collision explosion effects and no burning effects. Simply use a distance check (< EARTH_RADIUS × 1.2) to hide them directly, saving compute.
        *   Color is light gray (0.45–0.65 grayscale), `castShadow: false`.
    *   **The Moon does not rotate.** In reality the Moon is tidally locked, and rotation would cause the chunk positions to become inconsistent in the subsequent physics simulation.
    *   **The Moon has no shaking/vibration animation.** The breakup process is driven entirely by physical forces; no artificial shaking effect is needed.

---

**4. Animation Flow and Physics Simulation (purely physics-driven; implement strictly):**

All physics updates must be computed based on the dt (delta time) returned by `THREE.Clock`'s `getDelta()`, ensuring the animation speed is frame-rate independent.

**Core physics model (minimalist pure gravity, no cohesion force):** after release, each chunk is subject to only one force:
*   **Earth's gravity** (computed independently for each chunk): `F_gravity = G_earth / dist_to_earth²`, directed toward the Earth's center.

**Do not implement a cohesion force.** A previous version using cohesion caused an unnatural oscillation where chunks first clumped together and then rebounded after entering the Roche limit. The correct approach is: after entering the Roche limit, all chunks are instantly released as independent physics bodies subject only to Earth's gravity. Because each chunk's distance to the Earth differs, chunks closer to the Earth feel stronger gravity and accelerate faster, naturally flying out first; chunks farther from the Earth feel weaker gravity and accelerate more slowly. This **gravity differential** is itself the essence of tidal force, naturally producing an "onion-peeling" layered effect — **no cohesion decay, artificial release thresholds, erosion fronts, or shader clipping are needed**.

*   **Phase One — Approach + per-chunk detachment (Approach with per-chunk Roche detection):**
    *   Maintain a `moonCenter` position, initially one Moon diameter outside the Roche limit (`MOON_START_DISTANCE = ROCHE_LIMIT + MOON_RADIUS × 2`), so the player can clearly watch the Moon slowly approach from a distance.
    *   **The approach speed is extremely slow and gradually accelerates:** set the base speed to `APPROACH_BASE_SPEED = 0.005` (extremely slow). Use a quadratic acceleration curve: `speedMult = 1 + progress² × 9`, where `progress = distance traveled / total distance`.
    *   **Each chunk independently checks the Roche limit (key point! Not a whole-body release!):**
        *   Every frame, iterate over all unreleased chunks and first compute their world position `worldPos = moonCenter + localOffset`.
        *   If `worldPos.length() <= ROCHE_LIMIT` (the chunk itself has entered the Roche limit), then **release that chunk individually**, giving it moonCenter's current velocity.
        *   Chunks that have not crossed the line continue following moonCenter. Released chunks run independent physics (Earth's gravity).
        *   The shell mini-meteorites work the same way: individually detected, individually released; on release they additionally receive the tangential vortex velocity.
    *   **Effect:** chunks nearer the Earth reach the Roche limit first and detach toward the Earth first; chunks on the far side of the Moon are still following the Moon as it continues to approach. This produces a natural layer-by-layer peeling effect.
    *   **Mixed-state handling:** during the approach phase, some chunks are already falling, burning, and impacting, so impact detection and flame trails must also be active during the approach phase.
    *   A slight Y-axis sine-wave bobbing is allowed.
    *   Once all chunks have been released, enter Phase Two.

*   **Phase Two — Collective fall (Falling):**
    *   All chunks have been released, each independently accelerated by Earth's gravity, falling toward the Earth.
    *   **moonCenter continues, as a reference point, to be accelerated by Earth's gravity and falls toward the Earth** (used only for UI display and tracking; it does not affect chunk physics).
    *   **Important: do not use a cohesion force.** Cohesion would cause the unnatural oscillation of chunks clumping and then rebounding.

*   **Phase Three — Falling and Atmospheric Entry (Falling & Atmospheric Entry):**
    *   Chunks fall independently, with no chunk-to-chunk collision detection; only collisions with the Earth are detected.
    *   **Atmosphere interaction:** when a chunk's distance to the Earth's center < the atmosphere radius (Earth radius × 1.15):
        *   Set `material.emissive` to orange-red (#ff4400), with `emissiveIntensity` increasing the deeper it penetrates the atmosphere.
        *   Gradually lerp `material.color` toward orange.
        *   **An independent flame-trail particle system must be implemented:** a shared `THREE.Points` + custom shader, using `AdditiveBlending`.

*   **Phase Four — Impact and Explosion (Impact & Explosion):**
    *   A chunk is judged to have impacted when its distance to the Earth's center ≤ the Earth's radius.
    *   **Key implementation detail (bug-prone, please note):** when the chunk's `update()` method detects a collision, only set the `impacted = true` flag; **do not** set `alive = false` in the same frame. In the animation loop, you must **first iterate over and process all `impacted` chunks** (spawn explosion effects, hide the mesh, set alive=false), and **then** filter surviving chunks with `if (!alive) continue` for the flame trails. Otherwise, setting `alive=false` and `impacted=true` in the same frame and then skipping via `!alive` would mean the explosion effect never triggers.
    *   **Impact explosion effect (do not use a simple scaled disc!):**
        *   Explosion particle system: eject 30–60 particles from the impact point along the surface normal (bright yellow → orange → dark red).
        *   Ring shockwave: an additional 20–30 ring-spreading particles.
        *   When a large chunk impacts, create a temporary `PointLight` (orange, intensity 8, range 15) that decays and disappears after 1.5 seconds.
        *   Surface scorch marks: semi-transparent red/orange circular Meshes that slowly fade out.
    *   Once all chunks have impacted, enter the "simulation complete" state.

---

**5. UI and Interaction:**

*   **OrbitControls:** configure damping (`enableDamping: true`), limit the zoom range (`minDistance / maxDistance`), and place the initial camera at an elevated, angled bird's-eye view.
*   **Info panel:** place a semi-transparent frosted-glass style (`backdrop-filter: blur; rgba background`) info panel in the top-left corner, displaying in real time:
    *   The current phase name (color-coded, e.g. approach = blue, tidal disintegration = orange, impact = red)
    *   Phase description text
    *   Data such as distance / chunk count / surviving count / impact count
*   **Reset button:** centered at the bottom of the screen, with a gradient background and hover scale effect; clicking it clears all chunks, particles, and explosion effects and regenerates the Moon chunk cluster.
*   **Legend:** place a small color legend in the bottom-right corner, annotating the color meanings of the Roche limit line, the atmosphere, and the Moon chunk cluster.

---

Please generate the complete single HTML file code containing all the logic above (scene setup, procedural texture generation, spherical meteorite chunk arrangement + shell mini-meteorite coverage, slow gradual approach + pure-gravity physics simulation (no cohesion), multiple particle systems, material handling, UI panel). Please write the comments in the code in English.
