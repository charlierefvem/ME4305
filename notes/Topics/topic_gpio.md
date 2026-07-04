---
title: General Purpose IO (GPIO)
type: topic
tags:
  - mechatronics
  - gpio
  - input
  - output
  - push-pull
  - open-drain
source:
  course: ME405
  term: 2262
  lecture: 13
status: draft
---

# Motivation

General-purpose input/output (GPIO) pins are the primary interface between a microcontroller and the outside world. Understanding how GPIO circuitry works is essential for safely interfacing switches, LEDs, sensors, and external electronics.

# GPIO

## GPIO Logic Levels

General Purpose Input/Output (GPIO) refers to:
* Setting an output pin **logic high**
* Clearing/resetting an output pin **logic low**
* Reading the state of a digital input

Modern MCUs typically operate from **1.5–3.6 V**, with **3.3 V** now being the de facto logic standard. The  supply rail may be labeled:
* **VDD**: MOSFET terminology ("drain")
* **VCC**: BJT terminology ("collector")

For practical purposes these both represent the digital logic supply voltage.

## Push-Pull Outputs

The most common output configuration is called **push-pull** output and allows the MCU to actively drive the pin high (sourcing current) or low (sinking current). Push-pull outputs require two transistors in a configuration similar to a half-bridge.

![Simplified GPIO output stage showing complementary high-side and low-side MOSFETs capable of sourcing or sinking current.|700](images/gpio_push_pull.svg)

Characteristics:
* Can source current
* Can sink current
* Always drives a defined logic state
* Most common GPIO output mode

## Open-Drain Outputs

An alternative style of output is the **open-drain** or **open-collector** output. The internal circuitry in the MCU disables the high-side transistor to configure the output in open-drain mode. In this configuration the pin can be actively driven low or allowed to float in a high-impedance (Hi-Z) state.

![Open-drain output stage with only the low-side MOSFET populated.|700](images/gpio_open_drain.svg)

Open-drain outputs:
* Can actively pull the line low
* Otherwise leave the output floating (Hi-Z)
* Require an external (or internal) pull-up resistor when a logic high is needed

Most modern MCUs allow GPIO pins to be configured as either push-pull or open-drain.

**Insight**: Regardless of the output mode, a GPIO pin should be thought of as a switch rather than as a voltage source. The pin either connects the output to one of the supply rails or disconnects it; the external circuit determines how much current flows.
## Tri-State Outputs

A third output mode, technically not GPIO, but still relevant, is the **tri-state** output. Most MCUs do expose tri-state as a standard GPIO output mode, but instead use the mode in various alternate function modes. For example, the serial peripheral interface (SPI) requires tri-state configuration for some of the signals. A tri-state output may be actively driven high or low or may be left in a Hi-Z state.

![Push-pull output followed by a tri-state buffer capable of placing the output in high impedance.|700](images/gpio_tristate.svg)

A tri-state output has three possible states:
* Logic high
* Logic low
* High impedance (Hi-Z)

Typical use:
* Shared communication buses (for example SPI MISO), where only the selected peripheral actively drives the line while all others remain in Hi-Z.

## Digital Inputs

A general purpose input circuit consists of protection diodes and a Schmitt trigger. The protection diodes help clamp the input voltage between VDD and GND (within the diode's forward voltage) so that the internals of the MCU are protected from voltages that may cause damage. The Schmitt trigger is for glitch prevention and works by rejecting incomplete transitions between logic high and logic low.

![GPIO input showing Schmitt trigger and protection diodes.|700](images/gpio_input.svg)

Typical thresholds:
* Logic HIGH is typically anything above ⅔ of VDD
* Logic LOW is typically anything below ⅓ of VDD

### Schmitt Trigger

A Schmitt trigger provides hysteresis for improved noise immunity.
![Graph showing Schmitt trigger behavior.|700](images/gpio_schmitt_trigger.svg)

A Schmitt trigger prevents unwanted switching caused by noisy or slowly changing input voltages because small variations between ⅓ of VDD and ⅔ of VDD are ignored.

## Example 1

In this example a microcontroller input pin is connected to a motor driver's open-drain output pin. To guarantee that the MCU input pin always has a defined state, there must be a pull-up resistor connected to the circuit. The pull up defines the idle state of the input pin to be high, allowing the motor driver to actively drive the pin low when a fault occurs.

**Note**: In the schematic snippet below, an external pull-up is shown, however most MCUs allow internal pull-ups to be enabled in firmware.

![Example input circuits illustrating a pull-up resistor configuration.|700](images/gpio_pull_up.svg)

## Example 2

In this example a microcontroller output pin is connected to a motor driver's enable input pin. Typically a push-pull configuration would be used in this circumstance allowing the MCU to enable or disable the motor driver. A pull-down resistor is included for safety to make sure that the motor driver remains disabled in case the MCU fails to configure the pin in push-pull mode or in the case that the push-pull configuration occurs after startup.

**Note:** many motor drivers include internal pull-down resistors to implement this safety feature. Do not double-up with an external pull up in such a case.


![Example input circuits illustrating a pull-down resistor configuration.|700](images/gpio_pull_down.svg)

## Example 3

In this example a **wired-or** configuration is shown for a circumstance similar to Example 1, but involving multiple motor drivers. A wired-or configuration is made up of multiple open-drain outputs and a single pull-up resistor. The idle state of the MCU input pin will be pulled high until any of the motor drivers may actively drive the line low. The microcontroller can then detect that motor driver 1 **or** motor driver 2 **or** motor driver 3 has triggered a fault, but not which one.

![Example input circuit illustrating a wired-or configuration with a pull-up resistor.|700](images/gpio_wired_or.svg)
## Example 4

A microcontroller output pin can only source or sink up to a few milliamps. To drive loads that require more current, such as a high power LED, an external switch must be implemented. The two schematic snippets below show low-side and high-side switch configurations to drive an LED.

![Example GPIO circuit for driving an LED with a low side switch.|700](images/gpio_led_low_side.svg)

**Note:** for a low-side switch you should typically use an N-Channel MOSFET and for a high-side switch you should use a P-Channel MOSFET. These design choices will guarantee that the MCU is able to properly turn on and off the MOSFET.

![Example GPIO circuit for driving an LED with a high side switch.|700](images/gpio_led_high_side.svg)

**Caution**: in addition to protecting the MCU controller from excess current, you must also protect LEDs from excess current by placing current limiting resistors. For either circuit in this example the value of the current limiting resistor should be determined by the forward voltage and allowable current for the LED.

Suppose the LED is red and has a forward voltage of 1.7V and a maximum allowable current of 20mA and is to be driven directly from the VDD rail at 3.3V.
$$
\begin{aligned}
R &\ge \frac{V_{DD} - V_{f}}{i_f} \\
R & \ge \frac{3.3[V] - 1.7[V]}{20[mA]} \\
R & \ge 80[\ohm]
\end{aligned}
$$
A designer may then consider the typical standard values for a resistor and select an 82$\Omega$ resistor, or something larger if maximum current (brightness) is not required.

## Example 5

The same high-side and low-side switches presented for the LEDs in Example 4 can be repurposed to drive other types of loads like motors or in the case of this example, a magnetic relay. Inductive loads, like motors and relays, require an additional diode, reverse-biased, in parallel with the inductive load. The diode allows recirculation of current when the switch is turned off so that the coil current can decay slowly, thereby protecting the transistor acting as a switch. Without the diode, the current will decay rapidly, causing a large voltage spike due to the inductance; recall that for an inductor $V_L = L \frac{d}{dt} i_L$ so for rapidly changed $i_L$ the voltage $V_L$ will be very large.

![GPIO driving an inductive load with appropriate flyback protection.|700](images/gpio_inductive.svg)

## Example 6

This example shows two configurations, active-low and active-high, for interfacing a MCU input with a switch or button. The active-low configuration is named as such because the MCU pin will read low when the switch is pressed. In an active high configuration the switch being pressed will cause the MCU pin to read high.

![Example button circuit in active low configuration.|700](images/gpio_active_low_switch.svg)

**Note**: it is generally preferred to configure MCU pins and switches in an active-low configuration because many microcontrollers are slightly better at sinking current than sourcing it, and active-low signals integrate naturally with pull-up resistors and open-drain logic.

![Example button circuit in active high configuration.|700](images/gpio_active_high_switch.svg)

# Insights
* Push-pull outputs actively drive both logic states.
* Open-drain outputs only actively drive low.
* Multiple open-drain outputs can be connected in a wired-or setup.
* Tri-state outputs allow multiple devices to safely share a signal line.
* Schmitt triggers significantly improve digital input robustness.
* Pull-up and pull-down resistors establish a known logic state whenever an input would otherwise float.
* LEDs require current limiting resistors.
* Inductive loads require flyback diodes.

# Summary
GPIO circuitry is considerably more sophisticated than simply connecting a processor pin directly to the outside world. Understanding the internal driver circuits, input conditioning, and protection features allows reliable interfacing with common electronic components and forms the foundation for later topics involving digital communication and robust embedded hardware design.

# Candidate Static Notes
* \[\[GPIO\]\]
* \[\[Digital Logic Levels\]\]
* \[\[Pull-up Resistors\]\]
* \[\[Open-Drain Outputs\]\]
* \[\[Tri-State Outputs\]\]
* \[\[Schmitt Trigger\]\]
* \[\[Switch Bounce\]\]
* \[\[Debouncing\]\]
