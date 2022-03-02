# OpenGL
OpenGL is a **specification** that has implementations across different graphics cards that are maintained by the producers. 

**Immediate Mode:** Referred to as the **fixed function pipeline** -> easy way to draw graphics, but functionality is hidden
**Core-Profile Mode:** More low-level flexibility and more control

Generally, we don't need to use latest version of OpenGL, since only the most modern cards will support it

**Extensions** can be implemented in drivers and used. Developers need to query whether the extension is available before using

OpenGL can be thought of as a **State Machine**, where a collection of variables define how it operates (the **context**)

OpenGL libraries are written in **C** but it supports **objects** -> collection of options that represent a subset of OpenGL's state

# Creating a Window and Hello Window
Mostly setup content. Happily, my mac still supports OpenGL. My setup and CMake files can be found in the [OpenGLRasterizer](https://github.com/Chrezon/OpenGLRasterizer) repo

# Hello Triangle
Transforming 3D coordinates to 2D pixels is managed by the graphics pipeline (see reading notes)
- Transform 3D coordinates to 2D coordinates
- Transform 2D coordinates to pixels

Some of the shaders in graphics pipeline is customizable -> control over GPU, save CPU cycles
- Shaders are written in GLSL (OpenGL Shading Language)
- VERTEX SHADER -> Shape Assembly -> GEOMETRY SHADER -> Rasterization -> FRAGMENT SHADER -> Tests and Blending
- All caps = modifiable sections (again, see reading notes for what each stage does)
- (also tessellation and transform feedback loop exists)

OpenGL processes normalized device coordinates [-1.0, 1.0]
- transformed to screenspace coords via viewport transform -> specify data via glViewport

After defining vertex data, we send it to vertex shader
- Create memory on GPU to store the data
- Configure how OpenGL should interpret the memory
- Specify how to send data to graphics card

Then vertex shader can process this data from its memory. 

Memory is managed via VBO (Vertex Buffer Objects)
- stores large number of vertices in GPU memory -> can send large batches of data at once (since sending data is slow)

See initialization in OpenGLRasterizer

## Vertex Shader and Fragment Shader
We must set up a vertex shader and a fragment shader in order to use modern OpenGL. Shaders are written in GLSL (OpenGL Shading Language). To use a shader, OpenGL has to dynamically compile it at runtime from source code

Colours are very important when coding a fragment shader
- colours in OpenGL are represented by RGBA, where each (float) value is between 0.0 and 1.0

Linking into a **shader program object** is required to use recently compiled shaders
- then we can activate this shader program when rendering objects

## Linking Vertex Attributes
Recall that vertex shader allowed us to specify any input we weanted as vertex attributes. We need to specify which parts of input data belongs to which vertex attribute for the vertex shader. 

In our example, we know that we have 3 vertices of 3 32-bit floats

## Uniforms:
Like global variables -> have the same value for all vertices/fragments
