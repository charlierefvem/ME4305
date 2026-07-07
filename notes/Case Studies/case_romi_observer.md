---
title: Romi Observer
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
status: draft
---

### Another nuance in observing Romi

Recall Romi's output equations:

$$
\underbrace{
\begin{bmatrix}
s_L \\
s_R \\
\psi \\
\dot{\psi}
\end{bmatrix}}_{\underline{y}}
=
\underbrace{
\begin{bmatrix}
0 & 0 & 1 & -\frac{w}{2} \\
0 & 0 & 1 & \frac{w}{2} \\
0 & 0 & 0 & 1 \\
-\frac{r}{w} & \frac{r}{w} & 0 & 0
\end{bmatrix}}_{C}
\underbrace{
\begin{bmatrix}
\Omega_L \\
\Omega_R \\
s \\
\psi
\end{bmatrix}}_{\underline{x}}.
$$

The column space of this matrix describes every possible nonzero $\underline{y}$ given a nonzero $\underline{x}$. The $C$ matrix, as shown above, is only of rank 3, as observed by the first and second columns being linearly dependent. Because the $C$ matrix is a $4 \times 4$ matrix and has rank 3, there must exist a one-dimensional left nullspace.

The left nullspace, $\operatorname{null}(C^T)$, describes a new vector space orthogonal to the column space of $C$. In other words, it gives a way to identify combinations of outputs that cannot vary independently of one another.

The left nullspace is found by setting

$$
C^T\,\underline{v} = \underline{0}
$$

and then determining the space spanned by $\underline{v}$. For Romi,

$$
\begin{bmatrix}
0 & 0 & 0 & -\frac{r}{w} \\
0 & 0 & 0 & \frac{r}{w} \\
1 & 1 & 0 & 0 \\
-\frac{w}{2} & \frac{w}{2} & 1 & 0
\end{bmatrix}
\begin{bmatrix}
1 \\
-1 \\
w \\
0
\end{bmatrix}
=
\begin{bmatrix}
0 \\
0 \\
0 \\
0
\end{bmatrix}.
$$

Therefore, one vector in the left nullspace is

$$
\underline{v} =
\begin{bmatrix}
1 \\
-1 \\
w \\
0
\end{bmatrix}.
$$

The left nullspace can then be used to develop a set of constraints on $\underline{y}$. A physically consistent output vector must be orthogonal to this left-nullspace vector:

$$
\underline{y}^T\underline{v}=0.
$$

For the Romi output vector, this means

$$
\begin{bmatrix}
s_L & s_R & \psi & \dot{\psi}
\end{bmatrix}
\begin{bmatrix}
1 \\
-1 \\
w \\
0
\end{bmatrix}
=0.
$$

Therefore,

$$
s_L - s_R + w\psi = 0.
$$

Solving for heading gives

$$
\boxed{\psi = \frac{s_R-s_L}{w}.}
$$

This is the practical constraint: the encoder positions and the yaw angle cannot be initialized independently if the state and output values are intended to be mutually consistent.

> Technical Note (Instructor Review): The source phrasing describes the left nullspace as a space of nonzero $\underline{y}$ associated with $\underline{x}=\underline{0}$. Since $\underline{y}=C\underline{x}$ gives $\underline{y}=\underline{0}$ when $\underline{x}=\underline{0}$, this draft interprets the practical point as the output-consistency constraint $\underline{y}^T\underline{v}=0$ for every $\underline{y}$ in the column space of $C$.

## Insights

### Lab preparation outside the firmware

Outside of the firmware, probably using MATLAB, perform the following.

1. Find numeric values for the $A$, $B$, $C$, and $D$ matrices associated with the open-loop system. This should have been done in HW 0x03, but the \[\[Lecture 17\]\] notes may also be useful.
2. Either by hand, or with a pole placement tool such as the MATLAB `place()` function, design an \[\[Observer\]\] for the system and determine the observer gain $L$.
3. Find numeric values for

   $$
   A_O = A - LC
   $$

   and

   $$
   B_O = \begin{bmatrix}B & L\end{bmatrix}
   $$

   using the observer gain.
4. Either by hand, or with a discretization tool such as `c2d()`, transform the system matrices $A_O$ and $B_O$ from continuous time to discrete time to find $A_D$ and $B_D$.

### Firmware implementation checklist

In the firmware, perform the following by embedding code into the task-based structure.

#### 1. Initialize encoders and heading consistently

In one task, in an initialization state, make sure that the encoder values and heading values agree with one another. That is, do one of the following.

Set the initial heading / yaw angle based on the encoder values:

$$
\psi = \frac{s_R-s_L}{w}.
$$

Or set the initial encoder values based on the heading value:

$$
s_R = \frac{w}{2}\psi
$$

and

$$
s_L = -\frac{w}{2}\psi.
$$

#### 2. Concatenate the input and measurements

As the tasks run, collect values associated with the system input and the system measurements, and compose a vector from them by concatenation:

$$
\underline{u}^{*}
=
\begin{bmatrix}
\underline{u} \\
\underline{y}
\end{bmatrix}
=
\begin{bmatrix}
V_L \\
V_R \\
s_L \\
s_R \\
\psi \\
\dot{\psi}
\end{bmatrix}.
$$

#### 3. Run the observer update

As the tasks iterate, run the observer update equations:

$$
\hat{\underline{x}}_{k+1}
= A_D\,\hat{\underline{x}}_k + B_D\,\underline{u}^{*}_k
$$

$$
\hat{\underline{y}}_k = C\,\hat{\underline{x}}_k.
$$
