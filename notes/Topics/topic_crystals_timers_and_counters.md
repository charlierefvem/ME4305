---
title: Crystals and Timers
type: topic
tags:
  - timers
  - pwm
source:
  course: ME405
  term: 2262
  lecture: 5
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

The figure below shows how these components make up the Pierce oscillator circuit along with a simple RC oscillator circuit. A typical RC oscillator has accuracy on the order of 1% to 10% which correlates to a drift of minutes to hours per day. A typical crystal oscillator has an accuracy of 10 to 50 ppm which correlates to a drift of only a couple seconds per day.

![Schematic snippets for an RC oscillator and a Pierce oscillator. Also shown is a graph of the typical output from an oscillator.](images/timer_counter/oscillator.svg)

Notice that the oscillator does not output a digital waveform; that is, the signal produced is not a perfect square wave. The waveform is typically offset by $\frac{1}{2}V_{DD}$ (half of the MCU supply voltage) and only has an amplitude of about 1 V.
## Phase-Locked Loop

PLLs multiply and divide the crystal frequency to generate a derived system clock that is either faster or slower than the crystal oscillator frequency by some rational factor. Typically, PLLs are used to increase the clock speed so that the microcontroller can run at a higher speed.

In the example below, a PLL is used to match up 1 cycle of the 8MHz crystal output to 10 cycles of the 80MHz derived clock. The precise implementation details of a PLL are out of scope for this course, but the fundamental behavior is approachable to anyone that understands the nature of feedback. 

![Comparison of an 8 MHz crystal waveform and an 80 MHz PLL output illustrating phase locking and frequency multiplication.](images/timer_counter/phase_locked_loop.svg)

A PLL uses feedback from the phase error to increase or decrease the frequency of the derived clock; if 10 cycles of the PLL output lags 1 cycle of the crystal output the PLL frequency is increased; if the PLL output leads then the PLL frequency is reduced. Eventually the PLL will settle on a precise 10:1 ratio in frequencies, thereby "locking" the phase shift. Those familiar with control theory will recognize that this is similar to a PI controller.

$$
\begin{aligned}
f_{PLL} &= K \left( \phi_{crystal} - \phi_{pll} \right) \\
f_{PLL} &= K \int \left( 10\, f_{crystal} - f_{PLL} \right)\, dt
\end{aligned}
$$

# Timers and Counters

A counter can be configured to count up or down on each rising edge of the system clock output from the PLL. The count value can then be used to perform precise timing to either generate output signals or timestamp input signals or events.

In standard up counting mode, shown in the figure below, the counter will incrementally count edges until it hits the **Autoreload** value, which is the maximum count value. Instead of counting past the autoreload value the timer instead resets back to zero. In down counting mode the count resets at the autoreload value instead of counting below zero into negative numbers. In the example below the autoreload, often simplified to AR, is set to 3.

![Timing diagram of a free-running up counter with autoreload and associated clock waveform.](images/timer_counter/basic_upcounting.svg)

The period of the counter depends on the clock frequency, which determines the slope of the stair-step waveform shown above, and the autoreload value which determines the amplitude of the stair-step waveform (the total number of stair steps).

$$T=\frac{AR+1}{f_{CLK}}$$

Notice that $AR+1$ is used in the numerator instead of $AR$ since the timer must count zero explicitly.

## Prescalers

Modern microcontrollers run at very high clock speeds. The STM32 used in ME 4305 typically runs at 80 MHz. At this frequency it is impossible to generate low frequency output using a 16-bit timer. Consider the maximal 16-bit integer, 65,535; using this max value for the autoreload along with an 80 MHz clock frequency gives an approximate maximum period of $T \approx 800 uS$ which corresponds to a minimum frequency close to 1250 kHz.

The solution to producing low frequency output is to apply an additional scaling factor before counting. The prescaler, sometimes referred to as PS, determines the number of *additional* clock edges required to increment the count value and move up one stair-step. In the example below the prescaler is set to 1; consequently the count only increments after two rising edges on the system clock. That is, the prescaler can be considered as an additional factor that adjusts the slope of the stair-step waveform. In other words, prescalers divide the incoming clock before it reaches the counter.

![Comparison of timer operation with and without a prescaler showing reduced counting rate and longer period.](images/timer_counter/prescaled_upcounting.svg)

To determine the period of a prescaled counter, apply another factor equal to $PS+1$ to the previously derived formula for the timer period. 

$$T=\frac{(AR+1)(PS+1)}{f_{CLK}}$$

Now with a 16-bit timer the maximum period becomes approximately $T\approx54s$.

## Left-Aligned PWM

PWM is generated by comparing the counter against a compare register. When a compare value exceeds the present count the associated PWM channel is high and when the present count exceeds the compare value, the PWM output is low. The figure below shows this on a timing diagram. Notice that the duty cycle, or percentage of on-time to the counter period, is proportional to the compare value.

![Up-counting timer compared against two compare values producing left-aligned PWM waveforms with different duty cycles.](images/timer_counter/left_aligned_pwm.svg)
**Note:** in the preceding figure the stair-step waveform has been simplified to a standard sawtooth waveform. It should be imagined that zooming in on the sawtooth would reveal many small stair-steps as shown in previous figures in this lecture.

The duty cycle is computed as the ratio of on-time to the period:

$$D=\frac{CMP}{AR+1}$$

## Center-Aligned PWM

Changing the timer mode to up-down counting produces center-aligned PWM. Sometimes called "phase-correct" PWM, this style is often preferred in motor control as it guarantees that the phase alignment between PWM channels on a shared timer remains constant. With edge-aligned PWM, the relative phase of the PWM channels is a function of the duty cycle, but center-aligned PWM outputs always remain phase aligned except for right after the duty cycle is updated.

![Up-down counting timer producing center-aligned PWM waveforms that reduce phase asymmetry between channels.|700](images/timer_counter/center_aligned_pwm.svg)

In up-down counting mode, the timer counts from zero to the autoreload value and back. Therefore, each count value except for zero and the autoreload value occurs twice per period; consequently there are a total of $2AR$ individual stairsteps in up-down counting mode instead of $AR+1$ individual stairsteps. As a result, the formula to compute the period for center aligned PWM needs to be adjusted to account for the up-down counting:

$$T=\frac{2\,AR\,(PS+1)}{f_{CLK}}$$

The compare value applies while the count goes up or down, so the duty cycle formula is not affected by the factor of two. However, the denominator does need to be adjusted by removing the $+1$ applied to the autoreload value:

$$D=\frac{CMP}{AR}$$

## PWM and Motors

Students beginning to work with DC motors and PWM for the first time often ask why the motor doesn't speed up and slow down as the PWM waveform toggles on and off. The intuitive answer is that it does, by a small amount, but the motor requires more time to change speed noticeably than is available between PWM edges, so the effect is minimal.

From a more academic viewpoint, we can argue that DC motors naturally behave as low-pass systems and therefore attenuate the high frequency components of any applied input signal, like PWM.

Later in the quarter we will develop a mathematical model of the DC motor. That model captures the motor's finite dynamic response, including its inability to change speed (and therefore accelerate) instantaneously, the same behavior that motivates the low-pass analogy, and allows us to estimate the motor's true behavior from limited sensor measurements.

The figure below depicts three plots.
1) The first plot shows the spectrum of a PWM waveform with an assumed duty cycle of 50%. 
2) The second plot shows an approximation of the the frequency response function for a typical DC motor, shown as magnitude ratio vs frequency. Like all low-pass systems there is a passband at low frequencies with little attenuation and an attenuation band at high frequencies.
3) The final plot shows the magnitude of the motor output due each spectral component in the PWM waveform being affected by the frequency response function. That is, each spectral component from the first plot is scaled by the magnitude ratio at the same frequency on the second plot, to produce the spectral component of the output shown on the bottom plot.

For a DC motor to spin smoothly the PWM frequency must be large enough that the harmonics are all attenuated significantly compared to the DC component at zero frequency.

![Frequency-domain illustration showing PWM spectrum, motor low-pass response, and motor output spectrum dominated by the DC component.](images/timer_counter/motor_frequency_response.svg)

**Note:** in practice the floor for PWM frequency may actually be much higher than what satisfies the conditions shown above because it is common to choose ultrasonic PWM frequencies.

Frequencies in the hundreds of Hz to low kHz range will produce noticeable audible tones from the motor which are often undesirable, especially in a busy lab environment with many motors running simultaneously. Selecting PWM frequencies of 20 kHz or higher will guarantee that any produced tones are outside the audible spectrum of human hearing.

## Summary

-   Crystal oscillators provide an accurate timing reference.
-   PLLs generate higher-speed clocks.
-   Timers use autoreload and prescalers.
-   PWM duty cycle is controlled with the compare register.
-   Motors primarily respond to the average value of PWM.

# See Also:
* [[topic_hardware_overview|Hardware and Software Toolchain]]