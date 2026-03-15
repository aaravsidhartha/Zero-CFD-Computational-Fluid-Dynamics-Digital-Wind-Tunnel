# Zero-CFD-Computational-Fluid-Dynamics-Digital-Wind-Tunnel
A lightweight, real-time digital wind tunnel built entirely with pure vector calculus and raycast sensors. Zero fluid simulations.

## The Problem: Removing Need For Brute Computational Fluid Simulations
If you want to run fluid simulations or test aerodynamics, you usually need massive compute power and hours to bake. Standard physics engines calculate pressure, viscosity, and density across millions of voxels. It is incredibly heavy and notoriously slow, especially if your laptop is a literal potato.

I didn't want to wait hours just to visualize flow lines over a 3D mesh. I needed a real-time digital wind tunnel that runs instantly.

## The Solution: First-Principles Math & Geometry Nodes
Instead of relying on dedicated CFD (Computational Fluid Dynamics) software, I built a custom solver inside Blender's Geometry Nodes. 

Why Geometry Nodes? Because it isn't actually a physics engine, it's a raw, procedural logic environment. It gave me the exact framework I needed to bypass default 3D collision physics entirely. By treating the 3D viewport as a blank coordinate grid, I could use the exact vector calculus I just studied for my 12th-grade board exams to write my own visual algorithm. No heavy fluid simulations needed.

## Why This Matters in Engineering
In engineering and software architecture, efficiency is everything. This project isn't about perfectly simulating real-world molecular air pressure, it's more about extreme optimization in times where you don't need pin-point precision results, just an overall understanding of the way the object and air are interacting.

It proves that you don't always need to brute-force a massive physics simulation to get results. If you understand the fundamental math, like vector projection and boolean logic, you can engineer a lightweight, real-time visualizer that forces particles to mathematically wrap around complex geometry in milliseconds. 

---

## Phase 1: System Architecture & Initializing the Grid

To build a digital wind tunnel, you first need to generate the "air." In a procedural system, air isn't a volume, it's a highly dense array of independent data points.

Here is how I structured the initial emitter before feeding it into the solver:

* **The Array (Grid Node):** I generated a 5x5 meter mathematical grid and pushed the resolution to 200x200 vertices. This instantly gives us exactly 40,000 discrete coordinates in space. Let's treat these as our individual air particles.
* **Orientation (Transform Geometry):** A standard grid spawns flat on the floor. I rotated it 90 degrees on the X-axis to stand it up, acting as the primary inlet/fan facing the test geometry.
* **Data Isolation (Mesh to Points):** This is the crucial step. A standard grid is a connected mesh. If you push a mesh into a collision object, it stretches and tears. By piping it through a `Mesh to Points` node, I destroyed all the structural edges. Every single vertex is now a completely isolated, free-floating point in space with its own independent position data, ready to be injected with velocity.

