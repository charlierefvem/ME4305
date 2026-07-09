---
title: Path Planning and Dynamic Trajectory Generation
type: topic
tags:
  - path-planning
  - differential-drive
  - nonholonomic
  - trajectories
  - splines
  - romi
status: draft
---

## Point-Wise Transformation and Path Planning

Consider a thought experiment where Romi drives along two very similar paths, starting from the same beginning location with the same original orientation.
1) Romi drives forward for one unit, turns left in an arc of unit radius, then drives forward for three units.
2) Romi drives forward for three units, turns left in an arc of unit radius, then drives forward for one unit.

![A diagram showing Romi in a starting location and orientation and again in two final locations and orientations. The path Romi has taken between the start and each end are also shown as dashed lines.](images/romi/thought_experiment.svg)

In both scenarios Romi has driven in an L-shaped path; what information can we conclude by comparing the state before and after the path? Consider the following conceptual questions regarding this thought experiment:
* Is it possible to determine the final orientation or location of Romi, $\begin{bmatrix}X(t_f) & Y(t_f) & \psi(t_f)\end{bmatrix}^T$, after it has driven on a path by exclusively comparing the final values for the two wheel displacements, $\begin{bmatrix}s_L(t_f) & s_R(t_f)\end{bmatrix}^T$, to the initial wheel displacements, $\begin{bmatrix}s_L(t_0) & s_R(t_0)\end{bmatrix}^T$?
  > [!spoiler]-
  > It is possible to determine the final orientation of Romi, but not the final location.
* If not, would it be possible to determine the final orientation or location of Romi using a time-history of wheel displacements, $\begin{bmatrix}s_L(t) & s_R(t)\end{bmatrix}^T$?
  > [!spoiler]-
  > With a time-history it is possible to determine both the final location and orientation of Romi.
* If, instead, you know the initial and final location and orientation of Romi can you determine the change in displacement at each wheel? What if you know a time-history of the location and orientation?
  > [!spoiler]-
  > With only information about the starting and final states it is possible to determine the *difference* between left and right wheel displacements, but not the value of either, as long as the heading is known as an *unwrapped* value. If the heading is only known between $0$ and $2\,\pi$ then the difference in wheel displacements can only be known mod $2\,\pi\,w$ where $w$ is the track-width. If a full time-history is known then both wheel displacements can be determined explicitly.
### Configuration-Dependent Transformation

In conclusion, it appears that there is some information lost when transforming between local states, known in Romi's body frame, and global states, known in the global frame.

The apparent information loss does not occur because the local-to-global transformation is invalid. It occurs because endpoint wheel displacements are a compressed summary of the motion. They preserve the total amount of wheel travel, but discard the order in which translation and rotation occurred.

Therefore, the transformation into the global frame must be applied increment by increment using the heading $\psi$ which is different at each increment. Applying the transformation only at the endpoints discards the sequencing information needed to reconstruct $X$ and $Y$ in the global frame. Mathematically, this appears because the transformation from increments in wheel displacement to increments in global pose depends on $\psi$.

**Insight**: while it is not possible to apply the transformation once to an entire path by transforming the difference between endpoints, the transformation *can* be applied to the entire path by integrating pointwise along the path.

This is similar to the distinction between conservative and non-conservative fields in vector calculus and physics. If a differential relationship is conservative, the accumulated change depends only on the endpoints. But the differential-drive robot’s global position is obtained by integrating local motion through a changing heading. Those differential relationships are path dependent, so the endpoint values of $s_L$ and $s_R$ are not enough to determine $X$ and $Y$.

### Constant-Heading Special Case

If the robot drives without turning, the body frame remains aligned with a fixed direction in the global frame. The transformation from body displacement to global displacement is constant in this special case, so the net body-frame displacement can be transformed directly into net global displacement. Once the robot turns, however, the transformation changes with $\psi$. The same total distance traveled can produce different global displacements depending on when that distance was traveled relative to the heading changes.

When the heading is constant,
$$
\psi(t)=\psi_0,
$$
the transformation becomes constant:
$$
\begin{bmatrix}
\mathrm{d}X\\
\mathrm{d}Y
\end{bmatrix}
=
\begin{bmatrix}
\frac{1}{2}\cos\psi_0 & \frac{1}{2}\cos\psi_0\\
\frac{1}{2}\sin\psi_0 & \frac{1}{2}\sin\psi_0
\end{bmatrix}
\begin{bmatrix}
\mathrm{d}s_L \\
\mathrm{d}s_R
\end{bmatrix}.
$$

Now the differential relationship can be integrated directly:
$$
\begin{aligned}
\int dX
    &= \int \frac{1}{2} \cos\psi_0\, \mathrm{d}s_L
    + \int \frac{1}{2} \cos\psi_0\,\mathrm{d}s_R \\
\Delta X
    &= \frac{1}{2}\cos\psi_0
    \left(
        \int \mathrm{d}s_L
        + \int \mathrm{d}s_R
     \right) \\
    &= \frac{1}{2} \cos\psi_0\,\left( \Delta s_L + \Delta s_R \right), \\[4pt]
%
\int dY
    &= \int \frac{1}{2} \sin\psi_0\, \mathrm{d}s_L
    + \int \frac{1}{2} \sin\psi_0\,\mathrm{d}s_R \\
\Delta Y
    &= \frac{1}{2}\sin\psi_0
    \left(
        \int \mathrm{d}s_L
        + \int \mathrm{d}s_R
     \right) \\
    &= \frac{1}{2} \sin\psi_0\,\left( \Delta s_L + \Delta s_R \right).
\end{aligned}
$$

However, we also know that
$$
\Delta s = \frac{1}{2}\left( \Delta s_L + \Delta s_R \right),
$$
so the changes in displacement simplify to 
$$
\begin{aligned}
\Delta X &= \cos\psi_0\,\Delta s, \\
\Delta Y &= \sin\psi_0\,\Delta s.
\end{aligned}
$$

Therefore, in the special case of fixed heading, the endpoint value $\Delta s$ is enough to determine global displacement:
$$
\begin{bmatrix}
\Delta X\\
\Delta Y
\end{bmatrix}
=
\begin{bmatrix}
\cos\psi_0 \\
\sin\psi_0
\end{bmatrix}
\Delta s.
$$

In summary,
1. If heading is fixed, body-frame displacement maps cleanly into global displacement because the transformation remains the same at each point along the path.
2. If heading changes, the transformation changes during the path.
3. Therefore, the transformation must be applied increment-by-increment.

## Local Approximation and Waypoint Construction

This suggests a practical approximation. Although the transformation from local robot motion to global motion changes with heading, over a sufficiently short interval the heading can be treated as approximately constant. During that interval, the robot’s forward displacement maps to a short straight-line displacement in the global frame. A longer trajectory can then be represented as a sequence of these short local motions connected at waypoints. As the waypoint spacing becomes smaller, this piecewise-straight construction approaches the continuous integral model of the robot’s motion.

For one small segment,
$$
\begin{bmatrix}
\Delta X_k \\
\Delta Y_k \\
\Delta \psi_k
\end{bmatrix}
\approx
\begin{bmatrix}
\frac{1}{2}\cos\psi_k & \frac{1}{2}\cos\psi_k \\
\frac{1}{2}\sin\psi_k & \frac{1}{2}\sin\psi_k \\
-\frac{1}{w} & \frac{1}{w}
\end{bmatrix}
\begin{bmatrix}
\Delta s_{L,k} \\
\Delta s_{R,k}
\end{bmatrix}
$$

The trajectory can therefore be approximated as a sequence of short straight segments using a forward-Euler style odometry update:
$$
\begin{bmatrix}
X_{k+1} \\
Y_{k+1} \\
\psi_{k+1}
\end{bmatrix}
\approx
\begin{bmatrix}
X_k \\
Y_k \\
\psi_k
\end{bmatrix}
+
\begin{bmatrix}
\frac{1}{2}\cos\psi_k & \frac{1}{2}\cos\psi_k \\
\frac{1}{2}\sin\psi_k & \frac{1}{2}\sin\psi_k \\
-\frac{1}{w} & \frac{1}{w}
\end{bmatrix}
\begin{bmatrix}
\Delta s_{L,k} \\
\Delta s_{R,k}
\end{bmatrix}.
$$

This is a first-order approximation: the current heading is used to transform the local displacement over the next short interval, and then the heading is updated for the next interval.

The global path is reconstructed by repeatedly applying the local transformation at each waypoint. The key is that $\psi_k$ is updated along the way, so the transformation is not fixed for the whole trajectory; it is only treated as fixed locally.

In the limiting case, as the segment length goes to zero,
$$
\Delta s_{L,k} \to \mathrm{d}s_L,
\qquad
\Delta s_{R,k} \to \mathrm{d}s_R,
$$
and the waypoint update becomes the differential model:
$$
\begin{bmatrix}
\mathrm{d}X\\
\mathrm{d}Y\\
\mathrm{d}\psi
\end{bmatrix}
=
\begin{bmatrix}
\frac{1}{2}\cos\psi & \frac{1}{2}\cos\psi \\
\frac{1}{2}\sin\psi & \frac{1}{2}\sin\psi \\
-\frac{1}{w} & \frac{1}{w}
\end{bmatrix}
\begin{bmatrix}
\mathrm{d}s_L \\
\mathrm{d}s_R
\end{bmatrix}.
$$

Over the whole path,
$$
\begin{bmatrix}
X(t_f)\\
Y(t_f)\\
\psi(t_f)
\end{bmatrix}
-
\begin{bmatrix}
X(t_0)\\
Y(t_0)\\
\psi(t_0)
\end{bmatrix}
=
\int_{t_0}^{t_f}
\begin{bmatrix}
\frac{1}{2}\cos\psi & \frac{1}{2}\cos\psi \\
\frac{1}{2}\sin\psi & \frac{1}{2}\sin\psi \\
-\frac{1}{w} & \frac{1}{w}
\end{bmatrix}
\begin{bmatrix}
\dot{s}_L \\
\dot{s}_R
\end{bmatrix}
\mathrm{d}t.
$$

The constant-heading example is the path-independent special case. The changing-heading case is the path-dependent general case.

### Path versus Trajectory

A path specifies where the robot should go in the plane. A trajectory specifies how the robot moves through that path over time, including heading, velocity, and wheel motion. For a non-holonomic robot, the trajectory matters because not every geometric path can be followed with arbitrary orientation and timing.

A path with sharp corners is geometrically valid, but it is not smoothly traversable by a differential-drive robot unless the robot stops and pivots in place at the corner. If we want the robot to move continuously, the path should provide at least tangent continuity. Cubic splines are useful because they can enforce position and tangent (velocity) constraints. Quintic splines can also constrain endpoint accelerations. For a planar trajectory, those second-derivative constraints can be used to influence or match curvature, which is important because curvature determines the required relationship between the left and right wheel speeds.

### Curvature and Differential-Drive Motion

For a differential drive, curvature connects directly to wheel velocity difference. If the path is parameterized by arc length $s$,
$$
\kappa(s)=\frac{\mathrm{d}\psi}{\mathrm{d}s}.
$$

For the differential-drive robot,
$$
\mathrm{d}\psi=\frac{\mathrm{d}s_R-\mathrm{d}s_L}{w},
$$
and
$$
\mathrm{d}s=\frac{\mathrm{d}s_R+\mathrm{d}s_L}{2}.
$$

Therefore, curvature is
$$
\kappa
=
\frac{\mathrm{d}\psi}{\mathrm{d}s}
=
\frac{2}{w}
\frac{\mathrm{d}s_R-\mathrm{d}s_L}
{\mathrm{d}s_R+\mathrm{d}s_L}.
$$

In velocity form,
$$
\kappa
=
\frac{\dot\psi}{v}
=
\frac{\frac{v_R-v_L}{w}}{\frac{v_R+v_L}{2}}
=
\frac{2}{w}
\frac{v_R-v_L}{v_R+v_L}.
$$

That gives a strong reason why matched curvature matters: discontinuous curvature implies a discontinuous required ratio between left and right wheel velocities. A real robot can approximate that, but not instantaneously.

### Splines as the Practical Response to Non-Holonomic Kinematics

The differential-drive model shows that global position cannot be recovered from endpoint wheel displacements alone. The robot’s local motion must be accumulated along the path using the current heading. This motivates representing paths as sequences of local geometric constraints rather than as one large endpoint-to-endpoint transformation. At coarse resolution, those constraints might be waypoints connected by straight segments. At higher resolution, we want smooth curves whose tangent direction changes continuously. This leads naturally to spline-based path generation.

The non-holonomic constraint does not prevent the robot from reaching arbitrary nearby positions, but it does mean that the path and trajectory matter. The robot cannot simply “teleport” sideways onto a desired curve. It must arrive through a sequence of forward motions and heading changes. Smooth path planning is therefore about constructing geometric instructions that are compatible with the robot’s local motion constraints and can be converted into reasonable wheel commands. The figure below shows several possible paths for Romi to travel from point A to point B; some of the paths require Romi to pivot in place at the start or end of the path.

![A diagram showing multiple paths Romi can take to travel between two points. Four paths are shown: the first is a straight line between the points, requiring Romi to pivot at the start and end; the second is a smooth s-curve that is tangent to Romi's heading at the start and end, requiring no pivoting; the third is a curve that is tangent to Romi's heading at the start, but not at the end, requiring Romi to pivot at the end of its path; finally, the fourth is a curve similar to the third, but the pivot is required at the start instead of the end of the path.](images/romi/path_comparison.svg)

## Cubic and Quintic Splines

Splines are curves generated from endpoint constraints. A trajectory can be created using a spline by using geometric and kinematic constraints at the start and end of the trajectory to define the spline. Consider the table below which outlines kinematic constraints at two points in time, $t_0$ and $t_f$. Note that, in the table, the parameter $q(t)$ refers to any geometric coordinate.

|              | Start (at $t_0$)           | Stop (at $t_f$)            |
| ------------ | -------------------------- | -------------------------- |
| Position     | $q_0=q(t_0)$               | $q_f=q(t_f)$               |
| Velocity     | $\dot{q}_0=\dot{q}(t_0)$   | $\dot{q}_f=\dot{q}(t_f)$   |
| Acceleration | $\ddot{q}_0=\ddot{q}(t_0)$ | $\ddot{q}_f=\ddot{q}(t_f)$ |

The rows of this table can be used to develop cubic or quintic splines. Cubic splines allow the position and velocity to be constrained at the endpoints; a quintic spline can also constrain the acceleration at the endpoints. Each of these splines can be defined using a sufficiently-high-order polynomial. A cubic spline has four tunable parameters, allowing the four constraints mentioned above while a quintic spline has six tunable parameters, allowing the additional two acceleration constraints.

### Computing Splines with Linear Algebra

Now a cubic spline will be determined for an arbitrary coordinate $q(t)$. A third-order (cubic) polynomial can be written as
$$
q(t) = a + b\,t+c\,t^2+d\,t^3,
$$
where coefficients $a$, $b$, $c$, and $d$ define the shape of the polynomial. These four coefficients will be placed in a vector, $\underline{x}$, treated as an unknown:
$$
\underline{x} = 
\begin{bmatrix}
a \\ b \\ c \\ d
\end{bmatrix}.
$$

Now let a new vector $\underline{y}$ be defined by the known constraints to be applied:
$$
\underline{y} = 
\begin{bmatrix}
q_0 \\ \dot{q}_0 \\ q_f \\ \dot{q}_f
\end{bmatrix}.
$$

To determine the unknown coefficients in $\underline{x}$ from the known constraints in $\underline{y}$ , a matrix $M$ will be developed so that $\underline{y}=M\,\underline{x}$ leading to $\underline{x}=M^{-1}\,\underline{y}$. The rows of $M$ are determined by differentiating $q(t)$ the appropriate number of times for each constraint to apply.

For row one, apply the position constraint at $t=t_0$:
$$
q_0 = a + b\,t_0+c\,t_0^2+d\,t_0^3.
$$

We can rewrite this an inner product,
$$
q_0 = 
\begin{bmatrix}
1 & t_0 & t_0^2 & t_0^3
\end{bmatrix}
\begin{bmatrix}
a \\ b \\ c \\ d
\end{bmatrix}.
$$

For row two, apply the velocity constraint at $t=t_0$:
$$
\dot{q}_0 = b + 2\, c \,t_0 + 3\,d\,t_0^2.
$$

As an inner product,
$$
\dot{q}_0 = 
\begin{bmatrix}
0 & 1 & 2\,t_0 & 3\,t_0^2
\end{bmatrix}
\begin{bmatrix}
a \\ b \\ c \\ d
\end{bmatrix}.
$$

Rows three and four can be found the same way but using constraints at $t=t_f$. Once all four are found, the matrix $M$ can be assembled and finally inverted.

$$
\begin{aligned}
\begin{bmatrix}
q_0 \\ \dot{q}_0 \\ q_f \\ \dot{q}_f
\end{bmatrix}
&= 
\begin{bmatrix}
1 & t_0 & t_0^2 & t_0^3 \\
0 & 1 & 2\,t_0 & 3\,t_0^2 \\
1 & t_f & t_f^2 & t_f^3 \\
0 & 1 & 2\,t_f & 3\,t_f^2
\end{bmatrix}
\begin{bmatrix}
a \\ b \\ c \\ d
\end{bmatrix}, \\
\begin{bmatrix}
a \\ b \\ c \\ d
\end{bmatrix}
&= 
\begin{bmatrix}
1 & t_0 & t_0^2 & t_0^3 \\
0 & 1 & 2\,t_0 & 3\,t_0^2 \\
1 & t_f & t_f^2 & t_f^3 \\
0 & 1 & 2\,t_f & 3\,t_f^2
\end{bmatrix}^{-1}
\begin{bmatrix}
q_0 \\ \dot{q}_0 \\ q_f \\ \dot{q}_f
\end{bmatrix}.
\end{aligned}
$$

In some cases the matrix is further simplified by assuming that $t_0 = 0$ and $t_f = T$, making $M$ very sparse. Using this simplification may require the resulting trajectory to be time-shifted depending on how the trajectory is utilized.
$$
\left. M \right|_{t_0 = 0,~ t_f = T} = 
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
1 & T & T^2 & T^3 \\
0 & 1 & 2\,T & 3\,T^2
\end{bmatrix}
$$

For this simplified matrix, the inverse is also sparse:
$$
\left. M^{-1} \right|_{t_0 = 0,~ t_f = T} = 
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
-\frac{3}{T^2} & -\frac{2}{T} & \frac{3}{T^2} & -\frac{1}{T} \\
\frac{2}{T^3} & \frac{1}{T^2} & -\frac{2}{T^3} & \frac{1}{T^2}
\end{bmatrix}.
$$

The same procedure can be extended to multiple coordinate axes by repeating the same procedure for each axis. Fortunately, the produced matrix, $M$, is reusable since it only depends on $t_0$ and $t_f$.

**Note**: while general planar motion allows three degrees of freedom, $X$, $Y$, and $\psi$. Romi’s planar pose still has three configuration variables, $X$, $Y$, and $\psi$; however, its differential-drive kinematics do not allow arbitrary instantaneous motion in all three directions. The robot cannot move sideways in its own body frame, so its velocity must satisfy a non-holonomic constraint. For forward motion, this means the heading $\psi$ must be tangent to the trajectory.

This nonholonomic constraint can be written as
$$
-\sin\psi\dot{X} + \cos\psi \dot{Y}=0.
$$
### Transforming a Spline

Suppose that trajectories have been found for $X(t)$ and $Y(t)$ using cubic splines:
$$
\begin{aligned}
X(t) &= a_X + b_X\,t + c_X\,t^2 + d_X\,t^3, \\
\dot{X}(t) &= b_X + 2\,c_X\,t + 3\,d_X\,t^2, \\
\ddot{X}(t) &= 2\,c_X + 6\,d_X\,t, \\[4pt]
Y(t) &= a_Y + b_Y\,t + c_Y\,t^2 + d_Y\,t^3, \\
\dot{Y}(t) &= b_Y + 2\,c_Y\,t + 3\,d_Y\,t^2, \\
\ddot{Y}(t) &= 2\,c_Y + 6\,d_Y\,t, \\[4pt]
\end{aligned}
$$
Recall that
$$
\begin{aligned}
\mathrm{d}X &= \cos(\psi)\, \mathrm{d}s, \\
\mathrm{d}Y &= \sin(\psi)\, \mathrm{d}s, 
\end{aligned}
$$
and then solve for $\dot{s}$ and $\dot\psi$.

For $s$:
$$
\begin{aligned}
\left( \mathrm{d}X \right)^2 
+
\left( \mathrm{d}Y \right)^2
&=
\left( \cos(\psi)\, \mathrm{d}s \right)^2
+
\left( \sin(\psi)\, \mathrm{d}s \right)^2, \\
%
\left( \mathrm{d}X \right)^2 
+
\left( \mathrm{d}Y \right)^2
&=
\left( \cos^2(\psi) + \sin^2(\psi) \right) \left( \mathrm{d}s \right)^2, \\
%
\left( \mathrm{d}s \right)^2
&=
\left( \mathrm{d}X \right)^2 
+
\left( \mathrm{d}Y \right)^2, \\
%
\mathrm{d}s &= \sqrt{
    \left( \mathrm{d}X \right)^2 
    +
    \left( \mathrm{d}Y \right)^2
}, \\
\dot{s} &= \sqrt{ \dot{X}^2 + \dot{Y}^2},
\end{aligned}
$$
and for $\psi$:
$$
\begin{aligned}
\frac{\mathrm{d}Y}{\mathrm{d}X} 
&= \frac{\sin(\psi)\, \mathrm{d}s}{\cos(\psi)\, \mathrm{d}s}, \\
\frac{\mathrm{d}Y}{\mathrm{d}X} 
&= \tan(\psi), \\
\frac{\dot{Y}}{\dot{X}} 
&= \tan(\psi), \\
\psi &= \text{atan2}{(\dot{Y}, \dot{X}}).
\end{aligned}
$$

In the preceding analysis the four-quadrant version of $\arctan$,  called $\operatorname{atan2}$, is used so that $\psi$ evaluates correctly based on the correct quadrant and when $\dot X =0$. However to differentiate, it is more standard to use $\arctan$.

$$
\begin{aligned}
\dot\psi &= \frac{\mathrm{d}}{\mathrm{d}t} \arctan{\frac{\dot{Y}}{\dot{X}}}, \\
\dot\psi &= \frac{\ddot{Y}\,\dot{X} - \ddot{X}\,\dot{Y}}{\dot{X}^2 + \dot{Y}^2}.
\end{aligned}
$$

This expression assumes the trajectory has nonzero speed which is plausible except for perhaps the endpoints of a trajectory. If the robot stops, heading must be handled separately.

The velocity $\dot{s}$ and yaw-rate $\dot\psi$ can then be combined to find the left and right wheel velocities,
$$
\begin{aligned}
\dot{s}_L &= \dot{s} - \frac{w}{2}\dot\psi, \\
\dot{s}_R &= \dot{s} + \frac{w}{2}\dot\psi, 
\end{aligned}
$$
which may be integrated to find $s_L$ and $s_R$. The corresponding wheel displacements are  
$$
\begin{aligned}
s_L(t)  
&=  
s_L(t_0)  
+  
\int_{t_0}^{t}  
\left(  
\dot{s}(\tau)-\frac{w}{2}\dot{\psi}(\tau)  
\right)d\tau, \\
s_R(t)  
&=  
s_R(t_0)  
+  
\int_{t_0}^{t}  
\left(  
\dot{s}(\tau)+\frac{w}{2}\dot{\psi}(\tau)  
\right)d\tau.  
\end{aligned}
$$

**Insight**: if the trajectory is known ahead of time, the conversion from global coordinates to local coordinates does not need to be computed live during runtime, but it does need to be computed via integration. When paths are generated dynamically, this integration may need to be performed online or repeated over each planning horizon.

## Look-Ahead Horizon and Local Trajectory Generation

An advanced method for live trajectory planning is to use either a cubic or quintic spline to iteratively produce short intermediate trajectories that converge to a predetermined path. The path describes the geometry we want the robot to approach. At each update, the robot uses its current pose and a look-ahead point or look-ahead segment to generate a local curve. That local curve is chosen to match the robot’s current pose while also matching the desired path at the horizon. With a cubic curve, we can match position and tangency. With a quintic curve, we can also match curvature. Repeating this process at each update produces a sequence of feasible local trajectories that converge toward the desired path.

The animation below shows a simplified simulation of this style of iterative local trajectory generation. The simulated robot attempts to follow a large circular path at a specific traversal rate. The following update routine runs at a rate of 2Hz:
1) Determine the present location and velocity of Romi in the 2D plane. Treat these values as $X(0)$, $\dot{X}(0)$, $Y(0)$, $\dot{Y}(0)$.
2) Look ahead a fixed amount of time into the future and determine the location and velocity that Romi *should* have if it were following the path correctly at the predetermined traversal rate. Treat these values as $X(T)$, $\dot{X}(T)$, $Y(T)$, $\dot{Y}(T)$.
3) Use the cubic spline matrix, defined above as $M$, to determine the appropriate set of spline coefficients for a local trajectory $X(t), Y(t)$ from $X(0), Y(0)$ to $X(T), Y(T)$ that matches velocity at each endpoint between the local and desired trajectories.
4) Convert the local trajectory from global coordinates $X(t), Y(t)$ to state trajectories $s_L(t), s_R(t)$.
5) Apply closed-loop control over the interval of time between $t_0$ and $t_f$.

Note: for simplicity, the animation follows the algorithm presented just above, except for the final step. The animation does not use a simulation of Romi's dynamics; instead the animation assumes that Romi can successfully follow each local path.

![An animation showing the iteratively computed local trajectory generated from a look-ahead horizon. Many local trajectories are shown connecting waypoints to the desired path.|700](images/romi/path_animation.gif)

## Summary

1. Encoder and wheel motion are local.
2. Global motion requires accumulating local transformations.
3. Endpoint summaries lose ordering information.
4. Corners require discontinuous heading changes or in-place pivots.
5. Smooth paths avoid abrupt geometric demands.
6. Splines provide a systematic way to build smooth paths.
7. Look-ahead local planning turns a fixed path into feasible short-horizon trajectories.