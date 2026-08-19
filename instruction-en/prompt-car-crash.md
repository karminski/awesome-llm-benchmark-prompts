Toy Car Treadmill Elimination Race
---------------------------
(CC-BY-NC-SA 4.0 by karminski-牙医)

Role: Three.js & Cannon-es Physics Simulation Expert (Anti-Clipping Edition)

**Task Objective:** Write a 3D web application based on Three.js and Cannon-es simulating a "Toy Car Treadmill Elimination Race". The focus is on **absolute physics stability** — preventing cars from clipping through geometry, getting stuck, or falling out of the world.

---

### 1. Physics World Foundations (Core Anti-Clipping Setup)

To solve Cannon-es's common clipping/tunneling problems, the following initialization settings **must** be strictly applied:

*   **World Setup:**
    *   `gravity`: (0, -9.82, 0)
    *   `broadphase`: use `CANNON.SAPBroadphase(world)` (Sweep and Prune) — better performance with many bodies and more accurate collision detection.
    *   **Key setting A (precision):** `world.solver.iterations = 20;` (default is 10; raise to 20 to reduce error).
    *   **Key setting B (tolerance):** `world.defaultContactMaterial.contactEquationStiffness = 1e8;` (high stiffness) and `contactEquationRelaxation = 3;`.
    *   **Time step:** In `requestAnimationFrame`, use `world.step(1/60, deltaTime, 10)`. Note the third parameter `maxSubSteps` is set to 10, which prevents objects from tunneling through walls during frame drops.

### 2. The Treadmill: Separating Visuals from Physics (Deep Floor Technique)

*   **Visual Mesh:**
    *   Create a flat BoxGeometry belt, sized e.g. (10, 0.5, 20).
    *   Add a prominent texture or stripes, and scroll the texture offset (UV Offset) each frame according to speed to simulate the rolling motion visually.
*   **Physics Body — the anti-clipping core:**
    *   **Do not** use a Box as thin as the visual model.
    *   Create an **extremely thick** Static (or Kinematic) Body.
    *   **Shape:** `new CANNON.Box(new CANNON.Vec3(5, 50, 10))` — note the Y-axis half-height is set to 50 (i.e. total thickness 100).
    *   **Position:** Offset the physics body downward so that its **top surface** aligns exactly with the top surface of the visual model.
    *   **Rationale:** The belt physically becomes a bottomless-deep base; a car absolutely cannot penetrate a 100-meter-thick block.
    *   **Motion logic:** Set it to `Kinematic` type and force-set `.velocity.set(0, 0, speed)` every frame.

### 3. Vehicle Structure (Constraints & Initialization)

*   **Multi-body structure:** 1 chassis Box + 4 wheel Cylinders + 4 HingeConstraints.
*   **Physics corrections (to prevent lock-ups):**
    *   The constraints must set **`collideConnected: false`**.
    *   The wheel rigid bodies' **`angularDamping` set to 0**.
    *   Wheel material friction `friction > 10`, restitution `restitution = 0`.
*   **Spawn Strategy (Drop Strategy):**
    *   **Do not** attempt to compute the exact Y value at which the wheels touch the ground (this easily results in cars spawning stuck inside the floor).
    *   **Strategy:** Spawn all cars at **Y = 3.0** (or another safe height).
    *   Let the cars naturally **drop** onto the treadmill.
    *   To keep landing bounces from getting chaotic, gravity can be temporarily increased for the first 1 second after spawning, or the chassis can be given a small downward initial velocity.

### 4. Scene Boundaries & Interaction

*   **Invisible Walls:**
    *   Create static invisible physics walls (`mass: 0`) at the front, left, and right of the treadmill to prevent cars from falling off right at the start, leaving only the rear open as the exit.
*   **Elimination logic:**
    *   Detect `chassisBody.position.z` > the treadmill's rear edge.
    *   Mark the car as eliminated; do not remove the rigid body — let it fall naturally.
*   **UI:** A slider to control speed, a button to switch camera views (including OrbitControls).

### 5. Code Implementation Requirements

*   **Code structure:** Single HTML file structure, modular functions (`initPhysics`, `createCar`, `animate`).
*   **Error handling:** Make sure the Three.js and Cannon-es CDN links are correct (use unpkg or cdnjs).
*   **Primary goal:** Above all, ensure cars land firmly, grip the belt, and hold their position on the treadmill — no clipping, no sinking into the ground.
