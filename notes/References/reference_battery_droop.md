---
title: Battery Droop Compensation
type: reference
tags:
  - batteries
  - ADC
source:
  course: ME405
  term: 2262
status: draft
---

## Battery Discharge

As a battery discharges, its voltage droops significantly. For example, a fully charged NiMH cell has a voltage of 1.4V, but after about 10% or 15% discharge, the voltage drops to the nominal cell voltage of about 1.2V. The nominal voltage is relatively stable until the cell is about 90% discharged at which point the voltage drops to about 1V.

Batteries from reputable sources should have published discharge curves. For example, the Panasonic Eneloop batteries have plenty of published test data available on the internet. For those curious, consider looking at some of this data: https://eneloop101.com/batteries/eneloop-test-results/.

If battery droop is not compensated for in feedback systems, it may be challenging to tune the system because sensitivity to each gain will effectively be reduced by the drooping battery voltage.

### Example 1

Consider a model of a PWM controlled H-bridge that would be part of the "Actuator" model in a feedback system. Usually it is assumed that
$$
V_m = D\, V_{nom}.
$$

This model claims that the voltage going to the motor is equal to the nominal DC supply voltage, $V_{NOM}$, multiplied by the PWM duty cycle, $D$, represented as a number between -1.0 and 1.0. This model is accurate for constant voltage power supplies but not for batteries.

Once the battery voltage, $V_{BAT}$ starts drooping, the same duty cycle will not always represent the same output voltage. A more accurate model for a battery powered system is
$$
V_m = D\, V_{bat}.
$$
This new model claims that the duty cycle is only able to scale the present battery voltage, not its nominal voltage. Accordingly, the magnitude of effect of the duty cycle, $D$, depends on the state of the battery.

To restore the nominal behavior of the system we can introduce a new gain that compensates for the drooping battery. That is, 
$$
K_{bat}=\frac{V_{nom}}{V_{bat}}
$$
such that
$$
\begin{aligned}
V_m &= D\, K_{bat}\, V_{bat}, \\
V_m &= D\, \frac{V_{nom}}{V_{bat}}\, V_{bat}, \\
V_m &= D\, V_{nom}. 
\end{aligned}
$$

With this new gain included, the dynamic values of $V_{bat}$ cancel with each other, effectively removing the effects of the battery droop. The main consequence of this additional gain is that as the battery discharges, more is requested from it to compensate for the reduced voltage. This compensation will further expediate discharge. However, all robust battery driven systems monitor battery voltage and most enter a cutoff mode when the battery discharges past its nominal range.

## The Output or Actuator Gain

The block diagram below shows a simple modification to a standard feedback control loop that includes an additional gain in between the controller and the actuator. This gain can be used for unit conversion or, as previously mentioned, for compensating for a drooping battery.

![A block diagram representation of a closed loop controller with an extra dynamic gain on the controller output.](images/pid/Battery_Compensation.svg)

This output gain should generally remain outside of the controller itself. The controller should operate in physical units, while the output gain converts those units into whatever representation the actuator requires.
### Example 2

The block diagram above generalizes this method of compensation using an output gain $K$. For the battery driven H-bridge presented in Example 1, the parameters used in the diagram would be defined as shown in the table below.

| Gain or Signal | Definition in H-bridge Example                                                                                                                                                                  |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $a$            | The requested actuation value, equal to the voltage, $V_m$, that the controller requests from the the H-bridge and applied to the actuator.                                                     |
| $K$            | The output or actuator gain. Converts the requested voltage to an appropriate duty cycle, $D$. Depending on units for $D$, this gain may be $K=\frac{1}{V_{bat}}$ or $K=\frac{100\%}{V_{bat}}$. |
| $a^*$          | The requested actuation value after conversion to duty cycle, $D$.                                                                                                                              |

## Measuring $V_{bat}$

To measure the battery voltage safely, a voltage divider is necessary. The maximum raw battery voltage may be too high to read with a microcontroller's ADC, which often has an active range of 0V to 3.3V. To reduce the battery voltage to a safe range, a divider with a suitable ratio is required.

![A voltage divider circuit built from an upper resistor R1 and a lower resistor R2.](images/battery/battery_divider.svg)

### Example 3

In this example a voltage divider will be designed to allow measurement of a battery pack made up of 6 AA cells. A fully charged Alkaline battery can have a voltage of up to 1.6V; six of these in series result in a maximum possible voltage of 9.6V.

To drop the voltage below 3.3V at full charge, a divider of approximately $\frac{1}{3}$ is required. To achieve such a divider ratio, two resistors are needed, with one being equal to or slightly higher than double the value of the other resistor.

Additionally, the sum of the two resistor values should be large enough to mitigate significant current flow or risk draining the battery prematurely. Values of R1=10k and R2=4.7k will work nicely. The precise divider ratio of $$\frac{4.7\,k\Omega}{14.7\,k\Omega} \approx 0.32$$ will reduce the maximum voltage seen by the ADC to below 3.1V. Additionally, the steady-state current draw will only be $$\frac{9.6\,V}{14.7\,k\Omega} \approx 650 \mu A$$ which is assumed small compared to the draw from the rest of the system.

## See Also:
* [[reference_PID|PID Controllers]]
* [[topic_discrete_PID|Discrete PI Implementation]]