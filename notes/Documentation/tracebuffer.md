---
title: tracebuffer — Compact Task and State Trace Buffer
type: reference
tags:
  - python
  - micropython
  - debugging
  - tasks
  - finite-state-machine
  - api
source:
  course: ME4305
  repository: https://github.com/charlierefvem/micropython
  commit: c8e6f5f896349825cafbeef4037dee2dd21b93a0
  path: ports/stm32/boards/NUCLEO_L476RG/modules/tracebuffer.py
status: draft
---

The `tracebuffer` module records compact task and state events in a fixed-size byte buffer. Each record occupies four bytes: one byte for a task identifier, one byte for a state identifier, and an unsigned 16-bit value that may represent a timestamp, an event code, or other trace data.

> [!note] Source snapshot
> This page describes [`tracebuffer.py`](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/ports/stm32/boards/NUCLEO_L476RG/modules/tracebuffer.py) at commit [`c8e6f5f896349825cafbeef4037dee2dd21b93a0`](https://github.com/charlierefvem/micropython/commit/c8e6f5f896349825cafbeef4037dee2dd21b93a0).

## Standard use

Create a `TraceBuffer` with space for a fixed number of records. Call `log()` at the points in the program that should be traced. Later, use `get()` to remove records one at a time or `dump()` to print and remove every stored record.

Records are retrieved in reverse order: `get()` returns the most recently logged record first. This last-in, first-out behavior is useful when the events immediately preceding a fault are the most important.

## Event constants

The module provides three event identifiers. They can be stored in the 16-bit `halfword` field when these event meanings fit an application's trace design.

> [!table]
> Event constants defined by `tracebuffer`.
>
> | Constant | Value | Intended meaning |
> | --- | ---: | --- |
> | `EVENT_STATE_CHANGE` | `1` | State-change event. |
> | `EVENT_MODE_CHANGE` | `2` | Mode-change event. |
> | `EVENT_FAULT` | `3` | Fault event. |

## API overview

> [!table]
> Application-facing members of `tracebuffer`.
>
> | Name | Kind | Purpose |
> | --- | --- | --- |
> | `TraceBuffer` | Class | Allocate and manage fixed-size packed trace storage. |
> | `TraceBuffer.log()` | Method | Add one task, state, and value record. |
> | `TraceBuffer.get()` | Method | Remove and return the most recently logged record. |
> | `TraceBuffer.dump()` | Method | Print and remove all stored records. |

## `TraceBuffer`

```python
class tracebuffer.TraceBuffer(length)
```

Allocate storage for `length` records. Each record uses four bytes, so the internal byte buffer has a capacity of `4 * length` bytes.

> [!table]
> Parameter accepted by `TraceBuffer`.
>
> | Parameter | Type | Description |
> | --- | --- | --- |
> | `length` | positive `int` | Maximum number of records retained by the buffer. |

**Raises:** `ValueError` when `length` is zero or negative.

---

### `TraceBuffer.log()`

```python
stored = trace.log(task_id, state_id, halfword, overwrite=True)
```

Pack and store one record. The binary layout is `<BBH`: little-endian, one unsigned byte for `task_id`, one unsigned byte for `state_id`, and one unsigned 16-bit value for `halfword`.

> [!table]
> Parameters accepted by `log()`.
>
> | Parameter | Type | Default | Description |
> | --- | --- | --- | --- |
> | `task_id` | `int` | required | Task identifier stored in one unsigned byte (`0` through `255`). |
> | `state_id` | `int` | required | State identifier stored in one unsigned byte (`0` through `255`). |
> | `halfword` | `int` | required | Trace value stored in one unsigned halfword (`0` through `65535`). |
> | `overwrite` | `bool` | `True` | When the buffer is full, allow the new record to replace the oldest retained record. |

**Returns:** `True` when the record is stored. Returns `False` only when the buffer is already full and `overwrite` is `False`.

Values outside the ranges accepted by the packed field types cause `struct.pack_into()` to raise an exception.

---

### `TraceBuffer.get()`

```python
record = trace.get()
```

Remove the most recently logged record.

**Returns:** A three-item tuple `(task_id, state_id, halfword)`, or `None` when the buffer is empty.

---

### `TraceBuffer.dump()`

```python
trace.dump()
```

Print `Trace:` followed by every stored record in reverse logging order. Task and state identifiers are printed as two-digit hexadecimal values, and the halfword is printed as a four-digit hexadecimal value.

**Returns:** `None`.

> [!warning]
> `dump()` calls `get()` for every record, so dumping the trace empties the buffer.

## Attribution and license

Source: [`tracebuffer.py`](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/ports/stm32/boards/NUCLEO_L476RG/modules/tracebuffer.py) in the `charlierefvem/micropython` repository.

Original work Copyright © 2026 Charlie Refvem. Licensed under the [GNU General Public License, version 3.0 only](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/LICENSE).
