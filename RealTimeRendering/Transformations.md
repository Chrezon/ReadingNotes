# Transforms

**Transform:** Operation that takes points/vectors/colours and converts them in some way
- Position, reshape and animate objects, lights and cameras
- Ensure all operations are in the same coordinate system
- Project objects onto a plane

**Linear Transform:** preserves vector addition and scalar multiplication
	- f(x)+f(y)=f(x+y)
	- kf(x) = f(kx)
- e.g. scaling + rotation transforms
- These can be represented with 3x3 matrices for 3-element vectors

Translation and shear is **not** linear -> it is **affine** (preserves parallelism of lines, but not angles or lengths)
- We store these transformations in 4x4 matrices and use **homogeneous notation**
	- Basically, adding an additional *w* element to allow for translations
		- Since we don't want to translate direction vectors, they are generally (x, y, z, 0)
		- Points are (x, y, z, 1)

In general, we use 4x4 matrices to represent transformations, since affine transforms are pretty useful.

## Basic Transforms
The notation here is in **column major form**

**Row major form:** Vectors are rows instead of cols. This is notation difference. The order of matrices would be reversed.
- From my understanding, this is slightly better data packing. 

**rigid-body transform:** preserves distance between points transformed, and preserves handedness (no flips)

### Translation
```
|1 0 0 x|
|0 1 0 y|
|0 0 1 z|
|0 0 0 1|
```

To move a point by (x, y, z). Notice how direction vectors, where w=0 is unaffected (0x = 0 therefore no transformation).

The inverse of T(t) is T(-t), or just negate x, y, z

In row major form, (x, y, z) would be along the bottom row followed by a 1. 

This is (clearly) a rigid-body transform.

### Rotation
Rotation along a given **axis** passing through the origin. Also a rigid body transform

**Orientation matrix:** rotation matrix associated with a camera view/objects that defines its orientation in space -> directions for **up** and **forward**

You can derive these equations by considering vectors in polar form, then rotating them about the desired axis

The common rotations are Rx, Ry, Rz, which rotates around the x, y, z, axes respectively

(Search the matrices up on the internet)

We can delete the rightmost and bottommost row/col to get a 3x3 matrix that rotates a 3-elem vector (notice that the deleted row/col is for working with the w-value)

We also know that the **trace** (sum of diagonal in matrix) is always 1+2cos(phi) for rotating phi degrees around any axis (this constant is axis independent)

Rotation will leave any point on the rotation axis unchanged (we can choose any axis, not just x, y, z, covered later)

Rotation matrices have **determinant = 1** and are **orthogonal** (holds for compound rotations too)

The inverse of R(phi) is R(-phi) -> rotate opposite direction around same axis)

If we want to rotate about a certain point (e.g. an axis that does not pass through origin), we want to translate that point to the origin, rotate, then translate back. Something like:
- T(p)R(phi)T(-p)
- Recall we apply transforms from rightmost to leftmost

### Scaling
Suppose we want to scale by (sx, sy, sz), we use this matrix:
```
|sx 0  0  0|
|0  sy 0  0|
|0  0  sz 0|
|0  0  0  1|
```
**Uniform:** sx=sy=sz (nonuniform otherwise)
- Also **isotropic/anisotrpic** 

Inverse is S(1/sx, 1/sy, 1/sz)

We can also do uniform scaling by using this matrix:
```
|1  0  0  0 |
|0  1  0  0 |
|0  0  1  0 |
|0  0  0 1/x|
```
This manipulates the w factor (again only scales points)
- We must then perform **homogenization** which contains division (inefficient)

If any of the values are negative, we have a **reflection matrix**
- even number of reflections seem to give rotations
- Rotation concatenated with reflection is reflection 
- Reflections need special treatment -> order of vertices is reversed (CCW -> CW). This causes incorrect lighting + backface culling. 
- If the **determinant of the upper 3x3 is negative** the matrix is reflective. 

If we want to scale along an axis that is NOT x, y, z, we need to change the **basis**, then use standard scaling, then transform back
- X = FS(s)F^T

### Shearing
Used to warp a models appearance. There are 6 basic shearing matrices based on the axis of shear (2 axes each)
- First is the axis where coordinates will change
- Second is the coordinates that does the shearing (how far the shearing moves, the smaller the coord on this axis, the smaller the shear)

e.g. Hxz
```
|1 0 s 0|
|0 1 0 0|
|0 0 1 0|
|0 0 0 1|
```
Notice in the above, the value on the x axis is modified based on the value in the z axis (hence Hxz), note that z does not change

The inverse of Hij(s) is Hij(-s)

We can also do something like:
```
|1 0 s 0|
|0 1 t 0|
|0 0 1 0|
|0 0 0 1|
```
Where coordinates on 2 axes are sheared by the third. (Notice this is just the product of 2 of the original matrices)

The determinant of a shear matrix is always 1 -> shear is **volume preserving**

### Concatenation of Transforms
ORDER MATTERS -> in general, the composite matrix C is as follows:
- **C = TRS**
- Apply **scaling -> rotation -> translation**
- sometimes this can be grouped as (TR)(Sp), since we want a single rigid-body transform -> concatenation is **associative**

### Rigid-Body Transform 
Shape of the object is not affected, only **orientation** and **location**
- Rigid body transforms are characterized by **preserving lengths, angles, and handedness**

From above, a rigid-body transformation can be written as the concatenation of a rotation and a translation, or:
```
|r00 r01 r02 tx|
|r10 r11 r12 ty|
|r20 r21 r22 tz|
| 0   0   0   1|
```
The inverse of X=T(t)R is R-1T(t)-1 or R^T T(-t)
- upper 3x3 of rotation is transposed, and the translation change sign. These are then multiplied in reverse to get inverse

**e.g. Orienting the Camera**
Orienting the camera is clearly a rigid body transformation. 

TODO: Finish this part later

SKIPPED TO EULER TRANSFORM
















