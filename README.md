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

It proves that you don't always need to brute-force a massive physics simulation to get results. If you understand the fundamental math—like vector projection and boolean logic, you can engineer a lightweight, real-time visualizer that forces particles to mathematically wrap around complex geometry in milliseconds. 

---
