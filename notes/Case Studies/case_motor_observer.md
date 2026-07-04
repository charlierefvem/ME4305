---
title: Practical Motor Control
type: case-study
tags:
  - mechatronics
  - case-study
  - disturbance-observer
  - observers
  - state-estimation
  - discrete-control
  - encoder
  - pmdc-motor
  - state-space
source:
  course: ME 4305
  term: June 2026
  file: observer_case_study.tex
  author: Charlie Refvem
status: draft
---

# Motivation

This case study outlines the control and estimation theory needed to achieve accurate control of the permanent magnet DC (PMDC) motors that are part of the Romi robot platform used in ME 4305. Nonetheless, the topics covered are applicable to motor control in general.

This case study covers:
* Basics of discrete control, primarily through PI control in discrete time.
* Velocity-estimation techniques, including finite differences and moving averages.
* An [[reference_disturbance_observer|augmented disturbance observer]] for estimating motor velocity from encoder measurements in the presence of quantization noise and an unknown disturbance in acceleration.

# Discrete Control

In this section, the basics of discrete controls are presented primarily in the context of a [[PI controller]]. Refer to [[reference_z_domain|Discrete Time Systems]] for background on discrete signals, the z-domain, and difference equations.

## PI Control

Readers of this document should have passing familiarity with [[reference_PID|PID Controllers]] from study in this and other courses. For the remainder of this document, it will be assumed that the following standard form of a PI controller is well understood:
$$
u(t) = K_p\, e + K_i\, \int e\,\mathrm{d}\tau 
$$
where $u$ is the output of the controller representing actuation effort. Derivative control is not used due to its limited utility in systems with significant measurement or quantization noise. 

We approximate the integral using a rectangular update:
$$
I_{k+1} = I_k + e_k.
$$

The control law is therefore
$$
\begin{aligned}
u_k = K_p\,e_k + K_i\,I_k.
\end{aligned}
$$

**Note**: in this convention the value of $K_i$ already accounts for the sample period, $T_s$.

In firmware the actuation effort produced by the control law is saturated based on the available actuation limits. Anti-windup is also added in firmware using conditional-integration techniques. These nuances will be covered in greater detail at the end of this case study. For more information on anti-windup techniques see the pertinent section in [[reference_PID|PID Controllers]].

## Velocity estimation using encoders

Encoders directly measure displacement, $\theta$, rather than velocity, $\Omega = \dot{\theta}$, so an estimate of velocity must be obtained numerically before the control law can be applied. The next sections compare simple methods of velocity estimation from an encoder assumed to be low resolution. Here low resolution refers to the CPR, or counts per revolution, of the encoder.

In general, encoders provide information at insufficient update rates when operating at slow velocities to provide much useful velocity resolution. The lower the resolution, the worse the velocity measurement becomes.

### Finite-difference approximation

The simplest solution to numerical differentiation is the finite-difference method, which can be explained as the reverse of Euler's method, equivalent to the backward-difference method covered in [[reference_continuous_to_discrete|Continuous to Discrete Conversion]]. 

Applying this method to define the angular velocity, $\Omega$, gives
$$
\begin{aligned}
\Omega_k &= \frac{\theta_k - \theta_{k-1}}{T_s}.
\end{aligned}
$$

This method is computationally inexpensive and easy to implement. However, encoder measurements are quantized, meaning that the measured displacement changes only when a new encoder count is observed. At low rotational velocities, this quantization can produce significant error in the velocity estimate.

The resolution problem can be seen by considering the example when
$$
\left|\Omega_k\,T_s\right| < \frac{2\pi}{CPR},
$$
where $CPR$ is the encoder resolution in counts per revolution.

In this extreme case, the velocity is low enough that, for a sample period $T_s$, the quantization may cause no change to occur in displacement between samples. To handle speeds this low using a finite-difference approach, the sample time $T_s$ must increase, which may conflict with the intentions for controller design.

### Moving average

One simple method for reducing quantization noise is to average multiple finite-difference estimates, or equivalently, to apply the finite difference to the first and last values from a window of multiple measurements.

For example, we could calculate over $N$ samples
$$
\begin{aligned}
\hat{\Omega}_k &= \frac{1}{N}\sum_{i=0}^{N-1}\Omega_{k-i},
\end{aligned}
$$
where each $\Omega_{k-i}$ is computed using a finite-difference approach. However, in practice it may be simpler to instead calculate
$$
\begin{aligned}
\hat{\Omega}_k &= \frac{\theta_k - \theta_{k-N}}{N\,T_s},
\end{aligned}
$$
to avoid needing a running history of prior velocities.

This averaged estimate represents motion over a window of length $N\, T_s$​, so its effective timing is near the center of that window. The resulting delay is roughly $\frac{N\,T_s}{2}$ or $\frac{(N-1)\,T_s}{2}$ when viewed as the additional delay of an $N$-point moving average applied to already-computed velocity samples.

As the number of samples, $N$, gets large, the delay becomes large and may cause the measurement to lag enough to damage controller performance. Consequently, moving-average filtering represents a tradeoff between noise reduction and responsiveness. For very low resolution encoders it may be impossible to use a large enough window to provide sufficient smoothing without introducing enough lag to damage control performance.

### Velocity-measurement conclusions

The previous two sections show the shortcomings of direct numerical differentiation of encoder data to compute angular velocity. The finite-difference method has low delay but high noise, while the moving average has larger delay and lower noise.

A more effective solution for the Romi hardware, covered in the next section, is to use additional information accessible from the system itself to help "extrapolate", in a loose sense of the word, between measurement changes from the encoder so that velocity estimates remain accurate at low speeds.

# Motor Observer Design

## Observer-Based Velocity Estimation

Both finite-difference and moving-average approaches estimate velocity using only encoder measurements. Neither method directly incorporates knowledge of the motor dynamics. An [[reference_observer_design|observer]] provides an alternative approach by combining a mathematical model of the motor with encoder measurements and knowledge of the applied voltage to estimate the system state. This method allows the observer to reject measurement noise while maintaining a responsive estimate of velocity.

The purpose of this section is to build the augmented disturbance-observer model used for the lab implementation. The observer model is intended to estimate the internal state of a simple first-order model of a PMDC motor with robustness against inaccuracies in the plant model, such as an incorrect back-EMF constant or an inaccurate time constant.

### The plant model

We will focus on the DC motor model for this observer design, not the entire Romi model. The model will be based on a linear first-order representation of a DC motor; that is, it will be assumed that the motor behaves as a simple first-order low-pass filter defined by a gain, $K_m$, and a time constant, $\tau$. These parameters can be found experimentally or derived from physical properties of the motor like the terminal resistance, the back-emf coefficient, etc.

As a differential equation, we can write the simplified motor model as
$$
\begin{aligned}
\tau\,\dot{\Omega} + \Omega &= K_m\,u.
\end{aligned}
$$

The model can also be written as a state-space model by including a state representing the angular displacement of the motor. We can produce the state-space model by augmenting the system with an additional state,
$$
\theta = \int \Omega\,\mathrm{d}t.
$$

Combining the new equation with
$$
\begin{aligned}
\tau\,\dot{\Omega} + \Omega &= K_m\,u \\
\dot{\Omega} &= -\frac{1}{\tau}\Omega + \frac{K_m}{\tau}\,u
\end{aligned}
$$
yields the state-space model for the system,
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}
\begin{bmatrix}
\Omega \\
\theta
\end{bmatrix}
&=
\begin{bmatrix}
-\frac{1}{\tau} & 0 \\
1 & 0
\end{bmatrix}
\begin{bmatrix}
\Omega \\
\theta
\end{bmatrix}
+
\begin{bmatrix}
\frac{K_m}{\tau} \\
0
\end{bmatrix}
\begin{bmatrix}
u
\end{bmatrix}.
\end{aligned}
$$

### Modeling for uncertainty

The linearized model derived in the previous section assumes that the parameter $K_m$ is known exactly. In practice, however, the value of $K_m$ varies from motor to motor and may also change due to unmodeled behavior. Additionally, the preceding model has relatively high sensitivity to the parameter $K_m$ because it directly relates the input $u$ to the state $\Omega$ once the system has reached constant velocity. Therefore, even modest parameter error in $K_m$ will manifest as significant steady-state velocity bias.

We now aim to allow uncertainty in the model by introducing a disturbance state, $d$, that will represent inaccuracy in $K_m$. That is, to improve robustness, we augment the model with a disturbance state. Rather than attempting to identify the exact source of model error, the disturbance state is allowed to absorb uncertainty in the velocity dynamics. This allows the observer to compensate for model mismatch while retaining a linear state-space representation.

Let
$$
\begin{aligned}
K_m = \hat{K}_m + \Delta K_m
\end{aligned}
$$
be a model of the uncertainty in $K_m$, where $\hat{K}_m$ is the assumed value for $K_m$ used in modeling and control, and $\Delta K_m$ is the error or deviation from the true value representing the plant. Substitution into the ODE lets us factor out a disturbance state $d$ representing the uncertainty in $K_m$. That is,
$$
\begin{aligned}
\tau\dot{\Omega} + \Omega &= \left(\hat{K}_m + \Delta K_m\right)u \\
\dot{\Omega} + \frac{1}{\tau}\Omega &= \frac{\hat{K}_m}{\tau}\,u + \frac{\Delta K_m}{\tau}\,u \\
\dot{\Omega} + \frac{1}{\tau}\Omega &= \frac{\hat{K}_m}{\tau}\,u + d,
\end{aligned}
$$
where $d$ is the disturbance state. The disturbance state should be interpreted as a lumped acceleration disturbance. If the only modeling error were a motor-gain error, then this disturbance would be precisely
$$
d = \frac{\Delta K_m}{\tau}\,u.
$$

In the observer, however, we do not estimate $\Delta K_m$ directly. Instead, we estimate a slowly varying acceleration correction $d$ that can absorb motor-gain error, friction effects, battery variation, and other model mismatch over short time windows.

The disturbance enters the velocity dynamics because uncertainty in $K_m$, torque uncertainty, and time constant uncertainty all manifest as an unknown acceleration-producing input, meaning the disturbances directly affect the rate of change of the velocity and only indirectly affect the displacement. Therefore, the disturbance is introduced through the same channel as the motor actuation.

For constant input $u$, the disturbance $d$ should remain constant, and for changing input the disturbance is assumed to change slowly. Therefore, we can claim that
$$
\begin{aligned}
\dot{d} = 0
\end{aligned}
$$
is approximately true for slowly changing inputs. In reality, a more accurate model would be $\dot{d} \propto \dot{u}$, but we aim to simplify the model and avoid nonlinearities.

The disturbance state can compensate for moderate model mismatch, including some effects of incorrect gain or time constant, especially over limited operating regions. It should *not* be interpreted as an exact estimator of every physical parameter error.

Now define the augmented state vector as
$$
\begin{aligned}
\underline{x} =
\begin{bmatrix}
\Omega \\
\theta \\
d
\end{bmatrix}.
\end{aligned}
$$

**Note**: the state vector, $\underline{x}$ is a different variable than the control variable $x$ presented in the controller section above.

The state-space representation of the system now has order three:
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}
\begin{bmatrix}
\Omega \\
\theta \\
d
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
d
\end{bmatrix}
+
\begin{bmatrix}
\frac{\hat{K}_m}{\tau} \\
0 \\
0
\end{bmatrix}
\begin{bmatrix}
u
\end{bmatrix}.
\end{aligned}
$$

Planning for observer design, we recognize that only the angular displacement $\theta$ can be measured directly. Therefore, we can conclude the state-space output is $y=\theta$, or more formally,
$$
\begin{aligned}
\begin{bmatrix}
\theta
\end{bmatrix}
&=
\begin{bmatrix}
0 & 1 & 0
\end{bmatrix}
\begin{bmatrix}
\Omega \\
\theta \\
d
\end{bmatrix}
+
\begin{bmatrix}
0
\end{bmatrix}
\begin{bmatrix}
u
\end{bmatrix}.
\end{aligned}
$$

### Observer design

We can now collect matrices and begin the observer design:
$$
\begin{aligned}
A &=
\begin{bmatrix}
-\frac{1}{\tau} & 0 & 1 \\
1 & 0 & 0 \\
0 & 0 & 0
\end{bmatrix}, \\
B &=
\begin{bmatrix}
\frac{\hat{K}_m}{\tau} \\
0 \\
0
\end{bmatrix}, \\
C &=
\begin{bmatrix}
0 & 1 & 0
\end{bmatrix}, \\
D &=
\begin{bmatrix}
0
\end{bmatrix}.
\end{aligned}
$$

In preparation for observer design, we should confirm that the system is observable. For the continuous-time model above, the observability matrix is
$$
\mathcal{O} = 
\begin{bmatrix}
C \\
C\,A \\
C\,A^2
\end{bmatrix}= 
\begin{bmatrix}
0  & 1 & 0 \\
1 & 0 & 0 \\
-\frac{1}{\tau} & 0 & 1
\end{bmatrix},
$$
which has full rank for finite $\tau$. For the actual implementation, it is recommended to also check the discretized pair $(A_d,C_d)$, perhaps using MATLAB:

```matlab
rank(obsv(Ad, Cd))
```

The observer will be a standard augmented disturbance observer, as covered in [[reference_disturbance_observer|Disturbance Observer Design]]. The observer will be designed in discrete time, so the preceding matrices for the continuous-time open-loop plant model, already augmented, need to be discretized using a sample time, $T_s$.

Augmenting before discretization is equivalent to assuming that the disturbance state is constant over each sample interval, just like the zero-order-hold assumption treats the motor command as constant over each sample interval.

For implementation purposes, the observer is formulated directly in discrete time rather than designing a continuous-time observer and discretizing the resulting gain. We will use the zero-order hold method for discretization, leading to
$$
\begin{aligned}
A_d &= \exp(A\,T_s), \\[4pt]
B_d &= \left(\int_0^{T_s}\exp(A\,\lambda)\,\mathrm{d}\lambda\right)B, \\[4pt]
C_d &= C, \\[4pt]
D_d &= D.
\end{aligned}
$$

Discretized, the system is now assumed to follow
$$
\begin{aligned}
\underline{x}_{k+1} &= A_d\,\underline{x}_k + B_d\,\underline{u}_k \\
\underline{y}_k &= C_d\,\underline{x}_k + D_d\,\underline{u}_k.
\end{aligned}
$$

So, to tune the observer error dynamics, we must place the eigenvalues of
$$
A_o = A_d - L\,C_d.
$$

In this example, we choose a dominant second-order pair from settling-time and overshoot specifications, then add one additional real pole because the augmented observer has three states.  
  
The continuous-time observer poles are
$$
\begin{aligned}
s_{1,2} &= -\zeta\,\omega_n \pm i\,\omega_n\sqrt{1-\zeta^2}, \\
s_3 &= -r\,\zeta\,\omega_n
\end{aligned}
$$
where $r$ is a pole-ratio that scales the third pole proportionately to the real part of the two second-order dominant poles.

The continuous-time poles $s_{1,2,3}$ are converted to discrete-time poles using
$$
\begin{aligned}
z_i = \exp\left(s_i\,T_s\right) \quad \forall~ i \in \{1,2,3\}.
\end{aligned}
$$

The observer gain is then determined using pole placement on $A_d^T$ and $C_d^T$:
$$
\begin{aligned}
L = \operatorname{place}\left(A_d^T, C_d^T, [z_1\;z_2\;z_3]\right)^T.
\end{aligned}
$$

For a more detailed explanation of this procedure see pertinent sections of [[topic_state_feedback|Fundamentals of State Feedback]].

With the observer gain determined, we can now produce the state-space matrix representation of the observer. This formulation allows the observer to be implemented directly as a discrete-time state-space system using standard simulation or embedded-control tools.

$$
\begin{aligned}
A_o &= A_d - L\,C_d, \\
B_o &= \begin{bmatrix}
\left(B_d - L\,D_d\right) & L
\end{bmatrix},
\end{aligned}
$$
and
$$
\begin{aligned}
\underline{w}_k =
\begin{bmatrix}
\underline{u}_k \\
\underline{y}_k
\end{bmatrix},
\end{aligned}
$$
such that the observer reduces to a standard state-space representation in discrete time:
$$
\begin{aligned}
\hat{\underline{x}}_{k+1} &= A_o\,\hat{\underline{x}}_k + B_o\,\underline{w}_k.
\end{aligned}
$$

This formulation is particularly convenient for embedded implementation, as the observer can now be executed using the same state-update pattern used for any other discrete-time state-space system.

The estimated plant output used inside the observer is $\hat{\theta}_k$​. The signal returned to the controller is the estimated velocity $\hat{\Omega}_k$​.

Therefore, let
$$
\begin{aligned}
\hat{\Omega}_k &=
    \begin{bmatrix}1&0&0\end{bmatrix}
    \,\hat{\underline{x}}_k.
\end{aligned}
$$

In conclusion, the resulting observer estimates velocity, displacement, and disturbance in acceleration while requiring only displacement measurements from the encoder. Only the estimated velocity is returned for control purposes. The disturbance state allows the observer to compensate for moderate model mismatch, improving robustness relative to a conventional second-order observer.

## Implementation Sequence

In the sections above, a discrete-time PI controller was developed to control the velocity of a PMDC motor. To overcome significant quantization noise in the control variable, a disturbance observer was developed to estimate angular velocity accurately even at low speeds.

The equations describing the controller and observer can be implemented together in firmware, but the equations should be applied in the correct sequence to make causal sense. When implementing a discrete-time control system, each computation must depend only on information available at the current sample instant. Furthermore, the implementation should avoid circular dependencies where two quantities depend on one another simultaneously. To make this concrete, the table below outlines a nine-step procedure for implementing the observer and controller in firmware.

The implementation sequence obeys the following rules:
- No use of future information.
- The left-hand side of an equation has indices greater than or equal to the right-hand side.
- The most recent (current) measurement at step $k$ is $\theta_k$.
- Equations avoid mixing present and future samples on the same side of an equation.
- Circular equations are avoided.

| Step | Operation                                                                        | Purpose                                                                                                                                                                                                                                                                   |
| ---- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | $\hat{\Omega}_k = \begin{bmatrix}1&0&0\end{bmatrix} \,\hat{\underline{x}}_k$     | Extracts the angular velocity for this sample, from the estimate predicted by the previous observer step.                                                                                                                                                                 |
| 2    | $e_k = \Omega_r(t_k) - \hat{\Omega}_k$                                           | Computes the control error using the angular velocity estimate, and the reference velocity at the present sample.                                                                                                                                                         |
| 3    | $u_{req,k} = K_p e_k + K_i I_k$                                                  | Computes the requested (pre-saturation) actuation effort at the present sample using the PI control law.                                                                                                                                                                  |
| 4    | $u_k = \text{sat}_{[u_{\min},u_{\max}]} \left( u_{req,k} \right)$                | Computes the saturated actuation effort that is applied to the system, by clipping the requested actuation effort at the saturation limits for the actuator.                                                                                                              |
| 5    | $r_k = u_{req,k}-u_k$                                                            | Computes the saturation residual: the difference between the pre- and post-saturation actuation efforts. This residual is zero when the requested command is within the actuator limits and otherwise represents how far the requested command was clipped by saturation. |
| 6    | $\gamma_k = \mathbb{1} \left[ r_k e_k \le 0 \right]$                             | Defines the conditional-integration gate using the indicator function written as $\mathbb{1}$.<br>    $\gamma_k=1$: the integrator is allowed to update.<br>    $\gamma_k=0$: the integrator is held constant.                                                            |
| 7    | $I_{k+1} = I_k + \gamma_k\, e_k$                                                 | Updates the state of the integrator only when permitted by the conditional-integration gate. This prevents the integrator from accumulating error when doing so would drive the actuator command farther into saturation.                                                 |
| 8    | $\underline{w}_k = \begin{bmatrix} u_k \\ \theta_k \end{bmatrix}$                | Builds the vector of known information from the post-saturation actuation value and a new displacement measurement from the encoder.                                                                                                                                      |
| 9    | $\hat{\underline{x}}_{k+1} = A_o\,\hat{\underline{x}}_k + B_o\, \underline{w}_k$ | Updates the observer estimate using the known-information vector.                                                                                                                                                                                                         |

# Insights

The finite-difference velocity estimate is responsive but noisy. The moving-average estimate is smoother but delayed. The disturbance observer provides a middle path: it uses the motor model to "extrapolate" between encoder measurement changes while using measurement feedback to correct model error.

Adding the disturbance state makes the observer more robust to moderate mismatch in motor gain. This is especially important when controlling inexpensive DC motors, where motor-to-motor variation and nonlinear friction effects may be significant compared to the simplified first-order model.

The firmware implementation order matters as much as the mathematical model. A correct observer-controller design can still fail in software if the update sequence accidentally uses future information or creates circular dependencies.

# Summary

This case study developed a discrete-time PI controller and an augmented disturbance observer for PMDC motor velocity control. The controller uses a discrete integrator, while the observer estimates angular velocity from encoder displacement measurements and a first-order motor model.

The observer begins with a two-state motor model in $\Omega$ and $\theta$, then augments the state vector with a disturbance state $d$ to account for uncertainty in $K_m$. After discretization with zero-order hold, an observer gain is selected by pole placement. The resulting observer can be implemented as a standard discrete-time state-space update using the measured encoder position and applied motor command.