---
title: Crystals and Timers
type: lecture
topics:
  - Crystal Oscillators
  - PLL
  - Timers
  - PWM
tags:
  - timers
  - pwm
source: ME405-2262 Lecture 5
status: draft
---

# Motivation

Modern microcontrollers derive timing from accurate hardware clocks. This lecture introduces crystal oscillators, PLLs, timers, counters, prescalers, and pulse width modulation (PWM).

>Candidate static notes:
>* \[\[Crystal Oscillator\]\]
>* \[\[Phase-Locked Loop\]\] 
>*  \[\[Hardware Timers\]\]
>* \[\[Pulse Width Modulation\]\]

# Crystal Oscillators

The STM32 Nucleo uses an 8 MHz crystal in a Pierce oscillator circuit to generate a clock for the system. Depending on the Nucleo variant, this may be part of the application MCU circuit on the main portion of the Nucleo or the application MCU may get its clock from the oscillator on the ST-Link as is the case for the Nucleo L476RG used in ME 4305. The image below shows a closeup of the Nucleo.

![Annotated Nucleo board highlighting the crystal oscillator components.](images/nucleo.png)

The Pierce oscillator circuit on the Nucleo L476RG is made up of the following components located on the ST-Link:
* **X1**: 8 MHz crystal sets the frequency of oscillation.
* **C3**, **C8**: Load capacitors stabilize the oscillation.
* **U2**: The inverter gate causing the oscillation is part of the ST-Link MCU.
A typical RC oscillator has accuracy on the order of typical accuracy of 1% to 10% which correlates to a drift of minutes to hours per day. A typical crystal oscillator has an accuracy of 10-50 ppm which correlates to a drift of a couple seconds per day.

## Phase-Locked Loop

PLLs multiply and divide the crystal frequency to generate a derived system clock that is either faster or slower than the crystal frequency by some rational factor. Typically, PLLs are used to increase the clock speed so that the microcontroller can run at a higher speed.

In the example below, a PLL is used to match up 1 cycle of the 8MHz crystal output to 10 cycles of the 80MHz derived clock. The precise implementation details of a PLL are out of scope for this course, but the fundamental behavior is approachable to anyone that understands the nature of feedback. 

![Comparison of an 8 MHz crystal waveform and an 80 MHz PLL output illustrating phase locking and frequency multiplication.|700](images/phase_locked_loop.svg)

A PLL uses feedback from the phase error to increase or decrease the frequency of the derived clock; if 10 cycles of the PLL output lags 1 cycle of the crystal output the PLL frequency is increased; if the PLL output leads then the PLL frequency is reduced. Eventually the PLL will settle on a precise 10:1 ratio in frequencies, thereby "locking" the phase shift.
$$
\begin{aligned}
f_{PLL} &= K \left( \phi_{crystal} - \phi_{pll} \right) \\
f_{PLL} &= K \int \left( 10\, f_{crystal} - f_{PLL} \right)\, dt
\end{aligned}
$$
# Timers and Counters

A counter can be configured to count up or down on each rising edge of the system clock output from the PLL. The count value can then be used to perform precise timing to either generate output signals or timestamp input signals or events.

In standard up counting mode, shown in the figure below, the counter will incrementally count edges until it hits the **Autoreload** value, which is the maximum count value. Instead of counting past the autoreload value the timer instead resets back to zero. In down counting mode the count resets at the autoreload value instead of counting negative numbers below zero.

![Timing diagram of a free-running up counter with autoreload and associated clock waveform.|700](images/basic_upcounting.svg)

The period of the counter depends on the clock frequency, which determines the slope of the stair-step waveform shown above, and the autoreload value which determines the amplitude of the stair-step waveform.
$$T=\frac{AR+1}{f_{CLK}}$$
Notice that $AR+1$ is used in the numerator instead of $AR$ since the timer must count zero explicitly.
## Prescalers

Modern microcontrollers run at very high clockspeeds. The STM32 used in ME 4305 typically runs at 80 MHz. At this frequency it is impossible to generate low frequency output using a 16-bit timer. Consider the maximal 16-bit integer, 65,535; using this max value for the autoreload along with an 80 MHz clock frequency gives an approximate maximum period of $T = 800 uS$ which corresponds to a minimum frequency of 1250 Khz.

The solution to producing low frequency output is to apply an additional scaling factor. The prescaler determines the number of additional clock edges required to increment the count value and move up one stair-step. That is, the prescaler can be considered an additional factor that adjusts the slope of the stair-step waveform. In other words, prescalers divide the incoming clock before it reaches the counter.

![Comparison of timer operation with and without a prescaler showing reduced counting rate and longer period.|700](images/prescaled_upcounting.svg)

To determine the period of a prescaled counter, apply another factor equal to $PS+1$ to the previously derived formula for the timer period.
$$T=\frac{(AR+1)(PS+1)}{f_{CLK}}$$

## Left-Aligned PWM

PWM is generated by comparing the counter against a compare register. When the compare value exceeds the present count the PWM output is high and when the present count exceeds the compare value, the PWM output is low. The figure below shows this on a timing diagram. Notice that the duty cycle, or percentage on on time to the counter period, is proportional to the compare value.

![Up-counting timer compared against two compare values producing left-aligned PWM waveforms with different duty cycles.|700](images/left_aligned_pwm.svg)
**Note:** in the preceding figure the stair-step waveforms have been simplified to a standard sawtooth waveform. It should be imagined that zooming in on the sawtooth would reveal many small stair-steps as shown in previous figures in this lecture.

The duty cycle is computed as the ratio
$$D=\frac{CMP}{AR+1}$$

## Center-Aligned PWM

Changing the timer to up-down counting produces center-aligned PWM. This style of PWM is often preferred in motor control as it guarantees that the phase alignment between PWM channels remains constant. With edge-aligned PWM the relative phase of the PWM channels is a function of the duty cycle, but center-aligned PWM outputs always remain phase aligned except for exactly one cycle following compare changes.

![Up-down counting timer producing center-aligned PWM waveforms that reduce phase asymmetry between channels.|700](images/center_aligned_pwm.svg)

In up-down counting mode, the timer still only counts stair-steps at count=0 and  and count=AR once each, which eliminates the need for incrementing the AR value in the denominator. That is, for center-aligned PWM.
$$D=\frac{CMP}{AR}$$
Similarly, the period for center aligned PWM needs to be adjusted to account for the up-down counting.
$$T=\frac{2\,AR\,(PS+1)}{f_{CLK}}$$

## PWM and Motors

Students working with DC motors and PWM for the first time will often ask why the DC motor doesn't speed up and slow down as the PWM waveform toggles on and off. The intuitive answer is that the motor requires more time to change speed than is available between PWM edges.

From a more academic viewpoint, we can argue that DC motors naturally behave as low-pass systems and therefore attenuate the high frequency components of any applied input signal. The figure below depicts three plots.
1) The first plot shows the spectrum of the PWM waveform with an assumed duty cycle of 50%. 
2) The second plot shows an approximation of the magnitude of the frequency response function for a typical DC motor. Like all low-pass systems there is a passband at low frequencies and an attenuation band at high frequencies.
3) The final plot shows the change in magnitude of the motor output due each spectral component in the PWM waveform.
For a DC motor to spin smoothly the PWM frequency must be large enough that the harmonics are all attenuated significantly compared to the DC component at zero frequency.

![Frequency-domain illustration showing PWM spectrum, motor low-pass response, and motor output spectrum dominated by the DC component.|700](images/motor_frequency_response.svg)
**Note:** in practice the floor for PWM frequency may actually be much higher than what satisfies the conditions shown above because it is common to choose ultrasonic PWM frequencies. Frequencies in the hundreds of Hz to low KHz range will produce noticeable tones from the motor which may not be desirable. Selecting PWM frequencies of 20 KHz or higher will guarantee that any produced tones are outside the audible spectrum of human hearing.

## Summary

-   Crystal oscillators provide an accurate timing reference.
-   PLLs generate higher-speed clocks.
-   Timers use autoreload and prescalers.
-   PWM duty cycle is controlled with the compare register.
-   Motors primarily respond to the average value of PWM.
