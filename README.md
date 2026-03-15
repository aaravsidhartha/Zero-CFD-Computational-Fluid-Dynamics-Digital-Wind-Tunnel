# Zero CFD (Computational Fluid Dynamics) Digital Wind Tunnel
A lightweight, real-time digital wind tunnel built entirely with pure vector calculus and raycast sensors. Zero fluid simulations.

![Image](https://github.com/user-attachments/assets/35a987b6-8020-4f05-a766-b10f74f2d24a)

![Image](https://github.com/user-attachments/assets/d267b723-05ed-4044-9d7d-5af1db2186d3)

## The Problem: Removing The Need For Brute Computational Fluid Simulations For Simple Visualisation
If you want to run fluid simulations or test aerodynamics, you usually need massive compute power and hours to bake. Standard physics engines calculate pressure, viscosity, and density across millions of voxels. It is incredibly heavy and notoriously slow, especially if your laptop is a literal potato.

I didn't want to wait hours just to visualize flow lines over a 3D mesh. I needed a real-time digital wind tunnel that runs instantly.

## The Solution: First-Principles Math & Geometry Nodes
Instead of relying on dedicated CFD (Computational Fluid Dynamics) software, I built a custom solver inside Blender's Geometry Nodes. 

Why Geometry Nodes? Because it isn't actually a physics engine, it's a raw, procedural logic environment. It gave me the exact framework I needed to bypass default 3D collision physics entirely. By treating the 3D viewport as a blank coordinate grid, I could use the exact vector calculus I just studied for my 12th-grade board exams to write my own visual algorithm. No heavy fluid simulations needed.

<img width="1569" height="718" alt="Image" src="https://github.com/user-attachments/assets/e2fed2c9-424c-4e6b-9808-495f8479d01a" />

## Why This Matters in Engineering
In engineering and software architecture, efficiency is everything. This project isn't about perfectly simulating real-world molecular air pressure, it's more about extreme optimization in times where you don't need pin-point precision results, just an overall understanding of the way the object and air are interacting.

It proves that you don't always need to brute-force a massive physics simulation to get results. If you understand the fundamental math, like vector projection and boolean logic, you can engineer a lightweight, real-time visualizer that forces particles to mathematically wrap around complex geometry in milliseconds. 

---

## Phase 1: System Architecture & Initializing the Grid

To build a digital wind tunnel, you first need to generate the "air." In a procedural system, air isn't a volume, it's a highly dense array of independent data points.

Here is how I structured the initial emitter before feeding it into the solver:

<img width="1576" height="665" alt="Image" src="https://github.com/user-attachments/assets/86f89459-189c-4466-9964-067e302b07ed" />

* **The Array (Grid Node):** I generated a 5x5 meter mathematical grid and pushed the resolution to 200x200 vertices. This instantly gives us exactly 40,000 discrete coordinates in space. Let's treat these as our individual air particles.
* **Orientation (Transform Geometry):** A standard grid spawns flat on the floor. I rotated it 90 degrees on the X-axis to stand it up, acting as the primary inlet/fan facing the test geometry.
* **Data Isolation (Mesh to Points):** This is the crucial step. A standard grid is a connected mesh. If you push a mesh into a collision object, it stretches and tears. By piping it through a `Mesh to Points` node, I destroyed all the structural edges. Every single vertex is now a completely isolated, free-floating point in space with its own independent position data, ready to be injected with velocity.

---

## Phase 2: State Machine & Integration

If we want real-time aerodynamics, we need a system that remembers its own state frame-by-frame. Standard nodes only calculate once. To get continuous flow, I built the engine inside a **Simulation Zone**, which acts as a `while` loop executing discrete time steps.

Here is the exact kinematic integration loop:

<img width="1580" height="673" alt="Image" src="https://github.com/user-attachments/assets/fdb955c3-31de-4e7d-8326-c95af2096a83" />

* **The Loop (Simulation Input/Output):** Everything inside this zone recalculates 60 times a second. It reads the previous frame's data, applies the math, and outputs the new state.
* **Kinematic Update (Set Position):** We aren't "animating" anything. We take our calculated aerodynamic velocity vector and pipe it directly into the `Offset` socket. This mathematically translates the XYZ coordinates of every single particle through space.
* **System Memory (Store Named Attribute):** A particle has to remember how fast it is moving and in what direction so the next frame can build on it. By storing our final vector as a custom `Velocity` attribute, we give the particles memory.

---

## Phase 3: The Sensor & Vector Calculus (The Brains)

This is where the 12th-grade math actually executes. Standard particle systems are blind; they crash straight through collision meshes. We need a system to detect the geometry *before* impact and mathematically force the velocity vector to curve along the surface.

Here is the exact logic flow happening every single frame:

<img width="1571" height="669" alt="Image" src="https://github.com/user-attachments/assets/b0e23b66-fdae-4b41-89aa-89cafd935eac" />

**1. The Boolean Sensor (Raycast & Switch)**
The `Raycast` node acts as an invisible proximity sensor, almost like a RADAR. It fires a ray `0.12m` ahead of the particle's current trajectory. 
* If `Is Hit` returns **False**, the `Switch` node routes the default, unobstructed wind velocity. The air flows straight.
* If `Is Hit` returns **True**, it extracts the exact `Hit Normal` ($\hat{n}$) of the geometry and triggers the math.

**2. The Tangential Math (Project & Subtract)**
When the sensor triggers, we isolate the exact force of the crash and delete it. 
* **Project:** We take the initial wind velocity vector and project it onto the `Hit Normal`. This calculates the component of the air that is pushing *directly into* the mesh.
* **Subtract:** We mathematically subtract that crashing vector from our initial velocity. 

$$\vec{v}_{tangent}=\vec{v}_{initial}-(\vec{v}_{initial}\cdot\hat{n})\hat{n}$$

What is left over is pure tangential velocity. The particles now mathematically have no choice but to glide seamlessly across the surface.

**3. The Edge Case Bumper (Scale & Add)**
If you look closely at the top `Scale` node, there is a built-in fail-safe. If particles only slide tangentially, they can get trapped in tight, concave geometry (like the ears of the mesh) or clip through low-poly faces. To solve this, I took the `Hit Normal`, scaled it down by a microscopic `0.005`, and added it back to the tangential velocity. 

It acts as a fractional repulsion field, pushing the flow just slightly off the surface to prevent friction and clipping.

---

## Phase 4: Telemetry & Data Visualization (The Flow Lines)

A point in 3D space is just a coordinate. To actually visualize aerodynamic drag, we need to turn those coordinates into physical flow streaks that align with the wind direction.

Here is how the system renders the telemetry:

<img width="1574" height="553" alt="Image" src="https://github.com/user-attachments/assets/f5996b9b-7b9b-48bb-96df-c6bd1ef79ecc" />

**1. Building the Aerodynamic Streak (Curve to Mesh)**
Instead of rendering heavy 3D arrows, I used a `Curve Line` and gave it a microscopic `Curve Circle` profile (Radius: 0.002m). This creates an extremely lightweight, aerodynamic streak.

**2. Vector Alignment (Align Euler to Vector)**
This is the most crucial visualization step. A streak is useless if it doesn't point in the direction of the airflow. I pulled the custom `Velocity` attribute we stored inside the simulation loop, and plugged it directly into an `Align Euler to Vector` node. This mathematically rotates every single streak to face the exact trajectory of its travel vector.

**3. Instancing (Instance on Points)**
Finally, we instance that perfectly aligned 3D streak onto all 40,000 data points simultaneously. Because they are instances and not unique meshes, the memory footprint remains incredibly low, keeping the system running in real-time.

---

## Phase 5: Thermal Shader & Velocity Mapping

A digital wind tunnel is useless if you can't read the data. I built a custom material to visually map the aerodynamic drag in real-time.

Here is the telemetry breakdown:

<img width="1283" height="457" alt="Image" src="https://github.com/user-attachments/assets/bc042847-888d-4089-88e4-27ed1ac27794" />

**1. Extracting Speed (Attribute & Length)**
I pulled the custom `Velocity` vector attribute we generated in the Geometry Nodes. But color requires a scalar float, not a 3D vector. By passing it through a `Length` math node, I extracted the magnitude of the vector, giving us the absolute speed of every single particle.

**2. Calibration (Multiplier & Color Ramp)**
I multiplied that raw speed by `7.0` to calibrate the data range, and fed it into a Color Ramp. 
* **Cyan:** Fast-moving, unobstructed airflow.
* **Red:** Slow-moving, trapped air dragging behind the mesh (aerodynamic drag). 

**3. Engine Optimization (Light Path & Emission)**
To make the flow lines glow like a thermal camera without destroying the engine's frame rate, I routed the data into an `Emission` shader. By utilizing a `Light Path` node to drive the emission strength, the particles glow intensely for the camera but don't cast heavy, scene-crashing light bounces onto the rest of the geometry.

---

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/90c317c1-f909-42ac-9721-7af7cc88a70d" />

## System Limitations & The Next Iteration

This isn't a massive, multi-million dollar CFD simulator running on server farms. It is a highly optimized, real-time visualizer. Because of that, it has one distinct limitation, i.e, **Discrete Time Steps.**

The system calculates position exactly once per frame (60hz). It does not use sub-stepping. If the initial wind velocity is set too high, a particle can theoretically move so far in a single frame that it completely skips past the Raycast sensor's 0.12m detection radius and clips through thin, low-poly geometry. 

**The Current Workaround:** Keeping the collision mesh sufficiently subdivided ensures the normals are dense enough to catch the rays, and artificially capping the maximum inlet velocity keeps the system stable.

**Future R&D:**
* **Turbulence (Curl Noise):** Injecting mathematical curl noise into the post-collision velocity to simulate real-world micro-vortices and unstable air behind the object.
* **Multi-Bounce Raycasting:** Expanding the loop to allow for secondary aerodynamic bounces inside complex, enclosed geometry (like an intake manifold).
