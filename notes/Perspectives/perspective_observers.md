---
title: General Observer Intuition
type: perspective
status: dirty
---
## Disturbance Observers vs Integral Action — Key Insights

### 1. Separation Principle with Augmented Disturbance States

The separation principle still holds when using an augmented-state observer that estimates disturbances.

For a plant

$$
\dot{x} = Ax + Bu + Ed
$$

with a constant disturbance model

$$
\dot{d} = 0,
$$

define the augmented state

$$
x_a =
\begin{bmatrix}
x \\
d
\end{bmatrix}.
$$

An observer can estimate both plant states and disturbance:

$$
\hat{x}_a =
\begin{bmatrix}
\hat{x} \\
\hat{d}
\end{bmatrix}.
$$

The closed-loop poles are the union of:

- Controller poles of the augmented system
- Observer poles of the augmented system

The controller and observer may therefore still be designed independently.

---

### 2. The Observer Does Not Compensate Disturbances

A key realization:

The observer's job is **estimation**, not compensation.

The observer estimates

$$
\hat d
$$

but does not apply any control action.

Compensation occurs only when the controller uses that estimate:

$$
u = u_{fb} - u_{dist}.
$$

For example,

$$
u = u_{fb} - \frac{\hat d}{K_m}.
$$

---

### 3. The Important Quantity Is Estimation Error

The most important insight from the discussion:

The observer is not trying to drive

$$
d \rightarrow 0
$$

or

$$
\hat d \rightarrow 0.
$$

Instead it is trying to drive

$$
\tilde d = d - \hat d
$$

to zero.

The observer dynamics are fundamentally estimation-error dynamics.

A perfect observer converges to

$$
\hat d = d,
$$

which is generally nonzero.

---

### 4. Why Disturbance Compensation Does Not Hide the Disturbance

Suppose

$$
\dot{\omega}
=
-\frac{1}{\tau}\omega
+
K_m V
+
d
$$

and

$$
V
=
V_{fb}
-
\frac{\hat d}{K_m}.
$$

Substituting:

$$
\dot{\omega}
=
-\frac{1}{\tau}\omega
+
K_m V_{fb}
+
(d-\hat d).
$$

The plant no longer sees the full disturbance.

It sees only the residual disturbance

$$
d-\hat d.
$$

If

$$
\hat d = d,
$$

the disturbance disappears from the plant dynamics.

This does **not** mean the observer has lost track of the disturbance.

It means the disturbance has been perfectly compensated.

---

### 5. Integral Action and Disturbance Estimation Solve Similar Problems

Integral control introduces

$$
\dot z = r-y.
$$

A disturbance observer introduces

$$
\dot{\hat d} = L_d(y-\hat y).
$$

Both are mechanisms for learning the steady-state bias required to reject a constant disturbance.

Integral action learns it indirectly through accumulated tracking error.

Disturbance estimation learns it directly through model mismatch.

Both ultimately generate the control bias needed to satisfy

$$
Bu + d = 0.
$$

---

### 6. Why Integral Action and Disturbance Feedback Can Feel Redundant

Both states can contribute a steady-state control bias:

$$
u = -Kx - k_i z - k_d \hat d.
$$

This can create the impression that there must be hidden dynamics.

The important distinction:

- Similar purpose does not imply identical dynamics.
- Similar purpose does not imply zero dynamics.

The integrator and disturbance estimate are generally driven by different signals:

$$
\dot z = r-y
$$

vs.

$$
\dot{\hat d} = L_d(y-\hat y).
$$

Therefore they usually evolve differently.

Only special choices of gains and structure would make them dynamically equivalent.

---

### 7. Connection to Pole-Zero Cancellation

The redundancy intuition is closer to pole-zero cancellation than to classical zero dynamics.

If two controller states learn the same bias term and evolve identically, one state direction can become effectively invisible to the plant.

This resembles:

- Non-minimal realizations
- Pole-zero cancellation
- Hidden internal modes

However, this only occurs under special circumstances.

In general, the states remain dynamically distinct.

---

### 8. State Counting for the Motor Example

Plant states:

$$
x =
\begin{bmatrix}
\theta \\
\omega
\end{bmatrix}
$$

Plant order = 2

Observer states:

$$
\hat{x}_a =
\begin{bmatrix}
\hat\theta \\
\hat\omega \\
\hat d
\end{bmatrix}
$$

Observer order = 3

Total without integrator:

$$
2 + 3 = 5
$$

Adding integral control:

$$
2 + 3 + 1 = 6
$$

Using disturbance feedback instead of integral action:

$$
2 + 3 = 5
$$

Thus disturbance feedback can reduce controller order by one state in this implementation.

---

### 9. Why Integral Action May Still Be Better

A major practical insight:

The disturbance estimate is generated from observer innovation:

$$
y-\hat y.
$$

If measurements are quantized, then

$$
\hat d
$$

contains:

- True disturbances
- Model mismatch
- Quantization artifacts

Therefore

$$
\hat d
$$

may be highly active and noisy.

Using disturbance feedback creates a direct path:

Measurement noise
→ Observer
→ Disturbance estimate
→ Control input

which can inject noise into the actuator.

---

### 10. Integral Action Naturally Filters Quantization Effects

The integrator evolves according to

$$
\dot z = r-y.
$$

An integrator naturally accumulates persistent bias and tends to average out high-frequency quantization effects.

Therefore:

- Disturbance feedback may react to quantization artifacts.
- Integral action primarily reacts to long-term offset.

In systems where the disturbance state is being used by the observer to absorb measurement quantization or model mismatch, integral action may produce cleaner control behavior despite appearing theoretically redundant.

---

## Final Takeaway

A disturbance observer and an integrator both provide mechanisms for rejecting constant disturbances.

The disturbance observer:
- Learns the disturbance explicitly.
- Can provide faster compensation.
- Can reduce controller order if an augmented observer already exists.
- May reintroduce measurement noise through the disturbance feedback path.

Integral action:
- Learns the required compensation indirectly through accumulated error.
- Is highly robust to modeling errors.
- Naturally suppresses high-frequency measurement artifacts.
- Often produces cleaner behavior when disturbance estimates are noisy.

For systems with significant quantization or observer-induced disturbance-estimate noise, integral action may be preferable even when a disturbance estimate is available.

## See Also
* [[reference_observer_design|Observer Design]]
* [[reference_disturbance_observer|Disturbance Observer Design]]
* [[case_motor_observer|Practical Motor Control]]