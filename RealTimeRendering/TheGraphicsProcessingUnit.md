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



















