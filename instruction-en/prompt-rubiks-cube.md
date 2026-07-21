
5x5 Rubik's Cube
----------------------------
(CC-BY-NC-SA 4.0 by karminski-牙医)


**Role:**
You are a senior front-end engineer with deep expertise in Three.js material management, WebGL, and Rubik's Cube algorithms.

**Task Update:**
Please rewrite the previous 5x5 Rubik's Cube demo code. We need to fix the model's material logic and ensure the correctness of the "solve" feature.

**Core Fixes (must be strictly followed):**

1.  **Material fix: internal faces must be gray**
    *   When building each cubie of the 5x5x5 cube, do **not** simply paint the entire cubie one color.
    *   You must use a **material array**: `const materials = [right, left, top, bottom, front, back]`.
    *   **Position-based logic:**
        *   Iterate x, y, z from -2 to 2 (corresponding to order 5).
        *   **Only when a cubie lies on the cube's surface** does the corresponding outward-facing face get its standard color.
        *   **All internal faces** (e.g. the right face at x=0, or any occluded middle layer) must have their material set to **dark gray** or a black plastic look.
        *   *Example:* only when `x == 2` is the cubie's "right face" material red; otherwise it is gray.

2.  **State fix: initial state must be "six solid-colored faces"**
    *   **Initial state:** on startup, the cube must be in the **perfectly solved state** (top white, bottom yellow, front green, back blue, left orange, right red).
    *   **Left-side logic diagram colors:** the dot colors on the left-side Canvas must correspond one-to-one with the cube's initial state on the right (e.g. all dots belonging to the top layer initialize to white).

3.  **Solve logic: stack-based inverse operations**
    *   **Scramble:** starting from the "perfectly solved state", perform 20-30 random rotations. After each move, push that move (e.g. "R", "U'", "F2") onto an array `historyStack`.
    *   **Solve:** when Solve is clicked, do not attempt to compute a solving algorithm. Simply pop moves off `historyStack`, invert them (e.g. "R" becomes "R'"), and play the animation. This guarantees the cube **returns 100% to its initial six-solid-color state**.

**Other features to keep unchanged:**

*   **Left-side logic transition diagram (Permutation Graph):** keep the previous Canvas orbit design.
    *   When the cube is in the "solid-color solved state", the dots on the left-side chart should look fairly ordered (e.g. colors clustered together).
    *   When the cube is scrambled, the colored dots on the left-side chart mix thoroughly along the orbits.
*   **UI:** the "Random Scramble" and "Demo Solve" buttons in the bottom-right corner.
*   **Animation:** use Tween to achieve smooth rotations.

**Implementation hints:**

*   Please use `THREE.BoxGeometry` together with an array of 6 `THREE.MeshBasicMaterial` to create each cubie.
*   Color constants:
    *   `Right (x=2)`: Red (`0xff0000`)
    *   `Left (x=-2)`: Orange (`0xff8800`)
    *   `Top (y=2)`: White (`0xffffff`)
    *   `Bottom (y=-2)`: Yellow (`0xffff00`)
    *   `Front (z=2)`: Green (`0x00ff00`)
    *   `Back (z=-2)`: Blue (`0x0000ff`)
    *   `Internal`: Dark Gray (`0x222222`)

Please generate the complete, corrected single-file HTML/JS code.
