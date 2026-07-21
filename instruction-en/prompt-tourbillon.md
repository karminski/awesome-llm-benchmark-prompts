Tourbillon Movement 3D Demo Prompt
------------------------

(CC-BY-NC-SA 4.0 by karminski-牙医)


**Role:** You are a creative coding expert with deep mastery of Three.js, WebGL shaders, and the mechanical principles of precision horology.

**Goal:** Create a browser-based, single-file HTML interactive 3D scene that builds, entirely from code (Procedural Generation), a mechanically accurate demonstration of a Swiss tourbillon.

**Key modification notes:**
1.  **Visual style:** Must use a bright background with a **light blue to white gradient**, creating a modern, fresh laboratory feel rather than the traditional dark background.
2.  **Mechanical principle:** Must follow true **planetary gear** motion logic: the central seconds wheel is fixed and stationary, the tourbillon cage rotates around it, driving the rotation of the escape wheel mounted on the cage.

---

## 1. Tech Stack Requirements

*   **Core library:** Three.js (r150+ via CDN)
*   **Interaction:** OrbitControls (with smooth damping)
*   **Animation:** Use `requestAnimationFrame` with `THREE.Clock`; GSAP optional for special transitions.
*   **Post-processing:** `EffectComposer`, `RenderPass`, `UnrealBloomPass` (must be fine-tuned for the bright background).
*   **Modeling approach:** **Purely code-generated** (using combinations of ExtrudeGeometry, LatheGeometry, TorusGeometry, etc.); loading external models is strictly forbidden.

---

## 2. Scene and Environment Setup (Adapted for a Light Palette)

*   **Background:** 
    *   Create a radial gradient texture (CanvasTexture).
    *   Center color: pure white (`#FFFFFF`).
    *   Edge color: clear ice blue (`#E0F7FA` or `#B2EBF2`).
    *   Effect: as if standing inside a bright watchmaking atelier.
*   **Lighting system:**
    *   **Important principle:** Because the background is bright white/light blue, lighting intensity must be restrained. ACES tone mapping compresses highlights; overly strong lighting will blow out metal surfaces and lose detail.
    *   **Tone mapping exposure:** set `renderer.toneMappingExposure` to **0.85**, slightly below the default, to avoid overall overexposure against the bright background.
    *   **Ambient light:** intensity **0.4**, bluish-white color (`#F0F8FF`), used only to prevent pitch-black shadows; should not be too high.
    *   **Key light (Directional):** positioned upper right, intensity **1.0~1.2**, producing crisp but not harsh highlights, with shadows enabled (`castShadow: true`).
    *   **Fill light (Point):** positioned rear left, cool-toned, intensity **0.4~0.6**, used to outline metal edges.
    *   **Shadow configuration:** shadows should not be pure black but a semi-transparent deep gray-blue; set `shadowMap` resolution to 2048.
*   **Environment map:**
    *   **Crucial:** against a white background, highly reflective metals (chrome/steel) become invisible if they only reflect white.
    *   Use `THREE.PMREMGenerator` to generate a virtual high-contrast environment map (containing blocks of dark gray and blue) so that metal parts reflect clear dark contours.
    *   Keep the material's **`envMapIntensity`** between **0.7~1.0**; do not exceed 1.0, to avoid reflections so bright the metal surfaces become unreadable.

---

## 3. Detailed Mechanical Component Specifications

Build the model according to the following hierarchy, ensuring correct parent-child relationships for animation:

### 3.1 The Fixed Fourth Wheel - [Stationary Reference]
*   **Position:** absolute center of the scene (0,0,0).
*   **State:** **absolutely stationary**, does not rotate with any component.
*   **Appearance:** a fairly large brass gear (about 60 teeth).
*   **Material:** brushed brass (`color: 0xD4AF37`, `roughness: 0.3`, `metalness: 0.9`).
*   **Principle:** it is the "sun gear"; the tourbillon cage revolves around it.

### 3.2 Tourbillon Cage - [Revolving Body]
*   **Hierarchy:** a child of the scene root node.
*   **Appearance:** a classic Breguet-style three-arm or two-arm cage with elegant lines.
*   **Material:** polished steel (`color: 0xAAAAAA`, `metalness: 1.0`, `roughness: 0.1`), with a slight blue sheen.
*   **Animation:** rotates uniformly around the Y axis (revolution), with a period of 60 seconds per revolution.
*   **Contained components:** the balance wheel, pallet fork, and escape wheel are all mounted inside this cage.

### 3.3 Escape Wheel Assembly - [Planetary Motion]
*   **Hierarchy:** a child of the **tourbillon cage** (revolves with the cage).
*   **Composition:**
    1.  **Escape wheel body:** a distinctive star-shaped gear (about 15 teeth), mounted on the upper part of the arbor.
    2.  **Escape Pinion:** mounted on the lower part of the arbor, meshing with the **fixed fourth wheel from 3.1**.
*   **Position:** off-center, on one side of the cage.
*   **Material:** bright golden brass.
*   **Animation logic (core):** 
    *   No need to hand-write a rotation animation.
    *   Because it revolves with the cage as a child object, and its pinion meshes with the stationary central wheel, it should visually exhibit the self-rotation of a planetary gear.
    *   In code this can be simulated in simplified form: `rotation.y += time * speed`, but the speed must be much faster than the cage's.

### 3.4 Pallet Fork
*   **Hierarchy:** a child of the **tourbillon cage**.
*   **Position:** between the escape wheel and the balance wheel.
*   **Animation:** performs a rapid reciprocating oscillation (Tick-Tock) at a frequency of about 4Hz.

### 3.5 Balance Wheel & Hairspring
*   **Hierarchy:** a child of the **tourbillon cage**.
*   **Position:** occupies most of the space in the cage.
*   **Balance wheel:** a golden ring with timing screws as counterweights and spokes at the center.
*   **Hairspring:** 
    *   An **extremely fine blued-steel spiral** (`color: 0x2244CC`).
    *   Build an Archimedean spiral using `THREE.Line` or an extremely thin `TubeGeometry`.
*   **Animation:** 
    *   Performs simple harmonic oscillation (sinusoidal motion): `rotation = maxAngle * sin(time * frequency)`.
    *   The hairspring should contract and expand with the balance wheel's rotation in a "breathing" effect (implemented via scaling).

### 3.6 Jewels (Jewel Bearings)
*   **Material:** ruby (`color: 0xE91E63`).
*   **Properties:** high transparency (`transmission: 0.8`), high index of refraction, but against the bright background add a touch of red `emissive` glow to ensure they remain vividly visible.
*   **Position:** at every arbor connection point.

---

## 4. Post-processing and Effects

*   **Bloom:**
    *   **Threshold:** must be set between **0.90 - 0.95**.
    *   **Reason:** the background is white; too low a threshold would cause Bloom on both the background and large metal highlight areas, washing the image out into a white blur and obscuring object detail. The threshold must be high enough that only the very brightest specular points produce a faint glow.
    *   **Strength:** **0.1 ~ 0.2** (very subtle, serving only to soften the image).
    *   **Radius:** 0.4.

---

## 5. Output Code Requirements

1.  **HTML structure:** provide a complete index.html file, including a CSS reset (margin: 0, overflow: hidden).
2.  **Code logic:**
    *   Initialize the scene, camera, and renderer.
    *   Component construction functions: `createFixedWheel()`, `createCage()`, `createBalance()`, `createEscapement()`.
    *   **Animation loop:** compute `deltaTime` centrally in `animate()` to ensure all components move in sync and smoothly.
3.  **Comments:** add brief Chinese comments to the key mathematical calculations and material parameters.
4.  **Performance:** keep gear tooth counts reasonably optimized; use `BufferGeometry`.

Please generate the code, presenting a precise, elegant tourbillon movement that gleams against the light background.
