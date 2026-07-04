---
title: Disturbance Observer Design
type: reference
tags:
- observers
- state-estimation
- observability
- pole-placement
- state-space
- disturbance
- augmentation
source:
course: ME405
term: 2262
lecture: 18
status: draft
---
# Motivation

A standard observer, as derived in [[reference_observer_design|Observer Design]], only works well in practice for systems with an accurate mathematical model, with accurate model parameters, and without the presence of unmodeled disturbances.

This limitation can be partially mitigated by building into the mathematical model of the system a model of the expected disturbances. The observer can then estimate the internal state of the system including the disturbances.

To embed the disturbance model into the system model the disturbance is initially treated as an additional input to the system. A discrete-time plant model with an unknown disturbance can be written as
$$
\begin{aligned}
\underline{x}_{k+1}
&= A_d\, \underline{x}_k
+ B_d\, \underline{u}_k
+ E_d\, \underline{d}_k, \\
\underline{y}_k
&= C_d\, \underline{x}_k
+ D_d\, \underline{u}_k.
\end{aligned}
$$
Here, $\underline{d}_k$ is an unknown disturbance vector, and $E_d$ describes how that disturbance enters the system dynamics.

For example, a load torque disturbance may not directly affect position, but it may directly affect angular acceleration. The matrix $E_d$ encodes that physical relationship.

# Effect on the nominal observer

The nominal observer has the form
$$
\hat{\underline{x}}_{k+1}
= A_d\, \hat{\underline{x}}_k
+ B_d\, \underline{u}_k
+ L
\left(
\underline{y}_k-\hat{\underline{y}}_k
\right),
$$
where
$$
\hat{\underline{y}}_k
= C_d\, \hat{\underline{x}}_k
+ D_d\, \underline{u}_k.
$$

Define the estimation error as
$$
\underline{e}_k
= \underline{x}_k-\hat{\underline{x}}_k.
$$

Then,
$$
\begin{aligned}
\underline{e}_{k+1}
&=
\underline{x}_{k+1}
-
\hat{\underline{x}}_{k+1} \\[4pt]
&=
\left(
A_d\,\underline{x}_k
+
B_d\,\underline{u}_k
+
E_d\,\underline{d}_k
\right)
-
\left(
A_d\,\hat{\underline{x}}_k
+
B_d\,\underline{u}_k
+
L
\left(
\underline{y}_k-\hat{\underline{y}}_k
\right)
\right).
\end{aligned}
$$

The known input terms cancel:
$$
\underline{e}_{k+1}
=
A_d\,
\left(
\underline{x}_k-\hat{\underline{x}}_k
\right)
+
E_d\,\underline{d}_k
-
L
\left(
\underline{y}_k-\hat{\underline{y}}_k
\right).
$$

Since
$$
\underline{y}_k-\hat{\underline{y}}_k
=
C_d\,
\left(
\underline{x}_k-\hat{\underline{x}}_k
\right),
$$
the error dynamics become
$$
\boxed{
\underline{e}_{k+1}
=
\left(
A_d-L\,C_d
\right)
\underline{e}_k
+
E_d\,\underline{d}_k.
}
$$

This shows the limitation of the nominal observer. The observer error still has the designed dynamics $A_o = A_d-L\,C_d$, but the unknown disturbance continually forces the error dynamics.

# Modeling the disturbance as an additional estimated state

To estimate the disturbance, the observer needs a model for how the disturbance changes. A common simple assumption is that the disturbance is constant or slowly varying over the time scale of interest:
$$
\underline{d}_{k+1}
=
\underline{d}_k.
$$

This lets us define an augmented state vector:
$$
\underline{x}_{a,k}
=
\begin{bmatrix}
\underline{x}_k \\
\underline{d}_k
\end{bmatrix}.
$$

Using the disturbed plant model,
$$
\underline{x}_{k+1}
=
A_d\,\underline{x}_k
+
B_d\,\underline{u}_k
+
E_d\,\underline{d}_k,
$$
and the disturbance model,
$$
\underline{d}_{k+1}
=
\underline{d}_k,
$$
the augmented dynamics are
$$
\begin{bmatrix}
\underline{x}_{k+1} \\
\underline{d}_{k+1}
\end{bmatrix}
=
\begin{bmatrix}
A_d & E_d \\
0 & I
\end{bmatrix}
\begin{bmatrix}
\underline{x}_k \\
\underline{d}_k
\end{bmatrix}
+
\begin{bmatrix}
B_d \\
0
\end{bmatrix}
\underline{u}_k.
$$

Therefore,
$$
\boxed{
\underline{x}_{a,k+1}
=
A_a\underline{x}_{a,k}
+
B_a\underline{u}_k
}
$$
where
$$
A_a
=
\begin{bmatrix}
A_d & E_d \\
0 & I
\end{bmatrix},
\qquad
B_a
=
\begin{bmatrix}
B_d \\
0
\end{bmatrix}.
$$

The output equation becomes
$$
\underline{y}_k
=
\begin{bmatrix}
C_d & 0
\end{bmatrix}
\begin{bmatrix}
\underline{x}_k \\
\underline{d}_k
\end{bmatrix}
+
D_d\underline{u}_k.
$$

Therefore,
$$
\boxed{
\underline{y}_k
=
C_a\,\underline{x}_{a,k}
+
D_d\,\underline{u}_k
}
$$
where
$$
C_a
=
\begin{bmatrix}
C_d & 0
\end{bmatrix}.
$$

# Augmented observer

The disturbance observer is now just a standard observer applied to the augmented system:
$$
\hat{\underline{x}}_{a,k+1}
=
A_a\,\hat{\underline{x}}_{a,k}
+
B_a\,\underline{u}_k
+
L_a\,
\left(
\underline{y}_k-\hat{\underline{y}}_k
\right),
$$
where
$$
\hat{\underline{y}}_k
=
C_a\,\hat{\underline{x}}_{a,k}
+
D_d\,\underline{u}_k.
$$

The augmented observer estimates both the original plant state and the disturbance:
$$
\hat{\underline{x}}_{a,k}
=
\begin{bmatrix}
\hat{\underline{x}}_k \\
\hat{\underline{d}}_k
\end{bmatrix}.
$$

The augmented estimation error is
$$
\underline{e}_{a,k}
=
\underline{x}_{a,k}
-
\hat{\underline{x}}_{a,k}.
$$

The augmented observer error dynamics are
$$
\boxed{
\underline{e}_{a,k+1}
=
\left(
A_a-L_a\,C_a
\right)\,
\underline{e}_{a,k}.
}
$$

Therefore, the augmented observer gain $L_a$ can be selected by placing the eigenvalues of
$$
A_a-L_a\,C_a.
$$

# Augmented observability

The augmented system must be observable. If the disturbance does not leave a distinguishable signature in the measured output, then no observer can reliably estimate it.

The augmented observability matrix is

$$
\mathcal{O}_a
=
\begin{bmatrix}
C_a \\
C_a\,A_a \\
C_a\,A_a^2 \\
\vdots \\
C_a\,A_a^{n_a-1}
\end{bmatrix},
$$

where $n_a$ is the number of augmented states.

The augmented system is observable if
$$
\boxed{
\operatorname{rank}(\mathcal{O}_a)=n_a.
}
$$

# Useful output selections

The augmented observer internally estimates both the original plant state and the disturbance. The estimated plant state can be extracted using
$$
\hat{\underline{x}}_k
=
\begin{bmatrix}
I & 0
\end{bmatrix}\,
\hat{\underline{x}}_{a,k}.
$$

The estimated disturbance can be extracted using
$$
\hat{\underline{d}}_k
=
\begin{bmatrix}
0 & I
\end{bmatrix}\,
\hat{\underline{x}}_{a,k}.
$$

# Continuous-time form
  
The same idea can be written in continuous time as  
$$
\begin{aligned}
\dot{\underline{x}}
    &=
    A\underline{x}
    +
    B\underline{u}
    +
    E\underline{d}, \\
\underline{y}  
    &=
    C\underline{x}
    +
    D\underline{u}.
\end{aligned}
$$
  
If the disturbance is modeled as constant or slowly varying, then
$$
\dot{\underline{d}}=0.
$$
  
The augmented continuous-time model is
$$
\begin{bmatrix}
\dot{\underline{x}} \\
\dot{\underline{d}}
\end{bmatrix}
=
\begin{bmatrix}
A & E \\
0 & 0
\end{bmatrix}
\begin{bmatrix}
\underline{x} \\
\underline{d}
\end{bmatrix}
+
\begin{bmatrix}
B \\
0
\end{bmatrix}\,
\underline{u},
$$
with output equation
$$
\underline{y}
=
\begin{bmatrix}
C & 0
\end{bmatrix}
\begin{bmatrix}
\underline{x} \\
\underline{d}
\end{bmatrix}
+
D\,\underline{u}.
$$