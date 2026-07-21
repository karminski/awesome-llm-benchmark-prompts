Elephant Toothpaste Test
---------------

(CC-BY-NC-SA 4.0 by karminski-牙医)

Using three.js, implement a realistic 3D demo of the "elephant toothpaste" chemistry experiment. All code (including HTML, CSS, JavaScript) must be encapsulated in a single standalone HTML file.

**Scene setup:**
1.  **Plane:** Create a 1000*1000 gray, smooth horizontal plane that can receive shadows.
2.  **Erlenmeyer flask:**
    *   **Shape:** Place a transparent glass Erlenmeyer flask at the center of the plane. The flask should have a clear silhouette: a cylindrical neck, a gradually widening conical body, and a flat bottom. Do not substitute a simple cone.
    *   **Material:** The flask material should be highly transparent glass, with suitably high transmission, low roughness, and a correct index of refraction to simulate glass. Set ```{ color: 0xffffff, transparent: true, opacity: 0.9, roughness: 0.95, metalness: 0.35, clearcoat: 1.0, clearcoatRoughness: 0.03, transmission: 0.95, ior: 1.5, side: THREE.DoubleSid}```. The background and the refraction of light passing through the liquid should be visible.
    *   **Liquid:** The flask is pre-filled with fluorescent pink liquid to about one third of its height. The liquid surface should be flat and conform to the flask's inner walls. Note: reference the shape of the Erlenmeyer flask model — ideally, duplicate part of the flask's conical shape; do not let the liquid model spill outside the Erlenmeyer flask.
    *   **Note:** The entire Erlenmeyer flask model must be oriented upward.

**"Elephant toothpaste" eruption effect simulation:**
1.  **Trigger:** The page provides a button in the footer position; clicking it starts the eruption.
2.  **Foam form and texture:**
    *   The ejected foam should consist of **large numbers of tiny, translucent particles that can merge into one another**, simulating the **dense, fluffy texture** of real foam and avoiding the look of isolated small spheres. The color is fluorescent pink, and some brightness variation may be added for layered depth.
    *   Consider using a simple **noise texture** or a procedural approach to add some surface detail to the foam particles, making them look more like porous foam rather than smooth spheres.
3.  **Ejection dynamics and fluid simulation:**
    *   **Initial eruption:** Foam erupts violently upward from the flask mouth, forming a continuously rising column (tightly packed foam stacked together into a columnar shape). The height must be at least 3 times the height of the Erlenmeyer flask; the foam column only begins to disperse as it approaches its maximum height, with high initial ejection velocity and pressure.
    *   **Pressure decay:** The ejection intensity (velocity and foam generation rate) should **gradually weaken** over time, causing the foam column's height and ejection distance to gradually decrease, simulating the depletion of the chemical reactants.
    *   **Particle motion:** Particle trajectories should simulate **basic fluid-dynamic influences** rather than just simple parabolas. For example, there should be slight repulsive forces between particles to simulate the foam's sense of expansion, or random velocity perturbations around the main jet stream. Everything is affected by gravity.
4.  **Foam-environment interaction and deformation:**
    *   **Gravity and accumulation:** Ejected foam particles fall under gravity. When foam particles contact the plane or the flask's outer wall, they should **simulate a compression-deformation effect**, e.g. flattened vertically (reduced Y-axis scale) and slightly expanded horizontally (increased X- and Z-axis scale).
    *   **Accumulated form:** Foam landing on the plane and on the flask should gradually **pile up, forming a covering layer of some thickness**, rather than disappearing. Accumulated foam should also show similar deformation and merging effects.
    *   **Foam settling:** After landing on an object's surface, foam slides a short distance and then stops moving.
5.  **Liquid depletion:**
    As the eruption proceeds, the liquid level inside the Erlenmeyer flask gradually falls, demonstrating the depletion of the liquid. Note this is not the liquid shrinking as a whole, but the effect of the liquid's top surface gradually descending.

**Lighting and rendering:**
1.  **Lights:**
    *   Set up a key light to produce crisp shadows.
    *   Add an ambient light to brighten the scene's dark areas, ensuring there are no pure-black regions.
2.  **Shadows:** Enable the renderer's shadow map. The plane should receive shadows; the flask and the ejected foam should cast shadows.
3.  **Reflection and refraction:** The flask material should show faint reflections of the environment and the refraction of light passing through the liquid and glass.

**Camera and controls:**
*   The camera should look down at the scene center (the Erlenmeyer flask's position) from an oblique 45-degree angle, ensuring the entire eruption process and the final accumulation effect can be observed clearly.
