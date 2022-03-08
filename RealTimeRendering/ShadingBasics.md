# Shading Basics

We want out models to have the right visual appearance. To achieve this, we need shading

## Shading Models
**Shading Model:** Describes how the object's colour should vary based on other factors
- e.g. **Gooch shading model:** non-photorealistic "cool to warm" shading for technical illustrations
	- Compare surface normal to light location
		- If towards -> warmer tone
		- If away -> cooler tone
	- Angles in between interpolate between the tones
- Have **properties** that control appearance variation -> setting these values determine appearance
	- e.g. surface colour, surface normals (usually normalized)
- Common functions: Clamping (usually between 0 and 1), Linear Interpolation (lerp/mix), reflect (reflected light vector)

## Light Sources
Lighting can be complex (multiple sources with own size/shape/colour/intensity/direct/indirect)

Shading model may react to presence/absense of light in a binary/continuous way -> distance from light, shadowing
- Usually continuous = interpolated between absence + full -> bounded range for intensity
	- Common option is to add an intensity *k* times appearance when fully lit to the appearance when unlit (c_shaded = unlit appearance + sum of(intensity * lit appearance))
	- Usually used for light not explicitly placed (e.g. skybox or bounce light)
- Combination of factors to distinguish the cases (e.g.):
	- Distance from light source
	- Shadowing
	- Facing towards/away from light source

Effect of light on a surface can be visualized as the **density** of rays
- The more dense, the more intense the light
- It is **inversely proportional** to the **cosine between the normal and light direction**
	- The smaller the angle, the more dense the light
	- recall this is just the dot product
	- This is also why we define **light vector** to be opposite to its direction of travel**
	- If the number is negative, the light is behind the surface -> clamp negatives to zero
	- This is **lambertian** shading -> ideal diffusely reflecting surfaces

Light interacts with models via 2 parameters: The light vector **l** and the light colour **c**

Popular light sources, as seen from the shaded surface location is an exact **point** (point light), but there are also area lights.
