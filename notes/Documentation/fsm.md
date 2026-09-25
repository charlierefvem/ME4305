---
title: fsm — Finite-State-Machine Task Base Class
type: reference
tags:
  - python
  - micropython
  - finite-state-machine
  - tasks
  - api
source:
  course: ME4305
  repository: https://github.com/charlierefvem/micropython
  commit: fd90403415b3f8962d778ab96c0ff5427c5e1773
  path: ports/stm32/boards/NUCLEO_L476RG/modules/fsm.py
status: draft
---

[[index|← ME4305 firmware and API documentation]]

The `fsm` module provides a small base class for cooperative tasks organized as finite-state machines. A subclass supplies its own `run()` method and uses `transition_to()` to record state changes.

> [!note] Source snapshot
> This page describes [`fsm.py`](https://github.com/charlierefvem/micropython/blob/fd90403415b3f8962d778ab96c0ff5427c5e1773/ports/stm32/boards/NUCLEO_L476RG/modules/fsm.py) at commit [`fd90403415b3f8962d778ab96c0ff5427c5e1773`](https://github.com/charlierefvem/micropython/commit/fd90403415b3f8962d778ab96c0ff5427c5e1773).

## Standard use

Create a subclass of `FSM`, implement the subclass's task behavior in `run()`, and call `transition_to()` when the task should move to a different state. The leading underscores on `_state` and `_last_state` identify them as implementation attributes, but the current base class does not provide public state-access properties; a subclass therefore reads `_state` when selecting the code for its current state.

## API overview

> [!table]
> Application-facing members of `fsm`.
>
> | Name | Kind | Purpose |
> | --- | --- | --- |
> | `FSM` | Class | Store the current and previously run states for a finite-state-machine task. |
> | `FSM.run()` | Method | Placeholder for the task behavior supplied by a subclass. |
> | `FSM.transition_to()` | Method | Change the current state and return the state that just finished. |

## `FSM`

```python
class fsm.FSM(initial_state=0)
```

Create the base state-machine object.

> [!table]
> Parameter accepted by `FSM`.
>
> | Parameter | Type | Default | Description |
> | --- | --- | --- | --- |
> | `initial_state` | any state identifier | `0` | Value stored as the current state when the object is created. |

### Subclass state attributes

> [!table]
> Implementation attributes used when writing an `FSM` subclass.
>
> | Attribute | Description |
> | --- | --- |
> | `_state` | State that the subclass should run next. It begins as `initial_state`. |
> | `_last_state` | State saved by the most recent successful call to `transition_to()`. It begins as `0`. |

---

### `FSM.run()`

```python
fsm_object.run()
```

The base implementation is a placeholder and does nothing.

**Returns:** `None` in the base class.

Subclasses are expected to override this method with the behavior needed by the task.

---

### `FSM.transition_to()`

```python
previous_state = fsm_object.transition_to(new_state)
```

Move the state machine to `new_state`. When `new_state` is not `None`, the method saves the current `_state` in `_last_state` and then stores `new_state` in `_state`.

> [!table]
> Parameter accepted by `transition_to()`.
>
> | Parameter | Type | Description |
> | --- | --- | --- |
> | `new_state` | any state identifier or `None` | Next state to run. Passing `None` leaves both stored states unchanged. |

**Returns:** The value of `_last_state`. After a transition, this is the state that was current immediately before the call. If `new_state` is `None`, the method returns the previously stored `_last_state`.

## Attribution and license

Source attribution: [`fsm.py`](https://github.com/charlierefvem/micropython/blob/fd90403415b3f8962d778ab96c0ff5427c5e1773/ports/stm32/boards/NUCLEO_L476RG/modules/fsm.py) in the `charlierefvem/micropython` repository maintained by Charlie Refvem.

The source file does not contain a copyright or license notice, and the pinned repository snapshot does not contain a root license file.

TODO (Instructor Review): Confirm the authorship and license terms for `fsm.py` before public release.
