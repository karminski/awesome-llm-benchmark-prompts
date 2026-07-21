Maze
-----
(CC-BY-NC-SA 4.0 by karminski-牙医)

## Role
You are a senior game developer with deep expertise in Python and computer graphics, especially skilled at physics simulation and algorithm visualization using Pygame.

## Goal
Please write a complete Python script using the `pygame` library to implement a fluid pathfinding demo.

## Requirements

## 1. Window and Environment Setup
- Window resolution: 1920x1080.
- Background color black; frame rate capped at 60 FPS.
- A `SEED` variable must be definable at the top of the script for the random number generator, ensuring deterministic maze generation.

## 2. Maze Generation
- Use the **Recursive Backtracker** algorithm to generate a 5x5 perfect maze (guaranteeing all paths are connected).
- The maze must auto-scale to fill most of the central screen area (leaving a margin).
- **Entrance**: open a gap at a randomly chosen position in the maze's top wall.
- **Exit**: open a gap at a randomly chosen position in the maze's bottom wall.
- When rendering, draw walls as white rectangles.

## 3. Fluid Particle System (key focus)
- Continuously spawn circular particles at the maze entrance (simulating flowing water), colored blue (varying shades of blue may be used for visual effect).
- **Physics simulation**:
  - Particles accelerate downward under gravity.
  - Particles need a simple repulsive force or volumetric collision between them to prevent infinite overlap, simulating the "piling up" behavior of liquid.
  - Particles must bounce or slide when colliding with maze walls.
- **Objective**: the water flow should naturally fill dead ends and ultimately find a path to flow out through the bottom exit.

## 4. Performance Optimization (critical)
- Due to Python's performance limits, you must implement a **Spatial Hash Grid** or **Uniform Grid** algorithm for collision detection.
  - Using an O(N^2) double loop for all-pairs particle collision detection is **strictly forbidden**.
  - Only test particles/walls within the same grid cell or neighboring cells.
- Keep the maximum particle count under reasonable control (e.g. a cap of 1500-2000); recycle old particles when they fall below the screen or exceed the count limit.

## 5. Code Structure
- The code must be encapsulated in classes (e.g. `Particle`, `Maze`, `Simulation`).
- Include clear comments explaining the logic of the physics calculations and collision algorithms.
- The demo should start immediately when the script is launched.

Please provide complete, directly runnable code.
