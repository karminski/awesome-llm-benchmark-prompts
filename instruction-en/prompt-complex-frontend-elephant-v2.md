Elephant Toothpaste v2
-----------
(CC-BY-NC-SA 4.0 by karminski-牙医)

Using three.js, implement a realistic 3D demo of the "elephant toothpaste" chemistry experiment. All code (including HTML, CSS, JavaScript) must be encapsulated in a single standalone HTML file.

**Scene setup:**
1. **Background:**
    * A skybox with a dawn effect: a gradient from blue at the top toward pink
2.  **Plane:** Create a 1000*1000 gray, smooth horizontal plane that can receive shadows.
3.  **Conical (Erlenmeyer) flask:**
    *   **Shape:** Place a transparent glass conical flask at the center of the plane. The flask should have a clear silhouette: a rounded mouth, a cylindrical neck, a gradually widening conical body, and a flat bottom. Do not substitute a simple cone.
    *   **Material:** The flask material should be highly transparent glass, with suitably high transmission, low roughness, and a correct index of refraction to simulate glass. The background and the refraction of light passing through the liquid should be visible.
    *   **Liquid:** The flask is pre-filled with fluorescent pink liquid to about one third of its height. The liquid surface should be flat and conform to the flask's inner walls. Note: reference the shape of the conical flask model — ideally, duplicate part of the flask's conical shape; do not let the liquid model spill outside the conical flask.
    *   **Note:** The entire conical flask model must be oriented upward.

**"Elephant toothpaste" eruption effect simulation:**
1.  **Trigger:** The page provides a button in the footer position; clicking it starts the eruption.
2.  **Foam form and texture:**
    *   Use 2000 fluorescent pink foam particles of varying sizes to simulate the experiment.
    *   The ejected foam should consist of **large numbers of tiny, translucent particles that can merge into one another**, simulating the **dense, fluffy texture** of real foam and avoiding the look of isolated small spheres. The color is fluorescent pink, and some brightness variation may be added for layered depth.
    *   Consider using a simple **noise texture** or a procedural approach to add some surface detail to the foam particles, making them look more like porous foam rather than smooth spheres.
3.  **Ejection dynamics and fluid simulation:**
    * **Expansion phase:** Foam is generated at the liquid surface inside the flask and, driven by the reaction's gas pressure, moves upward (+Y axis).
    * **Boundary constraints:** While rising inside the flask, particle coordinates must be strictly confined within the mathematical boundary of the flask's inner walls:
        - When $y < 20$, the maximum radius is limited to $R(y) = 10 - 0.4 \times y$.
        - When $y \ge 20$, the maximum radius is limited to $R(y) = 2$.
        - When a particle collides with the boundary, it should slide upward along the flask wall and must never penetrate the flask body (clipping through the model).
    * **Nozzle acceleration effect (Nozzle Effect / Venturi Effect):**
        - As the flask narrows, the fluid is compressed. A particle's upward velocity $v_y$ must be proportional to the reciprocal of the cross-sectional area at that height (or accelerate in proportion to it).
        - The particles form a foam column erupting upward,
        - When particles enter the neck ($y > 20$), the upward ejection velocity should reach its peak, forming a high-initial-velocity, high-pressure "jet" at the flask mouth ($y = 30$). The initial ejection height must reach at least 3 times the flask height.
    *   **Initial eruption:** Foam erupts violently upward from the flask mouth, forming a continuously rising column (tightly packed foam stacked together into a columnar shape). The height must be at least 3 times the height of the conical flask; the foam column only begins to disperse as it approaches its maximum height, with high initial ejection velocity and pressure.
    *   **Pressure decay:** The ejection intensity (velocity and foam generation rate) should **gradually weaken** over time, causing the foam column's height and ejection distance to gradually decrease, simulating the depletion of the chemical reactants.
    *   **Particle motion:** Particle trajectories should simulate **basic fluid-dynamic influences** rather than just simple parabolas. For example, there should be slight repulsive forces between particles to simulate the foam's sense of expansion, or random velocity perturbations around the main jet stream. Everything is affected by gravity.
4.  **Foam-environment interaction and deformation:**
    *   **Gravity and accumulation:** Ejected foam particles fall under gravity. When foam particles contact the plane, they have some elasticity: they bounce after impact like fluid striking a hard surface, then spread out on the plane, and finally, as the gas inside the particles gradually escapes, they should **simulate a compression-deformation effect** — flattened vertically and slightly expanded horizontally.
    *   **Collision with the flask body:** If large amounts of foam strike the flask body, they should bounce off as well; a small fraction of foam particles cling to the flask wall due to surface tension and gradually slide down, or particularly small particles stick directly to the flask body.
    *   **Accumulated form:** Foam landing on the plane should gradually **pile up, forming a covering layer of some thickness**, rather than disappearing. Accumulated foam should also show similar deformation and merging effects.
5.  **Liquid depletion:**
    As the eruption proceeds, the liquid level inside the conical flask gradually falls, demonstrating the depletion of the liquid. Note this is not the liquid shrinking as a whole, but the effect of the liquid's top surface gradually descending.

**Lighting and rendering:**
1.  **Lights:**
    *   Set up a key light to produce crisp shadows.
    *   Add an ambient light to brighten the scene's dark areas, ensuring there are no pure-black regions.
2.  **Shadows:** Enable the renderer's shadow map. The plane should receive shadows; the flask and the ejected foam should cast shadows.
3.  **Reflection and refraction:** The flask material should show faint reflections of the environment and the refraction of light passing through the liquid and glass.

**Camera and controls:**
*   The camera should look down at the scene center (the conical flask's position) from an oblique 45-degree angle, ensuring the entire eruption process and the final accumulation effect can be observed clearly.


Please write a complete, standalone, ready-to-run HTML file that uses Three.js to implement a 3D animated demonstration of the "elephant toothpaste" chemistry experiment with extremely realistic physical detail and visual effects.
All code (HTML, CSS, JavaScript, including the CDN imports of the Three.js library and its OrbitControls) must be entirely encapsulated within this one file.
