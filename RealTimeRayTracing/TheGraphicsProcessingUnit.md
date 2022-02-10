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

