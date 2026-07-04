---
title: Introduction to Feedback and Controls
type: topic
tags:
  - control
source:
  course: ME405
  term: 2262
  lecture: 10
status: draft
---

# Motivation

In ME 4305 you will need to write code that controls real hardware. Other courses in our curriculum will focus on the mathematical backend for control theory while this course will focus on the practical application of control theory and only dip into mathematical derivations as needed to support the practical theory.

# Basics of Feedback Control

Before understanding specific controller topologies it is paramount to understand the basics of **feedback**. Some people, your instructor included, use feedback as a model for most of reality as a means of understanding the world.

Feedback is simply the process of measuring the output of a system and using that knowledge to make choices on how to affect the system through its input.

An example of feedback is the hand-eye loop we use as people to interact with the world. Your brain uses feedback from your senses to enable articulate and precise motion of your body.

# PI~~D~~ Controllers

The controller topology that has been used frequently for many decades is the PID controller. The PID amplifies error in a system's output to determine an input to the system that corrects the output error. A standard PID controller follows an explicit **control law** involving several parameters and gains:
$$
a = K_p (r - \hat{x})
  + K_i \int_0^t (r - \hat{x})\, d\tau
  + K_d \frac{d}{dt}  (r - \hat{x})
$$

| Term      | Definition                                                       |
| --------- | ---------------------------------------------------------------- |
| $r$       | The reference or setpoint for the system output.                 |
| $x$       | The true output of the system to be controlled.                  |
| $\hat{x}$ | The measurement of the true system output.                       |
| $e$       | The system error, defined as $e = r - \hat{x}$.                  |
| $K_p$     | The proportional gain which amplifies error.                     |
| $K_i$     | The integral gain which amplifies the integrated error.          |
| $K_d$     | The derivative gain which amplifies the rate of change of error. |
A good starting point is proportional control, adding integral and derivative action only when needed.

![Annotated PID controller block diagram with actuator, plant, and feedback.|700](images/ControlLoopDiagrams_Standard_PID.svg)

PID control is often taught as if integral action and derivative action should always be included. In practice, however, the nature of the system and the available sensors to measure the system output greatly determines the effectiveness of each component.

**Caution**: Most controllers *do not benefit from derivative action* due to noise. Differentiation causes amplification of high-frequency signals which often damages the performance of a controller.

## Proportional Action

The proportional component drives the system until the measurement is close to the setpoint, but  steady-state error should be expected, with the amount depending on the type of the system and the value of $K_p$. Systems that require effort to maintain steady-state will have large steady-state error with proportional-only controllers. Too much proportional action (large $K_p$) can cause overshoot or oscillation before effectively reducing steady-state error.

## Integral Action

The integral component drives the system the last bit of the way toward the setpoint and holds it there, rejecting disturbances. Integral control is slow by nature due to the need to accumulate integrated error. Too much integral action can also cause overshoot or oscillation similar to an overly aggressive P controller. Integral control also has some side-effects like windup which will be covered in a different lecture.

## Derivative Action

The derivative component allows more aggressive proportional and integral control with less overshoot because derivative action pushes back against proportional and integral control when the error is shrinking over time. That is, a well functioning derivative control will allow higher $K_p$ and $K_i$ gains without allowing overshoot.

In practice, however, derivative control often causes amplification in noise since differentiation amplifies the high frequency content of a signal. This can be noticed intuitively by differentiating a simple sinusoidal signal: $\frac{d}{dt}\sin(\omega\,t)=\omega\cos(\omega\,t)$ so for large values of $\omega$ the amplitude increases.

## Expectations

A controls engineer must be careful to maintain their expectations. A novice engineer will design a controller under the false assumption that the goal is to reduce system error to zero in the shortest time possible. A seasoned engineer will design a controller with the aim to achieve *suitable performance* as determined by the constrains acting on the system. Fortunately this philosophy aligns well with the philosophy of mechatronics outlined on day one of ME 4305:
>The goal is to make the machine accomplish its intended task.

Consider a cruise control system in a vehicle. Is it more important that the vehicle travel at the precise velocity with zero error? Or is an error of 1% allowable? Would you prefer your car to accelerate as quickly as possible when you change the set velocity, or would you prefer the car to gradually reach the new setpoint?

To validate your controller you should first come up with performance metrics: quantitative tools to compare the output of one controller to another. Thorough discussion is out of scope of this lecture, but common metrics include parameters such as:
* Rise Time
* Maximum Over Shoot
* Settling Time
* MSE (Mean Squared Error)
* Total Fuel Cost
* Mean Squared Actuator Effort

## Implementing a Controller in Code

For your code to be reusable the controllers should be agnostic with respect to sensors and actuators. That is, you should implement the controller parameterized by:
* An arbitrary reference, $r$
* An arbitrary measurement, $\hat{x}$
* An arbitrary set of gains, $K_p$, $K_i$, etc.

# Feedforward Control

In most cases the typical PI or PID controller will achieve desired performance metrics, but some systems may be more challenging to tune than others. 

PID controllers are poorly suited to handle certain nonlinearities, including a deadband nonlinearity like static friction in a mechanical system. With a standard PI controller it is usually the integral component that must slowly build to overcome the deadband and cause change in the output. Such cases can even lead to a limit-cycle in which the integral action continues to build up, overshoot the set point, change direction, build up again, undershoot, and then start over.

Standard PID controllers also do poorly in circumstances where a large amount of effort is required to maintain steady-state relative to the effort needed to maintain stable and reject disturbances. In such cases the integral term is again overburdened as it must hold a large accumulated integral to remain near the setpoint.

There are a variety of additional features that may be added to a PI controller to make tuning easier. One addition to the standard PID that can help in either of the two cases mentioned above is feed forward control. Feedforward control uses information only from the setpoint and a model of the plant dynamics without using any feedback from the state of the system.

The simplest feedforward control schemes add an extra actuation term that is proportional to the setpoint, as shown in the example below. A modified control law implementing proportional feedforward would be the same as for a traditional PID controller, but with an additional feedforward term that uses $K_{ff}$, the feedforward gain.
$$
a = K_{ff}\, r
  + K_p (r - \hat{x})
  + K_i \int_0^t (r - \hat{x})\, d\tau
  + K_d \frac{d}{dt}  (r - \hat{x})
$$
A block diagram representation of this control law is shown in the figure below.

![Feedforward controller block diagram.|700](images/ControlLoopDiagrams_Feedforward.svg)

In another case it may be enough to add an actuation component that is constant, but the same sign as the setpoint to compensate for deadband nonlinearities like the aforementioned static friction.
$$
a = a_{ff}\, \text{sgn}(r)
  + K_p (r - \hat{x})
  + K_i \int_0^t (r - \hat{x})\, d\tau
  + K_d \frac{d}{dt}  (r - \hat{x})
$$
More complicated feedforward control schemes may use an inverse model of the plant to try and feed forward the exact signal needed to produce a desired output. For example, consider the preceding figure showing a block diagram with feedforward action. If the gain, $K_{ff}$, were replaced with an inverse plant model equal to $(G_2\,G_1)^{-1}$  then the entire path from the setpoint, $r$, to the output, $x$, would reduce to unity and the feedback controller would be unnecessary.

 However, in practice, this doesn't always work well; because physical systems are causal, inverse models of the systems become acausal and therefore impossible to implement. Methods do exist to overcome this difficulty, the simplest of which is to define a constant feedforward gain based only on the steady-state relationship between the actuator input and the plant output.

# Cascaded Control

A powerful technique used in industry is cascaded control. A cascaded controller is made by using several nested controllers each meant to control a different part of the actuator or plant. For example, an "inner loop" controller may be designed to control just the actuator, but not the plant, with the expectation that an "outer loop" controller commands the inner loop appropriately to control the plant. This controller structure does two things:
* Most importantly, cascaded control loops can be tuned incrementally, instead of all at once.
    * The inner loop can often be tuned first, aggressively, so that commands from the outer loop quickly affect the outputs.
    * The outer loop can be tuned afterwards, less aggressively, now trusting inner loop to follow the commands given.
* Saturation limits and additional controller features like feedforward can also be applied separately to the inner and outer loops.
* Nesting controllers may increase the overall order of the controller which may in some cases lead to improved performance.

## Example

In this example the classic "servo loop" is shown that is implemented in many motor control applications. In the diagram below, the actuators and plants are lumped together in pairs to declutter the diagram.
1) Start by examining the inner loop, consisting of $C_1$, $G_1$, and $H_1$. This loop implements current control. That is, the controller, $C_1$, takes the error in commanded current, and outputs a voltage to the motor, $G_1$, which provides feedback on the current through the current sensor, $H_1$. This inner loop can be tuned on its own. After tuning, the entire inner loop can be collapsed to a single block, used in the tuning of the intermediate loop.
2) The intermediate follows a similar principle, but is meant to control velocity by commanding motor current (and therefore torque),
3) The outer loop is tuned last to control position, treating both the intermediate loop and inner loop as if they are part of the plant for the outer loop.
With this set up, each layer can be tuned individually for performance, and may impose it's own saturation limit: the voltage saturation done in the inner loop handles the finite range of voltage from the power supply, the current saturation done in the intermediate loop protects the actuator from overheating or imposes an acceleration limit, and finally the velocity saturation done in the outer loop limits the maximum requested velocity, or slew-rate, while changing position.

![Servo motor cascaded control example showing current, velocity, and position loops.|700](images/ControlLoopDiagrams_Cascaded_Loops.svg)

## Summary

* Start with proportional control.
* Add integral only as required.
* Avoid derivative control unless you have low-noise sensors.
* Feedforward compensates for predictable plant behavior.
* Build reusable controller classes.
* Cascaded controllers simplify complex systems.
