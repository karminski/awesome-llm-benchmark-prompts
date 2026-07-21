Heaven's Door
---------------
(CC-BY-NC-SA 4.0 by karminski-牙医)




**Role positioning**: You are a world-class WebGL expert and senior Three.js developer, proficient in 3D graphics, custom shader authoring, particle physics simulation, and front-end performance optimization.

**Task objective**: Please write a **complete, unabridged, directly runnable single-file HTML** (containing HTML, CSS, and JavaScript) that uses Three.js to implement an artistically striking, visually powerful "Heaven's Door" volumetric fog and dynamic lighting demo.

---

### I. Scene and Visual Design (Visual Requirements)
1. **Base scene**:
   * At the center of the scene is a **gray-white staircase (Staircase)** tilted upward at a specific angle.
   * At the top of the staircase stands a minimalist **gray-white doorframe (Doorframe)**.
   * The overall color tone leans toward cool gray, minimalism, and a sense of the sacred; use ambient light (AmbientLight) and directional light (DirectionalLight) to sculpt delicate light-dark transitions and a sense of volume on the steps and doorframe.
2. **Volumetric Fog and vortex effect**:
   * The fog is milky white, initially **gradually pouring out** from inside the doorframe, then **spreading and diffusing** downward along the staircase.
   * Near the doorframe and during its descent, the fog must produce a **vortex-like (Vortex) flow effect**, expressing air turbulence and a lively, ethereal quality — not simple translation or uniform scaling.
3. **Dynamic Tyndall light (God Rays / Volumetric Light)**:
   * A beam of sacred **intense white light** shines out from deep within the doorframe, illuminating the staircase below.
   * The light must exhibit a clear **Tyndall effect** (i.e., light beams visible in the air).
   * **Deep coupling between the light and the fog**: the beam's brightness and shape must dynamically flicker in intensity, be occluded, and scatter in response to the fog's flow and density changes.

---

### II. Technical Implementation Constraints (Technical & Shader Constraints)
1. **Core technical path selection**:
   * Given that Three.js's native `THREE.Fog` is merely distance fog and has no volumetric quality, you must use one of the following two advanced approaches:
     * **Option A (recommended, higher visual ceiling)**: a custom shader material (`ShaderMaterial`) or post-processing Pass, using **Raymarching** and **3D noise (such as 3D Simplex Noise or Curl Noise)** to compute volumetric fog and light-beam scattering and occlusion in real time in the fragment shader.
     * **Option B (performance compromise)**: design a high-density **GPU/CPU particle system (THREE.Points)** combined with custom vertex/fragment shaders, simulating a vortex force field on the particles via mathematical formulas, and using post-processing (`EffectComposer` with `GodRaysShader`, etc.) to achieve the visual fusion of light beams and particle fog.
   * It is **forbidden** to use non-native, large third-party physics engines; all physical flow and light-scattering formulas must be hand-written in native JS or GLSL.
2. **Strict prohibition of "hallucinated APIs"**:
   * It is forbidden to call any fictitious APIs that do not exist in Three.js (for example, imaginary `THREE.VolumetricLight` or `THREE.VortexFog` classes). All complex volumetric computation and fluid motion must be solved via custom Shaders or hand-written mathematical formulas.
3. **Performance and rendering optimization**:
   * Volumetric computation is extremely expensive; be sure to include performance-optimization considerations in the code (such as limiting the maximum number of Raymarching sample steps, using half-resolution rendering, or temporal dithering/blending techniques, and explain your optimization strategy in the code comments).
4. **Responsiveness and interaction**:
   * The page must support window resizing (Window Resize), and include `OrbitControls` so the user can rotate and zoom the view to observe the volumetric fog and the beam's penetration effect from different angles.

---

### III. Output Specifications (Output Formats)
1. **Completeness requirements**:
   * You must provide **100% complete** code; it is **strictly forbidden** to use placeholders such as `// omitted here...`, `// same as above...`, or `// implement the logic yourself...`. All Shader code (GLSL) must be written out in full.
2. **Code structure**:
   * Consolidate all code into a single `.html` file.
   * You must use public, stable CDNs to import Three.js, OrbitControls, and any necessary post-processing libraries (if used).
   * The code should include comments at key steps, explaining the volumetric fog algorithm you implemented (such as the use of noise, the integration principle of raymarching, etc.).

Please begin your creation and demonstrate your coding prowess as a top-tier WebGL expert!
