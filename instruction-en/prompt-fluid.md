Cup Pouring Test
---------------

(CC-BY-NC-SA 4.0 by karminski-牙医)


Using Python and the pygame library, create a 2D fluid simulation program. The program must simulate liquid (represented by a large number of particles) pouring out of a tilting cup under the influence of gravity.

## Core Requirements:

### Create the Pygame environment:

Initialize a pygame window, with a suggested size of 1920x1080 pixels.
Set up a main loop to handle events, update the physics state, and render the frame.
Set the background color to white.

### Define the cup container:

The cup is two-dimensional, formed by several line segments making up a container shape. The cup's mouth is open and faces upward. You may define its shape with a list of vertices, e.g. [(x1, y1), (x2, y2), ...].
In the initial state, the cup should stand upright in the middle of the screen.
The cup's outline should be rendered, in black.

### Create the particle system:

The liquid is represented by a large number of particles (e.g. 400).
Each particle is a small circle with the following properties:
- Position (x, y)
- Velocity (vx, vy)
- Diameter (e.g., 4 pixels)
- Color (e.g., blue)

It is recommended to create a Particle class to manage these properties.
Initially, all particles are spawned gradually at a position above the cup, so that all particles flow into the cup's interior.


### Implement the physics simulation:

#### Gravity:
At every time step (frame), apply a constant downward acceleration (e.g. g = 0.1) to all particles.

#### Collision with the container:
Implement collision detection between particles and the line segments forming the cup.
Prevent particles from leaking out through the joints between line segments.
When a particle hits a wall, its velocity should be reflected. To simulate energy loss, the post-bounce velocity should be multiplied by a restitution coefficient less than 1 (e.g., 0.7).

#### Particle-particle interaction (simplified fluid behavior):
This is the key to the simulation. To prevent particles from clumping or overlapping unnaturally, implement a simple repulsive force.
When the distance between two particles is less than some threshold of the sum of their radii, apply a small outward force along the line connecting their centers. This can be done simply by directly adjusting their positions or velocities to gently push them apart.

#### Motion update:
Use Euler integration to update each particle's position:
velocity += acceleration * dt
position += velocity * dt
(For simplicity, you may assume dt=1 and add the acceleration directly to the velocity.)

#### Screen bottom:
Particles should land on the bottom of the screen rather than falling out of it.

### Implement the animation:

Spawn particles: first spawn the particles above the cup, and let them all flow into the cup.
Cup pouring: then, the cup must rotate slowly over time.
Rotate the cup around a point at its center.
The rotation angle may increase slowly from 0 degrees to about 135 degrees, simulating the pouring process.

### Code structure:
- Use an object-oriented approach, e.g. create a Particle class and a Flask class.
- Add comments at the key parts of the code, especially the physics calculations (gravity, collisions, inter-particle forces) and coordinate transformations (cup rotation).
- Note that the cup must be drawn before the particles, so the particles do not simply fall straight through.
- All code should be written in English. Put all code in a single .py file.
