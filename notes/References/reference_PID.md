---
title: PID Controllers
type: reference
tags:
  - control
  - pid
  - feedback
source:
  course: ME405
  term: 2262
status: draft
---

# PID Control Basics

Typical control laws are of the form P, PI, PD, or PID, where "P" stands for proportional, "I" stands for integral, and "D" stands for derivative. A proportional-integral-derivative (PID) controller is shown below in a typical feedback loop. These three aspects each apply to how the error signal is converted into an actuation signal.

The proportional component is most important and may be sufficient to achieve desired performance without the integral and derivative components; however, in many cases improved performance can be achieved by including integral and/or derivative action, and for some systems one or the other may be required for stability.

Consider the block diagram shown below; this diagram depicts a stand PID controller with no extra features included.

- The signal $r$ is the setpoint or reference. Often the setpoint is set to zero for "regulator" type controllers, but more commonly the setpoint is a nonzero value. If the setpoint changes over time, the controller must be tuned to allow good tracking performance because controller performance will be different for regulation, constant setpoints, and moving setpoints if tuned the same.
  
- The blocks labelled $G_1$ and $G_2$ represent the actuator and plant models, respectively; the actuator model and plant model are often lumped together and treated as a single plant $G = G_2\,G_1$, however it is important to understand that disturbances can enter the system in between the actuator dynamics and the plant dynamics due to unmodeled effects.
  
  The input to the actuator represents the "requested" effort from the controller, and is labelled as $a$ on the diagram. The actuator is what converts the numerical value of the requested effort, as produced by the control law, into a physical actuation parameter such as torque which then applies to the plant. The output of the plant is labelled as $x$ on the diagram and should be understood as the *true physical output of the system*, not a numerical value used in the control law.
  
- The block labeled $H$ is a model of the sensor used to measure the physical output of the system plant, $x$, and represent that measurement as a numerical value $\hat{x}$. The sensor model may or may not include dynamics; in many cases, unity feedback is assumed or a simple gain is used in the feedback loop to represent the sensor, but some types of sensors may have internal dynamics that need to be accounted for.
  
- The error signal, $e$ is produced by the difference between the system setpoint, $r$, and the system output, as measured by the sensor, $\hat{x}$; that is, $e = r - \hat{x}$. This signal is what goes into the PID controller to determine the actuation value, $a$, requested from the actuator. For a complete PID controller, the feedback (control) law is $$a = K_p\, e + K_i\int e\, \text{d}t + K_d \frac{\text{d}}{\text{dt}}e.$$The control law is often represented as a transfer function as well; for a full PID the transfer function representation becomes $$C = K_p + \frac{K_i}{s} + K_d\,s$$ so that $a = C\,e$.

![Block diagram representation of a PID control loop](images/pid/Standard_PID.svg)

Intuitive understanding of PID controllers comes from experience and practice working with them. Nonetheless, the table below is an attempt at summarizing the contribution of each of the three components of the PID.

**Summary of PID Components**

| Component    | Gain  | Purpose                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Potential Drawbacks                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------ | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Proportional | $K_p$ | The proportional action is the primary driving component for the system to approach its target.                                                                                                                                                                                                                                                                                                                                                                                                                      | Large proportional gains may be needed to achieve suitable performance and such gains can lead to oscillation or instability. P-only controllers also suffer with respect to disturbance rejection and dynamic tracking.                                                                                                                                                                                                                                                    |
| Integral     | $K_i$ | The integral action is the component that keeps the system at or near the target value and helps to reduce the system error. Depending on the system type and the reference profile, integral control can lead to very low steady-state error.                                                                                                                                                                                                                                                                       | Integral control can also lead to instability issues for larger gains. Other issues such as integral windup and reset windup are very common drawbacks, but each can be handled easily with a few tweaks to the algorithm.<br><br>Integral action can also lead to issues for systems that require a large amount of effort to reach the target refence in comparison to the effort required to overcome disturbances, as the same gain will not handle both cases equally. |
| Derivative   | $K_d$ | The derivative action is the component that allows the controller to aggressively command the system as it approaches the target value while mitigating large overshoot. In other words, as the system approaches the target value the derivative action will help slow down the approach.<br><br>Controllers with derivative action can have higher proportional and integral gains for the same amount of overshoot. Derivative control also makes the system respond faster to changes in the reference setpoint. | Derivative control has the effect of amplifying noise greatly, as most noise is of relatively high frequency but low amplitude. For some systems with low performing sensors, the derivative control is made useless by the amount of noise produced by the sensors. <br><br>Additionally, derivatives are challenging to compute accurately using numerical methods, such as with the finite difference method, which may lead to further numerical instability.           |

# Control Loop Modifications

In many cases controls engineers will modify the control structure from the standard PID implementation in order to improve controller performance.

## Actuator Saturation

One of the most important limitations of real world hardware is saturation. Actuators can only output so much power before they hit limits or fail. In some cases this has little effect on controller performance, but in general actuator saturation can have large ramifications.

Most importantly, the control law should never request more actuation effort than the actuator can safely supply. This is for two reasons: first, the control law should not request so much effort from the actuator to cause damage (due to overheating, torque limits, etc.) and second, the control law should be aware when actuator saturation occurs, even if no damage is expected from the actuator.

A modified control loop with actuator saturation is shown in the diagram below. The saturation block limits the true actuation value $a^{\ast}$ to be between some fixed upper and lower limits even if the requested actuation value $a$ exceeds those limits.

![A block diagram representation of a closed loop feedback controller with actuator saturation](images/pid/Saturation.svg)

In battery-powered systems, the actuator limits may themselves change over time. For example, as the battery voltage decreases, the maximum achievable motor voltage decreases proportionally. Consequently, it is often preferable to compute saturation limits dynamically from the measured battery voltage rather than assuming fixed limits.

**Insight**: almost all real-world PID implementations implement saturation, so it should be considered a standard feature in practice.
## Anti-Windup

One of the unintended consequences of actuator saturation occurs in systems with integral control and is known as integrator windup, saturation windup, or reset windup.

Saturation windup occurs when actuator saturation persists for a significant amount of time. During this period of time, the error continues to integrate even though the increasing integral value does not increase the actuator output. In other words, even though the controller is requesting more and more effort, the actuator clamps the effort at the saturation limit. The unbounded integration can cause unintended system behavior.

A common outcome of windup is prolonged or sustained overshoot. If the integrator has already grown large by the time the system reaches its setpoint then the actuation value will remain large even once overshoot occurs. Only after a sufficient amount of negative error during the overshoot period will the integrator value reduce enough to stop saturating the actuator output, eventually causing the system to reach a steady state.

The most common method of handling integrator windup is to "turn off" the integrator when the controller output is saturated. That is, as soon as the saturation takes place, the integrator should stop integrating the system error. This will keep the integrator value close to the threshold that just barely causes saturation.

![A block diagram representation of a PI controller with anti-windup implemented using naive conditional integration.](images/pid/Anti_Windup_Clamping.svg)

This method is not robust however, as it does not allow the integrator to wind back down when the error becomes negative unless the saturation is removed by the proportional term (or the derivative term if it is used). A more robust method uses slightly more complex logic to stop integrating. Using the robust method, the integrator is only switched off when the saturation occurs and the sign of the error matches the sign of the actuation value.

![A block diagram representation of a PI controller with anti-windup implemented using robust conditional integration.](images/pid/Anti_Windup_Advanced_Clamping.svg)

Other anti-windup techniques use feedback to reduce the integrator value dynamically depending on the amount of saturation that is occurring.

![A block diagram representation of a closed-loop PI controller with anti-windup implemented using feedback.](images/pid/Anti_Windup_Feedback_Method.svg)

An important concept in anti-windup construction is that once the saturation disappears the anti-windup mechanism must also disappear so that it is invisible during linear operation.

**Insight**: Adding anti-windup is essential for any controller implementing integral action when actuator saturation is expected.

## IP and IPD Controllers

Another common modification to the standard PID controller is meant to mitigate sensitivity to abrupt changes in setpoint. With a standard PID controller and a step input in setpoint, the system actuation includes a step change, from the proportional term, and an impulse, from the derivative term. These large spikes in actuation value can sometimes cause problems in implementation and in general they are unkind to the actuators in the system.

$$
a = -K_p\, \hat{x} + K_i\int e\, \text{dt} - K_d \frac{\text{d}}{\text{d}t}\hat{x}
$$

It should be noted that, partially through intentional design, the system will not respond as quickly if set up as an IPD controller instead of a PID controller.

![A block diagram representation of an IPD feedback controller.](images/pid/IPD.svg)

**Insight**: IP and IPD controllers are used when the setpoint changes abruptly and actuator stress is a concern.

## Feedforward Control

Yet another example modification that improves performance is the addition of a feedforward controller in parallel with the feedback controller. An example control loop is shown below as a block diagram.

The feedforward controller can be thought of as an open-loop controller with the objective of choosing an actuation value that, through the dynamics of the actuator and plant, cause the output to match the setpoint. In theory, an inverse model of the actuator and plant,

$$
\frac{1}{G_2 \, G_1},
$$
would act as a perfect feedforward controller as it would cancel out all dynamics between the setpoint and the output.

In practice, inverse plant models don't work due to mathematical and practical restrictions. Primarily, it is impossible to implement inverse plant models that result in improper systems, as indicated by transfer functions with higher-order numerators than denominators. Commonly, feed forward controllers use a simple proportional gain applied to the reference signal instead of a full inverse plant model, but it is also typical to combine a filter with the feedforward gain so that quickly changing setpoints don't affect the system right away.

The feedback controller therefore only needs to work on the small error between the open-loop output and the setpoint. In this way a higher performing controller can be implemented because the feedback controller only needs to respond to the smaller fluctuations in the error and is not responsible for maintaining steady-state output.

![A block diagram representation of a feedback controller with an additional feedforward path.](images/pid/Feedforward.svg)

**Insight**: Feedforward is useful when the plant is reasonably predictable and the required steady-state effort is known.

## Pseudo-derivative Feedback

It has been argued, above, that differentiation is often more problematic than helpful. The IPD controller mitigates this somewhat, by only differentiating the measured feedback signal instead of the system error, but noise in the measurement is still amplified through differentiation.

One tactic exists to overcome this challenge, but it only applies to specific systems. Instead of feeding back data measured from one sensor and then differentiating that data to use with $K_p$, a PDF controller feeds back back an additional measurement of the rate of change. That is, instead of performing a numerical derivative, the dynamics of the plant perform the differentiation which is then measured directly. The control law is very similar to that of an IPD controller otherwise. The control law for this setup is 

$$
a = -K_p\, \hat{x} + K_i\int e\, \text{dt} - K_d\, \hat{v},
$$

where $\hat{v}$ represents a measurement of the derivative of $x$.

![A block diagram representation of an PDF feedback controller.](images/pid/PDF.svg)

Pseudo-derivative feedback is an early example of a broader design philosophy that uses more information about the system in addition to the output measurement. Rather than computing additional information numerically (such as a derivative), it is often preferable to measure that information directly whenever practical. Later in the course we will extend this idea further using state feedback and observers.

**Insight**: Pseudo-derivative feedback is used when a direct measurement of the derivative is available through an additional sensor.

# Candidate static references
* \[\[State Feedback\]\]
* \[\[Observers\]\]
* \[\[Finite Difference Method\]\]
* \[\[Filtered Derivatives\]\]
* \[\[Gain Scheduling\]\]