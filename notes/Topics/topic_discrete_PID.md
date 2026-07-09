---
title: Discrete PID Implementation
type: topic
tags:
  - controls
  - discrete-control
  - PID-control
  - anti-windup
  - firmware
---

## Motivation

Mechanical systems evolve continuously over time, however it is uncommon to apply closed-loop control continuously. While it is possible to build a purely analog controller a modern approach is to design a digital control system instead. Digital control loops are implemented discretely, typically at a fixed sample rate.

This topic focuses on development of a full generic PID controller with filtered derivative and anti-windup, implemented discretely, that can act on a mechanical system. Some of the material in this topic relies on understanding of the [[reference_z_domain|z-Domain]] and [[reference_continuous_to_discrete|Continuous to Discrete Conversion]]. It is also assumed that the reader has familiarity with the standard [[reference_PID|PID Controller]] in continuous time.

**Note**: we are not deriving the only correct digital PID. We are deriving one clean, inspectable, firmware-ready discrete PID implementation from a familiar continuous-time PID form.
## Discrete Integration and Differentiation

The structure of the discrete PID will be found by considering a continuous-time PID controller and then applying a continuous-to-discrete conversion.

### Continuous PID

Recall the standard form for a PID controller in continuous time represented by a transfer function in the s-domain:
$$
C(s) = K_p + \frac{K_i}{s} + K_d\,s.
$$

As mentioned in the reference on [[reference_PID|PID Controllers]], the derivative action implemented by a standard PID can cause amplification of measurement noise. For this derivation the derivative term will be modified to include a filter coefficient $N$; that is, the derivative term will be replaced by a combined low-pass filter and differentiator. The modified control law can be written as a transfer function in the s-domain as:
$$
C(s) = K_p + \frac{K_i}{s} + K_d\,\frac{N\,s}{s+N},
$$
where the simple differentiator $s$ has been replaced by the filtered derivative $\frac{N\,s}{s+N}$. The time-constant associated with the filtered derivative is $\tau=\frac{1}{N}$; therefore as $N$ grows larger the degree of filtering decreases. In the limiting case, the filtered derivative reduces to the standard derivative:
$$
\lim_{N\rightarrow\infty} \left( \frac{N\,s}{s+N} \right)=s.
$$

### Discretization Using the Forward-Difference Method

For this lecture topic the forward-difference method will be used to discretize the system. It would also be valid to use the Bilinear (Tustin) transform, but the resulting coefficients would be different from those derived below. Recall that the forward-difference method for continuous-to-discrete conversion uses the following transformation
$$
s = \frac{z-1}{T_s}.
$$

Plugging this transformation into the continuous-time filtered PID gives
$$
\begin{aligned}
C(z) &= \left. C(s) \right|_{s = \frac{z-1}{T_s}}, \\[2pt]
%
C(z) &= K_p
     + \frac{K_i}{\frac{z-1}{T_s}}
     + K_d\,\frac{N\,\frac{z-1}{T_s}}{\frac{z-1}{T_s}+N}, \\[2pt]
%
C(z) &= K_p
    + K_i\,\frac{T_s}{z-1}
    + K_d\, \frac{N(z-1)}{z-1+N\,T_s}, \\[2pt]
%
C(z) &= K_p
    + K_i\,\frac{T_s\,z^{-1}}{1-z^{-1}}
    + K_d\, \frac{N(1-z^{-1})}{1+(N\,T_s-1)z^{-1}}.
%
\end{aligned}
$$

From here the PID controller can be converted from a discrete-time transfer function to a discrete-time state-space representation, or the terms in the transfer function may be split apart into individual proportional, integral, and derivative components. The latter approach will be used in this lecture topic.

Consider applying the discretized and filtered PID to an error signal $e(z)$:
$$
\begin{aligned}
u(z) &= C(z)\, e(z), \\
u(z) &= K_p\, e(z)
    + K_i\,\frac{T_s\,z^{-1}}{1-z^{-1}} e(z)
    + K_d\, \frac{N(1-z^{-1})}{1+(N\,T_s-1)z^{-1}} e(z).
\end{aligned}
$$

At this point a small change in variables will be employed so that the sample time is included in the integral and derivative gains:
$$
\begin{aligned}
u(z) &= C(z)\, e(z), \\[2pt]
%
u(z) &= K_p\, e(z)
    + K_i\,T_s\,\frac{z^{-1}}{1-z^{-1}} e(z)
    + \frac{K_d}{T_s}\, \frac{N\,T_s(1-z^{-1})}{1+(N\,T_s-1)z^{-1}} e(z), \\[2pt]
%
u(z) &= K_p\, e(z)
    + K_i^\prime\,\frac{z^{-1}}{1-z^{-1}} e(z)
    + K_d^\prime\, \frac{N\,T_s(1-z^{-1})}{1+(N\,T_s-1)z^{-1}} e(z), \\[2pt]
%
u(z) &= K_p\, e(z)
    + K_i^\prime\,\frac{z^{-1}}{1-z^{-1}} e(z)
    + K_d^\prime\, \frac{\alpha\,(1-z^{-1})}{1-\beta\,z^{-1}} e(z),
\end{aligned}
$$
where $K_i^\prime= K_i\,T_s$ is the new integral gain, $K_d^\prime=\frac{K_d}{T_s}$ is the new derivative gain, and $\alpha = N\,T_s$ and $\beta=1-N\,T_s$ are coefficients used in the differentiator.

With this method of forward-difference discretization a discrete pole is located at $z = \beta = 1-N\,T_s$. Using the discrete-time stability condition of $\lvert z \rvert < 1$, we can conclude that the derivative filter is stable only if $0 < N\,T_s < 2$. This is one reason Tustin or other discretization methods are often preferred for filtered derivative terms.

The actuation signal can be broken up into three parts:
$$
\begin{aligned}
u_p(z) &= K_p\, e(z) &
       &= K_p\, e(z), \\[2pt]
u_i(z) &= K_i^\prime\,\frac{z^{-1}}{1-z^{-1}} e(z) &
       &= K_i^\prime\, I(z), \\[2pt]
u_d(z) &= K_d^\prime\, \frac{\alpha\,(1-z^{-1})}{1-\beta\,z^{-1}} e(z) &
       &= K_d^\prime\, D(z),
\end{aligned}
$$
where $I(z)$ is the discretely computed integral of $e(z)$ and $D(z)$ is the discretely computed derivative of $e(z)$.

**Insight**: the primed gains are *firmware* gains. They include the effect of the sample period. This means that changing the sample period without recomputing $K_i^\prime$  and $K_d^\prime$​ changes the behavior of the controller, so these should be converted as part of controller initialization and each time the gains are updated. In other words, continuous-time gains and discrete implementation gains should not be mixed casually. If $K_i$​ and $K_d$​ are tuned in continuous-time units, the primed gains must be recomputed whenever $T_s$​ changes.

To find $I_k$ we can convert the discrete-time transfer function for $I(z)$ to a difference equation:
$$
\begin{aligned}
I(z) &= \frac{z^{-1}}{1-z^{-1}} e(z), \\[2pt]
\left(1 - z^{-1}\right)\,I(z) &= z^{-1}\,e(z), \\[2pt]
I_k - I_{k-1} &= e_{k-1}, \\[2pt]
I_k &= I_{k-1} + e_{k-1},
\end{aligned}
$$
or, after re-indexing,
$$
I_{k+1} = I_k + e_k.
$$

To find $D_k$ we can convert the discrete-time transfer function for $D(z)$ to a difference equation:
$$
\begin{aligned}
D(z) &= \frac{\alpha\,(1-z^{-1})}{1-\beta\,z^{-1}} e(z), \\[2pt]
\left( 1 - \beta\, z^{-1} \right)\, D(z) &= \alpha\,(1-z^{-1})\, e(z), \\[2pt]
D_k - \beta\,D_{k-1} &= \alpha\,(e_k  - e_{k-1}), \\[2pt]
D_k &= \alpha\,(e_k  - e_{k-1}) + \beta\, D_{k-1}.
\end{aligned}
$$

The resulting linear control law becomes:
$$
u_k = K_p\, e_k + K_i^\prime\, I_k + K_d^\prime\, D_k.
$$

Or, with saturation applied,
$$
\begin{aligned}
u_{req,k} &= K_p\, e_k + K_i^\prime\, I_k + K_d^\prime\, D_k, \\[2pt]
u_{sat,k} &= \operatorname{sat}_{[u_{\min},u_{\max}]}(u_{req,k}).
\end{aligned}
$$

Although the combined controller transfer function is second order, this separated implementation stores three persistent quantities: the integral accumulator $I_k$​, the stored filtered derivative value $D_{k-1}$​, and the previous error $e_{k-1}$​. This is not a minimal realization, but it is more transparent for firmware implementation because the integral accumulator and derivative filter remain separate. This separation makes debugging easier and allows anti-windup to be applied directly to the integral term.

### Anti-windup

Since this controller uses integral action an anti-windup mechanism needs to be incorporated so that the integrator stops accumulating during actuator saturation. The clamping method presented in [[reference_PID|PID Controllers]] will be applied here. Consider the circumstances in which the integrator should stop integrating; both of the following conditions must apply:
1. The actuator has saturated
2. The error has the same sign as the saturation

The preceding two conditions, considered together, indicate that not only is the actuator saturated, but the integrator is accumulating error in a manner that increases the level of saturation. The two conditions can be handled simultaneously by interpreting the sign of
$$
e_k\,r_k,
$$
where
$$
r_k = u_{req,k} - u_{sat,k}
$$
is the saturation residual.

The sign of $e_k\,r_k$ can be interpreted to check if integration should be clamped:
* If the expression is negative, it implies that the sign of the error, $e_k$, is opposite that of the saturation residual, $r_k$, which indicates that the integrator is *unwinding* and should not be clamped.
* If the expression is zero, then there is either zero error, or $u_{req,k} = u_{sat,k}$, indicating that no saturation has occurred. In either case the integrator does not need to be clamped.
* Finally, if the expression is positive, it means that the sign of  $e_k$ matches that of $r_k$, which means that the integrator would wind up unless it is clamped.

The expression can be used as an integration gate by using the indicator function, often expressed as $\mathbb{1}_{A}(x)$. This function returns a value of 1 for inputs $x$ within a certain range $A$ and returns a value of 0 for inputs $x$ outside of range $A$.  Therefore, the integration gate is
$$
\gamma_k = \mathbb{1}_{\le0} \left( e_k\,r_k \right).
$$

Finally, the integration gate can be applied by using it as a coefficient for $e_k$ in the update equation for the integrator. That is,
$$
I_{k+1} = I_k + \gamma_k\,e_k.
$$

## Final Implementation of the Filtered Saturation-Aware Discrete PID

The preceding derivation defines the control law and update equations for the integral, $I_k$, and the derivative, $D_k$. To apply these properly the update equations must be used in the correct order.

First, on initialization, prepare all the initial conditions, compute the firmware gains, and finally compute the coefficients used in the differentiator. That is,
$$
\begin{aligned}
I_0 &= 0, \\[2pt]
D_0 &= 0, \\[2pt]
e_{-1} &= 0, \\[2pt]
K_i^\prime &= K_i\,T_s, \\[2pt]
K_d^\prime &= \frac{K_d}{T_s}, \\[2pt]
\alpha &= N\,T_s, \\[2pt]
\beta &= 1-N\,T_s,
\end{aligned}
$$
with the stability condition
$$
0 < N\, T_s < 2.
$$

In firmware, $e_{-1}$​ is often initialized to the first measured error value to avoid a derivative transient on the first control update.

Then, during runtime, apply the following steps in the following explicit sequence:

| Step | Operation                                                         | Purpose                                                                                                                                  |
| ---- | ----------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | $e_k = x_{ref}(t_k) - x_{meas,k}$                                 | Measure the control parameter and then compute the control error using the present setpoint and the measurement.                         |
| 2    | $D_k = \alpha\,(e_k  - e_{k-1}) + \beta\, D_{k-1}$                | Update the filtered derivative using the present and previous control error.                                                             |
| 3    | $u_{req,k} = K_p\, e_k + K_i^\prime\, I_k + K_d^\prime\, D_k$     | Compute the linear controller output before saturation is applied.                                                                       |
| 4    | $u_{sat,k} = \operatorname{sat}_{[u_{\min},u_{\max}]}(u_{req,k})$ | Saturate the value based on the minimum and maximum actuation values. This post-saturation value should then be applied to the actuator. |
| 5    | $r_k = (u_{req,k} - u_{sat,k})$                                   | Find the saturation residual to check for saturation.                                                                                    |
| 6    | $\gamma_k = \mathbb{1}_{\le0} \left( e_k\,r_k \right)$            | Compute the integration-gate for the anti-windup method.                                                                                 |
| 7    | $I_{k+1} = I_k + \gamma_k\,e_k$                                   | Update the integrator taking into the integration gate into account.                                                                     |
In firmware, $I_k$, $D_{k-1}$, and $e_{k-1}$ must persist between control updates. The integrator and derivative-filter values are updated directly by their recurrence equations, while the previous error must be updated explicitly after the derivative calculation.

The preceding algorithm, when implemented in firmware, forms a suitable baseline for real-world implementation of a filtered PID with anti-windup. This baseline implementation should be adjusted and adapted as needed for the particular system being controlled.

## Practical Notes and Common Variants

As mentioned above, this PID is one of many implementations and the final implementation should be modified to handle specific nuances appropriately:
1. The discretization performed here used the forward-difference method for concision as it leads to simple update equations. Using the bilinear transform (Tustin method) is an alternative choice that may lead to a controller with better stability or numerical behavior for the filtered derivative.
2. Derivative control is, in general, something to be handled carefully. Systems with significant measurement noise may not benefit from derivative control even with a filtered derivative. The derivative action can be turned off simply by setting $K_d=0$ or the architecture may be refined to eliminate the differentiator entirely.
3. Derivative action also amplifies changes in the reference since the reference appears in the error term being differentiated; consequently, abrupt changes in setpoint cause "derivative kick", large spikes in actuation effort, which can be harmful to actuators. A suitable method to eliminate the derivative kick is to use a PI-D structure or an IPD structure like introduced in [[reference_PID|PID Controllers]]. Switching to a PI-D structure is as simple as revising the update equation for the derivative to $D_k = \alpha\,(x_{k-1}  - x_k) + \beta\, D_{k-1}$ so that the derivative is computed from only the measurement and not the system error.
4. The primed gains and differentiator coefficients depend on the sample time, $T_s$, and must be recomputed any time the sample time is changed.