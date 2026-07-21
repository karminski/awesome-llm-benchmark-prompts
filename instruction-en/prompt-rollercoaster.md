Three.js Roller Coaster Animation 
-----------------------------

(CC-BY-NC-SA 4.0 by karminski-牙医)

## Goal

Create a complete, self-contained HTML file implementing a roller coaster animation using Three.js. The animation features a small ball traveling along a complex 3D track (a tube), eventually returning to the starting point and looping forever.

---

## 1. Tech Stack

- HTML, CSS, JavaScript
- Use the Three.js library (latest version via CDN)
- **Single HTML file** that runs directly in the browser

---

## 2. Scene Setup

### 2.1 Renderer
- Create a full-screen `WebGLRenderer` with antialiasing enabled

### 2.2 Scene
- Create a `Scene` object
- Background color: pale pink
- Ground: grayish-white plane, no grid

### 2.3 Camera
- Use a `PerspectiveCamera`
- Initial view: an overview showing the entire track

### 2.4 Lighting
- `AmbientLight`: provides base ambient illumination
- `DirectionalLight`: creates shadows and highlights, adding depth

---

## 3. Roller Coaster Track

### 3.1 Path Definition
Use `CatmullRomCurve3` to create a 3D curve path containing the following elements:

- **Climbs and dives**: dramatic changes in height
- **Spirals/helixes**: climbing or descending while rotating around the Y axis
- **Sharp turns**: large-amplitude turns in the XZ plane
- **Closed loop**: the end point connects smoothly back to the start

**CatmullRomCurve3 parameter settings:**
```javascript
new THREE.CatmullRomCurve3(points, true, 'chordal', 0.1)
```
- The `'chordal'` type is recommended; it avoids sharp corners better than `'centripetal'` when handling unevenly distributed points
- A tension value of 0.1-0.3 is recommended; smaller values produce smoother, rounder curves
- The `'chordal'` type is especially well suited to curves with high point density, producing more natural rounded transitions

### 3.2 Track Model
- Use `TubeGeometry` to generate a tube from the path
- Material: acrylic-like, semi-transparent white

**Recommended TubeGeometry parameters:**
- **Path segments (tubularSegments)**: recommended 800-1500 to keep the tube sufficiently smooth
  - Higher segment counts make the tube smoother in regions of high curvature change
  - For complex tracks, 1000 or more is recommended
- **Radial segments (radialSegments)**: recommended 12-16
  - Makes the tube cross-section rounder and avoids a polygonal look
  - 8 segments makes the tube look octagonal; 16 segments is much closer to a circle
- **Performance trade-off**: higher segment counts increase rendering cost; balance performance against visual quality

### 3.3 Track Structure Types

It is recommended to write helper functions (e.g. `generateHelixPoints()`, `generateFunnelPoints()`, etc.) to generate the path points for each segment, then combine them into the complete track.

#### Helix Section
- A tight spiral climbing or descending track
- Use trigonometric functions to control the X and Z coordinates while changing Y linearly

#### Spring/Wave Section
- A continuously undulating up-and-down structure
- Apply sine/cosine functions to the Y coordinate

#### Funnel Drop
- The ball enters a wide mouth from above, spirals downward, and shoots out of a narrow opening
- Gradually shrink the spiral radius

#### Launch & Return
- A parabolic track where the ball is "launched" into the air and falls back onto the track
- Simulate with parabolic path points

#### Multi-level Crossover
- The track crosses and weaves at different height levels
- Similar to a highway interchange structure

#### Loop-the-Loop
- A complete 360-degree vertical loop
- Draw a full circular path

#### Pinball-style Segments
- Locally widened tube or visualized branches
- Add decorative "bumpers" or "obstacles"
- Create an oscillation zone (a path that sways side to side or zigzags)

### 3.4 Ensuring Smooth Transitions (Important)

To avoid right angles or sharp corners at track junctions:

#### Exact junction-point matching
- Each generator function accepts a `startPoint` parameter and returns an `endPoint`
- Ensure the last point of the previous segment is exactly identical to the first point of the next segment

#### Add transition segments
- Insert 3-5 transition points between segments of different shapes
- Transition points should smoothly blend from one segment's tangent direction into the next
- Linear interpolation or Bezier curves can be used

#### Tangent continuity
- Consider the direction of motion at the end of each segment
- Ensure the angular difference in direction between adjacent segments is less than 30 degrees
- Use `curve.getTangentAt()` to verify tangent smoothness

#### Sample point density

**Internal point generation strategy:**
- Each track segment generator function must internally generate points densely enough to avoid polylines or sharp corners within the segment
- **Helix and funnel segments**: set the point count dynamically based on the number of turns; at least 50-60 points per turn is recommended
  - For example: a 3-turn helix needs at least 150-180 points
- **Wave segments**: adjust point count dynamically based on frequency; higher frequency requires more points
  - It is recommended to compute the point count as `Math.max(100, frequency * 50)`
  - At least 100 points to ensure a smooth waveform
- **Vertical loops**: at least 60-80 points recommended to form a perfect circle
- **Parabolic segments**: 50-60 points recommended to keep the parabola fluid

**Applying a point-smoothing algorithm:**
- Before each generator function returns its point array, apply smoothing to the raw points
- The **Chaikin smoothing algorithm** is recommended:
  - Iteratively inserts new points between adjacent points
  - Inserts one point each at the 25% and 75% positions between two points
  - 1-2 iterations are recommended
  - 1 iteration roughly doubles the point count; 2 iterations roughly quadruple it
- This algorithm is especially suitable for regular shapes generated by mathematical formulas
- It effectively eliminates tiny kinks produced by sampling

**Dynamic point density:**
- Regions with large curvature changes should have increased point density
- After generating points, you can compute the angular change between adjacent points
- If the angular change exceeds a threshold (e.g. 15-20 degrees), add interpolated points in that region
- Straight segments can use fewer points; turning segments need more

#### Validation and debugging
- Check the angular change between adjacent points
- An angular change exceeding 30 degrees indicates a possible sharp corner
- Adding a `validateTrackSmoothness()` function is recommended
- During development, place markers at junctions and use different colors to distinguish segments

### 3.5 Track Quality Assurance

#### Curve continuity levels
- Aim for **G1 continuity** (continuous tangent direction) or the higher **G2 continuity** (continuous curvature)
- **G1 continuity** means adjacent segments share the same direction at the junction, with no sharp corners
  - The ball experiences no sudden direction changes when passing through
  - The track looks natural and fluid
- **G2 continuity** means the curvature of turns also changes smoothly
  - No sudden acceleration changes
  - Provides the best ride experience

#### Testing methods
- During development, draw tangent vectors along the track to visualize the continuity of the track direction
- Check whether the tube generated by `TubeGeometry` has any twisting, folding, or unnatural angles
- Testing with a smaller tube radius (e.g. 0.5-1.0) makes track unevenness easier to spot
- Play the animation at slow speed in the browser and observe whether the ball exhibits any "jumping" or "stuttering"

#### Common issue troubleshooting
- **Issue: the tube shows an obvious "kink" or "polyline" effect somewhere**
  - Cause: insufficient point density at that spot, or too large an angular change between adjacent points
  - Fix: increase the sample point count for that segment, or apply a point-smoothing algorithm
  
- **Issue: the tube twists or self-intersects**
  - Cause: insufficient radial segments in `TubeGeometry`, or path curvature too high
  - Fix: increase radial segments to 16, or reduce the tube radius
  
- **Issue: the overall curve looks "hard-edged"**
  - Cause: the tension value of `CatmullRomCurve3` is too high
  - Fix: lower the tension value to 0.1-0.2
  
- **Issue: sharp corners appear at junctions**
  - Cause: discontinuous tangent directions between adjacent segments
  - Fix: add transition points between segments, using Bezier curve interpolation

---

## 4. Ball Animation

### 4.1 Model
- Use `SphereGeometry` and `MeshStandardMaterial`
- Color should contrast with the tube so the ball is clearly visible

### 4.2 Animation Implementation
- Use `Clock` inside the `animate` loop to obtain time
- Compute the ball's position on the path from the time (the `.getPointAt()` method)
- Update the ball's `position`
- Loop the animation: restart from the beginning after reaching the end

---

## 5. Camera Controls

### 5.1 First-person View
- The camera follows the ball
- Positioned slightly behind or directly above the ball
- Use `.lookAt()` to point the camera ahead along the ball's direction of travel
- Creates an immersive ride experience

### 5.2 Third-person View (OrbitControls)
- Mouse wheel: zoom
- Left mouse button: rotate the camera
- Right mouse button: pan the camera

---

## 6. Track Support System

### 6.1 Goal
Add support pillars to the elevated track to enhance realism and visual quality.

### 6.2 Implementation

#### Sample track points
- Sample along the track at fixed intervals (40 units)
- Use `curve.getPointAt(t)` to obtain path points

#### Generate support pillars
For each sampled point:
- Drop a vertical line downward (-Y direction) from the track point to the ground
- Compute the pillar height: **pillar height = track point height - track tube radius - ground height**
  - The pillar should connect to the outer wall of the tube, not the center path line
  - The `TubeGeometry` radius value must be subtracted
- Use `CylinderGeometry` to create a cylinder (diameter 0.2 units)
- Place the pillar: base on the ground, top touching the outer wall of the tube

#### Height filtering
- Only generate pillars where the track is above a height threshold (10 units) off the ground
- Avoids overly dense pillars at low sections

#### Avoiding overlaps (optional)
- **Raycaster detection**: cast a ray downward to detect intersection with any existing objects
- **Offset strategy**: when an overlap is detected, offset 3 units along the tangent direction; if it still overlaps, keep offsetting

### 6.3 Visual Refinement
- Use metallic `MeshStandardMaterial`
- Add decorative pieces (small discs or blocks) to the tops and bottoms of pillars
- Add a flat base plate at the bottom of each pillar for a sturdier look

---

## 7. Implementation Recommendations

1. **Modular design**: write an independent generator function for each track structure type
2. **Parameterized control**: use parameters to control track dimensions, density, etc.
3. **Performance optimization**: keep pillar counts and geometry complexity reasonable
4. **Debugging tools**: add visualization aids during development (e.g. path point markers)
5. **Curve smoothness first**: prioritize track smoothness during implementation
   - Better to generate extra points than to allow sharp corners and polylines
   - Modern browsers can easily handle curves with thousands of points
   - A smooth track matters more than performance optimization, because an unsmooth track severely degrades the visual experience
   - Consider reducing point counts only during an optimization phase, rather than starting with the fewest points possible
