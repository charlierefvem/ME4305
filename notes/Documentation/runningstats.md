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
  commit: c8e6f5f896349825cafbeef4037dee2dd21b93a0
  path: ports/stm32/boards/NUCLEO_L476RG/modules/runningstats.py
status: draft
---

The `runningstats` module accumulates summary statistics one sample at a time. It uses Welford's online algorithm, so it does not need to keep a list of all earlier samples. The `cotask` scheduler uses this class for duration and latency profiles.

> [!note] Source snapshot
> This page describes [`runningstats.py`](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/ports/stm32/boards/NUCLEO_L476RG/modules/runningstats.py) at commit [`c8e6f5f896349825cafbeef4037dee2dd21b93a0`](https://github.com/charlierefvem/micropython/commit/c8e6f5f896349825cafbeef4037dee2dd21b93a0).

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

**Returns:** The largest sample seen since construction or the most recent `reset()`. Returns `0.0` when there are no samples. The first sample establishes the maximum, so streams containing only negative values are handled correctly.

---

### `RunningStats.reset()`

```python
stats.reset()
```

Clear the sample count and restore the accumulated mean, variance state, and maximum to `0.0`.

**Returns:** `None`.

## Attribution and license

Source: [`runningstats.py`](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/ports/stm32/boards/NUCLEO_L476RG/modules/runningstats.py) in the `charlierefvem/micropython` repository.

Original work Copyright © 2026 Charlie Refvem. Licensed under the [GNU General Public License, version 3.0 only](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/LICENSE).
