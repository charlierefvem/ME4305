---
title: Introduction to State Feedback
type: topic
tags:
  - controls
  - state-space
  - state-feedback
  - controllability
  - pole-placement
  - pmdc-motor
  - tracking-control
source:
  course: ME405
  term: 2262
  lecture: 17
status: draft
---

## Motivation

In a previous lecture topic, the [[topic_intro_to_feedback_and_controls|PID]] controller was presented along with several modifications to tailor the controller to a specific application. A PID controller is an example of a *classical* single-input-single-output (SISO) controller. Now, *modern* controls will be introduced through state-feedback, a multi-input-multi-output (MIMO) controller.

State-space models describe how the internal state of a system changes over time due to the system inputs. If those states can be measured or estimated, we can use them directly in the control law to determine the system inputs. This provides a much more flexible controller design method than only feeding back a single output variable.

The central concept is that, for controllable systems, full state feedback allows the closed-loop poles of the system to be placed intentionally. Since the poles determine the dynamics of the system response, pole placement gives a direct way to design the system response from performance criteria such as settling time and overshoot.

## State Feedback

A system implements **state feedback** when the control law defines the system input, $\underline{u}$, as a function of the system state, $\underline{x}$.

If only some states are used, the controller implements **partial state feedback**. If all states are used, the controller implements **full state feedback**. Full state feedback provides a lot of freedom in controller design because the controller can act on every state in the model.

### Controllability

A system is **controllable** if a finite input, $\underline{u}$, can steer the system to any finite state, $\underline{x}$, in finite time.

For a linear time-invariant (LTI) system described by a state-space model,
$$
\dot{\underline{x}} = A\,\underline{x} + B\,\underline{u},
$$
the system is controllable if the controllability matrix,
$$
\mathcal{C}
=
\begin{bmatrix}
B & A\,B & A^2\,B & \cdots & A^{n-1}\,B
\end{bmatrix},
$$
has rank $n$, where $n$ is the system order. That is, if $\mathcal{C}$ is full row-rank, the system is controllable.

If an LTI system is controllable, then it is mathematically possible to place the poles of the system arbitrarily using full state feedback. The exception is that complex poles must appear in complex-conjugate pairs. Otherwise, individual complex poles would make the system model itself complex, which is not valid for a real physical system.

### Pole Placement

**Pole placement** is the process of choosing feedback gains so that the closed-loop system has desired poles. It is desirable to have full control over pole placement because the system poles determine the overall dynamics of the system response. By placing those poles with careful intention, we can force the system dynamics to behave in a desired way.

### State Feedback Regulators

A standard full state feedback controller solves the **regulation problem**: steering all states to zero. Regulator-type controllers are designed to drive all states in the state vector, $\underline{x}$, to zero, or equivalently, to drive the state vector to the origin of the state space.

Consider a standard LTI system,
$$
\dot{\underline{x}} = A\,\underline{x} + B\,\underline{u}.
$$

A full state feedback regulator uses the control law
$$
\underline{u} = -K\,\underline{x},
$$
where $K$ is an $r \times n$ matrix of gains; $r$ is the number of input variables, and $n$ is the order of the system equal to the number of state variables.

Substituting the control law into the system model gives
$$
\begin{aligned}
\dot{\underline{x}} & = A\,\underline{x} + B\,\underline{u}, \\
\dot{\underline{x}} &= A\,\underline{x} - B\,K\underline{x}, \\
\dot{\underline{x}} &= (A - B\,K)\underline{x}.
\end{aligned}
$$

This is a new *homogeneous* LTI system,
$$
\dot{\underline{x}} = A_c\,\underline{x},
$$
where
$$
A_c = A - B\,K.
$$

The poles of $A_c$ can be placed by choosing appropriate values for the elements of the $K$ matrix.

### Characteristic Polynomial Matching

Recall that
$$
P(s) = \det(sI - A)
$$
is the characteristic polynomial for the matrix $A$, which may be interpreted as the characteristic polynomial for the open-loop system.

For the closed-loop system,
$$
\begin{aligned}
P(s) &= \det(sI - A_c), \\
P(s) &= \det(sI - A + BK).
\end{aligned}
$$

One way to place poles is to build a desired characteristic polynomial and then match each term with the characteristic polynomial for the closed-loop system. The desired characteristic polynomial can be formed directly from desired pole locations, or the pole locations can first be determined from performance criteria such as settling time and maximum overshoot.

A common workflow is:
1. Choose a desired settling time, $t_s$, and desired overshoot, $M_p$.
2. From the performance criteria, determine the required damping ratio, $\zeta$, and natural frequency, $\omega_n$, for a second-order system.
3. From $\zeta$ and $\omega_n$, determine the associated second-order poles.
4. Choose additional poles as needed to match the system order $n$, making sure each additional pole is significantly larger in magnitude than the second-order poles.
5. Construct the desired characteristic polynomial from the pole set.
6. Match terms with the characteristic polynomial found from $A_c$ and solve for the gains in $K$.

The workflow may be summarized by the following steps:
$$
(t_s, M_p)
\rightarrow
(\zeta, \omega_n)
\rightarrow
(s_1, s_2)
\rightarrow
(s_1, \dots, s_n)
\rightarrow
P(s)
\rightarrow
K.
$$

For systems with more than one input, there may be multiple possible solutions.


### Example 1

In this example, a regulator will be designed for a PMDC motor.

For a DC motor with inertial and viscous loads, the LTI model can be represented by
$$
\frac{d}{dt}
\begin{bmatrix}
i_L \\
\Omega_J
\end{bmatrix}
=
\begin{bmatrix}
-\frac{R}{L} & -\frac{K_v}{L} \\
\frac{K_v}{J} & -\frac{b}{J}
\end{bmatrix}
\begin{bmatrix}
i_L \\
\Omega_J
\end{bmatrix}
+
\begin{bmatrix}
\frac{1}{L} \\
0
\end{bmatrix}
\begin{bmatrix}
V_m
\end{bmatrix}
$$
where
$$
\underline{x}
=
\begin{bmatrix}
i_L \\
\Omega_J
\end{bmatrix},
\qquad
\underline{u}
=
\begin{bmatrix}
V_m
\end{bmatrix}.
$$

Let the control law be defined by
$$
\begin{bmatrix}
V_m
\end{bmatrix}
=
-
\begin{bmatrix}
K_1 & K_2
\end{bmatrix}
\begin{bmatrix}
i_L \\
\Omega_J
\end{bmatrix}.
$$

Substituting the feedback law into the state equations gives
$$
\frac{d}{dt}
\begin{bmatrix}
i_L \\
\Omega_J
\end{bmatrix}
=
\begin{bmatrix}
-\frac{R}{L} & -\frac{K_v}{L} \\
\frac{K_v}{J} & -\frac{b}{J}
\end{bmatrix}
\begin{bmatrix}
i_L \\
\Omega_J
\end{bmatrix}
-
\begin{bmatrix}
\frac{1}{L} \\
0
\end{bmatrix}
\begin{bmatrix}
K_1 & K_2
\end{bmatrix}
\begin{bmatrix}
i_L \\
\Omega_J
\end{bmatrix}
$$
or
$$
\frac{d}{dt}
\begin{bmatrix}
i_L \\
\Omega_J
\end{bmatrix}
=
\left(
\begin{bmatrix}
-\frac{R}{L} & -\frac{K_v}{L} \\
\frac{K_v}{J} & -\frac{b}{J}
\end{bmatrix}
-
\begin{bmatrix}
\frac{K_1}{L} & \frac{K_2}{L} \\
0 & 0
\end{bmatrix}
\right)
\begin{bmatrix}
i_L \\
\Omega_J
\end{bmatrix}.
$$

Therefore,
$$
A_c
=
\begin{bmatrix}
-\frac{R + K_1}{L} & -\frac{K_v + K_2}{L} \\
\frac{K_v}{J} & -\frac{b}{J}
\end{bmatrix}
$$

The closed-loop characteristic polynomial is
$$
\begin{aligned}
P(s) &= \det \left( sI - A_c \right) \\
P(s) &=
\begin{vmatrix}
s + \frac{R + K_1}{L} & \frac{K_v + K_2}{L} \\
-\frac{K_v}{J} & s + \frac{b}{J}
\end{vmatrix} \\
P(s) &= s^2 + a_1s + a_0.
\end{aligned}
$$

As expected, the characteristic polynomial is second order because the PMDC motor model has two states. The coefficients $a_1$ and $a_0$ depend on $K_1$ and $K_2$.

### Example 2
In this example, the characteristic polynomial found in Example 1 will be matched to a characteristic polynomial designed from desired performance criteria. To perform polynomial matching, we start with criteria such as settling time and overshoot, then reverse-engineer the desired characteristic polynomial.

For example, we can start with the approximate second-order relationships of
$$
t_s = \frac{3}{\zeta \omega_n}
$$
for 5% settling time, and
$$
M_p = \exp\left(\frac{-\zeta \pi}{\sqrt{1-\zeta^2}}\right)
$$
for overshoot.

Then we solve for $\zeta$ and $\omega_n$ to build the desired second-order characteristic polynomial in canonical form,
$$
P(s) = s^2 + 2\zeta\omega_n s + \omega_n^2.
$$

Then, we match the characteristic polynomial found from $A_c$ to the desired second-order polynomial,
$$
s^2 + a_1s + a_0
=
s^2 + 2\zeta\omega_n s + \omega_n^2.
$$

Matching coefficients gives two equations with two unknowns:
$$
\begin{aligned}
a_1 &= 2\zeta\omega_n \\
a_0 &= \omega_n^2
\end{aligned}
$$

Finally, we can resolve $K_1$ and $K_2$ in terms of $a_0$ and $a_1$. This algebra is in general tedious to do by hand so a symbolic solver can help expedite the procedure and allow rapid retuning.

### Example 3

The regulation controller designed in Example 1 and Example 2 is an important foundation, but not very useful in practice. A regulator drives the states to zero, but for a system like a PDC motor, we often want the system to be driven to a specific location in the state space, not always the origin.

For practical control of a motor's velocity, we can augment the state vector to include an error state. Let
$$
x_3 = \int e\,dt
$$
where
$$
e = r - x_2
$$
and $r$ is the velocity setpoint for
$$
x_2 = \Omega_J.
$$

The augmented state vector is therefore
$$
\underline{x}
=
\begin{bmatrix}
i_L \\
\Omega_J \\
\int e\,dt
\end{bmatrix}.
$$

The augmented system model is
$$
\frac{d}{dt}
\begin{bmatrix}
i_L \\
\Omega_J \\
\int e\,dt
\end{bmatrix}
=
\begin{bmatrix}
-\frac{R}{L} & -\frac{K_v}{L} & 0 \\
\frac{K_v}{J} & -\frac{b}{J} & 0 \\
0 & -1 & 0
\end{bmatrix}
\begin{bmatrix}
i_L \\
\Omega_J \\
\int e\,dt
\end{bmatrix}
+
\begin{bmatrix}
\frac{1}{L} \\
0 \\
0
\end{bmatrix}
\underline{u}
+
\begin{bmatrix}
0 \\
0 \\
1
\end{bmatrix}
\underline{r}
$$
or, compactly,
$$
\dot{\underline{x}} = A\underline{x} + B_1\underline{u} + B_2\underline{r}.
$$

Now we can apply full state feedback using
$$
\underline{u} = -K\underline{x}.
$$

After substituting the control law,
$$
\dot{\underline{x}}
=
(A - B_1K)\underline{x}
+
B_2\underline{r}.
$$

The same pole placement process can be used to find $K$, assuming temporarily that the reference input $\underline{r}$ is zero.

For this augmented system,
$$
\begin{bmatrix}
V_m
\end{bmatrix}
=
-
\begin{bmatrix}
K_1 & K_2 & K_3
\end{bmatrix}
\begin{bmatrix}
i_L \\
\Omega_J \\
\int e\,dt
\end{bmatrix}
$$

The gain terms behave like an **IPD** or **PDF** controller:
* $K_1$ acts on current, which is related to acceleration, and therefore similar to a derivative gain on velocity.
* $K_2$ acts on velocity and is therefore similar to a proportional gain.
* $K_3$ acts on accumulated position-like error, and is therefore similar to an integral gain.

Recall that an IPD controller is like a PID controller, but $K_p$ and $K_d$ apply to feedback terms rather than directly to the error.

Because the integral state is defined using the conventional tracking error, $e=r−\Omega_J$, the corresponding feedback gain, $K_3$, may be negative. This is not a problem; the overall control law is $\underline{u} = -K\underline{x}$, so changing the sign convention for a state simply changes the sign of the corresponding gain without changing the controller's behavior.

**Insight**: the preceding setup is designed so that with $\underline{r}$ set to zero the system behaves like a regulator, driving the system state to zero. When $\underline{r}$ is nonzero it acts as a deliberate disturbance on the state-space system. In this scenario the regulator will still attempt to drive the states to zero, but it will be unable to do so because a steady value of $x_2 = \Omega_J = 0$ would cause unbounded growth in $x_3 = \int r - \Omega_J\,dt$. Instead, the system will settle on a new location in the state-space in which all states are bounded; this requires $x_3 = \int r - \Omega_J\,dt$ to be constant at steady-state which is only the case for $\Omega_J = r$.

### Example 4

In Example 3, the system state was augmented to include an integral state, converting the regulator into a setpoint controller. Consequently the system increased from second- to third-order, requiring now three pole locations and three gains.

This example will use the dominant pole approximation to construct a set of three poles that will behave like a designed second-order system. A common strategy is to choose the dominant pair of second-order poles from the desired second-order behavior, then place the remaining pole or poles farther to the left in the complex plane so that they are significantly faster.

For a third-order system, one possible desired polynomial is
$$
P(s)
=
\left(
s^2 + 2\zeta\omega_n s + \omega_n^2
\right)
(s + a)
$$
where the additional real pole is placed at
$$
s = -a.
$$

For the extra pole to have minimal effect on the overall dynamics it must be about 10 times faster, so we can set
$$
a = 10\,\omega_n.
$$

The second-order pole pair remains dominant because the additional pole is much farther left in the complex plane and decays faster.

![Dominant pole approximation diagram. The complex plane shows a second-order dominant pole pair near the imaginary axis and an additional real pole placed farther left on the real axis. A note says the extra pole should be about 10 times faster, with a equals 10 omega_n.](images/state_feedback/second_order_dominant_pole_map.svg)

Expanding gives
$$
P(s)
=
s^3 + a_2s^2 + a_1s + a_0
$$
which can be matched to the three poles from the augmented system derived in Example 3.

**Insight**: the dominant pole approximation is a useful design heuristic, not a design rule. Additional poles should be placed according to the purpose of the additional states. Fast poles are common in controller design because they have little influence on the visible response, but other applications, such as disturbance modeling and observer design, may intentionally use slower poles to represent slowly changing dynamics.

## Practical Strategies for Gain Determination

As mentioned in Example 2, the algebra needed to compute gains from performance criteria involves main steps, each of which depends on the previous step. This process can be tedious and error prone if done by hand. One strategy suggested above is to use a symbolic tool, like the MATLAB Symbolic Math Toolbox to perform the polynomial matching.

There are other formulae available, such as Ackermann formula, that make it easier to compute gains from a set of poles.

However, in practice, the most common approach is to use a fully featured tool like MATLAB's `place()` function or the `place_poles()` method belonging to Python's SciPy package. These methods take in $A$ and $B$ matrices along with a set of pole locations and return the matrix $K$ fully computed.

**Insight**: after practicing with characteristic polynomial matching to understand the design process, it is generally preferable to use software tools such as MATLAB's `place()` function or SciPy's `place_poles()` for practical controller design. These routines use numerically robust algorithms that are generally less sensitive to roundoff and conditioning issues than direct coefficient matching. For multi-input systems, they can also exploit the fact that multiple feedback matrices may produce the same pole locations, allowing solutions with improved numerical properties to be selected.


## Practical Limitations

State-feedback is an approach with tradeoffs like any other tool. One tradeoff was presented above already: state-feedback is best suited for regulation so setpoint or tracking controllers must use an augmented or modified implementation.

Another limitation is the assumption that all states are accessible to use for feedback. The state of the system is generally a hidden internal property and sensors *may* exist to access some of these properties.

In a future lecture topic on [[reference_observer_design|Observer Design]] the concepts of observability and state estimation will be covered; these are the missing pieces needed to feed back the entire system state even when measurements are limited to a subset of the system state.

## Summary

This lecture introduced full state feedback as a regulation control law where the input is computed from the full state vector. For an LTI system, substituting the control law $\underline{u}=-K\underline{x}$ produces the closed-loop matrix $A_c=A-BK$. If the system is controllable, the poles of $A_c$ can be placed by choosing the gains in $K$.

Full state feedback turns gain selection into a pole placement problem. Instead of tuning gains directly and hoping the response improves, we choose a desired set of closed-loop poles and solve for the gains that produce them.

A regulator and a tracking controller solve different problems. A regulator drives the state to zero. A tracking controller must include the reference and, often, an error state so that the output can track a nonzero setpoint.

The lecture then demonstrated polynomial matching using a PMDC motor speed regulation example. The closed-loop characteristic polynomial depends on the feedback gains, and those gains can be selected by matching the closed-loop characteristic polynomial to a desired polynomial based on settling time, overshoot, and dominant pole approximations.

The dominant pole approximation is a practical design shortcut. It lets a higher-order system be shaped primarily by a desired second-order response, while faster non-dominant poles are placed far enough left that they have less influence on the visible response.

Finally, the lecture extended the regulator idea to tracking control by augmenting the state vector with an integrated error state. This creates a controller that behaves like an IPD controller, where the feedback gains act on current, velocity, and accumulated error.

## Candidate Static Notes
* \[\[State Feedback\]\]
* \[\[State-Space Models\]\]
* \[\[Controllability\]\]
* \[\[Pole Placement\]\]
* \[\[Characteristic Polynomial\]\]
* \[\[Dominant Pole Approximation\]\]
* \[\[PMDC Motor\]\]
* \[\[Tracking Control\]\]
* \[\[PID Control\]\]
* \[\[Integral Control\]\]