---
title: Romi Observer Design Reflection
type: perspective
status: dirty
---

# Perspectives on Observer Design for the Romi

## Context and Motivation

This note summarizes a design discussion about observer architectures for a small differential-drive robot using wheel encoders, an IMU, and eventually camera-based "pseudo-GPS" measurements. The original motivation was to improve state estimation beyond simple dead reckoning by augmenting observers with disturbance states.

The early goal was ambitious:

- estimate wheel and chassis motion,
- account for motor/model mismatch,
- account for yaw slip,
- estimate IMU heading bias,
- fuse encoder and IMU information,
- eventually support higher-level path planning.

The conversation began with standard disturbance-observer ideas, then moved through output disturbances, redundant sensors, observability failures, cascaded observer architectures, and finally course-design implications.

The most important conclusion is that observer design is not just a matter of adding more states. The added states must represent quantities that are actually distinguishable from the available measurements. When the available sensors cannot separate two explanations for the same measurement error, the observer becomes unobservable or produces misleading estimates.

## Input Disturbances vs. Output Disturbances

A common disturbance-observer model places the disturbance in the plant dynamics:

$$
\dot{x} = Ax + Bu + Ed
$$

with a disturbance model such as

$$
\dot{d} = 0.
$$

This says that the disturbance directly changes the evolution of the physical state. A motor load torque is a good example. The load torque changes angular acceleration, so it belongs in the state equation.

The augmented state is

$$
x_a =
\begin{bmatrix}
x \\
d
\end{bmatrix}
$$

and the observer estimates both the physical state and the disturbance:

$$
\hat{x}_a =
\begin{bmatrix}
\hat{x} \\
\hat{d}
\end{bmatrix}.
$$

Output disturbances are different. They appear in the measurement equation instead:

$$
\dot{x} = Ax + Bu
$$

$$
y = Cx + Du + Ed.
$$

This says the disturbance does not push on the plant. Instead, it corrupts the measurement. Sensor bias is the natural example.

For a constant output disturbance,

$$
\dot{d} = 0,
$$

one can still augment the observer:

$$
x_a =
\begin{bmatrix}
x \\
d
\end{bmatrix}
$$

with

$$
A_a =
\begin{bmatrix}
A & 0 \\
0 & 0
\end{bmatrix}
$$

and

$$
C_a =
\begin{bmatrix}
C & E
\end{bmatrix}.
$$

The observer residual becomes

$$
r_y = y - C_a\hat{x}_a - Du.
$$

So output-disturbance observers are possible, but the interpretation changes. An input disturbance can often be compensated by applying an opposing input. An output disturbance is usually used to correct or reinterpret a measurement.

For example, if an output disturbance represents sensor bias, then the estimate is used to form a corrected measurement:

$$
y_{\text{corr}} = y - E\hat{d}.
$$

It is not usually used as a force or torque cancellation term.

## The Separation Principle Still Applies, But on the Augmented System

One early question was whether the separation principle still applies when the observer has extra disturbance states. The answer is yes, provided the disturbance is modeled as part of an augmented linear system and the required controllability/observability conditions hold.

For an augmented system,

$$
\dot{x}_a = A_a x_a + B_a u
$$

with observer

$$
\dot{\hat{x}}_a
= A_a\hat{x}_a + B_a u + L(y - C_a\hat{x}_a),
$$

and feedback

$$
u = -K_a\hat{x}_a,
$$

the combined dynamics can be written in terms of the augmented state and estimation error:

$$
e_a = x_a - \hat{x}_a.
$$

The closed-loop matrix has the triangular form

$$
\begin{bmatrix}
\dot{x}_a \\
\dot{e}_a
\end{bmatrix}
=
\begin{bmatrix}
A_a - B_aK_a & B_aK_a \\
0 & A_a - LC_a
\end{bmatrix}
\begin{bmatrix}
x_a \\
e_a
\end{bmatrix}.
$$

Therefore the total closed-loop poles are the union of:

- the controller poles of $A_a - B_aK_a$,
- the observer poles of $A_a - LC_a$.

The subtle point is that these are poles of the augmented system, not just the original plant. If the observer estimates an extra disturbance state, then the observer has extra error dynamics.

## Estimation Is Not Compensation

A key insight was separating the observer's role from the controller's role.

The observer estimates the disturbance. The controller compensates for it.

For a motor model,

$$
\dot{\omega}
= -\frac{1}{\tau}\omega + K_mV + d,
$$

an observer may estimate $\hat{d}$. But the disturbance is not compensated until the controller uses that estimate, for example:

$$
V = V_{fb} - \frac{\hat{d}}{K_m}.
$$

Substituting gives

$$
\dot{\omega}
= -\frac{1}{\tau}\omega + K_mV_{fb} + (d - \hat{d}).
$$

After compensation, the plant responds to the residual disturbance:

$$
\tilde{d} = d - \hat{d}.
$$

This was one of the central "aha" moments. The observer is not trying to drive $d$ to zero, and it is not trying to drive $\hat{d}$ to zero. It is trying to drive the estimation error to zero:

$$
\tilde{d} = d - \hat{d}.
$$

If $\hat{d} = d$, then the residual disturbance is zero. The disturbance itself may still be nonzero.

This also resolves the concern that compensation might hide the disturbance from the observer. If $\hat{d}$ were wrong, the prediction error would reappear. The estimate remains meaningful because removing or changing it would make the observer's model disagree with the measured plant behavior.

## Integral Action vs. Disturbance Estimation

Integral action and disturbance estimation both help reject constant disturbances, but they do so through different mechanisms.

An integrator uses accumulated tracking error:

$$
\dot{z} = r - y.
$$

The integrator state grows until the controller produces enough steady input to remove the error.

A disturbance observer tries to estimate the source of the offset directly:

$$
\hat{d} \rightarrow d.
$$

Then the controller can compensate using $\hat{d}$.

Both mechanisms often learn the same steady-state input bias. This can make them feel redundant. If both an integrator and a disturbance estimate are included, the controller may contain two states that are both trying to generate the same compensation.

This resembles a non-minimal realization or near pole-zero cancellation more than classical zero dynamics. The concern is not usually that the design is fundamentally invalid. The concern is that two internal states can compete or produce poorly conditioned dynamics.

However, practical details matter more than theoretical redundancy. In this robot, the disturbance estimate in the wheel observer may be very active because it absorbs encoder quantization and other model mismatch. Feeding that estimate back directly can reintroduce measurement artifacts into the actuator command.

Integral action can be more robust in that situation because it naturally emphasizes persistent low-frequency error rather than high-frequency observer activity.

So the practical conclusion was:

- disturbance feedback is attractive when the disturbance estimate is physically meaningful and clean,
- integral action is attractive when the disturbance estimate contains noise, quantization artifacts, or other unmodeled effects,
- using both may be redundant unless there is a clear reason.

## Redundant Sensors and Relative Biases

The next topic was output disturbances for redundant measurements. Suppose two sensors should measure the same signal:

$$
y_1 = Cx + b_1 + v_1
$$

$$
y_2 = Cx + b_2 + v_2.
$$

Here $b_1$ and $b_2$ are biases, while $v_1$ and $v_2$ represent random measurement noise or quantization.

With only these two measurements, one generally cannot estimate both absolute biases unless there is an independent reference for the true value $Cx$. What can be estimated well is the relative bias:

$$
y_1 - y_2 = b_1 - b_2 + (v_1 - v_2).
$$

This became important for the wheel encoder and IMU yaw problem. The IMU heading and encoder-derived heading can disagree, but their disagreement alone does not reveal which one is wrong.

The observer can estimate a relative disagreement. It cannot magically assign the error correctly to each sensor without another assumption or another reference.

## The Initial Large Observer

One proposed architecture was a large augmented observer, roughly combining:

- wheel motion,
- chassis position,
- yaw,
- wheel slip,
- IMU bias,
- possibly GPS-like position measurements.

This model was mathematically tempting because all sensor relationships and disturbance states could be written in one place. It also preserved the standard state-space observer framework.

However, it created several practical problems:

- the state dimension became large,
- the model mixed several different physical domains,
- pole placement produced gains that were difficult to interpret,
- the model became nonlinear through the conversion from heading and forward speed to global $X,Y$ motion,
- multirate sensing made tuning awkward,
- observability depended on combining many sensors at once.

The large observer was useful as a thinking tool, but it started to look like the wrong implementation target for the course and hardware.

## The Cascaded Observer Idea

The discussion then shifted to a cascaded observer structure. This aligned better with both the physics and the course's emphasis on abstraction.

The proposed observer cascade was:

```text
Wheel observers
    -> chassis motion observer
        -> global position observer
```

The corresponding controller cascade was:

```text
Wheel PI controllers
    -> yaw / chassis controller
        -> trajectory / global planner
```

This symmetry was attractive. Each layer creates a cleaner abstraction for the next layer.

Layer 1 estimates wheel quantities:

- wheel velocity,
- wheel displacement,
- motor disturbance or model mismatch.

Layer 2 uses layer-1 outputs as virtual measurements and inputs:

- wheel velocity estimates drive the chassis kinematics,
- wheel displacement estimates provide encoder-derived chassis information,
- IMU measurements provide yaw information,
- disturbance and bias states try to explain yaw slip and IMU offset.

Layer 3, eventually, would use local motion and pseudo-GPS/camera data:

- local forward motion and heading are transformed into global velocity,
- global position is corrected by delayed or low-rate external measurements.

This decomposition has strong pedagogical value. Instead of one intimidating observer, students can see how each layer solves a specific problem and exports a useful interface.

## Inputs, Measurements, and Feedthrough in the Chassis Layer

For the chassis observer, there was an early concern that the chassis model had no physical input in the same sense as a motor voltage. The chassis model is kinematic:

$$
\dot{\psi} = \frac{r}{w}(\hat{\Omega}_R - \hat{\Omega}_L) + d_\psi.
$$

Here the "inputs" are not actuator commands. They are velocity estimates from the lower-level wheel observers.

That is acceptable in a cascade. Higher layers often treat lower-layer estimates as inputs or virtual sensors.

There was also concern about the $D$ matrix because the yaw-rate measurement depends directly on the wheel velocity input:

$$
\dot{\psi}_{IMU}
= \frac{r}{w}(\hat{\Omega}_R - \hat{\Omega}_L) + d_\psi.
$$

That creates direct feedthrough in the output equation:

$$
y = Cx + Du.
$$

This is not a problem. It simply means the observer residual must be computed as

$$
r_y = y - C\hat{x} - Du.
$$

The feedthrough term is physically expected because the measured yaw rate is directly related to the wheel velocities.

## Pole Placement and Symmetry Problems

When pole placement was used for the chassis observer, the gain matrix behaved strangely. In a symmetric differential-drive robot, intuition suggests that wheel displacement residuals should affect the forward displacement estimate symmetrically:

$$
L_{s,\theta_L} = L_{s,\theta_R}
$$

and should affect the yaw estimate with opposite signs:

$$
L_{\psi,\theta_L} = -L_{\psi,\theta_R}.
$$

However, MATLAB's `place` returned a gain matrix that did not preserve this symmetry.

This was not necessarily a bug. MIMO pole placement is not unique. If many gain matrices produce the requested poles, the algorithm is free to choose one that is mathematically valid but physically unintuitive.

Changing measurement coordinates to sum and difference channels,

$$
y_s = \frac{r}{2}(\hat{\theta}_L + \hat{\theta}_R)
$$

$$
y_\psi = \frac{r}{w}(\hat{\theta}_R - \hat{\theta}_L),
$$

made the physical interpretation clearer but did not fully solve the issue. The pole-placement algorithm could still cross-couple the channels.

LQE/Kalman-style design behaved better because symmetric model assumptions and symmetric noise weights tend to produce symmetric gains. This is because LQE solves an optimization problem. If the system and weights are symmetric, the unique optimal solution should preserve that symmetry.

This led to a broader insight:

- pole placement asks where the observer poles should be,
- LQE asks how much we trust the model versus each measurement.

For sensor fusion, the second question is often more natural.

## Removing the Longitudinal State

The chassis observer originally tried to estimate both longitudinal displacement $s$ and yaw $\psi$. But there is no independent layer-2 measurement of true forward chassis motion. The wheel sum already gives the best available estimate:

$$
\hat{s} = \frac{r}{2}(\hat{\theta}_L + \hat{\theta}_R).
$$

Without GPS, optical flow, vision, or another external reference, the observer cannot determine whether the chassis slipped longitudinally. A longitudinal slip disturbance is indistinguishable from an error in integrated wheel displacement.

Yaw is different because there are two yaw-related information sources:

- encoder-derived yaw,
- IMU yaw and yaw rate.

Therefore, it made sense to remove $s$ from the observer and compute it directly from wheel motion. The observer could then focus on yaw only.

This simplified the model and removed artificial coupling between common-mode wheel motion and yaw.

## The Yaw-Only Observer and Its Limits

The yaw-only observer seemed promising at first. A natural model used states such as:

$$
x =
\begin{bmatrix}
\psi \\
e_{\dot{\psi},enc} \\
e_{\psi,IMU}
\end{bmatrix}.
$$

The qualitative assumptions were:

1. Encoder yaw may drift due to slip. This appears as a yaw-rate error:

$$
\dot{\psi} - \dot{\psi}_{enc}
\approx \text{constant}.
$$

2. IMU yaw may have a roughly constant bias:

$$
\psi_{IMU} - \psi
\approx \text{constant}.
$$

3. IMU yaw rate may be relatively trustworthy over short time intervals:

$$
\dot{\psi}_{IMU} - \dot{\psi}
\approx 0.
$$

This motivated a model with:

- true yaw,
- encoder yaw-rate error,
- IMU yaw bias.

However, adding both IMU bias and accumulated encoder yaw error led to loss of observability. The underlying reason was fundamental, not just algebraic.

With only encoder yaw and IMU yaw, the system can measure disagreement:

$$
\psi_{IMU} - \psi_{enc}.
$$

But that disagreement can be explained by:

- IMU bias,
- accumulated encoder slip,
- or some combination of both.

The observer cannot know how to allocate the error without another reference or another assumption.

In other words, the true yaw might lie closer to the encoder value, closer to the IMU value, or somewhere between them. The available measurements do not determine which interpretation is correct.

## Why Initial Conditions Are Not Enough

One tempting thought was that initial conditions might resolve the ambiguity. For example, if the robot starts aligned with the encoder yaw, perhaps the observer could estimate IMU bias from there.

Initial conditions can choose one point on the unobservable family, but they do not remove the unobservable direction from the model. If the model contains both an unknown true yaw offset and an unknown IMU yaw bias, then the measurement

$$
\psi_{IMU} = \psi + b_{IMU}
$$

only sees their combination.

The transformation

$$
\psi \rightarrow \psi + \alpha
$$

$$
b_{IMU} \rightarrow b_{IMU} - \alpha
$$

leaves the measured IMU yaw unchanged.

Therefore true yaw and IMU bias cannot both be estimated from IMU yaw alone. A yaw-rate measurement can help estimate rate error, but it does not anchor absolute yaw offset.

The practical conclusion was:

> With only encoder yaw and IMU yaw, and no independent absolute heading reference, true yaw, IMU yaw bias, and accumulated encoder slip are not all separately observable.

## The BNO055 Complication

The IMU used in the lab, the BNO055, adds another practical complication. It reports fused orientation using an internal proprietary fusion algorithm. The reported heading is not a raw physical measurement. It is the output of another observer running inside the sensor.

This matters because the heading can change when the sensor recalibrates or when the local magnetic environment changes. Indoors, magnetometers are often unreliable due to:

- steel structures,
- motors,
- power electronics,
- nearby computers,
- lab benches,
- distorted local magnetic fields.

So the BNO055 heading is not well modeled as

$$
\psi_{IMU} = \psi + b
$$

with constant $b$.

It is more like

$$
\psi_{IMU} = \psi + b(t),
$$

where $b(t)$ may drift or jump.

That behavior is dangerous for feedback and estimation. A heading jump from the IMU can look like a sudden robot rotation unless the estimator explicitly detects and rejects it.

This led to a practical reassessment. If the IMU is not trustworthy enough to serve as a yaw reference, then a sophisticated yaw observer may be trying to extract information that the sensor does not reliably contain.

## BNO08x and Better IMUs

The BNO08x series was considered as a possible improvement. It likely has better firmware and more flexible report modes than the BNO055, but it uses a more complicated packet protocol rather than the simple register model of the BNO055.

A better IMU could help if it provides a yaw signal trustworthy enough to serve as a reference. But if the sensor becomes that trustworthy, then the observer may not be needed for yaw in the same way. One could simply use the IMU yaw or gyro-integrated yaw directly, perhaps with basic filtering.

More importantly, a better IMU does not remove the fundamental observability issue. Without an independent heading reference, one still cannot fully distinguish:

- true yaw,
- IMU yaw bias,
- accumulated encoder slip.

So upgrading the IMU may improve signal quality, but it does not change the structure of the estimation problem.

## Returning to Dead Reckoning

The final practical conclusion was to step back from the yaw observer and return to robust dead reckoning for the current course scope.

The dead-reckoning equations are:

$$
\dot{s}
= \frac{r}{2}(\hat{\Omega}_L + \hat{\Omega}_R)
$$

$$
\dot{\psi}
= \frac{r}{w}(\hat{\Omega}_R - \hat{\Omega}_L).
$$

Then global motion can be computed from:

$$
\dot{X} = \dot{s}\cos\psi
$$

$$
\dot{Y} = \dot{s}\sin\psi.
$$

This is not globally accurate over long runs, but it is simple, predictable, and teachable.

The key is to use dead reckoning only over short intervals and repeatedly re-register against external features.

## What "Robust Dead Reckoning" Means

Dead reckoning is fragile when used as a long-term global reference. Even a small initial heading error creates lateral drift.

For a travel distance $L$ and initial heading error $\theta$, the lateral error is approximately:

$$
e_y \approx L\sin(\theta).
$$

For $L = 1$ m and $\theta = 0.5$ deg:

$$
e_y \approx 8.7\text{ mm}.
$$

That is already meaningful for a small robot on a game track.

Therefore robust dead reckoning means:

- use it for short relative maneuvers,
- do not rely on it as the global truth source,
- reset or correct using environmental references whenever possible,
- design tasks so drift cannot accumulate unchecked.

The more robust structure is:

```text
line or registration feature
    -> short dead-reckoned maneuver
        -> next line or registration feature
            -> re-register
```

This supports tasks such as:

- lane changes,
- parallel parking,
- pull-out maneuvers,
- obstacle bypasses,
- short spline trajectories,
- transitions between track features.

The design principle is:

> Use line sensing for global registration; use dead reckoning for local maneuvers between registration events.

## Lines, Magnets, and Future Pseudo-GPS

Line sensors are valuable because they provide an external reference. They are not just a way to implement line following. They can also act as localization features.

A line is a one-dimensional feature. It tells the robot something like:

> You are on or near this curve.

Magnets embedded at specific waypoints could provide zero-dimensional landmark events:

> You just passed this point.

This could support event-triggered localization without requiring vision. A single Hall sensor would mostly detect waypoint crossings, while richer magnet patterns or multiple sensors could encode more information such as lateral offset or waypoint identity.

Eventually, camera-based pseudo-GPS would change the problem significantly. Once the robot has an external position reference, the third observer layer becomes much more meaningful. At that point, a global observer can correct drift in $X$, $Y$, and possibly yaw.

Until then, line features are the most reliable pseudo-GPS available in the lab.

## Course Design Implications

The original hope was that the observer would become reliable enough to support path planning in the Fall course. The discussion made that less likely, at least without camera-based absolute measurements.

This does not make the path and trajectory material useless. It changes how it should be used.

Path and trajectory lectures can still cover:

- paths versus trajectories,
- curvature,
- splines,
- velocity profiles,
- feedforward motion commands,
- lane-change maneuvers,
- why localization matters.

But in lab, the trajectories should be short and tied to registration events.

For example:

```text
follow line
    -> detect marker
        -> execute spline lane change
            -> reacquire line
```

This may be a better educational design than asking students to trust a fragile observer over a long run.

## Why Not Teach the EKF Now?

A unified multirate EKF may eventually be the right advanced solution. It could combine:

- wheel dynamics,
- IMU data,
- slip models,
- delayed camera measurements,
- global position correction,
- uncertainty propagation.

But introducing an EKF too soon may be pedagogically harmful. It would hide many important modeling lessons behind covariance matrices and algorithmic machinery.

Students first benefit from learning:

- what the sensors actually measure,
- what the models assume,
- how observer residuals work,
- why some states are unobservable,
- why external references matter,
- how abstraction helps manage complexity.

The EKF is better framed as a downstream topic:

> What we built here is a modular, fixed-gain version of a larger estimation architecture. With more time and better hardware, these layers could be unified into a multirate EKF.

For this course, the cascaded observer and dead-reckoning discussion may be more valuable than a premature EKF implementation.

## Final Takeaways

Several important conclusions emerged.

First, output disturbances are valid observer states, but they are sensor-model errors, not plant forces. Their purpose is usually measurement correction or sensor calibration.

Second, disturbance observers estimate error. They do not compensate by themselves. Compensation happens only when the controller uses the estimate.

Third, integral action and disturbance estimation can solve similar steady-state problems. Which one is better depends on signal quality, noise, quantization, and implementation details.

Fourth, redundant sensors reveal disagreement, not truth. Without an independent reference, the observer cannot always decide which sensor is wrong.

Fifth, the IMU/encoder yaw problem has a fundamental observability limitation. True yaw, IMU heading bias, and accumulated encoder slip cannot all be estimated from encoder yaw and IMU yaw alone.

Sixth, the BNO055 heading output is difficult to use as a clean observer measurement because it is itself the result of a hidden fusion algorithm and can change when the sensor recalibrates.

Seventh, the cascaded observer architecture remains valuable as an abstraction, even if the yaw observer is not used in the final Fall implementation.

Eighth, for the current course, robust short-horizon dead reckoning plus frequent line or landmark registration is a better target than long-horizon observer-based localization.

Finally, this journey is itself pedagogically useful. It shows that good engineering is not adding complexity until the model looks sophisticated. Good engineering is knowing what information the sensors actually provide, what the model can legitimately infer, and when a simpler architecture is more honest.

