# The GPU
The only benefit over CPU is speed on graphics computations -> critical
- Now is highly programmable via algorithm, usually **shaders**
- Gain speed by focusing on narrow set of highly parallelizable tasks
	- Silicon for z0buffer
	- Rapid access for texture images and other buffers
	- Rapid finding pixels covered by a triangle

**Shader Core:** small processor that does relatively isolated task (e.g. transform a vertex, compute colour of pixel covered by triangle)
- Can be billions of **shader invocations**

**Latency:** Common issue. e.g. Accessing data may stall processor

## Data Parallel Architectures
**SIMD:** Single Instruction Multiple Data
- CPUs run mostly serially, have fast local cache, and avoid stalls using things like branch prediction, instruction reordering etc.
- GPUs have a large set of **shader cores** (processors) and is a **stream processor** (ordered sets of similar data processed)
	- Can process in massively parallel fashion
	- Invocations are as indep as possible
	- Optimized for **throughput** -> max rate data can be processed
	- However less cache + control logic area = more latency on shader cores compared to CPU
- Latency can be hidden via concurrency -> on block, process some other data
- SIMD is like this -> separate instruction from data, and executes same comman in lock-step on fixed number of shader programs
	- e.g. each pixel shader invocation is a **thread** -> contains input values and register spaces needed
	- Threads with same shader are bundled into **warps** or **wavefronts** (NVIDIA/AMD)
	- Warp/wavefront is scheduled on some # of GPU cores. (8-64)
	- In this way, each thread is mapped to a **SIMD lane**
	- On NVidia, warps have 32 threads, which execute simultaneously on 32 cores. Memory fetch stalls all 32 cores
	- Warps are also swapped in and out on stall
	- Threads have own registers and warp keeps track of current instruction. Swapping is just pointing cores to different warp -> no overhead
	- Because of this, warps swap out for short delays too. 
	- Fewer threads = fewer warps = hiding latency becomes problematic
- The number of registers used per thread has impact -> more registers, fewer threads can be on GPU at one time
	- The number of warps **in flight**, or the **occupancy** decreases
	- High occupancy = less likely to have idle processors
	- Low occupancy usually = poor performance
- **dynamic branching** -> if statements and loops
	- If even one thread in a warp takes alternate path, then both results must be computed, then discarded
	- This is **thread divergence** -> leaves other threads idle

## GPU Pipeline Overview:
Conceptual stages in Rendering Pipeline are implemented on the GPU via hardware stages
-This is a *logical* rather than *physical* model
- Reference Fig 3.2 on pg. 34 for summary

## The Programmable Shader Stage
Modern shader programs use a **unified shader design** -> vertex, pixel, geometry, and tessellation shaders share common programming model
- Have the same **instruction set architecture (ISA)**
- Porcessor that implements this is a **common-shader core (in DX)**
- A GPU that has these cores has a **unified shader architecture**

Shader processors can have many roles, and GPU should allocate appropriately -> better work distribution
- **Shading Languages:** DX - **High-Level Shading Language (HLSL)**, OpenGL - **OpenGL Shading Language (GLSL)** 
	- HLSL can compile to VM bytecode **intermediate language** (DXIL) -> hardware independent
	- also can compile and store offline

**Data Types:** 32-bit single precision floating point scalars and vectors (shader code, not hardware)
- Modern usually also 32-bit ints and 64-bit floats
- Aggregate data types are also supported

**Draw Call:** invokes graphic API to draw group of primitives -> shaders execute
- **uniform inputs:** values constant throughout draw call
- **varying inputs:** data that come from triangle's vertices or from rasterization
- e.g. colour of light source is uniform, the triangle's surface location is varying (per pixel)

The underlying VM has special registers for different types of IO
- A lot more **constant registers** for uniforms than varying IO
	- Varying need to be stored separately for each vertex -> limit to # needed
	- Uniform are stored once and reused across the draw call
- Also **temporary registers** -> scratch space

Operations common in graphics are efficiently run on modern GPUs, exposed by shading languages
- **intrinsic functions** optimized for GPU

**Flow control:** branching instructions. Shaders support 2 types
- **Static flow control:** based on values of uniform inputs -> flow is const over draw call
	- Allow same shader for variety of situation
	- No thread divergence
- **Dynamic flow control:** based on values of varying inputs -> each fragment can execute differently
	- More powerful but costs performance

## The Vertex Shader
First stage in the functional pipeline -> directly under programmer control
- Some data manip happens before this stage though.
- **input assembler (DX)** several steams of data can be woven together to form set of vertices/primitives sent through pipeline
	- e.g. combining position and colour arrays (so different combinations can be used)
	- **Instancing:** Object drawin several times with varying data per instance, but with single draw call

Triangle mesh = set of vertices on model surface (position, colour, texture, surface normal)
- Instead of triangle normal, vertex normals represent orientation of the actual underlying surface (not of the triangle mesh) -> better representation

Vertex shader is first stage to process triangle mesh -> does not know what triangles are formed, only operates on vertices
- modify, create, ignore values with each vertex
- Usually, transforms from **model space** to **homogeneous clip space** (minimum)
- Every vertex is processed, which then outputs that are interpolated across triangle/line
	- No create/destroy vertices
	- results cannot be passed to another vertex
- This independence = parallel processing of vertices on GPU

## The Tessellation Stage
For **rendering curved surfaces** -> Take surface description and turn into representative set of triangles
- Curve description more compact than triangles (less memory)
	- Prevent CPU to GPU bus bottleneck for things that change shape per frame
- Surface rendered efficiently -> appropriate # of triangles for given view
	- Further = less triangles
- Controlling the **level of detail (LOD)**
	- Control program performance -> less triangles on weaker GPU for frame rate
- Triangle mesh flat surfaces can be warped
- Also can be for reducing expensive shading computations

Three elements:
1. **Hull Shader (Tessellation Control Shader)**
	- Input  is a **patch primitive**-> control points defining a subdivision surface/Bezier patch/other curved element
	- Tells tessellator how many and what configuration triangles to generate
	- Processes each control point
	- May modify patch (add/remove control points)
	- Outputs set of control points and tesselation control data to domain shader
2. Tessellator (Primitive Generator):**
	- Fixed function stage, only used with tessellation shaders
	- Adds several new vertices for domain shader to process
	- Hull shader informs what type of tessellation surface is wanted:
		- triangle
		- quadrilateral
		- isoline (sets of line stripes)
	- Hull shader also sends **tessellation factors/levels** -> inner/outer edge
		- Inner: how much tessellation inside triangle/quad
		- Outer: how much each exterior edge is split
			- matching edges avoids cracks + shading artifacts where patches meet
			- adjacent curved surface matching
			- 0 or NaN indicates discard
	- Generates mesh and sends to domain shader
3. **Domain Shader (Tessellation Evaluation Shader):**
	- Control points from hull shader used by each invocation to compute output values for each vertex
	- Input vertex from tesselator and generates a corresponding output vertex
	- These triangles then passed down pipeline

Generally, just structured this way for efficiency. Each shader can be simple

## The Geometry Shader
Goal is to turn primitives into other primitives (e.g. triangle mesh into wireframe or quads)
- Usage is optional
- Input: single object and its associated vertices (triangles in a strip, a line segment, or a point)
	- Can also define extended primitives
		- e.g. 3 additional vertices outside a triangle (defining 3 more triangles) or the 2 adjacent vertices to the polyline (segment with 3 parts)
- Tessellation stage is still more efficient for patch generation
- Output: 0 or more vertices, treated as points/polylines/strips of triangles
	- 0 or more means mesh can be selectively modified
- For modifying incoming data OR making a limited number of copies
- DX11/OpenGL4.0 introduced ability for geometry shader to use **instancing** -> can run set # of times on primitive
- Can also output to up to 4 **streams** -> one of which is down the rendering pipeline
- Guaranteed FIFO (affects performance) -> don't use this to replicate/create large amount of geometry
- FULLY PROGRAMMABLE (however rarely used since does not map to GPU strengths)

**stream output:** processed vertices can be outputted in a stream (ordered array)on the side
- Data can be sent back through the pipeline (iterative procession)
	- useful for flowing water/other particle effects
	- also used to skin a model and have the vertices available for reuse
- only return in floating point format ->high memory cost
	- works on **primitives** and not vertices
	- Vertex sharing between primitives is lost
	- Usually just send vertices as primitives

## The Pixel Shader
Primitives are clipped and set up for rasterization , relatively fixed process (somewhat configurable)
- Traverse each triangle to find which pixels it covers
	- Can also determine area of pixel covered
	- A piece of triangle covering pixel is a **fragment**
- Values at vertices (including z-value) are interpolated across triangle surface for each pixel
	- Then passed to **pixel shader/fragment shader** which processes the fragment

Type of interpolation performed across triangle is specified by the pixel shader program
- **perspective-correct interpolation**
- **screen-space interpolation**: perspective projection not taken into account

Inputs are the vertex shader's outputs + some extras
- e.g. screen position, visible side of triangle 

Pixel shader typically computes and outputs a fragment's colour (also possibly opacity and modified z-buffer)
- Values used to modify what's stored at the pixel during merging

Pixel shaders can also discard incoming fragments + generate no output
- clip volumes

**Multiple Render Targets (MRT):** we can generate multiple sets of values + save to different buffers (**render targets**)
- Dependant on GPU could be 4 or 8 available
- For example, leads to **deferred shading** -> visibility and shading done in separate passes
	- First pass stores object location and material of each pixel
	- Successive passes then apply illumination + effects
- later pass can allow us to access data from neighbouring pixels generated in previous passes

Limitation: normally only able to write to render target at the specific fragment location, and cannot read from neighbouring pixels
- Not severe as we can have several passes. 
- Exceptions: can immediately access information for adjacent fragments during computation of gradient or derivative information
	- Pixel shader has amounts which any interpolated val changes per pixel along x and y axis -> texture addressing, picking mipmap levels?
	- Quad = 2x2 group of fragments -> when requesting gradient (slope) values, the difference between adjacent fragments is returned.
		- Gradient used by other things in pixel shader
		- Kept in different threads on the same warp -> therefore we cannot access gradients in shaders with dynamic control flow (since fragments will not execute the same instruction set, and some may not be meaningful for computing gradients)

Now there is buffer type *unordered access view (UAV)/shader storage buffer object (SSBO)* that allows write access to any location 
- This may create data race condition -> therefore GPU uses dedicated atomic units -> Therefore there may be stalling as we wait to read/modify/write
- May want specific order of execution -> *raster order views (ROVs)* to enforce order of execution
	- Like UAVs but guarantee proper order
	- Enable pixel shader to write own blending methods
	- However if out of order is detected, pixel shader may stall

## The Merging Stage
Depths and colours of fragments generated by pixel shader are combined with the frame buffer 
- Where stencil-buffer and z-buffer operations occur 
- Also does colour blending
- Blending of fragment and stored colour is common for transparency and compositing operations

Fragment shader operations on hidden fragments is a waste -> Most GPUs perform some merge testing before pixel shader is executed 
- Visibility is tested (with z-depth, stencil buffer, scissoring etc.) and fragment is culled if hidden
	- Called **early-z**
	- If fragment discard/z-depth modification exists in fragment shader, generally turn early-z off. 

This stage is configurable, bur not fully programmable -> colour blending can be set up to perform large number of different operations. 

API requires that this stage has guaranteed draw order (output invariance)

## The Compute Shader
Non-graphical uses of shaders in other fields -> GPGPU (CUDA, OpenCL)
- Basically GPU is just a massively parallel processor
- Use C or C++ with extensions + libraries made for GPU

**Compute Shader:** form of GPU computing -> Shader not locked into a location in the graphics pipeline
- Invoked by graphics API, so tied with rendering process
- Draws from same pool of unified shader processors as others in pipeline
- Warps + threads more visible (each gets a thread index in a **thread group**) -> e.g. loop parallelization
	- Thread groups specified by x, y, z coordinates (simplicity of use in shader code)
	- Each thread group also has small amount of memory that is shared (DX = 32kB)
	- Compute shaders are executed by thread groups -> all threads guaranteed to run concurrently

Advantage: they can access data generated on the GPU -> since sending data from GPU to CPU = delay, we have performance improvements if everything is on GPU
- **Post processing** is common use case
	- Shared memory means intermediate results from sampling image pixels can be shared with neighbouring threads.
- Determining distribution or average luminance of an image

Useful for particle systems, mesh processing (facial animation), culling, image filtering, improving depth precision, shadows, depth of field etc.

## Further Reading
