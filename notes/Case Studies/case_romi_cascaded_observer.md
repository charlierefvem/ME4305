---
title: Romi Cascaded Observer
type: case-study
tags:
  - case-study
  - disturbance-observer
  - observers
  - state-estimation
  - state-space
source:
  course: ME4305
  term: 2266
status: dirty
---

## Nominal Differential Drive Model

A hierarchical/cascaded observer structure will be considered with three layers:
1. Wheel velocity observer.
    1. Estimates wheel motion based on motor voltage and encoder measurements.
    2. Primary utility is filtering of the quantization noise from the low resolution encoders.
2. Chassis motion observer
    1. Estimates chassis motion based on wheel motion estimates and IMU measurements.
    2. Primary utility is eliminating drift caused by encoder/IMU disagreement.
3. Global position observer
    1. Estimates absolute position based on local chassis motion
    2. Primary utility is to localize the robot in 2D space to allow trajectory planning.

**Present Status**
1. The first layer is working and tested in hardware.
    * With little other code running it is easy to achieve 100Hz running both observers. With other tasks running, the wheel observers must be reduced to 50Hz to maintain cycle-by-cycle telemetry, or can run at 100Hz with reduced telemetry (1 out of 10 data points dropped due to GC interference).
    * See [[case_motor_observer|Practical Motor Control]] for a focused outline of the details for the first layer observer and controller setup.
2. The second layer is under construction and the main focus for this document.
    * Initial effort will be on observer design and tuning.
    * A matching second-layer controller may be added in future revisions.
    * See [[topic_romi_modeling|Romi Dynamic Modeling]] for details on the plant model adapted for this observer.
3. As of now this document does not cover the third layer.
    * In the short term, due to lack of available sensors for global positioning, the third layer will be replaced by a dead-reckoning approach that simply integrates the local motion produced in the second-layer to get a rough approximation of the global position.
    * A vision-based "pseudo-GPS" setup is under development, but not yet ready for integration with this observer setup. The pseudo-GPS will eventually provide low frequency measurements of heading and global position.

## Wheel Speed Observer

The wheel-speed observer is covered in great detail in [[case_motor_observer|Practical Motor Control]], some pertinent aspects are reproduced here as they are contextuality pertinent to the second-layer observer design below.

For the first observer layer, each wheel has its own three state disturbance observer.
* The estimated states include:
    * $\Omega$, the angular velocity of the wheel.
    * $\theta$, the angular displacement of the wheel.
    * $d_1$, an unknown disturbance in acceleration primarily due to inaccurate estimates of the motor constant $\hat{K}_m$.
* The system input $u$ is the applied motor voltage.
* The measured output is the angular displacement of the wheel as measured by an encoder, $\theta_{enc}$.

The following augmented plant model is used by the wheel observer.

$$
\underline{x}
=
\begin{bmatrix}
\Omega \\
\theta \\
d_1
\end{bmatrix},
\qquad
\underline{u}
=
\begin{bmatrix}
u
\end{bmatrix},
\qquad
\underline{y}
=
\begin{bmatrix}
\theta_{enc}
\end{bmatrix},
$$
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}
\begin{bmatrix}
    \Omega \\
    \theta \\
    d_1
\end{bmatrix}
&=
\begin{bmatrix}
    -\frac{1}{\tau} & 0 & 1 \\
    1 & 0 & 0 \\
    0 & 0 & 0
\end{bmatrix}
\begin{bmatrix}
    \Omega \\
    \theta \\
    d_1
\end{bmatrix}
+
\begin{bmatrix}
\frac{\hat{K}_m}{\tau} \\
    0 \\
    0
\end{bmatrix}
\begin{bmatrix}
    u
\end{bmatrix}, \\
%
\begin{bmatrix}
    \theta_{enc}
\end{bmatrix}
&=
\begin{bmatrix}
    0 & 1 & 0
\end{bmatrix}
\begin{bmatrix}
    \Omega \\
    \theta \\
    d_1
\end{bmatrix}.
\end{aligned}
$$

The observability matrix for this system is
$$
\begin{aligned}
\mathcal{O} &= \begin{bmatrix}C \\ C\,A \\ C\,A^2\end{bmatrix}, \\
%
\mathcal{O} &= 
\begin{bmatrix}
    0 & 1 & 0 \\
    1 & 0 & 0 \\
   -\frac{1}{\tau} & 0 & 1
\end{bmatrix}.
\end{aligned}
$$
The system is observable because $\operatorname{rank}(\mathcal{O})=3$, matching the number of states.

## Chassis Motion Observer

Refer to [[topic_romi_modeling|Romi Dynamic Modeling]] for background on Romi's dynamic model. Portions of it will be repurposed here to develop the observer model for the chassis dynamics.

For the second observer layer, one observer will be used to estimate the local state of the Romi chassis.
* Instead of treating wheel velocities as states in the chassis observer, their estimates, $\hat\Omega_L$ and $\hat\Omega_R$, from the first-layer observers will be treated as inputs to the second-layer observer.
* The primary estimated states for the chassis motion observer are:
    * $s$, longitudinal displacement.
    * $\psi$, yaw angle.
* Two displacement states, $d_2$ and $b_1$ will be included as well:
    * $d_2$, an additive yaw-rate disturbance representing rotational slip.
    * $b_1$, a constant bias in the IMU measurement.
* The measured outputs will be:
    * $\psi_{enc} = \frac{r}{w} \left( \hat\theta_R - \hat\theta_L \right)$, the angular displacement estimates for each wheel from the first-layer observers.
    * $\psi_{IMU}$ and $\dot\psi_{IMU}$, the yaw angle and yaw rate from the IMU.
    * $z=\psi_{IMU} - \psi_{enc}$, a fictious measurement representing a constant bias between the yaw angles produced by the IMU and from the wheel displacements.

The following augmented plant model is used by the chassis motion observer.

$$
\underline{x}
=
\begin{bmatrix}
    \psi \\
    d_2 \\
    b_1
\end{bmatrix},
\qquad
\underline{u}
=
\begin{bmatrix}
    \hat\Omega_L \\
    \hat\Omega_R
\end{bmatrix},
\qquad
\underline{y}
=
\begin{bmatrix}
    \psi_{enc} \\
    \psi_{IMU} \\
    \dot\psi_{IMU} \\
    z
\end{bmatrix},
$$
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}
\begin{bmatrix}
    \psi \\
    d_2 \\
    b_1
\end{bmatrix}
&=
\begin{bmatrix}
    0 & 1 & 0 \\
    0 & 0 & 0 \\
    0 & 0 & 0 \\
\end{bmatrix}
\begin{bmatrix}
    \psi \\
    d_2 \\
    b_1
\end{bmatrix}
+
\begin{bmatrix}
    -\frac{r}{w} & \frac{r}{w} \\
    0            & 0 \\
    0            & 0 \\
\end{bmatrix}
\begin{bmatrix}
    \hat\Omega_L \\
    \hat\Omega_R
\end{bmatrix}, \\
%
\begin{bmatrix}
    \psi_{enc} \\
    \psi_{IMU} \\
    \dot\psi_{IMU} \\
    z
\end{bmatrix}
&=
\begin{bmatrix}
    1 & 0 & 0 \\
    1 & 0 & 1 \\
    0 & 1 & 0 \\
    0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
    \psi \\
    d_2 \\
    b_1
\end{bmatrix}
+
\begin{bmatrix}
    0 & 0 \\
    0 & 0 \\
    -\frac{r}{w} & \frac{r}{w} \\
    0 & 0 \\
\end{bmatrix}
\begin{bmatrix}
    \hat\Omega_L \\
    \hat\Omega_R
\end{bmatrix}.
\end{aligned}
$$

The observability matrix for this system is
$$
\begin{aligned}
\mathcal{O} &= \begin{bmatrix}C \\ C\,A \\ C\,A^2\end{bmatrix}, \\
%
\mathcal{O} &= 
\begin{bmatrix}
    1 & 0 & 0 \\
    1 & 0 & 1 \\
    0 & 1 & 0 \\
    0 & 0 & 1 \\[4pt]
%
    0 & 1 & 0 \\
    0 & 1 & 0 \\
    0 & 0 & 0 \\
    0 & 0 & 0 \\[4pt]
%
    0 & 0 & 0  \\
    0 & 0 & 0  \\
    0 & 0 & 0  \\
    0 & 0 & 0 
\end{bmatrix}.
\end{aligned}
$$

The system is observable because $\operatorname{rank}(\mathcal{O})=3$, matching the number of states. Since $C$ has full column rank, every state appears directly in the output equations. Consequently the augmented system is observable without requiring higher powers of $A$.





$$
\underline{x}
=
\begin{bmatrix}
    \psi \\
    e_{\psi,IMU} \\
    e_{\dot\psi,enc} \\
\end{bmatrix},
\qquad
\underline{u}
=
\begin{bmatrix}
    \hat\Omega_L \\
    \hat\Omega_R
\end{bmatrix},
\qquad
\underline{y}
=
\begin{bmatrix}
    \dot\psi_{IMU} \\
    \psi_{IMU}
\end{bmatrix},
$$

$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}
\begin{bmatrix}
    \psi \\
    e_{\psi,IMU} \\
    e_{\dot\psi,enc} \\
\end{bmatrix}
&=
\begin{bmatrix}
    0 & 0 & 1 \\
    0 & 0 & 0 \\
    0 & 0 & 0 \\
\end{bmatrix}
\begin{bmatrix}
    \psi \\
    e_{\psi,IMU} \\
    e_{\dot\psi,enc} \\
\end{bmatrix}
+
\begin{bmatrix}
    -\frac{r}{w} & \frac{r}{w} \\
    0            & 0 \\
    0            & 0
\end{bmatrix}
\begin{bmatrix}
    \hat\Omega_L \\
    \hat\Omega_R
\end{bmatrix}, \\
%
\begin{bmatrix}
    \dot\psi_{IMU}  \\
    \psi_{IMU} 
\end{bmatrix}
&=
\begin{bmatrix}
    0 &  0 & 1 \\
    1 & -1 & 0
\end{bmatrix}
\begin{bmatrix}
    \psi \\
    e_{\psi,IMU} \\
    e_{\dot\psi,enc}
\end{bmatrix}
+
\begin{bmatrix}
    -\frac{r}{w} & \frac{r}{w} \\
    0 & 0 \\
\end{bmatrix}
\begin{bmatrix}
    \hat\Omega_L \\
    \hat\Omega_R
\end{bmatrix}.
\end{aligned}
$$

$$
\begin{aligned}
\mathcal{O} &= \begin{bmatrix}C \\ C\,A \\ C\,A^2\end{bmatrix}, \\
%
\mathcal{O} &= 
\begin{bmatrix}
    0 &  0 & 1 \\
    1 & -1 & 0 \\[4pt]
%
    0 &  0 & 0 \\
    0 &  0 & 1 \\[4pt]
%
    0 &  0 & 0 \\
    0 &  0 & 0
\end{bmatrix}.
\end{aligned}
$$