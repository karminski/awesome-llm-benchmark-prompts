Firecracker Chain Explosion Test
---------------

(CC-BY-NC-SA 4.0 by karminski-牙医)

Using three.js, implement a realistic "box of firecrackers" 3D chain-reaction explosion demo. All code (including HTML, CSS, JavaScript) must be encapsulated in a single standalone HTML file.

**Scene setup:**
1. **Plane:** Create a 1000×1000 gray, smooth horizontal plane that can receive shadows. The ground material uses ```{color: 0x808080, roughness: 0.8, metalness: 0.1}```.

2. **Firecracker modeling:**
   - **Body:** red cylinder, height 1, diameter 0.2, material ```{color: 0xff0000, roughness: 0.6, metalness: 0.2}```
   - **Clay seals:** a small earthen-yellow cylinder at both the top and bottom of the body, height 0.05, material ```{color: 0xdaa520, roughness: 0.8}```
   - **Fuse:** a thin teal cylinder protruding from the center of the top seal, length 0.5, diameter 0.05, material ```{color: 0x20b2aa, roughness: 0.4}```
   - **Collision volume:** firecrackers have collision volumes between them to prevent models from overlapping each other 
   - **Burning state:** the tip of the fuse must be able to display a small orange-red glowing sphere simulating a spark

3. **Transparent box:**
   - Dimensions: a 10×10×8 transparent glass box; the top has no collision volume while the other faces do; material ```{color: 0xffffff, transparent: true, opacity: 0.15, roughness: 0.1, metalness: 0.0}```
   - **Box interior:** when the animation starts, spawn firecrackers randomly starting from a height of 5 above the box top, in groups of 10, for a total of 10 groups; the firecrackers free-fall into the box
   - **Box position:** placed at the center of the plane, slightly toward the back

**Explosion and chain-reaction simulation:**

1. **Trigger mechanism:**
   - A "Start Fireworks" button at the bottom of the page triggers the animation
   - An already-burning firecracker falls from the top of the screen into the box along a parabolic trajectory

2. **Fuse burning effect:**
   - **Burning animation:** the fuse shortens gradually from the tip downward over 2 seconds, with the burning portion showing an orange-red glow effect

3. **Explosion effect:**
   - **Instant replacement:** the firecracker disappears at the instant of explosion and is replaced by a large amount of colored paper confetti fragments
   - **Confetti system:** generate 50-80 small colored square/rectangular fragments, colored red
   - **Explosion shock wave:** produce a spherically expanding transparent shock-wave effect that affects nearby firecrackers
   - **Fragment physics:** confetti has initial outward velocity, is affected by gravity, and has random tumbling animation

4. **Impact force and knock-back:**
   - **Effective range:** firecrackers within a radius of 2-3 units of the explosion center are hit by the impact
   - **Knock-back animation:** impacted firecrackers are flung away 1-5 units in random directions and with random force
   - **Rotation effect:** flung firecrackers have random tumbling rotation animations
   - **Collision detection:** flung firecrackers bounce off the box walls and ground

5. **Fire flash propagation and chain reaction:**
   - **Fire flash range:** each explosion produces a spherical fire-flash zone of radius 3 units, lasting 0.5-1 second
   - **Ignition mechanism:** the fuses of unexploded firecrackers within the fire-flash zone ignite automatically, starting a 1-second burn countdown
   - **Propagation delay:** ignition of different firecrackers has a random delay of 0.1-0.3 seconds, avoiding simultaneous explosions
   - **Chain spread:** forming a realistic domino-style chain reaction

6. **Environmental interaction effects:**
   - **Ground impact:** landing firecrackers and fragments produce small impact bounces on the ground
   - **Fragment accumulation:** confetti fragments eventually scatter across the ground and around the box, forming piles

**Advanced visual effects:**

1. **Enhanced particle systems:**
   - **Spark particles:** burning fuses and explosions produce bright sparks with trailing effects
   - **Smoke effect:** explosions produce gray-white smoke particles that rise slowly and gradually dissipate
   - **Glow effect:** explosions produce a bright momentary glow that lights up the surrounding area

2. **Dynamic lighting:**
   - **Explosion flash:** create a momentary point light at each explosion to simulate the flash
   - **Fuse glow:** burning fuses produce small dynamic point lights
   - **Color temperature:** light color varies from orange-red to bright white

**Lighting and rendering settings:**

1. **Light configuration:**
   - **Key light:** a 45-degree directional light ```DirectionalLight```, intensity 1.2, capable of casting shadows
   - **Ambient light:** ambient light with intensity 0.4, ensuring no pure-black areas in the scene
   - **Dynamic lights:** temporary point lights produced by explosions and burning

2. **Shadow system:**
   - Enable the renderer's shadow map ```renderer.shadowMap.enabled = true```
   - All firecrackers and the box can cast shadows ```castShadow = true```
   - The ground receives shadows ```receiveShadow = true```
   - Dynamic shadow updates to reflect moving objects

3. **Post effects:**
   - Enable a momentary screen white-flash effect during explosions
   - Optionally add a subtle camera-shake effect

**Camera and controls:**
- The camera looks down on the whole scene from an oblique 45-degree angle, about 15 units from the box
- Controllable with the mouse
- Ensure the full chain-reaction process and the scattering of fragments can be observed in their entirety

**Code structure requirements:**
- Create a standalone ```Firecracker``` class managing individual firecracker state
- Create an ```ExplosionSystem``` class managing explosion effects and particles
- Create a ```ChainReactionManager``` class controlling the chain-reaction logic
- All physics calculations use simplified but realistic equations of motion
- Add detailed comments in the code, especially for the explosion propagation and physics simulation parts
- Performance optimization: keep particle counts under reasonable control

**Interactive controls:**
- Provide a "Reset Scene" button to restart the demo
- Optionally add a "Single Detonation" mode: click a specific firecracker for a localized detonation test
- Display statistics showing the current count of unexploded firecrackers and the number of explosions that have occurred
