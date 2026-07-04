---
title: Digital Filters
type: reference
tags:
  - digital-filters
  - discrete-systems
  - z-transform
  - continuous-to-discrete
source:
  course: ME507
  term: 2264
  lecture: 16
status: draft
---

# Motivation

In an ideal world sensors would return perfect, noise-free measurements that precisely match physical quantities being measured. Real sensors include many kinds of noise on top of their measurement such as quantization noise, electrical noise, vibration, and random variation in their outputs. One strategy to reduce unwanted measurement noise is to use a digital filter, such as an FIR or IIR filter. The engineering tradeoff is that filtering usually makes the signal smoother but also adds delay or reduces responsiveness.

Digital filters can be designed directly in discrete time, or some filters can be designed first as continuous-time transfer functions and then converted to discrete time. See [[reference_continuous_to_discrete|Continuous to Discrete Conversion]] for multiple conversion techniques.

# Finite impulse response (FIR) filters

A finite impulse response (FIR) filter only depends on a recent history of inputs, or measurements. It does not depend on past filter outputs. Let $x_k$ be the sampled measurement entering the filter, and let $y_k$ be the filtered output.
$$
y_k = N_0\,x_k + \dots + N_n\,x_{k-n}
= \sum_{j=0}^{n} N_j\, x_{k-j}.
$$

The filter is of order $n$, meaning that the output depends on a weighted sum of the current measurement $x_k$ and $n$ past measurements. In smoothing filters, these coefficients are often chosen to form a weighted average.

The filter is called a finite impulse response filter because a discrete input sequence can be thought of as a collection of delayed unit samples. For an FIR filter, the effect of any one input sample disappears after a finite number of updates.

The same filter can be expressed as a transfer function in the [[reference_z_domain|z-domain]] with a denominator equal to unity:
$$
\frac{y(z)}{x(z)}
= N_0 + \dots + N_n\,z^{-n}
= \sum_{j=0}^{n} N_j\, z^{-j}.
$$

FIR filters with finite coefficients are inherently stable because their response to any single input sample ends after a finite number of steps.

# Infinite impulse response (IIR) filters

An infinite impulse response (IIR) filter depends on a recent history of inputs and also feedback from a recent history of past filter outputs.

$$
D_0\,y_k
= (N_0\,x_k + \dots + N_n\,x_{k-n})
- (D_1\,y_{k-1} + \dots + D_n\,y_{k-n}).
$$
For simplicity, this expression uses the same maximum order $n$ for the numerator and denominator. More generally, the input and feedback histories may have different lengths.

**Note**: often the coefficients are normalized so that $D_0=1$. This does not reduce tuning freedom, as long as $D_0\neq 0$, because all coefficients can be divided by the same value.

Equivalently,
$$
D_0\,y_k
= \sum_{i=0}^{n} N_i\,x_{k-i}
- \sum_{j=1}^{n} D_j\,y_{k-j}.
$$

The filter is called an infinite impulse response filter because each new measurement $x_k$ triggers a response in $y_k$, but past values $y_{k-1}$, $y_{k-2}$, etc. also trigger a response in $y_k$. Therefore, the response from $x_k$ may never actually decay to zero in finite time.

The transfer-function representation now includes a denominator:
$$
\frac{y(z)}{x(z)}
=
\frac{N_0 + \dots + N_n\,z^{-n}}
{D_0 + \dots + D_n\,z^{-n}}
=
\frac{\sum_{i=0}^{n} N_i\,z^{-i}}
{\sum_{j=0}^{n} D_j\,z^{-j}}.
$$

Because an IIR filter uses feedback from past outputs, its denominator coefficients must be chosen so the filter is stable; in z-domain terms, the poles must lie inside the unit circle.

# Summary

In firmware, both FIR and IIR filters are implemented as update equations that run once per sample period, just like discrete-time controllers and observers.