---
title: Cooperative Multitasking and the Scheduler
type: topic
tags:
  - scheduler
  - multitasking
source:
  course: ME405
  term: 2262
  lecture: 4
status: draft
---

# Motivation

Embedded firmware must be deterministic and robust. Throughout this course we will build firmware as a collection of cooperative tasks dispatched by a priority scheduler.

> **Candidate static notes**
> -  [[Scheduler Profiler]]

## Embedded Multitasking

In this course, *embedded* refers to firmware executing on hardware permanently or semi-permanently embedded within a product, almost always a microcontroller.

Python (MicroPython) is used in ME 405 because it provides a productive learning environment while teaching concepts that transfer to languages such as C++ and Rust.

On a single-core microcontroller only one task executes at any instant. Multitasking is achieved by switching between tasks.

------------------------------------------------------------------------

# Cooperative Scheduling

In previous courses you may have implemented a task based system without any additional tools to help with task timing. For example, a simple way to implement cooperative multitasking is to run each task within an unbounded loop with each task internally handling its own timing.

A more structured approach is to run a **Scheduler** - an algorithm that chooses when and in what order to run tasks. By scheduling tasks, the designer is able to specify a desired period or frequency for each task that the scheduler attempts to maintain during runtime.

Schedulers come in two main varieties
* Preemptive schedulers allow tasks to interrupt (preempt) one another and are often implemented as part of a Real Time Operating System (RTOS).
* Cooperative schedulers do not allow preemption, therefore each task is required to run quickly in order to cooperate with other tasks' timing requirements.

In this course you will utilize a cooperative scheduler, specifically one that implements **Priority** scheduling. The simple scheduling method mentioned above using an unbounded loop may be called a **Round-Robin** scheduling method as each task may be dispatched to run on every cycle. 

The logic flow of a round-robin scheduler is quite simple:
```
ALGORITHM RRScheduler(task_list)
    WHILE TRUE DO
        FOR EACH task IN task_list DO
            IF task.is_ready THEN
                Execute(task)
            ENDIF
        ENDFOR
    ENDWHILE
ENDALGORITHM
```

A priority scheduler extends this algorithm by using priority-based arbitration to determine which task should run first when more than one task needs to run at a time. That is, when two tasks need to be dispatched the one with the higher priority is dispatched first. If two tasks of the same priority both need to run the scheduler preserves the order in which the tasks were added to the scheduler's task list.

The logic flow of a cooperative priority scheduler is only a small amount more complex than for the round-robin scheduler:
```
FUNCTION DispatchNextTask(priority_list)
    FOR EACH priority IN priority_list DO
        FOR EACH task IN priority.task_list DO
            IF task.is_ready THEN
                Execute(task)
                RETURN
            ENDIF
        ENDFOR
    ENDFOR
ENDFUNCTION

ALGORITHM PriScheduler(priority_list)
    WHILE TRUE DO
        DispatchNextTask(priority_list)
    ENDWHILE
ENDALGORITHM
```
A *critical* detail in the implementation of the priority scheduler is that, unlike the round-robin scheduler, each pass only dispatches at most one task, whichever is of the highest priority, earliest in the task list, and needs to run.

If all tasks are assigned the same priority then the priority scheduler essentially behaves as a round-robin scheduler. Consequently, round-robin scheduling may be viewed as the degenerate case of priority scheduling. 

## Scheduling Examples

In this section you will see examples of cooperative round-robin and cooperative priority scheduling shown as informal timing diagrams. These timing diagrams are conceptual illustrations intended to demonstrate scheduler decision-making. They are not exact execution traces. Scheduler overhead, queue maintenance, and other implementation details have been simplified so that the scheduling policy can be understood more clearly.

### Round-Robin Scheduling

Round-robin scheduling dispatches each task sequentially in a repeating pattern.

![Three timing diagrams comparing round-robin scheduling. The diagrams show how task periods and execution durations affect processor utilization and demonstrate how long-running tasks delay later executions.|700](images/round_robin_scheduling.svg)

The lecture compares three cases with different task periods and execution times to illustrate how timing margin decreases as execution time increases.

### Priority Scheduling

Priority scheduling dispatches whichever ready task has the highest assigned priority.

![Three timing diagrams comparing priority scheduling. The diagrams show higher-priority tasks executing with lower latency while lower-priority tasks are deferred when processor time is limited.|700](images/priority_scheduling.svg)

Compared with round-robin scheduling, priority scheduling generally provides improved latency for critical tasks but introduces additional scheduling complexity.

## Processor Utilization

One clear lesson that can be learned from the preceding timing diagrams is that attempting to schedule tasks too quickly can overrun the scheduler. A processor only has so many available computation cycles and once those are used tasks are either skipped or run late.

Something not obvious on the timing diagram is that overrun can occur well below 100% utilization. Utilization can loosely be defined as the percentage of processor computation time claimed by a single task. To quantitatively assess the utilization compute
$$
U_{task} = \frac{D_{task}}{T_{task}}.
$$
That is, the utilization for one task can be expressed as the ratio of the duration of the task, $D_{task}$, (either average or worst-case) to the period of the task, $T_{task}$. The total utilization for all periodic tasks is therefore
$$
U_{periodic} = \sum_{n=1}^N U_n = \sum_{n=1}^N \frac{D_n}{T_n}
$$
where $N$ is the total number of tasks and $n$ is an index or ID for a given task.

This formula only accounts for periodic tasks. Most tasks *are* periodic; however, two kinds of aperiodic task can be handled by the scheduler and both utilize the processor even though the previous formula doesn't apply.
* Idle tasks are implemented by setting both the period and priority for a task to zero. Idle tasks are designed to run as many times as possible in between other tasks filling all free time and maximizing utilization.
* Triggered tasks have no defined period and are triggered by events that occur in other tasks.

As mentioned briefly above, the maximum practical utilization for any system will be less than 100%, as the true utilization for the system includes many additional factors outside the set of periodic tasks. For a real system the utilization is more accurately described by
$$
U_{system} = U_{periodic} + U_{idle} + U_{triggered} + U_{ISR} + U_{scheduler}
$$
where $U_{ISR}$ accounts for utilization by interrupt service routines and $U_{scheduler}$ accounts for utilization by the scheduler itself.

Depending on the number of tasks scheduled and their individual periods the scheduler utilization may either be inconsequential or very significant.

## Profiling
Due to the practical difficulties in accurately computing the total system utilization it is greatly preferred to *measure* the utilization rather than to simply approximate its value. Most schedulers come equipped with a profiler that can measure the task duration and latency and in some cases perform basic statistical analysis on each.

A good designer should always monitor the output of the profiler and verify that there is overhead or safety margin in the total system utilization.

------------------------------------------------------------------------

# Summary

-   Embedded firmware must be deterministic.
-   ME 4305 uses cooperative multitasking.
-   Tasks are dispatched by the scheduler.
-   Scheduling policy directly affects task latency and determinism.