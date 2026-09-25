---
title: runningstats — Online Running Statistics
type: reference
tags:
  - python
  - micropython
  - statistics
  - profiling
  - api
source:
  course: ME4305
  repository: https://github.com/charlierefvem/micropython
  commit: fd90403415b3f8962d778ab96c0ff5427c5e1773
  path: ports/stm32/boards/NUCLEO_L476RG/modules/runningstats.py
status: draft
---

[[index|← ME4305 firmware and API documentation]]

The `runningstats` module accumulates summary statistics one sample at a time. It uses Welford's online algorithm, so it does not need to keep a list of all earlier samples. The `cotask` scheduler uses this class for duration and latency profiles.

> [!note] Source snapshot
> This page describes [`runningstats.py`](https://github.com/charlierefvem/micropython/blob/fd90403415b3f8962d778ab96c0ff5427c5e1773/ports/stm32/boards/NUCLEO_L476RG/modules/runningstats.py) at commit [`fd90403415b3f8962d778ab96c0ff5427c5e1773`](https://github.com/charlierefvem/micropython/commit/fd90403415b3f8962d778ab96c0ff5427c5e1773).

## Standard use

Create one `RunningStats` object for a stream of measurements. Call `update()` once for each new value, then read the properties when a summary is needed. Call `reset()` before beginning an independent measurement interval.

## API overview

> [!table]
> Public members of `runningstats`.
>
> | Name | Kind | Purpose |
> | --- | --- | --- |
> | `RunningStats` | Class | Accumulate statistics without storing every sample. |
> | `RunningStats.update()` | Method | Add one sample. |
> | `RunningStats.mean` | Property | Read the running arithmetic mean. |
> | `RunningStats.variance` | Property | Read the sample variance. |
> | `RunningStats.std` | Property | Read the sample standard deviation. |
> | `RunningStats.max` | Property | Read the stored maximum. |
> | `RunningStats.reset()` | Method | Clear the accumulated statistics. |

## `RunningStats`

```python
class runningstats.RunningStats()
```

Create an empty accumulator. Before any calls to `update()`, `mean`, `variance`, `std`, and `max` all evaluate to `0.0`.

---

### `RunningStats.update()`

```python
stats.update(x)
```

Add one sample and update the mean, sample variance, sample standard deviation, and maximum.

> [!table]
> Parameter accepted by `update()`.
>
> | Parameter | Type | Description |
> | --- | --- | --- |
> | `x` | number | New sample to include in the accumulated statistics. |

**Returns:** `None`.

---

### `RunningStats.mean`

```python
stats.mean
```

**Returns:** The arithmetic mean of the samples seen since construction or the most recent `reset()`. Returns `0.0` when there are no samples.

---

### `RunningStats.variance`

```python
stats.variance
```

**Returns:** The sample variance, using a denominator of $n-1$. Returns `0.0` when fewer than two samples have been added.

---

### `RunningStats.std`

```python
stats.std
```

**Returns:** The square root of `variance`. This is the sample standard deviation and is `0.0` when fewer than two samples have been added.

---

### `RunningStats.max`

```python
stats.max
```

**Returns:** The largest stored value according to the accumulator's current implementation.

Technical Note: The stored maximum begins at `0.0` and changes only when a sample is greater than the current maximum. A data set containing only negative values therefore reports `0.0`, even though `0.0` was not observed.

---

### `RunningStats.reset()`

```python
stats.reset()
```

Clear the sample count and restore the accumulated mean, variance state, and maximum to `0.0`.

**Returns:** `None`.

## Attribution and license

Source attribution: [`runningstats.py`](https://github.com/charlierefvem/micropython/blob/fd90403415b3f8962d778ab96c0ff5427c5e1773/ports/stm32/boards/NUCLEO_L476RG/modules/runningstats.py) in the `charlierefvem/micropython` repository maintained by Charlie Refvem.

The source file does not contain a copyright or license notice, and the pinned repository snapshot does not contain a root license file.

TODO (Instructor Review): Confirm the authorship and license terms for `runningstats.py` before public release.
