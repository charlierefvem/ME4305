---
title: Romi Dynamic Modeling
type: topic
tags:
  - romi
  - kinematics
  - dynamics
  - differential-drive
  - state-space
source:
  course: ME405
  term:  2262
  lecture: 11
status: draft
---

# Motivation

The goal of this topic is to build a usable dynamic model for the \[\[Romi\]\] robot platform without making the model more complicated than it needs to be. In principle, Romi could be modeled using either a kinetic model or a kinematic model, but those two approaches emphasize different kinds of information.

Kinetic models use forces, mass properties, and similar quantities to derive system dynamics. These models are usually built using methods such as Newton-Euler rigid-body analysis, as commonly taught in undergraduate dynamics courses, or energy analysis such as the Euler-Lagrange equation. With this type of model, small factors such as the traction and slip at the tires must be included for the model to be accurate.

Kinematic models relate motion in the form of positions, velocities, and accelerations without requiring force or energy analysis. Therefore, information about mass properties, tire friction, and similar physical details must be replaced by modeling assumptions, such as the no-slip condition.

Which kind of model is better? It depends on several factors, primarily including the desired level of model fidelity and the accessibility of system parameters.

For example:
* How easy is it to measure tire traction forces?
* What about the moment of inertia of the entire Romi chassis?
* Is the model meant to account for small variations caused by tire slip?

Due to the simplifications needed for kinematic models, it tends to be the case that kinetic models are both higher-order and more complex than purely kinematic models.

In the context of Romi in ME 4305, it is probably best to keep the model as simple as reasonably possible by considering mostly kinematics while minimizing kinetic effects. In some cases it is possible to split the difference. For example, the Romi chassis kinematics can be merged with a simplified actuator model of the DC motors driving Romi's wheels.

# Kinematics

## Modeling Assumptions

For the Romi model developed here, the main simplifying assumption is that each wheel rolls without slipping. This makes it possible to relate wheel angular velocity directly to wheel-center translational velocity.

This assumption does not mean that real tires never slip. It means that the model intentionally ignores that behavior so that the core chassis motion can be described using geometry rather than detailed tire-force modeling.

A secondary assumption is that the DC motors driving Romi's wheels behave like ideal first-order systems. This implies that the inductance in the DC motor has little effect on the overall system dynamics and that nonlinearities are either not present or do not have a dominating effect on the model.

## Wheel kinematics

Start by considering a side view of Romi. If each wheel is assumed to roll without slip, the velocity at the center of each wheel is related to that wheel's angular velocity by the wheel radius $r$.

![An isometric view of the Romi robot chassis.](images/romi/isometric.svg)

![An right view of the right wheel attached to the Romi robot.](images/romi/wheel.svg)

The no-slip rolling relationships are
$$
v_R = r\,\Omega_R
$$
and
$$
v_L = r\,\Omega_L.
$$

Here $v_R$ and $v_L$ are the translational velocities at the centers of the right and left wheels, while $\Omega_R$ and $\Omega_L$ are the angular velocities of the right and left wheels.

## Chassis kinematics

Next, relate the wheel-center velocities to the chassis longitudinal velocity $v$ and yaw rate $\dot\psi$. Let $w$ be the wheelbase, or distance between the left and right wheel contact lines.

Using the relative velocity relationship from the left wheel to the right wheel,
$$
v_R\,\hat{\imath} = v_L\,\hat{\imath} + \dot\psi\,\hat{k}\times\left(-w\,\hat{\jmath}\right).
$$

This gives
$$
\left(v_R-v_L\right)\,\hat{\imath} = w\,\dot\psi\,\hat{\imath},
$$
so the yaw rate is
$$
\dot\psi = \frac{v_R-v_L}{w}.
$$

Substituting the wheel rolling relationships gives
$$
\dot\psi = \frac{r}{w}\left(\Omega_R-\Omega_L\right).
$$

The center velocity can be found using a similar relative velocity relationship from the left wheel to the center of the chassis:
$$
v\hat{\imath} = v_L\,\hat{\imath} + \dot\psi\,\hat{k}\times\left(-\frac{w}{2}\,\hat{\jmath}\right).
$$

This gives
$$
v\,\hat{\imath} = \left(v_L+\frac{w}{2}\dot\psi\right)\,\hat{\imath}.
$$

Combining this with the expression for $\dot\psi$ gives
$$
v = \frac{v_L+v_R}{2}.
$$

In terms of the wheel angular velocities,
$$
v = \frac{r}{2}\left(\Omega_L+\Omega_R\right).
$$

## Absolute motion of the chassis

The previous relationships describe Romi's motion in a local, body-fixed reference frame. To track Romi's position on the lab table, we also need to express the chassis velocity in a global, inertial reference frame.

Let $X$ and $Y$ be Romi's position coordinates in the global frame, and let $\psi$ be Romi's yaw angle. Romi's absolute velocity can be written as
$$
\dot{X}\,\hat{I} + \dot{Y}\,\hat{J}.
$$

![Annotated absolute-motion figure showing a global X-Y inertial frame and a Romi chassis with local x-y axes rotated by heading angle psi. The annotations show i-hat = cos(psi) I-hat + sin(psi) J-hat, j-hat = -sin(psi) I-hat + cos(psi) J-hat, dot X = v cos(psi), dot Y = v sin(psi), and dot psi = omega.](figures/lecture-11-absolute-motion.png)

The local basis vector $\hat{\imath}$ can be written in terms of the global basis vectors as
$$
\hat{\imath} = \cos\psi\,\hat{I} + \sin\psi\,\hat{J}.
$$

The local lateral basis vector is
$$
\hat{\jmath} = -\sin\psi\,\hat{I} + \cos\psi\,\hat{J}.
$$

Equivalently,
$$
\begin{bmatrix}
\hat{\imath} \\
\hat{\jmath}
\end{bmatrix}
=
\begin{bmatrix}
\cos\psi & \sin\psi \\
-\sin\psi & \cos\psi
\end{bmatrix}
\begin{bmatrix}
\hat{I} \\
\hat{J}
\end{bmatrix}.
$$

Since the chassis velocity is $v\,\hat{\imath}$,
$$
\dot{X}\,\hat{I} + \dot{Y}\,\hat{J} = v\,\hat{\imath}.
$$

Substituting the basis-vector relationship gives
$$
\dot{X}\hat{I} + \dot{Y}\hat{J}
= v\left(\cos\psi\,\hat{I} + \sin\psi\,\hat{J}\right).
$$

Therefore,
$$
\dot{X} = v\,\cos\psi
$$
and
$$
\dot{Y} = v\,\sin\psi.
$$

# Nonlinear State-Space Model

Now that the kinematics have been determined, the next step is to put together a nonlinear state-space model using the equations from the previous steps.

Use the following definitions for the state variables, input variables, and output variables:
$$
\underline{x}
=
\begin{bmatrix}
\Omega_L \\
\Omega_R \\
s \\
\psi \\
X \\
Y
\end{bmatrix},
\qquad
\underline{u}
=
\begin{bmatrix}
u_L \\
u_R
\end{bmatrix},
\qquad
\underline{y}
=
\begin{bmatrix}
s_L \\
s_R \\
\psi \\
\dot\psi \\
X \\
Y
\end{bmatrix}.
$$

In the output vector:
- $s_L$ and $s_R$ are associated with the wheel [[topic_encoders|encoders]].
- $\psi$ and $\Omega$ are associated with the [[topic_inertial_measurement_units|IMU]].
- $X$ and $Y$ would come from GPS if available.

The new variables $u_L$ and $u_R$ are the input voltages for the left and right motors. The new variable $s$ is the total arc length traveled by Romi's center. This arc length can be considered similar to an odometer reading; it tells us how far along a path Romi has traveled.

Similarly, $s_L$ and $s_R$ are the arc lengths traced out by the centers of the left and right wheels. These can be measured by the wheel encoders.

## First-order motor model reminder

It may help to recall that a DC motor can be modeled as a linear first-order system. For example, the system model for the right motor can be represented by the following differential equation:
$$
\dot{\Omega}_R = \frac{1}{\tau}\left(K_m\,u_R-\Omega_R\right).
$$

Here $K_m$ is the motor gain and $\tau$ is the motor time constant. Likewise, the left motor can be represented by
$$
\dot{\Omega}_L = \frac{1}{\tau}\left(K_m\,u_L-\Omega_L\right).
$$

## Model Construction

It will be the student's responsibility to complete assembly of the state-space model to use in homework for simulation. Specifically, student's will need to consider the derivation shown above, add in any missing relationships, and then formulate state and output equations. The state equations must be represented by
$$
\dot{\underline{x}} = \underline{f}\left(\underline{x},\underline{u}\right)
$$
and the output equations by
$$
\underline{y} = \underline{g}\left(\underline{x},\underline{u}\right).
$$

**Note**: due to the few trigonometric expressions in the preceding kinematics, the state and output equations are not linear equations, so the standard matrix form for LTI systems that students are familiar with, $\dot{\underline{x}}=A\underline{x}+B\underline{u}$, cannot be used in this context.

The preceding wheel, chassis, and absolute-motion kinematics provide the relationships needed to assemble these equations.

# Insights

The main modeling decision in this lecture is to keep Romi's chassis motion mostly kinematic while keeping a simple first-order dynamic model for each wheel motor. This avoids requiring a full tire-force or chassis-inertia model while still giving the system enough dynamics to be useful for simulation and control.

The kinematic relationships also show why a differential-drive robot is naturally nonlinear once absolute position is included. The velocity components $\dot{X}$ and $\dot{Y}$ depend on $\cos\psi$ and $\sin\psi$, so the state equations are nonlinear even though the motor model is linear.

The output vector is also a reminder that the variables we care about are not necessarily the same as the variables we can measure directly. Encoders, IMUs, and external position sensors each provide different pieces of information about the robot's motion.