---
title: Observer Design
type: reference
tags:
  - observers
  - state-estimation
  - observability
  - pole-placement
  - state-space
source:
  course: ME405
  term: 2262
  lecture: 18
status: draft
---

# Motivation

In state-feedback control, it is convenient to write control laws as though every internal state of the system is available for direct measurement. In practice, that is usually not true. Sensors normally provide only a limited set of system outputs.

An observer is a feedback algorithm used to estimate the internal state of a dynamic system, $\underline{x}$, based exclusively on a mathematical model of the system along with knowledge of the system input, $\underline{u}$, and measurements of the system output, $\underline{y}$.

In general, the system output does not provide enough information to determine the internal state of the system explicitly. Therefore, additional information is required for the observer to estimate the internal state of the system accurately. Observers are therefore also often referred to as state estimators.

The output of the observer is an estimate of either the internal system state, denoted by $\hat{\underline{x}}$, or an estimate of the system output, denoted by $\hat{\underline{y}}$. Vectors with "hats" refer to estimated versions of the otherwise "unhatted" vectors.

The core principle behind observer design is feedback from the estimation error, or difference, between the estimated output and the measured output:
$$
\underline{y} - \hat{\underline{y}}.
$$

**Note**: the derivation below will be done using a discrete-time formulation to better align the derived equations with the intended digital implementation on a microcontroller. Therefore it may be useful to review the [[reference_z_domain|z-domain]] and methods for [[reference_continuous_to_discrete|continuous-to-discrete conversion]].

# Open-loop state-space model

In a discrete-time representation, the dynamics for the open-loop system are governed by the following state and output equations:
$$
\begin{aligned}
\underline{x}_{k+1}
    &= A_d\,\underline{x}_k 
     + B_d\,\underline{u}_k, \\
\underline{y}_k
    &= C_d\, \underline{x}_k
     + D_d\, \underline{u}_k.
\end{aligned}
$$

# Observability

In [[reference_state_feedback|state-feedback]] design, the notion of controllability is fundamental. Observability is a similar property of a dynamic system, but with a slightly different definition.

A system is observable if the measured output history contains enough information to distinguish the system’s internal state. In other words, different initial states must produce distinguishably different output histories when the input is known. Practically, this means each independent state direction must leave some detectable signature in the measured output, either directly through $C_d$ or indirectly through the system dynamics.

Like controllability, the observability of a system can be determined by checking the rank of a matrix. For the discrete-time model presented above, the observability matrix is
$$
\mathcal{O}
=
\begin{bmatrix}
C_d \\
C_d\,A_d \\
C_d\,A_d^2 \\
\vdots \\
C_d\, A_d^{n-1}
\end{bmatrix}.
$$

The same idea applies in continuous time using $A$ and $C$.

If the observability matrix has full column rank, meaning rank equal to the system order $n$, then the system is observable. If a system is observable it is possible to design an observer that reconstructs an estimate of the internal state of the system using limited information from the system output.

# Observer equations

In the open-loop state equations above, $\underline{x}$ is the state vector, $\underline{y}$ is the output vector, and $\underline{u}$ is the input vector. In the observer equations, $\hat{\underline{x}}$ is the estimated state vector, and $\hat{\underline{y}}$ is the estimated output vector.

To help keep all of the parameters straight, refer to the table below.

| Symbol                  | Dimensionality           | Meaning                                                                                                                            |
| ----------------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| $\underline{x}_k$       | $\mathbb{R}^n$           | The true state vector.                                                                                                             |
| $\underline{u}_k$       | $\mathbb{R}^m$           | The input vector.                                                                                                                  |
| $\underline{y}_k$       | $\mathbb{R}^p$           | The true output vector.                                                                                                            |
| $\underline{w}_k$       | $\mathbb{R}^{m+p}$       | The known-information vector used with the observer. Consists of $\underline{u}_k$ vertically concatenated with $\underline{y}_k$. |
| $\hat{\underline{x}}_k$ | $\mathbb{R}^n$           | The estimated state vector. Tracks $\underline{x}_k$ for a working observer.                                                       |
| $\hat{\underline{y}}_k$ | $\mathbb{R}^p$           | The estimated output vector. Tracks $\underline{y}_k$ for a working observer.                                                      |
| $A_d$                   | $\mathbb{R}^{n\times n}$ | The discrete-time state-to-state coupling matrix.                                                                                  |
| $B_d$                   | $\mathbb{R}^{n\times m}$ | The discrete-time input-to-state coupling matrix.                                                                                  |
| $C_d$                   | $\mathbb{R}^{p\times n}$ | The discrete-time state-to-output coupling matrix.                                                                                 |
| $D_d$                   | $\mathbb{R}^{p\times m}$ | The discrete-time input-to-output coupling matrix.                                                                                 |
| $L$                     | $\mathbb{R}^{n\times p}$ | The discrete-time observer gain.                                                                                                   |
where $n$ is the system order, $m$ is the number of system inputs, and $p$ is the number of system outputs.

The observer model has a similar structure to the open-loop plant model, but it also includes a feedback term based on output estimation error:
$$
\begin{aligned}
\hat{\underline{x}}_{k+1}
    &= A_d\,\hat{\underline{x}}_k
     + B_d\,\underline{u}_k 
     + L\,\left(\underline{y}_k-\hat{\underline{y}}_k\right) \\
\hat{\underline{y}}_k
    &= C_d\,\hat{\underline{x}}_k 
     + D_d\,\underline{u}_k.
\end{aligned}
$$

The output estimation error, $\underline{y}_k-\hat{\underline{y}}_k$, is often referred to as the **innovation** or **measurement residual**.

The gain matrix $L$ determines how strongly the observer responds to the output estimation error. The observer is effectively a real-time simulation of the system model, corrected by feedback from the difference between measured output and estimated output.

Substitute the output-estimate equation into the observer state equation:
$$
\hat{\underline{x}}_{k+1}
    = A_d\,\hat{\underline{x}}_k
    + B_d\,\underline{u}_k
    + L\left(
        \underline{y}_k 
        - C_d\, \hat{\underline{x}}_k
        - D_d\, \underline{u}_k
    \right).
$$

Grouping terms gives
$$
\hat{\underline{x}}_{k+1}
= \left(A_d-L\,C_d\right)\,\hat{\underline{x}}_k
+ \left(B_d-L\,D_d\right)\,\underline{u}_k
+ L\,\underline{y}_k.
$$

This can be written as a new state-space model for the observer using block matrix notation:
$$
\hat{\underline{x}}_{k+1}
= \left(A_d-L\,C_d\right)\,\hat{\underline{x}}_k
+ \begin{bmatrix} B_d-L\,D_d & L \end{bmatrix}
\begin{bmatrix}
\underline{u}_k \\
\underline{y}_k
\end{bmatrix}.
$$

Define observer matrices
$$
A_o = A_d-L\,C_d
$$
and
$$
B_o = \begin{bmatrix} B_d-L\,D_d & L \end{bmatrix}.
$$

Also define the augmented known-information vector
$$
\underline{w}_k
=
\begin{bmatrix}
\underline{u}_k \\
\underline{y}_k
\end{bmatrix}.
$$

Using these new symbols, the observer can be written compactly as
$$
\hat{\underline{x}}_{k+1}
    = A_o\,\hat{\underline{x}}_k
    + B_o\,\underline{w}_k.
$$

The observer internally computes both a state estimate and an output estimate. The estimated plant output is
$$
\hat{\underline{y}} = C_d\, \hat{\underline{x}}_k + D_d\,\underline{u}_k.
$$

In many control implementations, the signal actually wanted from the observer is the estimated state itself:
$$
\hat{\underline{x}}_{\text{out},k} = \underline{x}_k.
$$

The observer dynamics depend on the gain matrix $L$ which can be tuned by considering the error dynamics for the observer.

# Error Dynamics

Let the observer error be
$$
\underline{e} = \underline{x} - \hat{\underline{x}}.
$$

Consequently at sample $k$,
$$
\underline{e}_k = \underline{x}_k - \hat{\underline{x}}_k,
$$
and at sample $k+1$,
$$
\underline{e}_{k+1} = \underline{x}_{k+1} - \hat{\underline{x}}_{k+1}.
$$

Substitution into the plant and observer dynamics allows the $B_d\, \underline{u}_k$ terms to cancel,
$$
\begin{aligned}
\underline{e}_{k+1}
    &= \left(
        A_d\, \underline{x}_k
        +B_d\, \underline{u}_k
    \right)
    -\left(
        A_d\, \hat{\underline{x}}_k
        +B_d\, \underline{u}_k
        +L
        \left(
            \underline{y}_k
            -\hat{\underline{y}}_k
        \right)
    \right), \\
\underline{e}_{k+1}
    &= A_d\, 
    \left(
        \underline{x}_k
        -\hat{\underline{x}}_k
    \right)
    - L
    \left(
        \underline{y}_k
        -\hat{\underline{y}}_k
    \right).
\end{aligned}
$$
This ideal error-dynamics derivation assumes the observer uses the same known input as the plant model and that model mismatch and measurement noise are neglected. In real systems, those effects appear as disturbances in the observer error dynamics.

Using
$$
\underline{y}_k
    = C_d\, \underline{x}_k
    + D_d\, \underline{u}_k,
$$
and
$$
\hat{\underline{y}}_k
    = C_d\, \hat{\underline{x}}_k
    + D_d\, \underline{u}_k,
$$
the output estimation error becomes
$$
\begin{aligned}
\underline{y}_k-\hat{\underline{y}}_k
    &= C_d\, \underline{x}_k
     + D_d\, \underline{u}_k
     - C_d\, \hat{\underline{x}}_k
     - D_d\, \underline{u}_k, \\
\underline{y}_k-\hat{\underline{y}}_k
    &= C_d\, \left(
        \underline{x}_k
        -\hat{\underline{x}}_k
    \right), \\
\underline{y}_k-\hat{\underline{y}}_k
    &= C_d\, \underline{e}_k.
\end{aligned}
$$

Therefore the error dynamics reduces to
$$
\begin{aligned}
\underline{e}_{k+1}
    &= A_d\,\underline{e}_k-L\,C_d\,\underline{e}_k, \\
\underline{e}_{k+1}
    &= \left(A_d-L\,C_d\right)\,\underline{e}_k,
\end{aligned}
$$
or, finally,
$$
\boxed{\underline{e}_{k+1} = A_o\,\underline{e}_k.}
$$

This is a new homogeneous system that governs the observer error dynamics. In other words, the eigenvalues of $A_o = A_d-L\,C_d$ determine how the estimation error evolves.

The clean cancellation above depends on the model and input being accurate. If the plant experiences an unknown input or disturbance, that disturbance appears in the error dynamics. One way to handle that case is to augment the state with a [[reference_disturbance_observer|Disturbance Observer Design]].

# Observer Tuning via Traditional Pole Placement

The observer error dynamics are
$$
\underline{e}_{k+1} = \left(A_d-L\,C_d\right)\,\underline{e}_k.
$$

Compare this to the closed-loop dynamics for full-state feedback,
$$
\underline{x}_{k+1} = \left(A_d-B_d\,K\right)\,\underline{x}_k.
$$

The general form of the expressions matches, but the placement of the gain matrix is different:
$$
A_d-B_d\,K
\qquad \text{versus} \qquad
A_d-L\,C_d.
$$

For full-state feedback, the gain $K$ appears on the right side of $B_d$. For the observer, the gain $L$ appears on the left side of $C_d$.

To make the observer pole-placement problem resemble the full-state-feedback problem, use the transpose of $A_o=A_d-L\,C_d$:

$$
\begin{aligned}
A_o^T &= \left(A_d-L\,C_d\right)^T \\
A_o^T &= A_d^T - \left(L\,C_d\right)^T \\
A_o^T &= A_d^T - C_d^T\,L^T.
\end{aligned}
$$

This transposed expression now has the same structure as the full-state-feedback expression $A_d-B_d\,K$. In the transposed observer problem, $A_d^T$ plays the role of $A_d$, $C_d^T$ plays the role of $B_d$, and $L^T$ plays the role of $K$.

In MATLAB, the observer gain can therefore be computed with
``` MATLAB
L = place(Ad', Cd', p_obs)';
```

For comparison, the full-state-feedback gain is computed with
``` MATLAB
K = place(Ad, Bd, p_ctrl);
```

Recall that for a discrete system stable poles are within the unit disk. See the "Pole Mapping" section of [[reference_continuous_to_discrete|Continuous to Discrete Conversion]] for details on mapping poles to discrete time.

**Insight**: according to the separation principle, for a linear model the controller poles and observer error poles can be assigned independently. In practice, the observer still affects the implemented control signal through noise, model error, saturation, and finite sampling effects, so observer poles should be chosen with engineering judgment.

The observer poles should be selected to be faster than any controller poles, so the state estimate converges quickly relative to the closed-loop response. In discrete time, “faster” generally means the observer poles are placed farther inside the unit disk, often closer to the origin than the dominant closed-loop controller poles. They should not be placed arbitrarily fast, because aggressive observer gains tend to amplify measurement noise and model mismatch.

# Summary

Observer design mirrors state-feedback design but focuses on estimation error dynamics instead of closed-loop system dynamics. In full-state feedback, the controller gain $K$ is chosen to shape the poles of $A_c = A_d-B_d\,K$. In observer design, the observer gain $L$ is chosen to shape the poles of $A_o = A_d-L\,C_d$.