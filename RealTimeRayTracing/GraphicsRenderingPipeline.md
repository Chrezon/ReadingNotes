# Introduction
Real-time rendering produces what the viewer sees as a **dynamic process**
- **Frames Per Second** (FPS) or **Herts** (Hz)
	- Games: 30/60/72+ FPS -> user focus on action and reaction
- Refresh rate is separate from display rate
- 10ms of temporal delay = interfere with interaction
	- VR -> 90 FPS to minimize latency
- **Interactivity** and **connection** to the 3D space = sufficient conditions for real time rendering
- **Grphics acceleration hardware** is often included in the definition -> although most devices now come with graphics processors

TODO: Go back and review the math section

# The Graphics Rendering Pipeline
## Introduction:
**Graphics Rendering Pipeline:** Function is to render a **2D image** given a **virtual camera**, **3D objects**, **light sources** + more
- **Location and Shape:** Depends on geometry, characteristics of environment, camera placement
- **Appearance:** Depends on material properties, light sources, textures, shading equations

## Architecture:
Stages execute in **parallel** and each stage depends on results of previous stage
- Idealy, we gain a speedup of *n* for *n* stages
- Stages are **stalled** until the slowest stage finishes -> **bottleneck**
	- The stage after is **starved**
Real time popeline is divided into 4 mian stages:
1. **Application:** What's running, implemented in software running on CPUs, usually with multiple *threads*
	- Common Tasks: Collision detection, global acceleration algorithms, animation, physics etc.
2. **Geometry processing:** Generally on GPU, start of GPU side of pipeline
	- Common Tasks: Transforms, projections, other geometry handling
3. **Rasterization:** Given 3 vertices (triangle) find all pixels that are in the triangle
4. **Pixel Processing:** program per pixel to determine its colour/visibility
	- Also blending + other operations per pixel

