Underwater Firecracker Explosion — Physics Simulation of Cavitation Bubbles and Water Ejection
-------------------------------------------

(CC-BY-NC-SA 4.0 by karminski-牙医)

Using three.js, write a single-file HTML/JS demo that simulates a lit firecracker being dropped into a lidless glass tank filled with water, going through the complete physical process: underwater fuse burning, explosion, cavitation bubble expansion and collapse, and water column ejection. The glass tank does not shatter in the explosion; the water is forced out through the tank's open top.

**The physical core of this demo:** Water is nearly incompressible. When a firecracker explodes underwater, the detonation products form a high-pressure, high-temperature bubble that expands rapidly, squeezing the surrounding water like a piston. In a lidded container this would burst the container, but in a lidless tank the water's only way out is upward — gushing out of the tank opening to form a spectacular water column. The bubble then over-expands until its internal pressure drops below ambient pressure, gets squeezed back by the surrounding water and collapses, oscillating this way several times, each cycle accompanied by violent heaving of the water surface.

---

## Scene Setup

**Environment and atmosphere:**
- A bright, soft indoor laboratory style. The background uses a gradient from light blue to white; the ground is a light gray, smooth plane with faint reflectivity, like a lab bench surface.
- Lighting is bright and even, ensuring the glass tank is clear and transparent from every angle and the phenomena inside the water are clearly visible. An environment map must be provided so the glass refraction material works correctly.
- Use ACES Filmic tone mapping with moderate exposure.

**Glass tank:**
- A lidless rectangular glass tank with proportions close to a real fish tank (width slightly greater than height).
- The glass walls use a physical transmission material, showing realistic glass refraction, reflection, and a faint teal-green tint. Double-sided rendering.
- The tank walls have a proper sense of thickness (not paper-thin) and remain intact through the explosion.
- The tank is filled with water, with about a 1cm gap between the water surface and the tank rim.

**Water body:**
- A translucent light-blue volume filling the tank, with realistic light attenuation — the deeper, the darker.
- At rest the water surface is flat, with faint caustic light patterns shimmering on the tank bottom.
- The water surface can deform dynamically in response to internal disturbances (ripples from rising bubbles, violent heaving after the explosion).

**Firecracker:**
- A classic red cylinder, sealed with packed clay at the top and bottom, with a waterproof fuse protruding from the top.
- The fuse is already lit, with a bright burning spark at its tip.
- Overall size is in reasonable proportion to the glass tank (firecracker length about 1/5 of the tank height).

---

## Animation Sequence

### Phase 1: Water Entry

After clicking the "Start" button, the lit firecracker falls from above the frame into the glass tank along a parabolic trajectory.

**Physics at the moment of entry:**
- As the firecracker passes through the water surface, it throws up an upward splash and ring-shaped ripples.
- Once submerged, the firecracker's speed drops sharply — water's drag is roughly 800 times that of air; the firecracker should show pronounced deceleration, sinking slowly rather than plunging straight to the bottom.
- The air entrained during entry forms a trail of accompanying bubbles that rise along the firecracker's wake.

### Phase 2: Underwater Fuse Burning (Visual Highlight #1)

The firecracker sinks slowly to near the tank bottom while the fuse keeps burning underwater.

**Fuse burning and smoke-filled bubbles:**
- The fuse's burning point continuously emits a bright orange-white glow underwater, producing an underwater halo via Bloom that illuminates the surrounding water. The glow flickers irregularly.
- The burning continuously produces bubbles. Key detail: these bubbles are not empty inside — they encapsulate the grayish-white smoke produced by combustion, appearing translucent and murky, with specular highlights on their surfaces.
- After being released from the burning point, the bubbles rise with a swaying motion under buoyancy, expanding slightly as water pressure decreases during ascent. Smaller bubbles may merge into larger ones.
- When bubbles reach the water surface they burst, releasing the enclosed smoke at the burst location; the smoke drifts upward and disperses above the water surface.
- As the fuse burns down, the water in the tank is gently stirred and fine ripples appear on the surface.

### Phase 3: The Moment of Explosion

The instant the fuse burns out, the firecracker explodes near the tank bottom.

**Detonation and shock wave:**
- The explosion produces an extremely intense flash — the color shifts rapidly from white to orange, and the Bloom effect bursts instantly to maximum. The entire body of water in the tank is lit up by the flash.
- The detonation produces a spherical shock wave in the water. Since the speed of sound in water is about 1500m/s, the shock wave reaches the tank walls almost instantly (visually this can be represented as an extremely fast-expanding sphere of refractive distortion). When the shock wave reaches the water surface, it kicks up a fine layer of water mist.
- The firecracker body is blown apart, producing a small amount of red fragments and debris tumbling and dispersing through the water.

### Phase 4: Cavitation Bubble Expansion — Water Column Ejection (Visual Highlight #2)

This is the most spectacular phase of the entire demo.

**Bubble expansion principle:** The detonation products (high-temperature, high-pressure gas) form a spherical bubble that expands violently, driven by extremely high internal pressure. Water is nearly incompressible, and the expanding bubble pushes it outward like a piston. In a lidless tank, the only clear escape route for the displaced water is the tank opening —

**Water column ejection:**
- As the bubble expands, the water surface inside the tank surges upward, doming up, then erupts violently out of the tank opening, forming a thick water column shooting into the air.
- The water column's ejection height can reach 2-3 times the tank height, with tremendous visual impact.
- The top of the water column gradually spreads and breaks apart into droplets and mist as it rises.
- At the same time, large amounts of water splash outward around the tank rim and stream down the outer walls.
- When the bubble reaches maximum expansion inside the tank, the water level drops sharply; the bubble's spherical boundary can even be briefly visible (a huge cavity occupying the lower half of the tank).

### Phase 5: Bubble Collapse and Oscillation

**Collapse principle:** After over-expanding, the bubble's internal pressure drops below the surrounding water pressure; the water squeezes back, collapsing the bubble rapidly. At minimum size the gas is recompressed, pressure rises, and the bubble re-expands — forming a damped oscillation, with the bubble's maximum radius shrinking each cycle.

**Visual representation:**
- As the bubble collapses, the previously ejected water column loses its support and begins to fall back, with large amounts of water dropping from the air back into the tank and onto the surrounding ground.
- Each re-expansion of the bubble pushes the water surface up again (with progressively smaller amplitude), producing secondary splashes.
- After 2-3 oscillation cycles the bubble stabilizes, breaking into multiple small bubbles that rise to the surface and burst.
- Throughout this process the water in the tank is violently churned, appearing murky and turbulent (mixed with smoke particles from the explosion products).

### Phase 6: Aftermath and Calm

- Large amounts of splashed water sit on the outer tank walls and the ground, with spreading water stains/puddles forming on the ground.
- The water level in the tank is clearly lower than at the start (a substantial portion of the water has been ejected).
- The water surface sloshes violently, producing standing waves that bounce back and forth between the tank walls with gradually decaying amplitude.
- The water is murky, with a few residual small bubbles rising sporadically.
- Firecracker fragments and debris settle on the bottom or drift slowly down through the water.
- A thin layer of steam/water mist rises from the surface and dissipates.
- After several seconds the water surface calms, and the scene comes to rest.

---

## Visual and Rendering Requirements

**Water rendering:**
- The water body must not be a simple translucent box. It needs dynamic surface deformation capability, able to depict bulging, depressions, waves, and splashes.
- The specular reflections and refraction of the water surface must update in real time with the deformation.
- Water ejected out of the tank should be depicted as a continuous stream gradually breaking up into droplets, not as discrete particles from the start.

**Bubble rendering:**
- Smoke-filled bubbles from the fuse: translucent spheres with visible murky smoke fill inside, and environment-reflection highlights on the surface.
- The explosion cavitation bubble: a huge, nearly transparent spherical surface with visible refractive distortion at its boundary, and a faint dark-orange glow of hot gas inside.

**Lighting and post-processing:**
- Bloom is used to enhance the visual brightness of the burning fuse spark, the explosion flash, and water-surface highlights.
- At the moment of explosion, the Bloom intensity has a pronounced pulse-like burst.
- Underwater light scattering and attenuation should be depicted — the farther from the light source, the darker.

**Performance notes:**
- Splash, droplet, and debris particles should be managed uniformly with InstancedMesh to maintain frame rate.
- Avoid frequently creating and destroying dynamic point lights during the animation. Fire glow effects should preferentially use emissive materials combined with Bloom.

---

## Interactive Controls

- **"Start" button:** triggers the firecracker toss-in animation.
- **"Reset" button:** restores the scene to its initial state.
- **Time-scale slider:** 0.1x - 1.0x slow-motion control, for observing the details of bubble expansion and collapse.
- **View switching:** provide at least three preset views — external panoramic top-down, close-up side-on at eye level, and underwater looking up from inside the tank.
- OrbitControls supports free mouse rotation and zooming.

---

## Code Requirements

- All code encapsulated in a single HTML file.
- Use sensible class/module divisions to manage the different systems (bubbles, water body, particles, physics simulation, etc.).
- Simple numerical integration is sufficient for the physics simulation; no external physics engine is needed.
- Emphasize the expressiveness of the two core visual climaxes: "smoke-filled bubbles rising underwater" and "the water column blasting skyward at the moment of explosion".
