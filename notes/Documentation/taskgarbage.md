---
title: taskgarbage — Cooperative Garbage Collection
type: reference
tags:
  - python
  - micropython
  - tasks
  - memory
  - garbage-collection
  - api
source:
  course: ME4305
  repository: https://github.com/charlierefvem/micropython
  commit: fd90403415b3f8962d778ab96c0ff5427c5e1773
  path: ports/stm32/boards/NUCLEO_L476RG/modules/taskgarbage.py
status: draft
---

[[index|← ME4305 firmware and API documentation]]

The `taskgarbage` module supplies a generator that performs garbage collection cooperatively. It lets an application make garbage collection one scheduled activity instead of leaving collection to occur automatically at an unpredictable point in another task.

> [!note] Source snapshot
> This page describes [`taskgarbage.py`](https://github.com/charlierefvem/micropython/blob/fd90403415b3f8962d778ab96c0ff5427c5e1773/ports/stm32/boards/NUCLEO_L476RG/modules/taskgarbage.py) at commit [`fd90403415b3f8962d778ab96c0ff5427c5e1773`](https://github.com/charlierefvem/micropython/commit/fd90403415b3f8962d778ab96c0ff5427c5e1773).

## Standard use

Call `taskgarbage.run()` to create its generator object, then give that generator to a cooperative scheduler such as [`cotask`](cotask.md). Each scheduled advance performs one explicit garbage-collection cycle and then yields.

> [!warning]
> The first advance of this generator disables automatic garbage collection. The module does not re-enable it. The application must therefore continue scheduling the generator often enough for its allocation pattern and available memory.

## API overview

> [!table]
> Application-facing member of `taskgarbage`.
>
> | Name | Kind | Purpose |
> | --- | --- | --- |
> | `run()` | Generator function | Disable automatic collection, then explicitly collect once per scheduled run. |

## `run()`

```python
garbage_task_generator = taskgarbage.run()
```

Create the cooperative garbage-collection generator. Calling the function creates the generator object; its body begins running only when the scheduler first advances that object.

On its first advance, the generator calls `gc.disable()`. On every advance, including the first, it calls `gc.collect()` and then yields `None`.

**Parameters:** None.

**Yields:** `None` after each garbage-collection cycle.

**Returns:** The generator is designed to run indefinitely and has no normal return value.

## Attribution and license

Source attribution: [`taskgarbage.py`](https://github.com/charlierefvem/micropython/blob/fd90403415b3f8962d778ab96c0ff5427c5e1773/ports/stm32/boards/NUCLEO_L476RG/modules/taskgarbage.py) in the `charlierefvem/micropython` repository maintained by Charlie Refvem.

The source file does not contain a copyright or license notice, and the pinned repository snapshot does not contain a root license file.

TODO (Instructor Review): Confirm the authorship and license terms for `taskgarbage.py` before public release.
