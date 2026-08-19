RBMK Reactor Accident Simulation Animation Prompt
--------------------------------

(CC-BY-NC-SA 4.0 by karminski-牙医)

**Theme:** Create a cinematic 3D simulation animation of an RBMK reactor accident using Three.js

**Reference notes:** This prompt is written based on real structural images of the RBMK-1000 reactor and Chernobyl accident materials. It covers the reactor core's vertical fuel channel layout, the upper biological shield cover with colored markings ("Elena"), and the physics details of the accident sequence.

**Role:** You are a creative coding expert proficient in Three.js, GLSL shader authoring, and WebGL visual effects.

**Goal:** Create a browser-based 3D animation sequence demonstrating an RBMK graphite reactor's progression from criticality, to runaway, to final explosion and meltdown. The demo is for cautionary/educational purposes; the visuals must be serious, striking, and physically believable.

**Tech stack requirements:**
1.  **Core library:** Use `Three.js` (loaded via CDN).
2.  **Animation library:** Use `GSAP` (GreenSock) to orchestrate the complex animation timeline.
3.  **Post-processing:** Must use `EffectComposer` and **UnrealBloomPass** — this is key to simulating a nuclear reactor's high-energy light effects and high-temperature thermal radiation.
4.  **Generation approach:** All models (core, lid, control rods) are generated procedurally in code (Procedural Generation). **Do not** load external model files, so the code runs directly after being copied.

**Scene composition:**

1.  **The Core:**
    *   **Overall structure:** A matrix of numerous vertical cylinders (fuel channels), arranged in an approximately 18x18 grid array (referencing the actual RBMK-1000 layout).
    *   **Fuel channels:** Each channel is a tall tubular structure running the full height of the core. Initially dark gray/black metallic finish.
    *   **Graphite moderator:** Dark graphite blocks (representable as low-poly cubes) fill the space between fuel channels, forming a honeycomb-like structure.
    *   **Control rod channels:** Vertical tubes scattered among the fuel channels, distinguished with dark blue or silver material. In the initial state, the control rods are partially inserted.
    *   **Bottom support structure:** Includes metal grating and piping, presenting an industrial-mechanical aesthetic.
    *   **Height:** The core's overall height should be 2-3 times its diameter, emphasizing its vertical character.

2.  **Upper Biological Shield (The Lid / "Elena"):**
    *   **Shape and size:** A huge circular concrete cover plate, its diameter spanning the entire core (about 15-17 meters), thickness about 2-3 meters.
    *   **Cover blocks:** The top surface is tiled with hundreds of square/rectangular metal cover plates (simulating the fuel loading/unloading channel covers), each about 25x25cm.
    *   **Color coding:** The cover plates carry markings in different colors:
        -   Green: normal fuel channels
        -   Yellow: control rod positions
        -   Red: high-power zones
        -   Blue: instrumentation positions
    *   **Numbered labels:** Position numbers in white or yellow shown along the plate edges (e.g. E31, K23), simulating the actual positioning system.
    *   **Material:** The surface should have a rough concrete/metal composite texture, with industrial grime and wear marks.
    *   **Structural details:** Prominent reinforcement rings and support beams around the edges.

3.  **Intermediate structures:**
    *   **Water-steam pipes:** Huge pipes (roughly 1 meter in diameter) extending from the sides of the core, connecting to the separator/steam generators.
    *   **Instrument penetrations:** Fine pipes and cabling between the lid and the core.

4.  **Environment:**
    *   **Lighting:** A dark industrial hall with only faint emergency lighting (cold white or yellow).
    *   **Atmospheric effects:** Light steam haze and dust particles floating in the air.
    *   **Background:** Dimly visible outlines of concrete walls, piping, and industrial equipment.

**Animation script (staged):**
Write a main function named `startSequence()` that uses a GSAP Timeline to execute the following phases in order:

*   **Phase 0: Initial State** *(lasts 2-3 seconds)*
    *   The camera slowly orbits the core, showing the overall structure.
    *   The core emits a faint blue-green glow (low-power operating state).
    *   The color markings on the lid are clearly visible.
    *   Ambient sound: a low hum, the sound of flowing coolant water (optional).

*   **Phase 1: Criticality Runaway** *(lasts 5-8 seconds)*
    *   **Control rod action:** All control rods (silver cylinders) rapidly insert downward into the fuel channels (triggered by the AZ-5 button), insertion speed about 0.4m/s.
    *   **Positive scram (tip) effect begins:**
        -   Fuel channels in the **bottom region** of the core begin shifting from blue to orange-red.
        -   The color spreads gradually from bottom to top, forming a clear temperature gradient.
        -   The graphite blocks begin emitting a dull red thermal glow.
    *   **Power spike:**
        -   Core brightness increases sharply; the Cherenkov blue glow is overwhelmed by red light.
        -   The fuel channel surfaces begin exhibiting heat-distortion effects (using a refraction shader).
        -   The red-marked zones on the lid begin flashing warnings.
    *   **Camera shake:** Slight jitter begins, gradually intensifying.
    *   **Sound cues:** Creaking of metal under stress, the shriek of steam pipes.

*   **Phase 2: First Steam Explosion** *(lasts 2-3 seconds)*
    *   **Lid rupture:**
        -   A few cover blocks at the center of the lid are pushed up first by steam pressure (simulating the rupture of fuel channel covers).
        -   Then the entire Elena lid (2000 tonnes) is violently thrown upward, to a height of about 10-15 meters.
        -   The lid tumbles about 30-45 degrees in the air, finally coming to rest upright beside the core or crashing back down onto it.
    *   **Particle effects:**
        -   A column of white high-pressure steam (1000°C+) jetting from the rupture point.
        -   Metal fragments, concrete chunks, and graphite debris ejected upward.
        -   Fiery-red fuel fragments flung into the air, trailing glowing streaks.
    *   **Lighting changes:**
        -   The blue glow is instantly replaced by blinding white/yellow light.
        -   Bloom strength increases sharply to 3-5x.
        -   Intense Lens Flare effects are produced.
    *   **Shockwave:**
        -   A ring of semi-transparent shockwave expands outward from the core.
        -   The surrounding haze is blown away.
    *   **Camera:** Violent shaking, simulating the shockwave impact.

*   **Phase 3: Second Explosion & Air Contact (Graphite Fire Ignition)** *(lasts 3-5 seconds)*
    *   **Graphite burning:** The exposed graphite moderator ignites on contact with air.
        -   Graphite blocks shift from dull red to bright orange-yellow flames.
        -   Large volumes of black/gray dense smoke are produced (radioactive graphite dust).
    *   **Second explosion:** Hydrogen and carbon monoxide deflagrate, producing an even larger fireball.
    *   **Flame particles:** Flames and sparks erupting upward from the core, reaching heights of 30-50 meters.
    *   **Radiation light effects:**
        -   Add an eerie ultraviolet-like halo.
        -   An "ionization glow" effect appears in the air (pale purple / blue-white).

*   **Phase 4: Core Meltdown** *(lasts 8-12 seconds)*
    *   **Structural collapse:**
        -   Fuel channels, having lost support, begin tilting toward the center, bending, and breaking.
        -   Graphite blocks collapse one by one to the bottom of the core.
    *   **Temperature shader effect:**
        -   Use a custom `ShaderMaterial` with `uniform float temperature` (range 600K - 3000K).
        -   **Color mapping:**
            -   600K-900K: dull red
            -   900K-1200K: bright red
            -   1200K-1700K: orange-yellow
            -   1700K-2500K: yellow-white
            -   2500K+: incandescent white
        -   The material gradually transitions from solid metal to a viscous molten-lava appearance.
    *   **Melt flow:**
        -   The molten mixture of fuel, graphite, and metal (the "Corium" elephant's-foot material) flows downward.
        -   Use UV-scrolling animation to simulate the flowing lava texture.
        -   The melt pools at the bottom, forming a glowing "lava pool".
    *   **Heat distortion:** Visual distortion (refraction effect) from intense rising hot air currents.
    *   **Glow intensity:** The Bloom effect stays at high intensity, simulating the self-luminescence of nuclear fuel.
    *   **Ember effect:** Glowing motes drifting up from the surface of the melt.

*   **Phase 5: Aftermath** *(lasts 5-8 seconds)*
    *   The camera slowly pulls back, revealing the full view of the destroyed core.
    *   Flames and dense smoke continue billowing upward.
    *   The melt pool at the bottom keeps emitting a terrifying orange-red glow.
    *   The surroundings are shrouded in a haze of radioactive dust.
    *   Gradually fade to a black screen, displaying a warning message (optional).

**Code output requirements:**
Provide a **complete, runnable single HTML file**. The file should include:

1.  **HTML structure and CDN imports:**
    *   Three.js (latest version)
    *   GSAP (GreenSock) animation library
    *   (Optional) dat.GUI for debugging parameters

2.  **CSS styling:**
    *   Full-screen black background (`body { margin: 0; background: #000; }`)
    *   Canvas fills the entire viewport
    *   Clean UI control button styling

3.  **JavaScript core logic:**

    **Initialization:**
    *   Scene, camera (perspective camera, FOV 60-75), and WebGL renderer initialization
    *   Enable antialiasing and shadows (if performance allows)
    *   Set up EffectComposer and UnrealBloomPass (Bloom strength adjustable)

    **Geometry generation:**
    *   `createReactorCore()`: generates the 18x18 fuel channel array (CylinderGeometry)
    *   `createGraphiteBlocks()`: fills graphite blocks between channels (BoxGeometry)
    *   `createControlRods()`: generates the control rods (slender CylinderGeometry)
    *   `createBiologicalShield()`: generates the Elena lid and the colored marking grid on its surface
    *   `createSteamPipes()`: adds side piping decoration (optional)

    **Material system:**
    *   **Custom ShaderMaterial** for the fuel channels, which must include:
        -   `uniform float uTime`: time variable
        -   `uniform float uTemperature`: temperature variable (600-3000)
        -   `uniform vec3 uBaseColor`: base color
        -   Fragment Shader implementing temperature-to-color mapping (black-body radiation)
        -   Optional: add a noise function to simulate surface texture and thermal perturbation
    *   **Standard materials** for graphite, concrete, metal, etc.

    **Particle systems:**
    *   Steam particles: white, semi-transparent, upward motion, with diffusion
    *   Debris particles: various colors, parabolic motion, with rotation
    *   Flame particles: orange-yellow, glowing, turbulent motion
    *   Smoke particles: black-gray, varying opacity, slowly rising

    **Animation timeline:**
    *   `startSequence()`: the main function, using a GSAP Timeline to chain all phases
    *   Each phase uses `timeline.to()` to control various properties:
        -   The materials' `uTemperature`, `emissive`, `opacity`
        -   Objects' `position`, `rotation`, `scale`
        -   The camera's `position`, shake effects
        -   Bloom's `strength`, `threshold`, `radius`

    **User interface:**
    *   A simple HTML button: `<button id="startBtn">Start Simulation</button>`
    *   (Optional) a dat.GUI panel, adjustable for:
        -   Bloom strength
        -   Animation speed
        -   Camera view
        -   Temperature range

    **Performance optimization:**
    *   Use InstancedMesh to render the repeated fuel channels and graphite blocks
    *   Keep particle counts reasonable (1000-5000 recommended)
    *   Use LOD (Level of Detail) techniques (optional)

4.  **Comment requirements:**
    *   Key parts of the code must have English comments
    *   A comment must precede the start of each animation phase
    *   Shader code must be commented in detail

**Visual tone:**
*   Dark, high-contrast, dangerous industrial-disaster style
*   Color scheme: cement-gray background + blue (normal) → red-orange (runaway) → white-yellow (explosion)
*   Emphasize vertical drama (low-angle, looking-up shots)
*   Cinematic lighting and post-processing

**Physical realism considerations:**
*   Explosion trajectories follow gravity (parabolic motion)
*   The lid conveys a sense of mass (slow acceleration and rotation)
*   Thermal radiation follows black-body radiation color temperatures
*   Particle velocities and densities must be plausible
