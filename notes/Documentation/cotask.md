---
title: cotask — Cooperative Task Scheduler
type: reference
tags:
  - python
  - micropython
  - tasks
  - multitasking
  - scheduler
  - api
source:
  course: ME4305
  repository: https://github.com/charlierefvem/micropython
  commit: fd90403415b3f8962d778ab96c0ff5427c5e1773
  path: ports/stm32/boards/NUCLEO_L476RG/modules/cotask.py
status: draft
---

[[index|← ME4305 firmware and API documentation]]

The `cotask` module provides a small cooperative task scheduler for MicroPython. A task is implemented as a Python generator: each call to the generator performs one short, bounded portion of work and then uses `yield` to return control to the scheduler.

The module supports:

- periodic tasks that become ready after a specified time;
- triggered tasks that become ready when their `go()` method is called;
- priority-based or round-robin scheduling; and
- optional measurements of task duration and scheduling latency.

> [!note]
> This module is intended for MicroPython. It imports MicroPython-specific modules and is normally frozen into the firmware, but application code imports it in the usual way with `import cotask`.

> [!note] Source snapshot
> This page describes [`cotask.py`](https://github.com/charlierefvem/micropython/blob/fd90403415b3f8962d778ab96c0ff5427c5e1773/ports/stm32/boards/NUCLEO_L476RG/modules/cotask.py) at commit [`fd90403415b3f8962d778ab96c0ff5427c5e1773`](https://github.com/charlierefvem/micropython/commit/fd90403415b3f8962d778ab96c0ff5427c5e1773).

## Quick start

The generator object passed to `Task` must already have been created by calling the generator function. The scheduler then advances that object by one `yield` each time the task is ready.

> [!block_listing] Creating and scheduling a periodic task
> ```python
> import cotask
>
> def task1_fun():
>     # A minimal two-state task used only for this example.
>     state = 0
>
>     while True:
>         if state == 0:
>             state = 1
>         elif state == 1:
>             state = 0
>
>         yield state
>
> # Calling task1_fun() creates the generator object used by Task.
> task1 = cotask.Task(task1_fun(), name="Task 1", priority=1,
>                     period=500, profile=True)
> cotask.task_list.append(task1)
>
> while True:
>     cotask.task_list.pri_sched()
> ```

The example gives the task a period of 500 ms, so it is eligible to advance twice per second. The scheduler loop should run continuously and should not contain a blocking delay.

> [!warning]
> Cooperative scheduling depends on every task yielding promptly. One task that blocks or performs an unbounded amount of work prevents the other tasks from running on time.

> [!note] Generator objects in the current API
> The current version of `cotask` expects a generator object such as `task1_fun()` or `task.run()`. Some older examples may instead pass the generator function or method without calling it because this API changed recently.

## API overview

> [!table]
> Public classes and module-level objects in `cotask`.
>
> | Name | Kind | Purpose |
> | --- | --- | --- |
> | `Task` | Class | Wrap a generator object with scheduling and optional profiling behavior. |
> | `TaskList` | Class | Store tasks and dispatch them using priority or round-robin scheduling. |
> | `task_list` | `TaskList` object | The module's shared task list, created automatically when `cotask` is imported. |

## `Task`

```python
class cotask.Task(run_fun, name="NoName", priority=0, period=None,
                  profile=False)
```

Create a cooperative task around a generator object. A `Task` may be periodic, in which case the scheduler determines when it is ready from elapsed time, or triggered, in which case another task or an interrupt marks it ready by calling `go()`.

The generator should normally contain an indefinite loop and reach `yield` once during each short task run. If the generator finishes, its next scheduled run raises `StopIteration`.

### Constructor parameters

> [!table]
> Parameters accepted by `Task`.
>
> | Parameter | Type | Default | Description |
> | --- | --- | --- | --- |
> | `run_fun` | generator object | required | Generator object that implements the task. Call the generator function or method, such as `task.run()`, before passing it to `Task`. |
> | `name` | `str` | `"NoName"` | Short, descriptive name shown in diagnostic and profiling output. |
> | `priority` | `int` | `0` | Scheduling priority. Larger numbers represent higher priorities. |
> | `period` | number or `None` | `None` | Time in milliseconds between task runs. Use `None` to create a task that runs only after `go()` is called. |
> | `profile` | `bool` | `False` | Set to `True` to collect task-duration and scheduling-latency statistics. |

Technical Note: Use a positive value for a periodic task's `period`. The source accepts `0`, but `Task.profile()` then treats the task like a triggered task while `TaskList.profile()` treats it like a periodic task, causing incompatible report formatting.

### Public attributes

> [!table]
> Publicly visible attributes of a `Task` object.
>
> | Attribute | Type | Description |
> | --- | --- | --- |
> | `name` | `str` | Name supplied to the constructor. |
> | `priority` | `int` | Task priority; higher numbers run first under priority scheduling. |
> | `period` | `int` or `None` | Stored period in **microseconds**, even though the constructor and `set_period()` accept milliseconds. `None` identifies a triggered task. |
> | `go_flag` | `bool` | `True` when the task has been marked ready. Application code should normally call `go()` instead of writing this flag directly. |

> [!warning]
> Set the final task priority before calling `task_list.append(task)`. Changing `task.priority` afterward does not automatically reorganize the task list.

---

### `Task.schedule()`

```python
task.schedule()
```

Run one iteration of the task generator if the task is ready. This method is normally called by a `TaskList` scheduler rather than by application code.

**Returns:** `True` if the generator was advanced, or `False` if the task was not ready.

---

### `Task.ready()`

```python
task.ready()
```

Check whether the task is ready to run.

- For a periodic task, `ready()` compares the current time with the task's next scheduled run time and marks the task ready when its period has elapsed.
- For a triggered task, readiness depends on whether `go()` has set `go_flag`.

**Returns:** `True` when the task is ready; otherwise, `False`.

Application code rarely needs to call `ready()` directly because `schedule()` performs this check.

---

### `Task.set_period()`

```python
task.set_period(new_period)
```

Change the task's scheduling mode or the time between its runs. Changing a periodic task's period starts the new interval from the time of this call.

> [!table]
> Parameter accepted by `set_period()`.
>
> | Parameter | Type | Description |
> | --- | --- | --- |
> | `new_period` | number or `None` | New period in milliseconds, or `None` to make the task trigger-based. |

**Returns:** `None`.

---

### `Task.reset_profile()`

```python
task.reset_profile()
```

Clear the run count and all collected task-duration and scheduling-latency statistics. This method may be called whether or not profiling is enabled.

**Returns:** `None`.

---

### `Task.go()`

```python
task.go()
```

Mark the task as ready. This method is primarily used with a task whose `period` is `None`, and it may be called from another task or from an interrupt service routine.

**Returns:** `None`.

`go()` records a trigger time when profiling is enabled so that the scheduler can measure how long the triggered task waited before running.

> [!note]
> Readiness is stored as one Boolean flag, not as a count. Multiple calls to `go()` before the task runs still request only one run.

#### Triggered-task example

```python
print_task = cotask.Task(user_interface.run(), name="Print",
                         priority=0, period=None, profile=True)
cotask.task_list.append(print_task)

# Another task or an interrupt can request one run.
print_task.go()
```

---

### `Task.profile()`

```python
task.profile()
```

Return the profiling values for one task as a tuple. This method supplies data to `TaskList.profile()`; most application code will find the formatted task-list report easier to use.

For a periodic task, the returned fields are:

```text
(name, priority, period_ms, runs,
 duration_mean_ms, duration_max_ms, duration_stdev_ms,
 latency_mean_ms, latency_max_ms, latency_stdev_ms)
```

For a triggered task, `period_ms` is omitted:

```text
(name, priority, runs,
 duration_mean_ms, duration_max_ms, duration_stdev_ms,
 latency_mean_ms, latency_max_ms, latency_stdev_ms)
```

All duration and latency values are reported in milliseconds.

> [!note]
> The run counter includes every profiled run. For periodic tasks, the current implementation omits the first two runs from the duration statistics so that startup behavior does not affect those statistics.

---

### Diagnostic representation

Calling `repr(task)` or entering a task name at the MicroPython REPL produces a compact description:

```python
Task(name='Task 1', priority=1, period=500.0, profile=True)
```

The period shown in this representation is in milliseconds. A triggered task displays `period=None`.

## `TaskList`

```python
class cotask.TaskList()
```

Store and schedule a group of `Task` objects. Most applications should use the module-level `cotask.task_list` object rather than create another `TaskList`.

Tasks are grouped by priority when they are appended. Tasks at the same priority are considered in round-robin order.

### Supporting public attribute

`TaskList.pri_list` stores the scheduler's priority groups. Each group contains a priority value, a round-robin index, and its tasks. Application code should normally use `append()` and the scheduler methods instead of editing this structure directly.

---

### `TaskList.append()`

```python
task_list.append(task)
```

Add a `Task` object to the list and place it in the correct priority group.

> [!table]
> Parameter accepted by `append()`.
>
> | Parameter | Type | Description |
> | --- | --- | --- |
> | `task` | `Task` | Task to add to the scheduler. |

**Returns:** `None`.

---

### `TaskList.rr_sched()`

```python
task_list.rr_sched()
```

Give every task in the list one chance to run. Higher-priority groups are visited first, but priority does not prevent a lower-priority task from being checked during the same call.

**Returns:** `None`.

Call this method repeatedly from the application's main loop when round-robin scheduling is desired:

```python
while True:
    cotask.task_list.rr_sched()
```

---

### `TaskList.pri_sched()`

```python
task_list.pri_sched()
```

Find the highest-priority task that is ready and advance its generator once. Tasks within the same priority group are considered in round-robin order. If no task is ready, the method returns without running a task.

**Returns:** `None`.

Call this method continuously from the application's main loop:

```python
while True:
    cotask.task_list.pri_sched()
```

> [!insight]
> One call to `pri_sched()` runs at most one task. Repeating the call as quickly as possible allows the scheduler to respond when tasks become ready.

---

### `TaskList.profile()`

```python
task_list.profile()
```

Create a formatted text table containing profiling results for every task in the list.

**Returns:** A `str` containing the report. The method does not print the report by itself.

```python
print(cotask.task_list.profile())
```

The report contains the following measurements:

> [!table]
> Columns in the scheduler profiling report.
>
> | Column | Meaning |
> | --- | --- |
> | `TASK NAME` | Task name, limited to 11 displayed characters. |
> | `PRI` | Task priority. |
> | `PERIOD (ms)` | Periodic task interval in milliseconds, or `-` for a triggered task. |
> | `RUNS` | Number of times the task generator has been advanced while profiling. |
> | `DURATION AVG/MAX/STDEV` | Statistics describing how long one task run takes. |
> | `LATENCY AVG/MAX/STDEV` | Statistics describing how late a periodic task ran or how long a triggered task waited after `go()`. |

All duration and latency measurements in this report are in milliseconds.

When profiling is disabled for a task, its run counter and collected statistics remain at their reset values.

## `task_list`

```python
cotask.task_list
```

The shared `TaskList` object created when the module is imported. A typical application appends each task during setup and then repeatedly calls one scheduler method:

```python
cotask.task_list.append(motor_task)
cotask.task_list.append(sensor_task)

while True:
    cotask.task_list.pri_sched()
```

## Attribution and license

Source: [`cotask.py`](https://github.com/charlierefvem/micropython/blob/fd90403415b3f8962d778ab96c0ff5427c5e1773/ports/stm32/boards/NUCLEO_L476RG/modules/cotask.py) in the `charlierefvem/micropython` repository.

Original work:

- Copyright © 2017–2023 JR Ridgely
- Released under the GNU General Public License, version 3.0.

Modifications:

- Copyright © 2026 Charlie Refvem
- Modified for Cal Poly Mechatronics coursework.
- Major changes include scheduler profiling, trigger-based tasks, revised task ownership conventions, and MicroPython-focused memory optimizations.

This software is intended for educational use, but its use is not limited thereto.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, version 3.0.

This program is distributed in the hope that it will be useful, but **without any warranty**; without even the implied warranty of **merchantability** or **fitness for a particular purpose**. See the GNU General Public License for more details.
