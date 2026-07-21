Volcanic Eruption
-------
(CC-BY-NC-SA 4.0 by karminski-牙医)


# Role
You are a senior WebGL / Three.js graphics engineer and VFX director, skilled in: procedural terrain, GPU particle state simulation, volumetric-feeling smoke, and "cinematic" lighting (IBL + dynamic key light + controllable post-processing + **per-phase exposure budgeting**, avoiding underexposed dead blacks and Bloom/exposure washing out the sky).

# Task Objective
Please output **a single-file `index.html` that can be opened directly by double-clicking (`<script type="module">`)**, using **Three.js (pin `three` and `examples/jsm` to the same version, e.g. r160+; state the exact version number and CDN import paths in a comment at the top of the file)** to implement:
a looping demo of **"crater particle bed + mushroom-cloud smoke build-up → explosive ejection → sustained lava overflow from the crater (not ballistic-only) → cooling and extinction"** (recommended **20s cycle**, seamlessly returning to phase one at the end or fading into a reset).

Must include: **OrbitControls**, **EffectComposer + Bloom (UnrealBloomPass or an equivalent BloomPass)**, **resize handling**, and a **reasonable performance-degradation strategy** (limit `devicePixelRatio`, optional half-resolution Bloom).

---

# 0) Hard Engineering Constraints (must be followed)
1. **Single-file runnable**: all logic inside one HTML file; use ES module `import`; mixing in the legacy global `THREE` UMD is forbidden.
2. **Version lock**: `three` and `examples/jsm/*` must be the same version; provide copy-pasteable import examples (`OrbitControls`, `EffectComposer`, `RenderPass`, `UnrealBloomPass`, `RoomEnvironment` or an equivalent environment-map solution).
3. **Camera and shake**: create a `THREE.Group` (e.g. `cameraRig`) and attach the `PerspectiveCamera` as a child; bind `OrbitControls` to `camera`. Eruption shake is **applied only to `cameraRig.position` (a tiny amount of rotation if necessary)**, with `controls.update()` every frame. Do not wrap the OrbitControls target inside a shaking parent node, which would break interaction.
4. **Terrain and particles share "one and the same height field"**: maintain a single `TERRAIN_GLSL` string in JS (containing FBM / domain warp with identical parameters), injected into both:
   - the `onBeforeCompile` of `MeshStandardMaterial` (modifying vertex height)
   - the particle system's compute/vertex shaders (sampling height, ground collision, flow along slopes)
   You must clearly specify the **world-space xz alignment rule** (the terrain mesh's transform must be consistent), and implement the **slope vector** via finite differences (debris flow slides tangentially along the slope + friction damping); "collision in words only" is forbidden.
5. **Terrain normals and built-in material integration**: when modifying heights on `MeshStandardMaterial`, the normals must stay consistent with the current Three version's vertex pipeline, so that `transformedNormal` / the lighting chain remains intact; avoid "plastic faceting" or structural errors where the shader fails to link (the implementation approach is the author's choice, judged by whether it compiles and the normals are trustworthy).
6. **CPU vs shader division of labor**: if terrain heights are initialized on the CPU side (e.g. particle spawn heights), they must use **the exact same math** as `TERRAIN_GLSL`; do not directly reuse GLSL built-in function names (such as `fract`) in JavaScript, to avoid runtime errors.
7. **Three.js modern pipeline and the "world coordinates" trap (self-check strongly recommended)**: when using world coordinates in the **fragment** stage of `onBeforeCompile` on `MeshStandardMaterial` / `MeshPhysicalMaterial` (pseudo secondary bounce, distance-to-hotspot `length(world.xz)`, etc.), you **must not assume** `vWorldPosition` exists. In the typical `meshphysical` shader of **r150+**, `varying vec3 vWorldPosition` **is often only declared and assigned when a few macros such as `USE_TRANSMISSION` are enabled**; referencing it directly without transmission causes **Fragment shader not compiled / VALIDATE_STATUS false**. The reliable approach: add a custom `varying` in the vertex stage (e.g. `vVcWorldPosition`), write it from the **displaced** vertex using `(modelMatrix * vec4(p, 1.0)).xyz`, and only read that varying in the fragment stage.
8. **Slicing substrings out of `TERRAIN_GLSL` for the sky and other ShaderMaterials**: if you use `String.prototype.split` to strip out the terrain functions, splitting on the substring `'getTerrainHeight'` is **forbidden** — it would cut through `float getTerrainHeight`, leaving an illegal orphan `float` in the GLSL and likewise causing fragment compilation failure. Use a complete declaration boundary (such as `'float getTerrainHeight'`), or maintain a separate `NOISE_GLSL` constant containing only `vc_hash` / `vc_noise` / `vc_fbm`, avoiding fragile string slicing.
9. **JavaScript template literals and shader source**: whenever an entire `vertexShader` / `fragmentShader` / `TERRAIN_GLSL` is wrapped in backticks `` ` ``, **no unescaped backticks may appear inside GLSL comments**, otherwise the browser will report `Unexpected identifier` (the template literal gets closed prematurely). When you need to annotate code fragments, use single quotes or plain-text descriptions, or escape the backticks.

---

# 1) "Very Good Global Illumination" (browser-feasible definition: must be implemented as specified here)
> Note: do not promise true path-traced GI in WebGL. Implement **high-quality IBL + dynamic key light + contact-shadow darkening + a lava secondary-illumination approximation**, and strictly follow this section's **exposure budget** and **per-phase brightness curves**, avoiding an overly dark build-up phase, a fully blown-out eruption phase, or a "washed-out sky".

## 1.0) Exposure Budget (brightness layering, must be discernible)
Treat final brightness as a **finite budget** to allocate; it is **forbidden** to "look bright" merely by maxing out `toneMappingExposure` or Bloom `strength`.

| Layer          | Responsibility                                               | Constraint                                                                                                                     |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| Sky / sky dome | Blue or dusk gradient + readable cloud-shape texture         | During eruption and overflow phases, **hue and mid-to-high-frequency detail** must be preserved; must not be bleached by Bloom/exposure into a textureless bright gray/pure white sheet |
| Terrain shadows | Moonlight + IBL + very weak hemisphere/ambient fill          | Near rock walls and distant slopes **must not smear into pure black**; shadowed areas must retain discernible silhouettes (lifted via IBL + weak `HemisphereLight` ground floor, not brute-force exposure) |
| Lava and crater rim | Primary heat source: point light + high-temperature material zones + **localized** Bloom | High-temperature overflow should be **concentrated at the crater rim, overflow channels, and ballistic trajectories**; do not turn the entire dome into "one big lamp" |
| Smoke          | Light-absorbing, scattering, may be slightly warmed at lava-lit edges | Additive/high-alpha layers during eruption must be restrained, to avoid lifting the frame's average brightness through the ceiling |

## 1.1) Required Technical Combination (all of these, none optional)
1. **IBL / environment map**: use `PMREMGenerator` to generate `scene.environment` from `RoomEnvironment` (or an equivalent HDR), feeding diffuse and specular reflection of `MeshStandardMaterial`; the IBL intensity must suffice to keep the terrain readable during **non-eruption phases** (if too dark, **first** slightly raise the `scene.environment` intensity or PMREM contribution — **do not** double the exposure first).
2. **Moonlight key light**: a cool-toned `DirectionalLight` (low angle, **restrained intensity**), paired with a **very weak** `HemisphereLight` or `AmbientLight`: **sky hemisphere cool-biased, ground hemisphere very weak warm-biased or neutral**, lifting shadow floors but **not** flattening the moonlit shadow shapes at the crater rim.
3. **Dynamic crater point light**: an orange-red `PointLight`, which **must** have reasonable `distance` / `decay` set (or the version-equivalent attenuation parameters), so energy falls mainly on the crater rim and near field; intensity is bound to `phaseT`; a non-attenuating point light washing half the mountain to uniform brightness is **forbidden**.
4. **Tone mapping and color space**: `renderer.outputColorSpace = SRGBColorSpace` (or the current version's equivalent API); use `ACESFilmicToneMapping` or the **safe default** `toneMapping` consistent with your Three version, avoiding linear output that makes everything gray or blown out.

## 1.2) Per-Phase Brightness Knobs (must be driven by JS via smooth curves on `phaseT`; hard step changes are forbidden)
The following are **suggested starting ranges** (fine-tune per camera scale, but the relative relationships must hold: at the **eruption peak it is forbidden** to simultaneously push exposure, point light, and Bloom all to their in-cycle maxima).

- **`renderer.toneMappingExposure`**: build-up **0.70–0.90**; eruption peak only **+0.03 ~ +0.10** relative to build-up (a +0.25-magnitude surge is forbidden); overflow **slightly above build-up or equal**; cooling **eases down** to **0.65–0.88**; at the cycle's seam, **flashing to a pure-black frame is forbidden**.
- **`UnrealBloomPass` (or equivalent)**:
  - `strength`: build-up **0.25–0.55**; eruption peak **≤ 1.1**; if the sky dome still smears white, **first** lower `strength` / tighten `radius`, **second** slightly raise `threshold`, and only **last** nudge exposure.
  - `threshold`: build-up **0.82–0.98**; may drop briefly to **0.65–0.88** during eruption; staying below **0.55** for extended periods, causing full-screen Bloom smear, is **forbidden**.
  - `radius`: **0.35–0.55** (may briefly tick up at the eruption peak); too large easily blurs the clouds and sky into one blob.
- **`PointLight.intensity`**: build-up roughly **8%–22%** of the eruption peak; the eruption peak is the cycle's thermal apex, but even combined with the slight exposure lift + medium Bloom, the crater-rim highlights must still show discernible layering; cooling decays toward **0** using `smoothstep` or exponential falloff.

**Same-frame constraint**: the **triple-stacked apex** of "`toneMappingExposure` at cycle max + `PointLight.intensity` at cycle max + Bloom `strength` at cycle max" is **forbidden**. The eruption's impact should come mainly from **particle density, ejection, camera shake, and localized highlight Bloom**, not from doubling global exposure.

## 1.3) Balancing Lighting and Post-Processing (avoid washing out the sky)
Point light, environment/hemisphere, exposure, and Bloom **stack linearly**; full-screen Bloom raises mid-to-high frequencies, so the sky may over-brighten even without direct point-light illumination. Principle: **better a slightly darker sky and clouds with full layering than one so bright that hue and cloud texture become indiscernible**; Bloom should make **only genuinely high-temperature pixels** bleed prominently. If using half-resolution Bloom, watch for cloud-layer aliasing; optionally suppress the sky material slightly in the pre-Bloom input, or composite the sky with a separate weight (implementation is free; acceptance criteria are what counts).

## 1.4) Pseudo Secondary Bounce (strongly recommended)
- Terrain `onBeforeCompile`: an extremely restrained **rim / bounce tint** (mixing a warm color into shadows facing the hotspot, **suggested mix weight peak roughly 0.04–0.12**), avoiding turning entire rock walls red.
- Smoke: additive/translucent + **depth-aware** thickness; bright edges may be warmed by lava, but the whole must not lift through the sky dome.

## 1.6) Anti-Patterns (explicitly forbidden)
- Using an excessive `toneMappingExposure` to compensate for underexposed IBL/fill light (it washes the sky bright as a side effect).
- Keeping Bloom `threshold` low for prolonged periods during eruption, smearing smoke, clouds, and distant scenery everywhere.
- Disabling or severely weakening the environment/hemisphere lights, relying solely on lava and Bloom to "light" the scene (easily makes the build-up phase too dark).

---

# 1.5) Interaction of Sky, Clouds, and the Smoke Column (recommended as a visual target)
- **Sky-dome base tone**: there must be a **recognizable blue or dusk gradient** (gradient or dome, either is fine), forming a **cool-warm contrast** against the dark terrain and warm lava; avoid a default pure-black background; the sky dome must maintain a readable hue band **through the full cycle** — pushing the gradient toward a near-**#FFFFFF** flat field during eruption is forbidden.
- **High-altitude clouds**: soft, large-scale cloud shapes (noise or textures), drifting slowly; the **maximum brightness** of the clouds' undersides at the eruption peak should still stay one stop below the brightest crater-rim lava, so cloud texture remains readable; if Bloom still eats the clouds, **prefer** slightly lowering the cloud material's albedo/emissive or the cloud pass's contribution to Bloom, rather than raising exposure further.
- **Smoke column punching through clouds**: during the build-up—eruption—overflow phases, as smoke-column density and upward momentum grow, one should be able to read the **clouds being pushed open, blasted apart, or locally thinned** (transparency, sampling offsets, weights synced to `phaseT`, etc.); this effect's intensity curve must be **in phase** with `cloudDissolveWeight` / exposure and Bloom curves in **# 3**, so the clouds and smoke don't each do their own thing. After cooling, the cloud-punching effect **subsides in sync with the smoke's momentum**.
- **Protecting the sky during eruption**: cloud-punching may locally whiten and turn transparent, but **banded or clumped cloud structure must remain discernible**; the entire sky dome and clouds melting into one structureless bright blob is forbidden (consistent with **acceptance criterion 9**).

---

# 2) Particle System Architecture (must be layered; a single particle system doing everything is forbidden)
At least three layers (each with independent parameters and blend mode), with the total count determined by the **GPGPU texture side length** (commonly all three layers share the same size, so the total particle count is roughly **3 × side²**). Where the target machine allows, a **larger side length** may be used (e.g. on the order of `128√10`) to substantially thicken the sampling density of **lava and smoke volumes**; low-end configs must support **stepping the side length down a full tier** while retaining the layered architecture and physical consistency. The delivery notes must explain the **performance vs side-length trade-off**.

## A) Crater "Particle Bed / Debris Layer"
- Pre-place a layer of **high-density stationary or weakly jittering particles** in the crater region (representing a loose layer of scoria/pumice).
- **Phase two, eruption**: apply a **radial + tangential impulse** to this layer's particles so the crater-rim material is **torn away and ejected** (not merely spawning new particles from a point source), creating the awe of "whole chunks of the rim being carried off".
- **Phase three**: the particle bed continuously provides the **overflow-lava source term**: emit "lava-stream particles" sampled from a ring/multiple points around the crater (not pure parabolic fireworks), with initial velocity outward along the slope, plus viscosity and separation forces (a simplification is fine: hash-neighborhood repulsion / low-density diffusion), forming a glowing network on the slope faces.
- **Motion believability**: overflow and slope sliding should exhibit **gravity and downhill tendency**; avoid physically unconstrained "decorative jitter" that oscillates horizontally, which would make the lava appear to **flow backwards or twitch spasmodically** on the slope in a counter-intuitive way.

## B) Explosive Ejection (ballistics-dominated)
- Short-window, high-density burst; trailing sparks optional.

## C) Smoke / Volcanic Ash
Under windless conditions, phase one must present:
1. **A smoke column rising slowly from the crater rim/vent** (rise speed increasing over time; particle radius/alpha/scattering intensity increasing over time).
2. **Mushroom-cloud structure**: once the smoke column rises past a threshold height, a **cap-like spreading** appears (horizontal radial velocity component increases, vertical velocity decreases), forming a "stem + cap" silhouette; layered noise may drive horizontal vortices/curl, but it **must not look like randomly drifting snowflakes**.
3. **Density progression**: from sparse to dense within 0–4s, occluding the moonlight and building pre-eruption tension.
4. **Clumpiness and non-axisymmetry**: the smoke body should suggest **flocculent, clumped** volume rather than a rigid cylinder; the stem may have **radius slowly varying with height and slight swaying**, the cap may have **tangential curl**, while the whole retains a readable mushroom silhouette.
5. **Continuous coupling with global time**: entering the cooling phase, **newly spawned smoke should gradually become scarce, and the rising and spreading forces gradually weaken**; existing smoke clumps' **opacity / saturation slowly decrease**, turning into grayish-white residual smoke; avoid hard cuts at phase boundaries where **emission and force fields drop to zero instantaneously**.

Implementation hints (pick one, but state it clearly):
- Multi-layer billboards / instanced quads + curl-noise offsets; or
- Volume approximation: multiple stacked passes of same-layer particles + soft particles (depth fade)

---

# 3) Global Timeline (must be driven by JS-fed uniforms; shader-only `mod(time)` is forbidden)
Use `phaseT ∈ [0,20)` (computed in JS) to drive the following quantities, applying **smoothstep / easing curves** on `phaseT` to **all** of them (hard cuts at phase boundaries are forbidden), so sky, clouds, smoke, light, and post-processing stay readable on the same timeline:

**Uniforms / parameters that must be bound (at minimum)**
`PointLight.intensity` (and attenuation-related), `renderer.toneMappingExposure`, `UnrealBloomPass.strength` / `threshold` / `radius` (or equivalent), each particle layer's **emission rate** and force-field strength, `cameraRig` shake amplitude, smoke-cap spread coefficient; if the sky dome and cloud-punching are implemented, also bind **`cloudDissolveWeight`** (or equivalent naming: a 0–1 weight modulating clouds being dispersed/thinned/transparency), cloud UV offset or deformation strength.

**Relative rhythm of lighting and post-processing (consistent with # 1.2; here as time phases)**
- **Build-up**: exposure and Bloom at **baseline**; weak point light; low `cloudDissolveWeight`.
- **Eruption**: point light **smoothsteps to its peak** first; Bloom `strength` may rise moderately, `threshold` may **briefly** dip; `toneMappingExposure` **only lifts slightly** (see # 1.2); all three peaking at once is **forbidden**.
- **Overflow**: point light may **ease back slightly** from the eruption peak while keeping the crater-rim heat feel; Bloom stays medium-high but `threshold` should not remain too low for long; exposure slightly below or equal to the eruption peak.
- **Cooling**: point light, Bloom `strength`, and exposure **ease down in sequence or in sync** (not zeroed in a single frame); `cloudDissolveWeight` falls with the smoke's momentum; by cycle's end exposure remains within a readable floor range (see # 1.2), avoiding a black flash when reconnecting to the build-up.

Suggested phases (may be fine-tuned, but the logic must be visible; light/smoke/Bloom within each phase must follow the relative rhythm above):
- **0–4s build-up**: mushroom-cloud smoke grows from small to large; the point light pulses faintly; the particle bed heaves slightly; `toneMappingExposure` / Bloom at baseline; `cloudDissolveWeight` eases up while keeping the sky dome readable.
- **4–6s eruption**: ballistic particles surge; the particle bed is ejected en masse; strong shake; the point light peaks (still obeying # 1.2's no-triple-apex rule); the smoke column and cap expand rapidly; `cloudDissolveWeight` hits its phase peak — cloud-punching is readable but the sky is protected (# 1.5).
- **6–14s overflow and debris flow**: **sustained overflow from the crater** + landed lava sliding along the slope (staying hot for at least several seconds — no instant cooling on touchdown); ballistic components decay; point light/Bloom/exposure follow the overflow-phase curves, avoiding a prolonged "washed sky".
- **14–20s cooling**: **new particles and strong emission taper off progressively** (rather than a sudden "gate slam" on some frame); the temperature field and brightness decline slowly globally; colors shift from bright yellow-red to dark rock; Bloom, point light, and `toneMappingExposure` **smoothly** fade toward near-extinction but **retain a weak readable base glow at the tail**; `cloudDissolveWeight` falls back; **existing smoke keeps drifting, spreading, and fading over several seconds**, ultimately becoming grayish-white residual smoke, then linking to the cycle start or fading into a reset.

## 3.1) Particle Lifetime and "Fluid Feel"
- Each layer's **visible persistence time** should match the camera scale: too short makes lava and smoke look like **flickering sparks/specks**, unable to form **coherent trails and clumped volume**; where performance allows, the smoke and overflow layers should use **relatively long lifetimes or slower age decay**, so viewers can read a full flow trajectory.
- **Lifetime decay must align with the phase curves**: after entering cooling, prefer **slowing the death rate** or **lowering the per-unit-time spawn weight**, so on-screen extinction stays in sync with **the dimming of lighting and emissive colors**, avoiding the mismatch of "the light hasn't gone out yet, but the particles are already all gone".

## 3.2) Scale and Framing ("spectacular")
- Under a point-sprite / billboard approach, use an **appropriate screen-space scale** (coordinated with distance and FOV) and **particle density near the crater rim** so the eruption and smoke column occupy **sufficient bulk** in the frame; merely raising texture resolution while individual particles stay tiny will still look thin from afar.

---

# 4) GPU State Simulation (state the implementation path clearly; avoid fake GPGPU)
Must use **GPUComputationRenderer (or equivalent ping-pong DataTexture)** to maintain particle state texture fields (at minimum: `pos`, `vel`, `life` or `age`, `temp`, `phaseTag`).
The vertex shader handles instanced display.
If performance is insufficient, reducing the particle count is allowed, but **"updating 100k particles in a CPU for-loop every frame" is not**.

---

# 5) Terrain Material (must be PBR-friendly)
The terrain uses `MeshStandardMaterial` + `onBeforeCompile` to inject the height field and roughness variation (smoother along lava paths, optional).
Replacing the terrain with a pure `ShaderMaterial` that cannot receive `scene.environment` IBL is forbidden (unless you fully rewrite PBR yourself, which is not recommended).

---

# 6) Delivery and Comments
- Output the complete `.html`.
- Key sections must be commented: `cameraRig`, the timeline, `TERRAIN_GLSL` reuse, mushroom-cloud parameters, particle-bed impulse, IBL/PMREM initialization, and **the lighting/Bloom/exposure/cloud-punch weight curves bound to `phaseT`**.
- At the end of the file, in **10 lines or fewer**, explain **how to tune parameters** (each line may cover several items; write parameter names in English): (1) smoke-cap height / stem thickness; (2) overflow strength; (3) cooling-phase smoke fade-out and spawn weight; (4) particle lifetime and `age` decay rate; (5) per-particle screen scale and crater-rim density; (6) **the curve of `cloudDissolveWeight` (or equivalent) over `phaseT`**; (7) **`toneMappingExposure` four-phase baseline/peak/end values**; (8) **`PointLight.intensity` and attenuation**; (9) **Bloom `strength` / `threshold` / `radius` with the suggested ranges from # 1.2**; (10) GPGPU texture side length and the low-end down-tiering strategy.

# Acceptance Criteria (model self-check)
1. During 0–4s one can clearly see **the smoke column thickening and rising + cap-like spreading** (mushroom-cloud silhouette); the smoke has **clumps and gentle swaying**, not a rigid thin column.
2. During eruption, the crater rim's **particle bed is carried away** (not just a central point source).
3. During 6–14s one can clearly see **sustained overflow from the crater** (not just explosive fireworks); slope flow **follows gravity and the downhill direction**, with no obvious **backflow or high-frequency spasmodic jitter**.
4. The terrain is lit jointly by **environment + directional light + point light + (optional) warm shadow tint**, no longer a "plastic plane"; during the **build-up phase (0–4s)**, distant slopes and rock-wall shadow areas remain silhouetted — large areas of dead black are **not allowed** (satisfying the # 1.0 exposure budget).
5. OrbitControls works throughout; shake does not break interaction.
6. **14–20s cooling**: one can perceive **new smoke and strong updrafts gradually diminishing, old smoke slowly fading and graying**; there must be no jarring break where **smoke and force fields all stop the instant cooling begins**; at the cycle seam back into build-up, **no full-screen black flash**.
7. Within a single cycle, overflow and smoke have **readable trajectories and sufficiently long lifetimes**; the whole reads closer to **fluid and volume** than sparse noise specks.
8. If the sky dome is implemented: a **blue or dusk sky with high clouds** is visible; while the smoke column is at full strength one can read **clouds being dispersed or locally thinned**; after cooling, cloud and smoke momentum **subside in sync**.
9. Under eruption and overflow highlights, **the sky and clouds still retain layering and hue**; the whole sky must never be bleached by Bloom/exposure into a **detail-free bright blob**; the # 1.2 **triple simultaneous apex of exposure + point light + Bloom** is **forbidden**.
10. **4–6s eruption peak**: the crater rim and ballistics may be extremely bright, but **the sky-dome gradient and cloud texture must remain discernible**; the frame must not sit for long under a "gray-white veil", nor may the crater-rim highlights **clip entirely into a structureless white block** covering most of the screen.
11. **Console self-check**: no `THREE.WebGLProgram` / `Fragment shader is not compiled` / `Unexpected identifier` or similar errors; if the terrain fragment shader needs world coordinates, it must satisfy the custom-`varying` constraint from **0.7** above — it must not assume `vWorldPosition` exists without `USE_TRANSMISSION`.
