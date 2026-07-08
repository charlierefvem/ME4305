---
title: Matrix Exponential
type: reference
tags:
  - mathematics
  - linear-algebra
  - matrices
  - state-space
source:
  course: ME405
  term: 2262
  lecture: 20
status: draft
---
## Motivation

The matrix exponential appears when solving systems of linear differential equations. It plays the same role for a state-space model that $e^{a\,t}$ plays for a scalar first-order ODE. In continuous-to-discrete conversion, the matrix exponential becomes especially important because the exact zero-input state update over one sample period is
$$
\underline{x}_{k+1}=e^{A\,T_s}\underline{x}_k
$$
## Matrix Exponential as the Solution to a Homogeneous LTI System

Consider the homogeneous system defined by $\dot{\underline{x}}=A\,\underline{x}$ with initial conditions $\underline{x}(0) = \underline{x}_0$.

Recall the eigenvalue decomposition of $A$. If $A$ has a full set of linearly independent eigenvectors, then it can be decomposed as
$$
A = Q\,\Lambda\, Q^{-1}
$$
where the columns of $Q$ are eigenvectors and $\Lambda$ is a diagonal matrix containing the corresponding eigenvalues.

Substituting the eigendecomposition into the zero-input state equation gives
$$
\dot{\underline{x}}(t) = Q\,\Lambda\, Q^{-1}\underline{x}(t).
$$

Pre-multiplying by $Q^{-1}$ gives
$$
Q^{-1}\,\dot{\underline{x}}(t) = \Lambda\, Q^{-1}\,\underline{x}(t).
$$

Then, let
$$
\underline{\eta}(t) = Q^{-1}\,\underline{x}(t)
$$
be a vector in the eigenbasis; consequently
$$
\dot{\underline{\eta}}(t) = Q^{-1}\,\dot{\underline{x}}(t).
$$

Substitution back into the state equations eliminates $\underline{x}$,
$$
\dot{\underline{\eta}}(t) = \Lambda\,\underline{\eta}(t).
$$

This is now a decoupled system of ODEs:
$$
\begin{bmatrix}
\dot{\eta}_1(t) \\
\vdots \\
\dot{\eta}_n(t)
\end{bmatrix}
=
\begin{bmatrix}
\lambda_1 & & \\
& \ddots & \\
& & \lambda_n
\end{bmatrix}
\begin{bmatrix}
\eta_1(t) \\
\vdots \\
\eta_n(t)
\end{bmatrix}.
$$

The solution to each decoupled ODE is an exponential:
$$
\eta_i(t) = \eta_i(0)\, e^{\lambda_i t}.
$$

Collecting the decoupled solutions into vector form,
$$
\begin{aligned}
\begin{bmatrix}
\eta_1(t) \\
\vdots \\
\eta_n(t)
\end{bmatrix}
&=
\begin{bmatrix}
e^{\lambda_1\,t} & & \\
& \ddots & \\
& & e^{\lambda_n\,t}
\end{bmatrix}
\begin{bmatrix}
\eta_1(0) \\
\vdots \\
\eta_n(0)
\end{bmatrix} \\[4pt]
\underline{\eta}(t) &= e^{\Lambda\,t}\,\underline{\eta}(0).
\end{aligned}
$$

The solutions can then be transformed back into the standard basis:
$$
\underline{x}(t) = Q\,\underline{\eta}(t).
$$

Therefore,
$$
\underline{x}(t) = Qe^{\Lambda\, t}\,\underline{\eta}(0).
$$

However, $\underline{\eta}(0) = Q^{-1}\,\underline{x}(0)$, so
$$
\underline{x}(t) = Q\,e^{\Lambda\, t}\,Q^{-1}\,\underline{x}_0.
$$

---

**Insight**: the matrix exponential is not found by exponentiating each element of $A$. It is a matrix function defined by the series representation
$$
e^{A\,t} = I + A\,t + \frac{(A\,t)^2}{2!} + \dots + \frac{(A\,t)^n}{n!}.
$$
The Taylor series may be manipulated by plugging in the eigenvalue decomposition, $Q\,\Lambda\, Q^{-1}$, in place of $A$ on the right-hand-side.
$$
\begin{aligned}
e^{A\,t} &= I + Q\,\Lambda\, Q^{-1}\,t 
        + \frac{(Q\,\Lambda\, Q^{-1}\,t)^2}{2!} + \dots 
        + \frac{(Q\,\Lambda\, Q^{-1}\,t)^n}{n!} + \dots \\[4pt]
e^{A\,t} &= I + Q\,\Lambda\, Q^{-1}\,t 
        + \frac{(Q\,\Lambda\, Q^{-1})\,(Q\,\Lambda\, Q^{-1})\,t^2}{2!} + \dots 
        + \frac{(Q\,\Lambda\, Q^{-1})^n\,t^n}{n!} + \dots \\[4pt]
e^{A\,t} &= Q\,I\,Q^{-1} + Q\,\Lambda\, Q^{-1}\,t 
        + \frac{Q\,\Lambda^2\, Q^{-1}\,t^2}{2!} + \dots 
        + \frac{Q\,\Lambda^n\, Q^{-1}\,t^n}{n!} + \dots \\[4pt]
e^{A\,t} &= Q\,\left( I + \Lambda\,t 
        + \frac{\Lambda^2\,t^2}{2!} + \dots 
        + \frac{\Lambda^n\, t^n}{n!} + \dots \right)\, Q^{-1} \\[4pt]
e^{A\,t} &= Q\,e^{\Lambda\,t}\, Q^{-1}
\end{aligned}
$$

For diagonalizable $A$, the derivation above shows that the matrix exponential can be computed from the Taylor series directly, or by using the eigenvector similarity transform to compute the exponential in the eigenbasis. This equivalence is referred to as the matrix exponential identity.

---


Using the matrix exponential identity, the final solution to the homogeneous system is
$$
\underline{x}(t) = e^{A\,t}\,\underline{x}_0.
$$

The derivation above assumes $A$ is diagonalizable. The final result, however, is more general, and remains valid even when $A$ cannot be diagonalized.

**Insight**: $e^{A\,t}$ is the state-transition matrix, sometimes referred to using the symbol $\Phi(t)$. It tells how the current state moves forward in time when there is no input. For sampled implementation, $e^{A\,T_s}$​ tells how the state evolves over one control-loop update period.

## Computing the Matrix Exponential

When computing matrix exponentials in software make sure to call the appropriate function. In MATLAB,
``` MATLAB
% Correct matrix exponential
Ad = expm(A*Ts)

% Incorrect matrix exponential (applies element by element)
Ad_wrong = exp(A*Ts)
```

And, in Python using SciPy,
``` Python
from scipy.linalg import expm, exp
import numpy as np

# Correct matrix exponential
Ad = expm(A * Ts)

# Incorrect matrix exponential (applies element by element)
Ad_wrong = np.exp(A * Ts)
```