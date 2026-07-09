---
title: Encoders and Decoding Quadrature Output
type: topic
tags:
  - timers
  - encoders
source:
  course: ME405
  term: 2262
  lecture: 6
status: draft
---
## Motivation

Encoders convert position into electrical signals. Both rotary and linear encoders exist, but rotary encoders are considerably more common. You may have already worked with rotary encoders attached to motors in other courses.

### Encoders, Tachometers, and Resolvers

Encoders encode position or displacement and include direction information. Tachometers measure speed only. Resolvers generally output analog position signals, often sine/cosine pairs and are primarily used in motor control.

Incremental encoders are the most common because they are inexpensive and easy to interface. Absolute encoders are preferable when absolute position is required immediately after startup.

## Absolute Encoders

Absolute encoders output a signal representing position with respect to an absolute datum.

### PWM Output

Many modern encoders output a PWM signal in which the duty cycle or on-time represents the angular displacement of the encoder. Some are physically bounded to a single rotation, but others allow continuous rotation, causing the PWM signal to reset once per rotation.

![Graph showing PWM duty cycle varying approximately linearly with shaft angle from 0 to 360 degrees.](images/encoder/absolute_pwm.svg)

### Gray Code

While less popular in modern applications, the Gray code encoder is a classic example of an absolute encoder. Gray code is a special bit pattern used encoder displacement in binary. The special feature provided by Gray code is that between any two adjacent states only one bit in the binary output changes. This allows the device reading the Gray code output to check for errors.

The table below shows the 3-bit Gray code pattern associated with the Gray code disk shown further below. With 3 bits, there are 8 possible states, therefore the resolution of the encoder is one part in eight, or 45°. Each of the concentric rings on the Gray code disk represents one of the bits in the binary representation of the Gray code state.

| State | B2    | B1    | B0    | Binary  |
| :---: | :---: | :---: | :---: | :-----: |
| 0     | 0     | 0     | 0     | `0b000` |
| 1     | 0     | 0     | 1     | `0b001` |
| 2     | 0     | 1     | 1     | `0b011` |
| 3     | 0     | 1     | 0     | `0b010` |
| 4     | 1     | 1     | 0     | `0b110` |
| 5     | 1     | 1     | 1     | `0b111` |
| 6     | 1     | 0     | 1     | `0b101` |
| 7     | 1     | 0     | 0     | `0b100` |

![Circular Gray-code encoder disk with eight angular sectors.](images/encoder/gray_code_disk.svg)

The following snippet of Python code shows how to convert between 3-bit Gray code and binary. Note that for larger Gray code disks the number of cumulative XOR operations will increase as well.
``` python
binary = gray
binary ^= (binary >> 1)
binary ^= (binary >> 2)
```

## Incremental Quadrature Encoders

Incremental encoders do not have a fixed datum. Instead they output increments of displacement using quadrature signals: two square-wave channels, 90° out of phase, provide information about displacement. That is, the phase shift between the two waveforms encodes direction - if A leads B, the encoder is moving one direction and if B leads A, it is moving the other direction.

![Quadrature channel A and B waveforms that are 90 degrees out of phase with A leading B.](images/encoder/quadrature_waveform.svg)

### Optical Encoders

Many high-performance (high resolution) encoders use optics to produce edges. In the figure below are two varieties:
* Transmissive encoders shine light, usually from an LED, through a spoked disk. As the disk rotates the spokes break the line of site between the LED and a pair of photodetectors. If the two detectors are placed carefully the resulting output will have the appropriate 90° phase shift.
* Reflective encoders work on a very similar principle, but instead of having gaps in the encoder disk, there are reflective strips instead. This allows the LED and photodetectors to be on the same side of the disk, making the encoder more compact and easier to assemble.

![Transmissive optical encoder example.](images/encoder/optical_transmissive.svg)
![Reflective optical encoder example.](images/encoder/optical_reflective.svg)

### Magnetic Quadrature Encoders

While optical encoders offer very high density in the form of many spokes they are more costly to produce than other varieties of incremental encoder. Most low-cost incremental encoders are made with multi-pole magnets and hall effect sensors. These work on the same principle as the optical encoders, but instead of detecting light the hall effect sensors detect north vs. south magnetic poles. The animation below shows a 3 pole-pair magnetic encoder.

![Magnetic quadrature encoder with Hall sensors and output waveforms.](images/magnetic_encoder_animation.gif)

### Encoder Resolution

Every quadrature encoder has a specific resolution defined by the smallest increment of change that the encoder can detect. Unfortunately this is one of the areas in engineering where notation and naming conventions lack standardization.

| Term | Definition                                                                                                                          |
| :--: | :---------------------------------------------------------------------------------------------------------------------------------- |
| CPR  | Cycles per revolution - the total number of full cycles including rising and fall edges on both quadrature channels.                |
| CPR  | Counts per revolution - the total number of individual transitions. Equal to $4\times$ the number of cycles per revolution.         |
| PPR  | Pulses per revolution - usually defined the same as counts per revolution, representing the total number of individual transitions. |
Due to the ambiguity in convention it is important for designers to read the documentation for a given encoder in detail to understand if the provided resolution already accounts for the $4\times$ factor.

## Decoding Quadrature

The encoder is responsible for encoding the physical displacement as a pair of electrical signals. However the signals must be decoded back into numbers to be practically useful. Two approaches are common which mirror a common dichotomy in embedded systems: interrupts vs polling.
* With an interrupt based strategy, on each individual edge, code or hardware checks the state of the signals and increments a counter up or down depending on the direction.
* With a polling based strategy, the edges are ignored, and instead the state of each signal is sampled frequently and compared to the previous state.

Up until the mid 2000s, quadrature decoding required specialized components in addition to the microcontroller, or the use of interrupts and firmware to decode the signals. Interrupt based methods require a significant amount of CPU time and the time scales with the resolution of the encoder and its rotational velocity. Dense (high resolution) encoders spinning at high velocity can produce quadrature output in the kHz or MHz range. Triggering interrupts at this rate may or may not be possible, but even if the interrupts can keep up, they may require enough CPU time to cause other parts of the application code to have problems.

Most modern microcontrollers now have built in hardware to decode quadrature signals. For the STM32 series, the quadrature decoding is part of the timer peripheral. The timer implements a polling based strategy at the system clock frequency, typically in the tens or hundreds of MHz.

### Edge Decoding

Direction can be decoded at every edge if we know both:
1.  The edge direction.
2.  The state of the opposite channel.

From this information, a lookup table, like the one shown below, can be used to determine direction.

| Ch. A | Ch. B | Direction      |
| :---: | :---: | :------------- |
| ↑     | H     | Down (Reverse) |
| ↑     | L     | Up (Forward)   |
| ↓     | H     | Up (Forward)   |
| ↓     | L     | Down (Reverse) |
| H     | ↑     | Up (Forward)   |
| L     | ↑     | Down (Reverse) |
| H     | ↓     | Down (Reverse) |
| L     | ↓     | Up (Forward)   |
**Note**: the arrows (↑ and ↓) shown in the table refer to rising and falling edges.

#### Example 1

In this example a basic case of decoding will be covered for an encoder rotating with constant velocity.

![Quadrature edge decoding waveform.](images/encoder/quadrature_decoding_simple.svg)

Examine the figure above and notice the highlighted edge on Channel A. In this figure the horizontal axes represent time, so reading the waveforms left-to-right makes the edge of Channel A a *rising edge*. When this rising edge occurs, the opposite channel is low. Therefore, according to row two of the table above, the encoder is rotating in the forward direction and the count should go up by one unit.

All edges in the waveform below will produce the same direction using the lookup table because the waveforms correspond to constant velocity rotation. It's easy to see this visually by noticing that Channel A leads Channel B for the entire duration shown.

### Polling Decoder

STM32 timer hardware compares the current AB state against the previous state instead of explicitly detecting edges. This comparison occurs rapidly, once per edge on the clock source for the timer. On the Nucleo L476RG all timers run directly off the system clock at 80MHz, so the polling occurs 80 million times per second. Built into the microcontroller is a lookup table, similar to the one shown below, that shows the count direction based on the present and previous AB states.

![Sixteen-state quadrature polling transition table.|700](images/quadrature_polling_table.png)

The timer can also XOR the two channels to generate a square wave whose frequency represents speed.

#### Example 2

In this example a more complicated motion will be decoded in which the encoder changes direction.

![Worked quadrature decoding example showing channel waveforms, XOR output, and accumulated count.](images/encoder/quadrature_decoding_advanced.svg)

There are four plots shown in the figure above:
1) The first plot shows the quadrature waveform for Channel A. Notice that it no longer has a fixed frequency; instead each low and high portion is of different length.
2) The second plot shows the corresponding waveform for Channel B.
3) The third plot shows the exclusive or (XOR) of Channel A with Channel B. The exclusive or, when applied to square waves, produces a new square wave with the frequency content of both input waveforms. In other words, the signal shown on the third plot includes every edge from both Channel A and Channel B. Overlaid on this plot are many marks indicating whether the edge represents an up-count or a down-count. These marks were produced by applying the AB state lookup table presented above.
4) The final plot shows the motion of the encoder as read from the accumulated up- and down-counting. The black line shows a true (but quantized) representation of the actual displacement of the encoder. Notice that as the encoder spins backwards its displacement crosses zero and goes negative.
   However, the timer count can only hold positive integer values. So, instead of the count going negative, it instead reloads at the autoreload value, which is assumed to be AR=5 for the example. The encoder displacement, as seen by the timer count, is shown in red.

From this example we can conclude that the hardware can effectively count encoder increments, but is unable to count arbitrarily high or low. In the example a reload while counting down, which may loosely be referred to as **underflow**, but the counter can also reload while counting up, which may loosely be referred to as **overflow**.

## Correcting Encoder Reload

To correct for the counter reloading during active operation we must read from the counter at a regular interval and detect overflow or underflow when it occurs.

### Example 3

The diagram below depicts an encoder that has been rotating with constant velocity for long enough that the counter was forced to overflow.

One approach to detecting overflow is to frequently compute the change in count between updates and then use a rule to determine whether the change in count is correct, or off due to overflow.

![Timer rollover compensation example illustrating overflow and underflow correction.](images/encoder/reload_algorithm.svg)

In the example waveform in the figure above, overflow occurs between update #4 and update #5. The change in count *should* be a small positive change, as indicated by the black-colored $\Delta45$ but the computed value will actually be a larger negative change, as indicated by the $\Delta45$ shown in red.

Two observations can be made about the incorrect  $\Delta45$ , it is both the wrong sign and the wrong magnitude. Instead of being small and positive the change is large and negative. If the encoder were rotating the opposite direction and underflowed a similar effect occurs: instead of a small negative change the underflow would cause the count to change to be a larger positive amount.

Therefore, to detect when overflow occurs we check both the sign and magnitude of the change, and if the magnitude is greater than a certain threshold we identify the delta as incorrect, and offset appropriately to compensate for the overflow.

**Algorithm:**
1.  Sample the timer count periodically and compute $\Delta$, the change in count since the last update: `delta = count - last_count`
2.  Validate $\Delta$ (check for reload):
    * Overflow: `delta < -(AR+1)/2` → `delta += AR+1`
    * Underflow: `delta > (AR+1)/2` → `delta -= AR+1`
3.  Accumulate the validated $\Delta$ values: `position += delta`

Astute readers will ask "how do we know that the overflow or underflow occurred and that the encoder didn't actually change direction rapidly when we flag an incorrect change in count?". To answer this question we need to determine the sample rate at which we apply the correction algorithm.

If the updates run sufficiently fast for a given maximum rotation rate of the encoder we can guarantee that the algorithm never fails. That is, we need to determine the minimum update rate for the algorithm such that between two updates the change is never larger than the threshold (half of AR+1) used to distinguish correct and incorrect changes.

$$
f_{update}\left[\frac{1}{sec}\right] \ge \frac{
\omega_{max}\left[\frac{rev}{min}\right] \cdot
\frac{1\,[min]}{60\,[sec]} \cdot
\frac{4\cdot CPR\,[ticks]}{1 [rev]}}{\frac{AR+1}{2}\,[ticks]}
$$
**Note:** when running the calculation above it is critical that you use the maximum angular velocity of the encoder disk itself, not the output velocity of the motor the encoder is attached to if the motor includes gear reduction. Otherwise the computed frequency will be off by a factor of the gear ratio.

In ME 4305, the maximum rotation rate for the 3 pole-pair magnetic encoder (before the gear reduction) will be approximately $\omega_{max} = 30,000\,[RPM]$. Using the maximum for a 16-bit timer, the autoreload is AR=65,535. With these numbers the update rate comes out to:

$$
\begin{aligned}
f_{update}\left[\frac{1}{sec}\right] &\ge \frac{
30,000\left[\frac{rev}{min}\right] \cdot
\frac{1\,[min]}{60\,[sec]} \cdot
\frac{4\cdot 3\,[ticks]}{1 [rev]}}{32,768\,[ticks]} \\[10pt]
& \ge 0.183\,[Hz]
\end{aligned}
$$
or about one update every 5.4 seconds.

**Caution**: This result should not be assumed to be universal. The extremely low minimum update rate is a direct result of the low resolution encoder: 3 CPR is an extremely low resolution. Most encoders used in industry have thousands of cycles per revolution, and will need a significantly faster update rate.

**Bonus Insight**: The criteria above, when considered in the context of the encoder count waveform, can be reinterpreted intuitively. To guarantee that every underflow and overflow is detected the update must run at least twice per period. Any readers familiar with signal processing will recognize this as the Nyquist sampling criteria. The Nyquist sampling criteria states that the frequency of a signal can be measured only if the signal is sampled at a frequency at least twice that of the signal.

### Summary

* Encoders measure displacement.
* Quadrature provides direction.
* STM32 timers decode quadrature in hardware.
* Correct timer rollover before accumulating position.
