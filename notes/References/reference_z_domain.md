---
title: Discrete Time Systems
type: reference
tags:
  - digital-filters
  - discrete-systems
  - z-transform
  - continuous-to-discrete
source:
  course: ME507
  term: 2264
  lecture: 16
status: draft
---

## Motivation

Physical signals and many physical systems are naturally continuous in time, but measurements on a microcontroller are sampled at discrete time intervals. Once a signal is sampled, filters and controllers are usually implemented as difference equations rather than continuous-time differential equations.

We can apply the Laplace transform to ODEs in continuous-time to develop transfer functions that exist in the s-domain. Likewise, we can apply similar transformations to discrete-time systems to develop transfer functions that exist in the z-domain.

Understanding the z-domain is necessary even if controllers and filters are designed in continuous-time because the final implementation must be handled digitally by a microcontroller.

## Discrete signals and the z-domain

In mechanical engineering courses, focus is usually spent on the s-domain, which allows modeling of continuous systems. Many continuous systems are modeled using ordinary differential equations, and through the Laplace transform these differential equations can be rewritten as transfer functions in the s-domain.

For stable continuous-time systems, all poles, or roots of the characteristic polynomial, must be in the left half-plane. That is, the real part of $s$ must be negative. One way to understand this is to consider the exponential function $e^{st}$. For this function to decay over time as $t \to \infty$, the value of $s$ must be in the left half-plane, or more specifically, $\text{Re}(s) < 0$.

![The s-domain stability diagram shows horizontal Re(s) and vertical Im(s) axes. The left half-plane is shaded and labeled "stable region," emphasizing that continuous-time poles must lie to the left of the imaginary axis for stable decay.](images/discrete_systems/stability_s_plane.svg)

Discrete signals are more pertinent when measurements are sampled at a regular interval. Purely discrete-time models are usually described using difference equations rather than differential equations. For example, instead of
$$
\dot{x} = f(x),
$$
where the rate of change of the states depends on the present states, we might write
$$
x_{k+1} = f(x_k),
$$
where future states are directly dependent on present states were $x_k=x(k\,T_s)$.

For difference equations, the z-transform replaces the Laplace transform, resulting in a transfer function in the z-domain rather than the s-domain. For linear discrete-time systems, stability depends on a geometric sequence, because the natural response contains terms of the form $z^k$. The question becomes: for what values of $z$ will the function $z^k$ decay as the number of samples grows as $k \to \infty$?

---

**Insight:** why do we care about $z^k$? As derived below, we will find that $z = e^{s\,T_s}$. Plugging this into the stability condition of decaying $e^{s\,t}$ while letting $t=k\,T_s$ we find
$$
\begin{aligned}
e^{st} &= e^{st}, \\
e^{st} &= e^{s\,(k\,T_s)}, \\
e^{st} &= e^{k\,(s\,T_s)}, \\
e^{st} &= \left( e^{s\,T_s} \right)^k, \\
e^{st} &= z^k.
\end{aligned}
$$
So the discrete-time stability condition that $z^k$ decays is equivalent to the stability condition in continuous time.

---

Rather than requiring poles to be in the left half-plane, a stable discrete-time system must have poles within the unit disk, or more specifically, $\lvert z \rvert < 1$. Poles on the boundary, such as the imaginary axis in continuous-time or the unit circle in discrete-time, are marginal cases rather than asymptotically stable cases.

![The z-domain stability diagram shows a circle centered at the origin. The inside of the circle is shaded and labeled "stable region," emphasizing that discrete-time poles must lie inside the unit circle. The source drawing labels the axes similarly to the s-domain sketch.](images/discrete_systems/stability_z_plane.svg)

### Unit delay and continuous-to-discrete transforms

The unit delay links intuition about the s-domain to intuition about the z-domain, including the way the continuous-time stability region in the s-plane maps into the discrete-time stability region in the z-plane. It is possible to convert between the s-domain and the z-domain. Converting from continuous time to discrete time is called [[reference_continuous_to_discrete|Continuous to Discrete Conversion]]. It is also possible to convert back from discrete time to continuous time.

Common methods include:
- The zero-order hold, which assumes the input applied to the continuous-time system is held constant between samples.
- The first-order hold, which assumes the input applied to the continuous-time system changes linearly between samples.
- The bilinear transform, which maps the left half of the s-plane to the unit disk in the z-plane, and maps the imaginary axis of the s-plane to the unit circle in the z-plane.

These rational approximations are not equally stability-preserving: forward difference can map stable continuous-time poles outside the unit circle if $T_s$​ is too large, while backward difference and the bilinear transform map the continuous-time left half-plane into the discrete-time unit disk.

Recall the Laplace transform of a causal time-delayed function:
$$
\mathcal{L}\left\{f(t-T_s)\right\}
= e^{-s\,T_s}\mathcal{L}\left\{f(t)\right\}.
$$

From this relationship, multiplying by $e^{-s\,T_s}$ in the s-domain is equivalent to applying a delay in the time domain.

Standard convention is to define the operator $z^{-1}$ as the unit delay:
$$
z^{-1} = e^{-s\,T_s}.
$$

In other words, multiplying a signal by $z^{-1}$ delays that signal by one sample:
$$
z^{-1}\,x(z) \leftrightarrow x_{k-1}.
$$

Therefore, the ideal relationship between $s$ and $z$ is
$$
z = e^{s\,T_s}
$$
and
$$
s = \frac{1}{T_s}\log(z).
$$

Strictly speaking, the complex logarithm is multivalued, so converting from $z$ back to $s$ is not unique unless a branch is chosen. This is closely related to aliasing: continuous-time frequencies separated by integer multiples of the sampling frequency map to the same discrete-time angle.

---

**Insight:** from $z = e^{s\,T}$ it is easy to show that the left half plane is mapped to the unit disk by applying Euler's formula after splitting $s$ into real and imaginary components.

$$
\begin{aligned}
z &= e^{s\,T_s} \\
z &= e^{(a+bi)\,T_s} \\
z &= e^{a\,T_s}e^{i\,b\,T_s} \\
z &= e^{a\,T_s} \left( \cos(b\,T_s) + i\sin(b\,T_s) \right) \\
\end{aligned}
$$

The preceding equation shows that for the complex number $s$, the real part, $\text{Re}(s) = a$, determines the distance (radius) between the complex number $z$ and the origin while the imaginary part, $\text{Im}(s) = b$, determines the direction $z$ extends away from the origin. If we then plug in values of $a \le 0$ we can see that the radius starts at $e^0=1$ and decays toward zero as $s$ grows more negative. 

---

Directly substituting the exact inverse relationship $s=\frac{1}{T_s}\log z$ into a continuous-time transfer function generally produces a non-rational expression in $z$. That direct substitution is therefore not usually a practical implementation method. Exact hold-equivalent methods, such as zero-order hold and first-order hold, instead discretize the underlying state equations and can still produce finite-order rational discrete-time models for finite-dimensional LTI systems. The finite-difference and bilinear substitutions below are approximate rational substitutions that preserve transfer-function algebra.

These approximate transformations simplify the ideal relationship so the order and rationality of the system transfer function can be preserved through the transformation.

Several methods approximate the relationship between $z$ and $s$ based on the Taylor series expansion of the exponential function. Before substituting the Taylor series expansion, it is useful to notice that the ideal transformation can be written in several equivalent ways, each leading to a different approximate transformation.


| Ideal Transform                         | Approximate Transform                    | Approximate Inverse  Transform            | Method                                                        |
| --------------------------------------- | ---------------------------------------- | ----------------------------------------- | ------------------------------------------------------------- |
| $z = e^{s\,T_s}$                        | $s \approx \frac{z-1}{T_s}$              | $z \approx 1 + s\,T_s$                    | Forward Difference Method                                     |
| $z= \frac{1}{e^{-s\,T_s}}$              | $s \approx \frac{z-1}{z\,T_s}$           | $z \approx \frac{1}{1-s\,T_s}$            | Backward Difference Method                                    |
| $z= \frac{e^{s\,T_s/2}}{e^{-s\,T_s/2}}$ | $s \approx \frac{2}{T_s}\frac{z-1}{z+1}$ | $z \approx \frac{1+s\,T_s/2}{1-s\,T_s/2}$ | Trapezoid Method (aka Tustin's Method aka Bilinear Transform) |

### Example 1

This example shows a derivation of the continuous to discrete transform based on the forward difference method and then motivates its interpretation through a simple example transformation.

For the forward difference method start with
$$
z = e^{s\,T_s},
$$
and expand the exponential function as a Taylor series:
$$
\begin{aligned}
z &= e^{s\,T_s} \\
z &= 1 + s\,T_s + \frac{(s\,T_s)^2}{2!} + \cdots + \frac{(s\,T_s)^n}{n!}.
\end{aligned}
$$

Keeping only the first two terms,
$$
z \approx 1 + s\,T_s.
$$

Solving for $s$ gives
$$
s \approx \frac{z-1}{T_s}.
$$

Now consider the simple equation
$$
y(t) = \dot{x}(t)
$$

after applying the Laplace transform, assuming no initial conditions, and applying the continuous-to-discrete conversion:

$$
\begin{aligned}
y(s) &= s\,x(s) \\
y(z) &= \frac{z-1}{T_s}x(z) \\
y(z) &= \frac{1}{T_s}\left(z\,x(z)-x(z)\right).
\end{aligned}
$$

Since $z$ corresponds to advancing one sample in this algebraic manipulation,
$$
y_k = \frac{1}{T_s}\left(x_{k+1}-x_k\right).
$$

The diagram below makes geometric sense of this definition for $y_k$.

![The forward-difference diagram shows two sample points, (t_k, x_k) and (t_{k+1}, x_{k+1}), connected by a dashed line. A vertical arrow labels Delta x = x_{k+1} - x_k and a horizontal arrow labels Delta t = t_{k+1} - t_k = Ts. An annotation identifies y_k as the slope at step k.](images/discrete_systems/c2d_forward.svg)

According to this transformation, the derivative is approximated by looking forward at the finite difference between the current sample and the next sample, hence the name forward finite difference.

As written, the forward difference estimates the derivative at step $k$ using $x_{k+1}$​, so it is not directly causal as a real-time differentiator. In a state-update equation, however, it is causal because the purpose of the update is to compute $x_{k+1}$​ from known values at step $k$. This can also be concluded by noticing that $\frac{z-1}{T_s}$ is an improper transfer function.

### Example 2

This example shows a derivation of the continuous to discrete transform based on the backward difference method and then motivates its interpretation through a simple example transformation.

For the backward difference method start with
$$
z = \frac{1}{e^{-s\,T_s}},
$$
and expand the exponential function as a Taylor series
$$
e^{-s\,T_s} = 1 - s\,T_s + \frac{(s\,T_s)^2}{2!} - \cdots + \frac{(-s\,T_s)^n}{n!}
$$
and then plug the expansion into the definition of $z$, keeping only the first two terms
$$
z = \frac{1}{e^{-s\,T_s}} \approx \frac{1}{1-s\,T_s}.
$$

Solving for $s$ gives
$$
s \approx \frac{z-1}{z\,T_s}.
$$

Now reconsider the simple equation
$$
y(t) = \dot{x}(t).
$$

Applying the backward difference conversion gives
$$
y_{k+1} = \frac{1}{T_s}\left(x_{k+1}-x_k\right).
$$

The diagram below makes geometric sense of this definition for $y_{k+1}$.

![The backward-difference diagram shows the same two sample points, (t_k, x_k) and (t_{k+1}, x_{k+1}). The finite difference still uses Delta x = x_{k+1} - x_k over Delta t = T, but the annotation identifies y_{k+1} as the slope at step k+1.](images/discrete_systems/c2d_backward.svg)

According to this transformation, the derivative is approximated by looking backward at the finite difference between the previous sample and the current sample, hence the name backward finite difference.

This can be made clearer by re-indexing,
$$
y_k = \frac{1}{T_s}\left(x_k-x_{k-1}\right).
$$

### Example 3

This example shows a derivation of the continuous to discrete transform based on the trapezoidal method, also called Tustin's method or the bilinear transform, and then motivates its interpretation through a simple example transformation.

For the trapezoidal method start with
$$
z = \frac{e^{s\,T_s/2}}{e^{-sT_s/2}},
$$
and expand the exponential functions as Taylor series. Using the first-order approximation gives
$$
z \approx \frac{1+s\,T_s/2}{1-s\,T_s/2}.
$$

Solving for $s$ gives the standard bilinear transform:
$$
s \approx \frac{2}{T_s}\frac{z-1}{z+1}.
$$

Now reconsider the simple equation
$$
y(t) = \dot{x}(t).
$$

Applying the bilinear transform gives
$$
\frac{1}{2}\left(y_{k+1}+y_k\right)
= \frac{1}{T_s}\left(x_{k+1}-x_k\right).
$$


The diagram below makes geometric sense of this definition for $\frac{1}{2}(y_{k+1} + y_{k})$.

![The trapezoidal-method diagram shows two sample points connected by a dashed line. The vertical change is labeled Delta x = x_{k+1} - x_k and the horizontal change is labeled Delta t = T. An annotation indicates that one-half times (y_{k+1} + y_k) is the average slope at steps k and k+1.](images/discrete_systems/c2d_trapezoidal.svg)

According to this transformation, the average of the derivative at the next sample and the current sample is approximated by looking at the finite difference between those two samples. This method is essentially the average of the forward and backward difference methods, which is equivalent to a trapezoidal difference method.

## Summary
1) Continuous-time stable poles satisfy $\text{Re}⁡(s)<0$.
2) Discrete-time stable poles satisfy $\lvert z \rvert < 0$.
3) Exact sampled exponentials obey $z=e^{s\,T_s}$​.
4) Practical C2D methods either discretize the state equations directly or use rational approximations such as forward difference, backward difference, and bilinear/Tustin.