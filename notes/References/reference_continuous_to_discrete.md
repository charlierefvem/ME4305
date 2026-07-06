---
title: Continuous to Discrete Conversion
type: reference
tags:
  - discrete-systems
  - state-space
  - observers
source:
  course: ME405
  term: 2262
  lecture: 20
status: draft
---

# Motivation

Physical systems operate continuously. Computing devices, like microcontrollers, that perform closed-loop control operate discretely, or only at certain discrete time intervals. Therefore, it is desirable to accurately simulate and control continuous systems using discrete means.

One option is to use solvers, like Euler's Method or one of the Runge-Kutta Methods to run the continuous model in steps. This methodology is convenient, but it comes with large performance overhead. Because of this overhead, general-purpose solvers are not often implemented directly in firmware.

A commonly used alternative is to convert the model itself from a continuous-time model, usually a differential equation, to a discrete-time model, usually a difference equation.

See [[reference_z_domain|Discrete Time Systems]] for background discrete-time systems.

Continuous-to-discrete conversion can mean several related things. Sometimes we convert desired pole locations. Sometimes we convert a transfer function into a difference equation. Sometimes we convert a continuous-time state-space model into a discrete-time state update. These are related ideas, but they are not the same calculation.

For controllers and filters expressed as transfer functions, the bilinear transform is commonly used because it produces a rational discrete-time transfer function that is convenient to implement. For physical plants represented in state-space form, zero-order hold is often preferred because it matches how a digital controller actually applies actuator commands between updates.
## Standard Procedure

The standard approach for implementing a continuous model in firmware can be broken down into five steps:
1) Develop a model in continuous time using the modeling techniques of your choosing.
2) If the model is a closed-loop system, tune the dynamics as preferred in continuous time.
3) Convert the system from continuous time to discrete time, or alternatively, extract the system dynamics in the form of pole locations and convert those to discrete time.
4) Implement the resulting discrete time model in firmware as a difference equation.
5) After conversion, the discrete-time poles and time response should still be verified, especially if the sample time is not much smaller than the fastest dynamics.
# Pole Mapping

Sometimes it is simplest to convert the *desired dynamics* of the system from a continuous-time representation to a discrete-time representation, rather than the system model. This allows a model to be developed directly in a discrete time representation, but utilize dynamics designed using continuous time performance metrics.

For a continuous-time state-space model, the natural dynamics are described by the eigenvalues of $A$. For a discrete-time model, the corresponding natural dynamics are described by the eigenvalues of $A_d$​. A continuous-time pole $s_i$​ maps to the equivalent discrete-time pole
$$
z_i = e^{s_i\,T_s}.
$$

The same pole-mapping idea will be used when choosing discrete-time observer poles from desired continuous-time error dynamics.
## Example 1

This example shows how to convert the poles associated with a continuous-time system to poles associated with an equivalent discrete-time system.

Consider the third order characteristic polynomial
$$
P(s) = (s^2 + 2\,\zeta\,\omega_n\,s + \omega_n^2)(s+a)
$$
where
$$
\zeta = 0.8, \quad \omega_n = 50, \quad  \text{and} \quad a = 350.
$$

These parameters correspond to pole locations of
$$
\begin{aligned}
s_1 &= -\zeta\,\omega_n + i\,\omega_n\, \sqrt{1-\zeta^2} & &= -40+30i,\\
s_2 &= -\zeta\,\omega_n - i\,\omega_n\, \sqrt{1-\zeta^2} & &= -40-30i,\\
s_3 &= -a & &= -350.
\end{aligned}
$$

These poles can be visualized on a pole map like shown below.

![Continuous pole diagram. The location of each of the third order systems poles is shown as an x on a complex plane.](discrete_systems/example_3o_poles_continuous.svg)

Each of these can be converted to a discrete time pole with $z_n = e^{s_n\,T_s}$. For this example assume $T_s = 1[ms]=0.001[s]$.

The resulting discrete time poles are 
$$
\begin{aligned}
z_1 &= 0.9604 + 0.0288i, \\
z_2 &= 0.9604 - 0.0288i ,\\
z_3 &= 0.7047.
\end{aligned}
$$

These poles can also be visualized on a pole map like shown below.

![Discrete pole diagram. The location of each of the third order systems poles is shown as an x on a complex plane.](images/discrete_systems/example_3o_poles_discrete.svg)

**Insight**: notice that the location of the continuous time poles relative to the imaginary axis visually resembles the location of the discrete time poles relative to the unit circle.

# Transfer Functions

To represent a transfer function in the z-domain we need to apply a conversion from the s-domain to the z-domain. This can be achieved a number of ways as covered in [[reference_z_domain|Discrete Time Systems]]. The most common method is the bilinear transform, also called the Tustin method or the trapezoidal method:
$$
s = \frac{2}{T_s}\frac{z-1}{z+1}.
$$

To apply this transformation plug it into each appearance of $s$ in a continuous time transfer function.

Consider a system defined as a continuous-time transfer function. This transfer function can represent a digital filter, a PID controller, or another linear system. 

Let $y$ be the output of the system, and $u$ be the input to the system. Finally, let the coefficients $a_0, \ldots, a_n$ and $b_0, \ldots, b_m$ define the denominator and numerator of the transfer respectively.

$$
\begin{aligned}
G(s) &= \frac{y(s)}{u(s)} \\
G(s) &= \frac{b_m\,s^m + \dots + b_0}{a_n\,s^n + \dots + a_0}.
\end{aligned}
$$

Apply the Tustin / bilinear transform:
$$
G(z) = G(s)\bigg|_{s=\frac{2}{T_s}\frac{z-1}{z+1}}.
$$

After much algebra,
$$
G(z)
= \frac{N_0\,z^n + \dots + N_n}{D_0\,z^n + \dots + D_n}
= \frac{N_0 + \dots + N_n\,z^{-n}}
{D_0 + \dots + D_n\,z^{-n}}.
$$

Here, $N_0$ through $N_n$ are the set of $n+1$ numerator coefficients and $D_0$ through $D_n$ are the set of $n+1$ denominator coefficients of the discrete-time transfer function. The numerator may have fewer meaningful terms than the denominator, but for implementation it is convenient to pad the numerator coefficients so both polynomials are written to order $n$. These two sets of parameters depend on the coefficients $a_0, \ldots, a_n$ and $b_0, \ldots, b_m$, from the continuous-time transfer function along with the sample time $T_s$.

## Difference-equation form

Now that the discrete-time transfer function has been determined, an appropriate difference equation can be found that will lead to implementation of the transfer function in code.

Start with
$$
y(z) = G(z)\,u(z).
$$

Substitute the discrete-time transfer function:
$$
y(z)
= \frac{N_0 + \dots + N_m\,z^{-m}}
{D_0 + \dots + D_n\,z^{-n}}
u(z).
$$

Distribute the denominator:
$$
({D_0 + \dots + D_n\,z^{-n}})\,y(z)
= (N_0 + \dots + N_m\,z^{-m})\,u(z).
$$

After distributing $z^{-1}$ as a unit delay,
$$
D_0\,y_k + \dots + D_n\,y_{k-n}
= N_0\,u_k + \dots + N_m\,u_{k-m}.
$$

This expression could be solved directly for $y_k$ and then implemented in code if histories of previous samples of $y$ and $u$ are stored in memory:
$$
y_k
= \frac{1}{D_0}\big( (N_0\,u_k + \dots + N_m\,u_{k-m})
- (D_1\,y_{k-1} + \dots + D_n\,y_{k-n}) \big).
$$

This direct implementation requires storing the previous outputs, $y_{k-1} \dots y_{k-n}$, and previous inputs, $u_{k-1} \dots u_{k-n}$. Depending on the implementation, the current input $u_k$​ may also be stored in the input buffer.

Any transfer function, even one with numerator dynamics, should only require $n$ state variables. With a bit more math, the dependency on $u_{k-1} \dots u_{k-n}$ can be eliminated.

### Introducing an internal state variable

Define a new variable $x(z)$ such that the numerator coefficients shape the final output:
$$
y(z) = (N_0 + \dots + N_m\,z^{-m})\,w(z).
$$

Then substitute $y(z)$ back into the difference equation:
$$
(D_0 + \dots + D_n\,z^{-n})\,(N_0 + \dots + N_m\,z^{-m})\,w(z)
= (N_0 + \dots + N_m\,z^{-m})\,u(z).
$$

Cancel the common numerator-coefficient terms to find
$$
(D_0 + \dots + D_n\,z^{-n})\,w(z) = u(z).
$$

In time-domain form,
$$
D_0\,x_k + \dots + D_n\,w_{k-n} = u_k.
$$

This recurrence only depends on $n$ previous values, which matches the fact that the system is $n$th order.

Now solve for the new internal value $w_k$:
$$
w_k = \frac{1}{D_0}\left(u_k - D_1\,w_{k-1} - \dots - D_n\,w_{k-n}\right).
$$

This difference equation now only requires memory of $n$ previous internal values, which matches the order of the system. It no longer requires separate histories of both the input and output. More algebra is required to find the actual output.

### Output equation

Start again with the relationship between $y(z)$ and $w(z)$:
$$
y(z) = (N_0 + \dots + N_m\,z^{-m})\,w(z).
$$

In time-domain form,
$$
y_k = N_0\,w_k + \dots + N_m\,w_{k-m}.
$$

Substitute the expression for $w_k$:
$$
y_k
= \frac{N_0}{D_0}\left(u_k -  D_1\,w_{k-1} - \dots - D_n\,w_{k-n}\right)
+ N_1\,w_{k-1} + \dots + N_m\,w_{k-m}.
$$

Collect terms:
$$
y_k
= \frac{N_0}{D_0}u_k
+ \left(N_1 - \frac{N_0D_1}{D_0}\right)\,w_{k-1}
+ \dots
+ \left(N_n - \frac{N_0\,D_n}{D_0}\right)\,w_{k-n},\
\quad \text{with}~N_j=0~\forall~j>m.
$$
### Discrete-time state-space form

The preceding equations can be assembled into a discrete-time state-space representation of the system in standard $\underline{w}_{k+1} = A_f\,\underline{w}_{k}+B_f\,\underline{u}_{k}$ form:
$$
\begin{bmatrix}
w_{k-n+1} \\
\vdots \\
w_{k-1} \\
w_k
\end{bmatrix}
=
\begin{bmatrix}
0                 & 1                 & \dots             & 0    \\
\vdots            & \vdots                & \ddots            & \vdots    \\
0                 & 0                     & \dots             & 1    \\
-\dfrac{D_n}{D_0} & -\dfrac{D_{n-1}}{D_0} & \dots & -\dfrac{D_1}{D_0}
\end{bmatrix}
\begin{bmatrix}
w_{k-n} \\
w_{k-n+1} \\
\vdots\\
w_{k-1}
\end{bmatrix}
+
\begin{bmatrix}
0 \\
\vdots \\
0 \\
\dfrac{1}{D_0}
\end{bmatrix}
[u_k].
$$

The output equation  $\underline{y}_{k} = C_f\,\underline{w}_{k}+D_f\,\underline{u}_{k}$ form is
$$
[y_k]
=
\begin{bmatrix}
N_n - \dfrac{N_0\,D_n}{D_0} &
N_{n-1} - \dfrac{N_0\,D_{n-1}}{D_0} &
\dots &
N_1 - \dfrac{N_0\,D_1}{D_0}
\end{bmatrix}
\begin{bmatrix}
w_{k-n} \\
w_{k-n+1} \\
\vdots\\
w_{k-1}
\end{bmatrix}
+
\begin{bmatrix}
\dfrac{N_0}{D_0}
\end{bmatrix}
[u_k].
$$

It may be tempting to use NumPy to implement the state equation in matrix form, however due to the sparseness and predictable shape of the $A_f$ matrix it would be computationally wasteful. All rows of the $A_f$ matrix except for the bottom row are simply shifting back samples $w_k$ to $w_{k-1}$ to $w_{k-2}$, etc., and can be handled better using slice assignment operations or NumPy's `roll()` function.


# State-space Systems

One strategy for continuous to discrete conversion that applies well to state-space systems is to solve the model explicitly, ahead of time, over a short window of time while assuming constant inputs. This assumption is referred to as the Zero-order hold. More elaborate methods assume the input changes linearly over the short window of time; this is referred to as the First-order hold. The first-order hold is more complicated and will not be considered here. The zero-order hold is an effective choice for a discretely implemented control loop because the actuation signal typically does remain constant during the time between each update.

Recall the equations for a standard continuous-time linear time-invariant (LTI) system written in state-space form:
$$
\begin{aligned}
\dot{\underline{x}} &= A\,\underline{x} + B\,\underline{u}, \\
\underline{y} &= C\,\underline{x} + D\,\underline{u}.
\end{aligned}
$$

To convert this system to a discrete-time LTI system, we need to solve these equations over a short window of time, typically denoted as $T_s$, the sample time. That is, we desire to find the solution $\underline{x}(T_s)$ due to some initial conditions defined by
$$
\underline{x}_0 = \underline{x}(0)
$$
while assuming that the input vector $\underline{u}$ remains constant between $T = 0$ and $T = T_s$.

This solution can be found through various methods, including the Laplace transform and convolution. Instead, we will use methods covered in linear algebra to solve the system using the zero-input / zero-state decomposition.

## Zero-input / zero-state decomposition

The total response of a linear system can be split into two pieces:
1. The **zero-input response**  is due only to the initial condition.
2. The **zero-state response** is due only to the input and initial conditions of zero.

This decomposition gives a useful way to derive the discrete-time state update without repeatedly running a numerical ODE solver in firmware.

### Matrix exponential

The matrix exponential appears naturally when solving linear systems of the form
$$
\dot{\underline{x}} = A\,\underline{x}.
$$

For these systems, the solution has the form
$$
\underline{x}(t) = e^{A\,t}\underline{x}_0.
$$

The matrix exponential therefore plays the same role for systems of linear ODEs that the scalar exponential plays for first-order scalar ODEs.

### Zero-input solution to linear systems

To solve the zero-input problem, assume the input is zero:
$$
\underline{u} = \underline{0}.
$$

The system model becomes
$$
\dot{\underline{x}}_{ZI} = A\,\underline{x}_{ZI},
$$
and is subject to initial conditions of
$$
\underline{x}_{ZI}(0) = \underline{x}_0.
$$

The solution to the homogeneous LTI system can be written in terms of the [[reference_matrix_exponential|matrix exponential]]; so, the zero-input solution is
$$
\boxed{\underline{x}_{ZI} = e^{A\,t}\underline{x}_0.}
$$

### Zero-state solution to linear systems

To solve the zero-state problem, assume
$$
\underline{x}_0 = \underline{0}.
$$

The system model becomes
$$
\dot{\underline{x}}_{ZS} = A\,\underline{x}_{ZS} + B\,\underline{u},
$$
and is subject to initial conditions of
$$
\underline{x}_{ZS}(0)=\underline{0}.
$$

Apply the zero-order hold by assuming
$$
\underline{u}(t) = \underline{u}_0.
$$

To find a particular solution, write
$$
\dot{\underline{x}}_p - A\,\underline{x}_p = B\,\underline{u}_0.
$$

For a constant particular solution $\underline{x}_p = \underline{K}$, $\dot{\underline{x}}_p = \underline{0}$, so
$$
\begin{aligned}
\underline{0} - A\,\underline{K} &= B\,\underline{u}_0 \\
\underline{K} &= - A^{-1}\,B\,\underline{u}_0.
\end{aligned}
$$
For the moment, it is assume $A$ is invertible, but this assumption will be removed below.

Thus,
$$
\underline{x}_p = -A^{-1}\,B\,\underline{u}_0
$$
is the particular solution.

Borrowing from the zero-input solution, the complementary solution has the form
$$
\underline{x}_c = e^{A\,t}\underline{c}.
$$

The zero-state solution is the sum of the complementary solution and the particular solution:
$$
\underline{x}_{ZS} = e^{A\,t}\,\underline{c} - A^{-1}\,B\,\underline{u}_0.
$$

Use the zero initial condition to resolve $\underline{c}$:
$$
\begin{aligned}
\underline{0} &= e^{A\cdot 0}\,\underline{c} - A^{-1}\,B\,\underline{u}_0, \\
\underline{c} &= A^{-1}\,B\,\underline{u}_0.
\end{aligned}
$$

Substituting this back into the zero-state solution gives
$$
\underline{x}_{ZS}
= e^{A\,t}\,A^{-1}\,B\,\underline{u}_0 - A^{-1}\,B\,\underline{u}_0.
$$

Therefore,
$$
\boxed{\underline{x}_{ZS} = \left(e^{A\,t}-I\right)A^{-1}B\,\underline{u}_0.}
$$

The total solution is the sum of the zero-input and zero-state solutions:
$$
\underline{x} = \underline{x}_{ZI} + \underline{x}_{ZS}.
$$

Therefore,
$$
\boxed{\underline{x} = e^{A\,t}\underline{x}_0 + \left(e^{A\,t}-I\right)\,A^{-1}\,B\,\underline{u}_0.}
$$

For $t=T_s$, a fixed sample time,
$$
\underline{x}(T_s)
= e^{A\,T_s}\,\underline{x}_0 
+ \left(e^{A\,T_s}-I\right)\,A^{-1}\,B\,\underline{u}_0.
$$

In general, let
$$
\underline{x}(T_s) \rightarrow \underline{x}_{k+1},
\qquad
\underline{x}_0 \rightarrow \underline{x}_k,
\qquad
\underline{u}_0 \rightarrow \underline{u}_k.
$$

Define
$$
A_d = e^{A\,T_s}
$$

and
$$
B_d = \left(e^{A\,T_s}-I\right)\,A^{-1}\,B.
$$

Finally, the discretized state-space system is
$$
\boxed{\underline{x}_{k+1} = A_d\,\underline{x}_k + B_d\,\underline{u}_k.}
$$

This is the form needed by firmware: each control-loop update computes the next state estimate or model state from the current state and the held input.

## What if $A$ is singular?

There are some cases in which the preceding analysis fails. One such case is when the $A$ matrix is singular, or noninvertible. In that case it is impossible to compute
$$
B_d = \left(e^{A\,T_s}-I\right)A^{-1}B.
$$

It is possible to directly compute $B_d$ as
$$
B_d = \int_0^{T_s} e^{A\,\tau}\, B\, d\tau
$$
which is valid for singular or nonsingular $A$ matrices.

An indirect approach to solving this integral is to replace $e^{A\,T_s}$ in the expression for $B_d$ with its Taylor series expansion, and then compute $B_d$ as a summation. This has the drawback that $B_d$ can only be approximated, as only a finite number of terms of the Taylor series expansion can be computed in practice. However, depending on the number of terms computed, the truncation error can be made negligible.

First, recall the Taylor series expansion of the exponential function:
$$
e^a = 1 + \frac{a}{1!} + \frac{a^2}{2!} + \frac{a^3}{3!} + \cdots + \frac{a^n}{n!}.
$$

This same definition applies to the matrix exponential as well:
$$
e^A = I + \frac{A}{1!} + \frac{A^2}{2!} + \frac{A^3}{3!} + \cdots + \frac{A^n}{n!}.
$$

By manipulating this Taylor series, it is possible to produce a series representation for $B_d$:
$$
\begin{aligned}
e^A &= I + A + \frac{A^2}{2} + \frac{A^3}{6} + \cdots\\
\left(e^{A\,T_s}-I\right)
    &= A\,T_s + \frac{A^2\,T_s^2}{2} + \frac{A^3\,T_s^3}{6} + \cdots\\
\left(e^{A\,T_s}-I\right)A^{-1}B 
    &= \left(T_s\,I + \frac{A\,T_s^2}{2} + \frac{A^2\,T_s^3}{6} + \cdots\right)B.
\end{aligned}
$$

Thus,
$$
\boxed{B_d = \sum_{n=1}^{\infty} \frac{A^{n-1}\,T_s^n}{n!}B.}
$$

This can be done in MATLAB using `c2d()` or with SciPy using `cont2discrete()`. The short snippet below shows how to do the conversion with MATLAB.

``` MATLAB
sys_c = ss(A, B, C, D);
sys_d = c2d(sys_c, Ts, 'zoh');

A_d = sys_d.A;
B_d = sys_d.B;
```

## Output Equations

While the state equations require transformation of the $A$ and $B$ matrices to produce $A_d$ and $B_d$ matrices, the output equations remain valid. This is because the state equations involve dynamics while the output equations are merely algebraic. Therefore, 
$$
\underline{y}_k = C_d\,\underline{x}_k + D_d\,\underline{u}_k
$$
where $C_d = C$ and $D_d = D$.