---
title: Euler Angles
type: reference
tags:
  - imu
  - euler-angles
  - rotations
source:
  course: ME 405
  term: 2262
  lecture: 16
status: draft
---

## Motivation

Euler angles describe a set of three sequenced rotations that can be used to define an arbitrary orientation for an object in 3D space. Euler angles come in many different conventions because the sequence of rotations can be selected in many different ways. The common convention, sometimes called the aerospace convention, is the ZYX or 321 convention. Each rotation transforms between a set of coordinate frames.

| Euler Angle | Meaning                                   |
| ----------- | ----------------------------------------- |
| $\psi$      | Yaw or heading about a global $z_1$ axis. |
| $\phi$      | Pitch about an intermediate $y_2$ axis.   |
| $\beta$     | Roll about a local/body $x_3$.            |
$$
(x_1, y_1, z_1)
\quad \xrightarrow{\psi} \quad
(x_2, y_2, z_1)
\quad \xrightarrow{\phi} \quad
(x_3, y_2, z_2)
\quad \xrightarrow{\beta} \quad
(x_3, y_3, z_3)
$$
## Rotations
### Example 1

In this example a rotation matrix will be derived that performs rotation on basis vectors. For the common ZYX convention, the first rotation is yaw / heading by $\psi$ about the global $z_1$ axis.

The basis changes from the global frame to the first intermediate frame:

$$
(x_1, y_1, z_1)
\quad \xrightarrow{\psi} \quad
(x_2, y_2, z_1)
$$
The rotation will be determined using projection in the $(x_1,y_1)$ plane.

![Euler angle derivation for the ZYX convention as yaw by psi about global z1.](images/imu/euler_angle_psi_rotation.svg)

From the diagram, the new basis vectors are
$$
\begin{aligned}
\hat{x}_2 &= c_\psi \hat{x}_1 + s_\psi \hat{y}_1 \\
\hat{y}_2 &= -s_\psi \hat{x}_1 + c_\psi \hat{y}_1
\end{aligned}
$$
where
$$
c_\psi = \cos(\psi), \qquad s_\psi = \sin(\psi).
$$

This can be written as
$$
\begin{bmatrix}
\hat{x}_2 \\
\hat{y}_2
\end{bmatrix}
=
\begin{bmatrix}
c_\psi & s_\psi \\
-s_\psi & c_\psi
\end{bmatrix}
\begin{bmatrix}
\hat{x}_1 \\
\hat{y}_1
\end{bmatrix},
$$

and extended to 3D as

$$
M_z(\psi)
=
\begin{bmatrix}
c_\psi & s_\psi & 0 \\
-s_\psi & c_\psi & 0 \\
0 & 0 & 1
\end{bmatrix}
$$
when writing the new basis vectors in terms of the old basis vectors.

**Note**: similar analysis can be performed for the $\phi$ and $\beta$ rotations to get two additional rotation matrices.
#### Rotation Matrix Sign Convention

The matrix in this example is for transforming basis vectors. The next section uses matrices that act on vector components, which have the opposite sign convention for the $z$ rotation. The matrix that acts on vector components is the transpose of the matrix acting on basis vectors.

This difference is because vector components and bases are *contravariant*: for the geometric object that a vector represents to remain invariant under change of basis the components of the vector must change in an opposite manner as the basis vectors. For example, if you are standing on a rotating frame, like a merry-go-round, watching the world around you, it may appear as you are stationary and instead the world is rotating around you the opposite direction.

### Example 2:

This example shows how to use rotation matrices to rotate components of a vector. When rotating vector components, the elementary rotation matrices are

$$
M_z(\psi)
=
\begin{bmatrix}
\cos\psi & -\sin\psi & 0 \\
\sin\psi & \cos\psi & 0 \\
0 & 0 & 1
\end{bmatrix}
$$

$$
M_y(\phi)
=
\begin{bmatrix}
\cos\phi & 0 & \sin\phi \\
0 & 1 & 0 \\
-\sin\phi & 0 & \cos\phi
\end{bmatrix}
$$

$$
M_x(\beta)
=
\begin{bmatrix}
1 & 0 & 0 \\
0 & \cos\beta & -\sin\beta \\
0 & \sin\beta & \cos\beta
\end{bmatrix}
$$

These matrices act on components, not on bases. To transform from one frame to another premultiply by the appropriate matrix. To go from the global frame to the first intermediate frame the transformation is
$$
\underline{v} = M_z(\psi)\,\underline{w},
$$
where $\underline{w}$ is a vector in the $(x_1,y_1,z_1)$ global frame and $\underline{v}$ is the same vector expressed in the $(x_2, y_2, z_1)$ intermediate frame.

For the `ZYX` convention, the complete transformation is

$$
\begin{aligned}
\underline{v}
&=
M_x(\beta)\left(M_y(\phi)\left(M_z(\psi)\,\underline{w}\right)\right) \\
\underline{v}
&=
\left(M_x(\beta)M_y(\phi)M_z(\psi)\right)\,\underline{w} \\
\underline{v}
&=
M_{zyx}(\psi,\phi,\beta)\,\underline{w}
\end{aligned}
$$
where $\underline{w}$ is a vector in the $(x_1,y_1,z_1)$ global frame and $\underline{v}$ is the same vector expressed in the $(x_3, y_3, z_3)$ local/body frame.

To reverse the transformation, the inverse operations must be applied in reverse order,
$$
\begin{aligned}
\underline{w}
&=
M_z^{-1}\left(M_y^{-1}\left(M_x^{-1}\underline{v}\right)\right) \\
\underline{w}
&=
\left(M_z^{-1}M_y^{-1}M_x^{-1}\right)\underline{v}.
\end{aligned}
$$

Therefore,
$$
M_{zyx}^{-1}
=
M_z^{-1}M_y^{-1}M_x^{-1}.
$$

**Insight**:  *You must put your socks on before your shoes, but you must take your shoes off before your socks.* That is, when undoing a sequence of transformations, not only does each transformation invert but the sequence reverses as well.

### Gimbal Lock and Singular Matrices

One of the shortcomings of Euler angles is a phenomenon called gimbal lock. Gimbal lock occurs both physically, in gimbal systems, but also occurs mathematically. The phenomenon occurs when the second of the three sequenced rotations causes the axis of the first rotation and the third rotation to become collinear. For the ZYX convention this occurs when the pitch angle is $\phi = \pm 90^\circ$ because these conditions cause the $x_3$ axis to be aligned with the positive or negative extension of the $z_1$ axis.

When gimbal lock occurs, the matrix $M_{zyx}$ representing the total rotation from global to local coordinates becomes singular. The matrix becomes singular because there exist only two degrees of freedom in 3D space.

Gimbal lock is an issue in systems that undergo arbitrary 3D rotations; however, for systems with bounded rotation in 3D space, gimbal lock can easily be avoided by choosing the right Euler angle convention such that the expected range of motion for the second rotation axis does not cause gimbal lock.

Another strategy for avoiding gimbal is to use a [[reference_quaternions|Quaternion]] to represent the rotation instead of Euler angles.

## Insights

Euler angles are intuitive because yaw, pitch, and roll are easy to visualize, but they depend heavily on convention and rotation order. The same three angle values can mean different things under different conventions.