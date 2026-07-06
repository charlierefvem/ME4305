---
title: Quaternions
type: reference
tags:
  - imu
  - quaternions
  - rotations
source:
  course: ME 405
  term: 2262
  lecture: 16
status: draft
---
# Motivation

Quaternions are a type of four-dimensional number that extends the behavior of complex numbers from the plane to three dimensions. Quaternions are what motivates the cross product often used in rigid body mechanics and other areas of engineering.

![A plaque that reads "Here as he walked by on the 16th of October 1843 Sir William Rowan Hamilton in a flash of genius discovered the fundamental formula for quaternion multiplication i^2=j^2+k^2+ijk=-1 and cut it on a stone of this bridge.](images/hamilton_plaque.png)

## Fundamentals

The fundamental law for quaternion multiplication is an extension of the definition of the complex unit to three dimensions.
$$
i^2 = j^2 = k^2 = ijk = -1
$$
The following identities are corollaries and should be familiar to anyone that has used a cross product before.
$$
\begin{aligned}
ij &= k & ji&=-k\\
jk &= i & kj&=-i \\
ki &= j & ik&=-j
\end{aligned}
$$
**Note**: quaternions are **anti-commutative**. That is, when you exchange the sequence of factors in a product the sign must flip.

A quaternion can be written as

$$
q = a + b\,i + c\,j + d\,k
$$
or as
$$
q = a + \underline{v}
$$
where $a$ is the real component and
$$
\underline{v} = bi + cj + dk
$$
is the vector or imaginary component.

The quaternion conjugate is
$$
q^* = a - b\,i - c\,j - d\,k
$$
or
$$
q^* = a - \underline{v}
$$

The squared magnitude is
$$
\lVert q \rVert^2 = qq^*
$$
which is similar to complex numbers.

## Example 1

In this example a quaternion will be used to rotate a vector.

Let $q = a + bi + cj + dk$ be a unit quaternion used for rotating vectors where $a=\cos\frac{\theta}{2}$, $b=\sin\frac{\theta}{2}\, x$, $c=\sin\frac{\theta}{2}\, y$, $d=\sin\frac{\theta}{2}\, z$ and $x^2+y^2+z^2 =1$.

Through substitution and factoring we can show
$$
q = \cos\frac{\theta}{2} + \sin\frac{\theta}{2} \left( x\, i + y\, j + z\, k \right)
$$
or
$$
q = \cos\frac{\theta}{2} + \sin\frac{\theta}{2} \hat{u}.
$$

This quaternion represents a rotation of angle $\theta$ about the unit vector
$$
\hat{u} = x\, i + y\, j + z\, k.
$$

The conjugate of $q$ is therefore
$$
q^* =
\cos \frac{\theta}{2}
-
\sin \frac{\theta}{2}
(x\,i + y\,j + z\,k)
$$
or
$$
q = \cos\frac{\theta}{2} - \sin\frac{\theta}{2} \hat{u}.
$$

To rotate a pure vector (a quaternion with no real component) defined by
$$
\underline{w} = \alpha\, i + \beta\, j + \delta\, k
$$
by the quaternion $q$, compute
$$
\underline{v} = q\,\underline{w}\,q^*
$$
The quaternion "sandwich" guarantees that for a pure vector $\underline{w}$, the product $q\,\underline{w}\,q^*$ is also a pure vector, meaning that it has no real component.

## Example 2

As a simple example, consider a vector lying in the $xy$-plane:
$$
\underline{w} = \alpha\, i + \beta\, j.
$$
To rotate $\underline{w}$ by angle $\theta$ about the $x$ axis, the rotation quaternion is
$$
q =
\cos \frac{\theta}{2}
+
\sin \frac{\theta}{2} i.
$$

After applying the quaternion "sandwich",

$$
q\,\underline{w}\,q^*
=
\alpha\, i + \beta\left(\cos\theta\,j + \sin\theta\,k\right).
$$

The $i$ component is unchanged because the rotation axis is the $x$ axis. The $j$ component rotates into a combination of $j$ and $k$.

![The example rotates w equals alpha i plus beta j by angle theta about the x-axis. A quaternion q equals cos(theta/2) plus sin(theta/2) i is written. Diagrams shows the original vector w and the rotated vector v.|700](images/imu/quaternion_rotation.svg)

# Insights

Quaternions are less visually intuitive at first, but they represent a 3D rotation as one axis and one angle. They are especially useful for orientation calculations because they avoid many of the bookkeeping problems associated with sequenced rotations.