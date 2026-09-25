---
title: registerinterface — Generated Motor Register Interface
type: reference
tags:
  - python
  - micropython
  - registers
  - motors
  - shared-memory
  - api
source:
  course: ME4305
  repository: https://github.com/charlierefvem/micropython
  commit: c8e6f5f896349825cafbeef4037dee2dd21b93a0
  paths:
    - tools/register_definitions.json
    - tools/register_map_generator.py
    - tools/README.md
status: draft
---

The generated `registerinterface` module provides one shared byte buffer with typed register views for the left- and right-motor systems. Code can work with individual named fields through `uctypes` objects or with related floating-point fields through `ulab` array views.

All public views share the same `buf`. Writing a value through one view immediately changes every other view that overlaps the same bytes.

> [!note] Reproducible generated source
> `registerinterface.py` is generated during the firmware build and is intentionally not tracked. This page is derived from the pinned [`register_definitions.json`](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/tools/register_definitions.json), [`register_map_generator.py`](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/tools/register_map_generator.py), and [generated-interface API contract](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/tools/README.md) at commit [`c8e6f5f896349825cafbeef4037dee2dd21b93a0`](https://github.com/charlierefvem/micropython/commit/c8e6f5f896349825cafbeef4037dee2dd21b93a0).

The matching build generated schema hash `638fb41ec62c612e`, API manifest `register_metadata_638fb41ec62c612e.json`, and a 256-byte register map.

## Standard use

Import the module, then read or write a field through the motor object that owns it:

```python
import registerinterface as reg

reg.lm.Parameters.Gains.Kp = 0.25
left_velocity = reg.lm.Signals.OutputEstimates.Velocity
```

Use an array view when an operation should address a complete related group:

```python
reg.lm_K[1] = 0.25  # The Kp element
```

The generated module depends on MicroPython, `uctypes`, and `ulab`; it is not intended to be imported by ordinary CPython.

## API overview

> [!table]
> Public members of `registerinterface`.
>
> | Name | Kind | Purpose |
> | --- | --- | --- |
> | `schema_hash` | `int` | Identify the register-definition schema used to generate the module. |
> | `REG_MAP_ALLOC_SIZE` | integer constant | Report the complete allocated register-map size in bytes. |
> | `buf` | `bytearray` | Store the bytes shared by all register and array views. |
> | `hexdump()` | Function | Print `buf` as hexadecimal bytes, four bytes per line. |
> | `lm`, `rm` | `uctypes.struct` objects | Provide named left- and right-motor register fields. |
> | Alias arrays | `ulab.numpy.ndarray` objects | Provide one-dimensional floating-point views of related fields. |

## Module attributes and function

### `schema_hash`

```python
registerinterface.schema_hash
```

**Value for this snapshot:** `0x638fb41ec62c612e`.

This integer contains the first 64 bits of the SHA-256 hash of the pinned register-definition JSON. It identifies the schema, not the firmware as a whole.

---

### `REG_MAP_ALLOC_SIZE`

```python
registerinterface.REG_MAP_ALLOC_SIZE
```

**Value for this snapshot:** `256` bytes.

---

### `buf`

```python
registerinterface.buf
```

The shared `bytearray` that backs all register objects and alias arrays. Its length is `REG_MAP_ALLOC_SIZE`.

Application code should normally use the typed views instead of calculating byte offsets itself.

---

### `hexdump()`

```python
registerinterface.hexdump()
```

Print every byte in `buf` as two hexadecimal digits followed by a space, inserting a newline after every fourth byte.

**Parameters:** None.

**Returns:** `None`.

## Motor register objects

`lm` represents `LeftMotor.Registers`; `rm` represents `RightMotor.Registers`. They have the same nested field structure. Prefix a relative path in the following table with either `lm.` or `rm.`.

> [!table]
> Fields available on both motor register objects.
>
> | Relative Python path | Type | Units | Declared range |
> | --- | --- | --- | --- |
> | `Commands.Active` | `UINT8` | — | Storage range |
> | `Parameters.Settings.SaturationMinimum` | `FLOAT32` | V | $-\infty$ to $\infty$ |
> | `Parameters.Settings.SaturationMaximum` | `FLOAT32` | V | $-\infty$ to $\infty$ |
> | `Parameters.Gains.Kf` | `FLOAT32` | `%*s/rad` | $-\infty$ to $\infty$ |
> | `Parameters.Gains.Kp` | `FLOAT32` | `%*s/rad` | $-\infty$ to $\infty$ |
> | `Parameters.Gains.Ki` | `FLOAT32` | `%*s/rad` | $-\infty$ to $\infty$ |
> | `Signals.Timekeeping.Time` | `UINT32` | `us` | $0$ to $\infty$ |
> | `Signals.Timekeeping.SampleIndex` | `UINT32` | samples | $0$ to maximum `UINT32` |
> | `Signals.Controller.Setpoint` | `FLOAT32` | rad/s | $-\infty$ to $\infty$ |
> | `Signals.Controller.Error` | `FLOAT32` | rad/s | $-\infty$ to $\infty$ |
> | `Signals.Controller.IntegralError` | `FLOAT32` | rad/s | $-\infty$ to $\infty$ |
> | `Signals.Controller.RequestedVoltage` | `FLOAT32` | V | $-\infty$ to $\infty$ |
> | `Signals.Observer.AppliedVoltage` | `FLOAT32` | V | $-\infty$ to $\infty$ |
> | `Signals.Observer.Displacement` | `FLOAT32` | tick | $-\infty$ to $\infty$ |
> | `Signals.OutputEstimates.Velocity` | `FLOAT32` | rad/s | $-\infty$ to $\infty$ |
> | `Signals.OutputEstimates.Displacement` | `FLOAT32` | rad | $-\infty$ to $\infty$ |
> | `Signals.OutputEstimates.Disturbance` | `FLOAT32` | rad/s/s | $-\infty$ to $\infty$ |
> | `Signals.StateEstimates.Velocity` | `FLOAT32` | rad/s | $-\infty$ to $\infty$ |
> | `Signals.StateEstimates.Displacement` | `FLOAT32` | rad | $-\infty$ to $\infty$ |
> | `Signals.StateEstimates.Disturbance` | `FLOAT32` | rad/s/s | $-\infty$ to $\infty$ |
> | `Signals.Motor.DCSupplyVoltage` | `FLOAT32` | V | $0$ to $\infty$ |
> | `Signals.Motor.DutyCycle` | `FLOAT32` | % | $-\infty$ to $\infty$ |

Every field is read-write. The declared range is documentation metadata and is not enforced by the generated module.

> [!note]
> `Signals.Timekeeping.Time` has presentation scale $10^{-6}$ from microseconds to seconds, and `Signals.Observer.Displacement` has presentation scale `0.004372027` from encoder ticks to radians. These scale values do not change the raw values stored in `buf`.

## Alias array views

The alias arrays provide read-write `ulab.numpy.ndarray` views over contiguous `FLOAT32` register groups. Left- and right-motor arrays have matching layouts.

> [!table]
> Generated alias arrays and their element order.
>
> | Left / right names | Shape | Elements in order |
> | --- | ---: | --- |
> | `lm_K` / `rm_K` | `(3,)` | `Kf`, `Kp`, `Ki` |
> | `lm_z` / `rm_z` | `(4,)` | `Setpoint`, `Error`, `IntegralError`, `RequestedVoltage` |
> | `lm_w` / `rm_w` | `(2,)` | `AppliedVoltage`, `Displacement` |
> | `lm_y_h` / `rm_y_h` | `(3,)` | `Velocity`, `Displacement`, `Disturbance` from `OutputEstimates` |
> | `lm_x_h` / `rm_x_h` | `(3,)` | `Velocity`, `Displacement`, `Disturbance` from `StateEstimates` |

## Errors and constraints

- A missing field raises the normal `AttributeError`.
- Invalid assignments and array operations are handled by the firmware's `uctypes` or `ulab` implementation.
- Importing the generated module without its MicroPython dependencies raises `ImportError`.
- Callers should not depend on CPython coercion or exception details for invalid values.
- Names beginning with `_`, generated descriptor dictionaries, byte padding, and descriptor construction are private implementation details.

## Attribution and license

The [register definitions](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/tools/register_definitions.json), [generator](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/tools/register_map_generator.py), generated module, generated API manifest, and [API contract](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/tools/README.md) are Copyright © 2026 Charlie Refvem.

Licensed under the [GNU General Public License, version 3.0 only](https://github.com/charlierefvem/micropython/blob/c8e6f5f896349825cafbeef4037dee2dd21b93a0/LICENSE).
