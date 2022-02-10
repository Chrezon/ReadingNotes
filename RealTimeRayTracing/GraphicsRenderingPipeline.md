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

Real time popeline is divided into 4 main stages:
1. **Application:** What's running, implemented in software running on CPUs, usually with multiple *threads*
	- Common Tasks: Collision detection, global acceleration algorithms, animation, physics etc.
2. **Geometry processing:** Generally on GPU, start of GPU side of pipeline
	- Common Tasks: Transforms, projections, other geometry handling
3. **Rasterization:** Given 3 vertices (triangle) find all pixels that are in the triangle
4. **Pixel Processing:** program per pixel to determine its colour/visibility
	- Also blending + other operations per pixel

### The Application Stage
Developer has full control as it executes on CPU -> control implementation and performance
- Changes here also change performance of later stages
	- e.g. Acceleration algorithms (certain culling algorithms)

Some work can be done by GPU with **compute shaders**
- Treat GPU as highly parallel general processor

At the end, **rendering primitives** are fed to geometry processing stage
- Include points, lines, triangles etc.

Software based -> not divided into sub stages. However, parallelism is usually used -> **superscalar** construction (Section 18.5)

**Collision Detection** is commonly implemented, as well as handling  various input

### Geometry Processing
Responsible for **per-triangle** and **per-vertex** operations. It is divided into 4 functional stages:
1. **Vertex Shading:** compute position of vertex and evaluate the vertex output data
2. **Projection:** Transforms the scene for clipping
3. **Clipping:**
4. **Screen Mapping:**

#### Vertex Shading
Here,  we compute the position for a vertex and evaluate the vertex output data specified by the programmer (e.g. normal, texture coordinates)
- Traditionally: vertex shading -> only compute lights to vertices based on location and normal. Store colour only in vertices and interpolate across the triangle
- Now, this is just the programmable vertex processing unit -> much more general, goal to set up data. 

Vertex position is a **set of coordinates**
- A model can be transformed into many *spaces* or *coordinate systems*
	- **Model Space** - no transformations
	- A **model transform** describes the models position and orientation
		- Can have multiple if we have multiple *instances* of the same model with different location/orientation/size
- Vertices are transformed by the model transform
	- After which the model is in **world coordinates/space**
	- World space is unique, all models in scene exists in the same space
- Canera also has a world space location and orientation
	- Everything is transformed with the **view transform** -> place camera at origin and aim it at -z, y-up, x-right
	- This is **camera/view/eye space**
- Both these transforms are **4x4 matrices**

Vertex shading step also models objects' appearance -> materials, light effects called **shading** and is computed using a **shading equation**
- Shading equations can be processed here or at per-pixel processing
- A set of data is stored (e.g. normal, color) and from this shading results are calculated and sent to next stages

#### Projection
Technically part of vertex shading. Transforms view volume into unit cubes (generally [-1, 1] on all axes)
- This is the **canonical view volume**

**Orthographic/Parallel Projection:** No perspective -> parallel lines
- View volume is usually a rectagular box, which is transformed into unit cube
- Combination of translation and scaling
**Perspective Projection:** With perspective -> convergence
- View volume is a **frustrum** (truncated pyramid) also transformed into unit cube

Projection is expressed as a 4x4 matrix + sometimes concatenated with other geometry transforms
- After projection, models are in **clip coordinates**
- Usually z-coordinates are stored in **z-buffer** rather than generated image -> projection from 3D to 2D

#### Optional Vertex Processing
1. **Tesselation**: Generate curved surface with appropriate number of triangles -> related to LOD
2. **Geometry Shading**: Also takes primitives to produce new vertices. Used for particle generation
3. **Stream Output**: Use GPU as geometry engine. Output processed vertices for further processing (by CPU or GPU)
Depends on capabilities of hardware and programmer need. These are independent steps. (Not commonly used)

#### Clipping
Which primitives are actually in the view volume (and thus should be passed on)?
- Fully inside are passed as-is
- Fully outside are dropped
- Partially inside requires **clipping**
	- We want vertices outside the view volume to be replaced by vertices at the intersection
	- Recall projection step -> clip against unit cube
	- Normally 6 clipping planes (user can define additional)

Clipping uses **4-value homogeneous coordinates** produced by projection
- In general values do not interpolate linearly for perspective space so we need 4th coordinate
- **Perspective division** places triangles' positions into **3D Normalized Device Coordinates** (or window coordinates)

#### Screen Mapping
x and y-coordinates transformed to **screen coordinates**. This plus the z-coordinates is the **window coordinates**
- e.g. translate followed by scaling. z-coords are also remapped to [0, 1] usually. 

Usually pixels are described with *integers* and *floats*
- Left edge of leftmost pixel is at 0.0 (OpenGL adn DX10+), same with 1.0, 2.0, etc.
- Center of a pixel is 0.5
- e.g the leftmost 10 pixels [0, 9] cover [0.0, 10.0)
- Simply: discrete = floor(cont), cont = discrete + 0.5
- OpenGL treats lower left as (0,0), DX (sometimes) treats upper left as (0,0), take care when moving APIs

### Rasterization
Given the transformed and projected vertices, want to find all pixels that are inside the primitive being rendered. This has 2 functional stages:
1. **Triangle Setup**
2. **Triangle Traversal**

#### Triangle Setup
Differentials, edge equations, and other data for triangle are computed. Done by fixed-function hardware

#### Triangle Traversal:
Finding which samples/pizels are inside a triangle. A **fragment** is generated for the part of the pixel that overlaps the triangle.
- Each fragment's properties are interpolated between vertices

### Pixel Processing
Per-pixel/per-sample computations and operations are performed on pixels or samples that are inside a primitive
1. **Pixel Shading**
2. **Merging**

#### Pixel Shading
Per-pixel shading computations, using interpolated shading data as input
- Results in 1+ colours to be passed onto next stage
- Executed by programmable GPU cores -> programmer supplies a program (pixel/Fragment shader)
	- E.g. **Texturing**

#### Merging
Information per pixel is stored in **colour buffer** -> rectangular array of colours (RGB components)
- **Merging** combines fragment color (pizel shading) with current buffer-stored colour
- Called **ROP** or **Raster Operations Pipeline** or **render output unit**
- GPU not fully programmable here, but configurable
- Also resolves visibility -> z-buffer (stores z-value of curr closest primitive)
	- z-buffer has *O(n)* convergence where n = num of primitives
	- Works when z0value can be computed for all relevant pixels
	- Primitives can be rendered in any order
	- No transparent primitives (do after opaque in back-to-front order, or another algo)
- *alpha channel* stores opacity value for the pixel -> used to be used for discard operation
	- Now a part of pixel shader program
- *stencil buffer* -> offscreen buffer to record locations of rendered primitives
	- 8 bits/pixel
	- Contents used to control rendering into colour and z-buffer
	- Pretty literally a stencil -> special effects
- These functions at end of pipeline are  **raster operations (ROP)** or **blend operations**
	- e.g. mix colour buffer with pixel -> transparancy
- **framebuffer** is all buffers on a system

After rasterizer stage, the screen displays primitives visible from POV of camera
	- **Double Buffering** -> scene rendering is offscreen in **back buffer** and once rendered, swapped with **front buffer** which was displayed on screen
	- Swapping via **vertical retrace**

## Further Reading
**Trip Down the Graphic Pipeline** -> software renderer from scratch
**OpenGL Programming Guide (Red Book)**
