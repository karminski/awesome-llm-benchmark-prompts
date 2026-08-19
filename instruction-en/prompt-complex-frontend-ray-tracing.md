Global Illumination Physical Path Tracing (Time-Freeze Frame Edition)
----------------------------------
(CC-BY-NC-SA 4.0 by karminski-牙医)



**Role:**
You are a top-tier graphics guru who has worked on offline renderer development and is extremely proficient in WebGL 2.0, Three.js internals, and the Cannon.js physics engine.

**Core Task:**
Write a single-page HTML file that, combined with the Cannon.js physics engine, **implements a dynamic collapse scene entirely in a Fragment Shader, supporting physically based materials (PBR) and Monte Carlo Path Tracing**.
Using Three.js's native materials and light components is strictly forbidden; all lighting must be computed in custom shaders by solving the Rendering Equation.

**Scene and Physics Setup (Cannon.js side):**
1. Finite ground plane: a `100 x 100` checkerboard plane.
2. Brick wall: a wall built from `1 x 2 x 4` bricks stacked in a staggered, offset pattern, 100 bricks in total.
3. Dynamic sphere A (primary light source): a high-quality emissive sphere A with a huge initial velocity, rolling along the ground and smashing into the wall, triggering a realistic physical chain-reaction collapse.
4. Static sphere B (secondary light source): another high-quality emissive sphere B, placed stationary behind the wall.

**Materials and Data Structures (CPU->GPU synchronization challenge):**
The bricks come in three materials: **rough gold**, **frosted glass (transmission with roughness)**, and **diffuse white wall**.
On the JS side you must pack each brick's `inverse world matrix` (16 floats) and its `PBR material parameters` (albedo, roughness, metallic, transmission, etc., at least 4 floats) into a one-dimensional `DataTexture` and upload it to the GPU. The parsing in the shader must be absolutely accurate!

**Ray Tracing Core Algorithm (shader side, extremely hardcore):**
1. **PBR and BRDF**: you must implement the Schlick Fresnel approximation, roughness importance sampling based on the GGX microfacet distribution, and cosine-weighted hemisphere sampling for pure diffuse. **Roughness-driven specular blur** must be visibly demonstrated.
2. **Global illumination and soft shadows**: introduce `dat.gui` or `lil-gui` to provide a floating control panel, which must include the following dynamically adjustable items, synced in real time to the shader's uniforms:
   - **Max Bounces**: range 1~60, default 3.
   - **SPP per frame**: range 1~100, default 10.
   - Inter-reflection (Color Bleeding) between the gold, the glass, and the ground must be visible. Emissive spheres A and B act as Area Lights and must produce physically real soft shadows.
3. **Environment light**: if a ray flies off into the distance, it must sample a procedural sky dome (e.g. a gradient, blue above and warm below) to provide baseline GI illumination.
4. **Spatial matrix math**: in the shader, after reading a brick's inverse matrix to transform into Local Space for the OBB intersection test, you **must correctly compute the adjugate matrix (or inverse-transpose matrix) to transform the local normal back into world space**.

**Progressive Accumulation and Denoising System (the core freeze-frame logic!):**
Achieving film-grade quality depends entirely on temporal accumulation and unbiased Monte Carlo integration. You must use `WebGLRenderTarget` to implement **ping-pong history-frame blended accumulation**, and follow this strict trigger logic:
- **Time-freeze trigger (important!)**: when the demo starts, start a counter; after 20s, **immediately stop calling Cannon.js's `world.step()`**, completely freezing the physics world and producing a "disaster freeze-frame" image.
- **Dynamic period (reset accumulation)**: before the physics world freezes, or while the camera's `OrbitControls` is being dragged, pass `u_accumulateWeight = 0.0` to the shader and reset `historyFrames = 0.0`.
- **Still period (accumulate like crazy)**: once the physics world is frozen and camera interaction has stopped, pass the true integration weight `u_accumulateWeight = historyFrames / (historyFrames + 1.0)` to the shader to begin unbounded accumulation, incrementing `historyFrames`.
- **Anti noise-deadlock mechanism**: you must ensure the random-number seed generated in the Fragment Shader incorporates `u_frameCount` and the pixel coordinates, guaranteeing the ray paths emitted each frame are truly random — so that during the still accumulation period, the noise gets "washed" into a flawless, CG-grade cinematic image within seconds!

**Output Requirements:**
Directly output an extremely clean, thoroughly commented HTML file. All PBR math formulas, the Monte Carlo random number generator, and the matrix intersection sections must include detailed explanations of the underlying principles in Chinese. Show your strength as a top-tier graphics developer!
