---
title: Lecture 4 - Finite State Machines and State Transition Diagrams
type: lecture
topics:
- Embedded Systems
- Cooperative Multitasking
- Finite State Machines
- Scheduling
tags:
- me405
- fsm
- multitasking
---

# Motivation

Embedded firmware must be deterministic and robust. Throughout this course we will build firmware as a collection of cooperative tasks implemented as finite state machines (FSMs).

> **Candidate static notes**
>
> -   \[\[Finite State Machine\]\]
> -   \[\[State Transition Diagram\]\]
> -   \[\[Cooperative Multitasking\]\]
> -   \[\[Round Robin Scheduling\]\]
> -   \[[Priority Scheduling](#priority-scheduling)\]

## Embedded Multitasking

In this course, *embedded* refers to firmware executing on hardware permanently or semi-permanently embedded within a product, almost always a microcontroller.

Python (MicroPython) is used in ME 405 because it provides a productive learning environment while teaching concepts that transfer directly to languages such as C++ and Rust.

On a single-core microcontroller only one task executes at any instant. Multitasking is achieved by switching between tasks.

Two common approaches are:

-   **Preemptive multitasking**
-   **Cooperative multitasking**

### Preemptive multitasking

Not covered in ME 405.

Characteristics include:

-   Typically requires an RTOS.
-   Tasks are scheduled according to priority.
-   Higher-priority tasks may interrupt lower-priority tasks.
-   Shared hardware resources become more difficult to manage.

------------------------------------------------------------------------

# Tasks and Finite State Machines

In ME 405 every cooperative task is designed as a finite state machine.

## Definitions

**Task**

A portion of code that executes asynchronously with respect to other tasks.

**State**
![A state represented by an ellipse. In the center is the label S0 followed by the state name INIT.](state_ellipse.png)

A mutually exclusive operating mode of a task.

Only one state is active at a time.

**Transition**
![A transition is labeled with its triggering condition and optional action.](images/state_transition.png)
A change from one state to another.

Every transition has:

-   a trigger or condition,
-   an optional transition action,
-   a destination state.

Transitions leaving a common state must be mutually exclusive so that exactly one transition is selected.

Finite state machines can be viewed as a discrete abstraction of the broader class of state-determined systems encountered in dynamics.

------------------------------------------------------------------------

# State Transition Diagrams

A state transition diagram is the graphical representation of a finite state machine.

![State transition diagram showing states drawn as ellipses connected by directed transitions. Each transition is labeled with its triggering condition and optional action, and the diagram includes a designated start transition.](images/state_transition_start.png)

General conventions:

-   States are drawn as ellipses.
-   Transitions are directed arrows.
-   Every FSM has an explicit start transition.
-   Transition labels contain the condition and, optionally, the
    transition action.

Stateflow (Simulink) is an example of software capable of implementing programs directly from state transition diagrams.

------------------------------------------------------------------------

# States versus Transitions

One of the most common design questions is deciding whether behavior belongs inside a **state** or on a **transition**.

Remember:

> Transitions are effectively instantaneous.

Consequently, transition actions should be "one-shot" operations.

## Door Example

A simple model might contain only:

-   Open
-   Closed

with *Opening* and *Closing* represented as transition actions.

A higher-fidelity model might instead define four states:

-   Open
-   Opening
-   Closing
-   Closed

The appropriate level of detail depends on the goals of the design.

Self-transitions are optional. Include them when they improve clarity; omit them when they merely clutter the diagram.

------------------------------------------------------------------------

# Cooperative Round-Robin Scheduling

Round-robin scheduling executes each task sequentially in a repeating pattern.

![Three timing diagrams comparing round-robin scheduling. The diagrams show how task periods and execution durations affect processor utilization and demonstrate how long-running tasks delay later executions.](images/round_robin_scheduling.png)

The lecture compares three cases with different task periods and execution times to illustrate how timing margin decreases as execution time increases.

------------------------------------------------------------------------

# Priority Scheduling

Priority scheduling executes whichever ready task has the highest assigned priority.

![Three timing diagrams comparing priority scheduling. The diagrams show higher-priority tasks executing with lower latency while lower-priority tasks are deferred when processor time is limited.](images/priority_scheduling.png)

Compared with round-robin scheduling, priority scheduling generally provides improved latency for critical tasks but introduces additional scheduling complexity.

------------------------------------------------------------------------

# Programming Example

![A state transition diagram with three states. State 0, the initialization state, always transitions to state 1, the run state. State 1 always transitions to state 2, the run thrice state. State 2 transitions to state 1 after it self-transitions enough times to increment count to 2.](images/state_transition_diagram.png)

The lecture concludes with a finite state machine programming example.

Source:
``` python
import time

S0_INIT = 0
S1_RUN = 1
S2_RUN_THRICE = 2

def main():
    # A variable to indicate what state the FSM
    # is about to run
    state = 0
    
    # A counter variable used to track runs through
    # state 2
    count = 0
    
    while(True):
        try:
            # Implement FSM inside while loop
            if (state == S0_INIT):
                # Run state zero code
                print("The state is ", state)
                state = S1_RUN
                
            elif (state == S1_RUN):
                # Run state one code
                print("The state is ", state)
                state = S2_RUN_THRICE
                
            elif (state == S2_RUN_THRICE):
                # Run state zero code
                print("The state is ", state)
                if (count == 2):
                    state = S1_RUN
                    count = 0
                else:
                    count += 1 # Increment count
                
            else:
                # If the state isnt 0, 1, or 2 we have an
                # invalid state
                raise ValueError('Invalid state')
            
            # Sleeping (delaying) for 1/2 second to slow down
            # the printing
            time.sleep(0.5)
        
        # Trying to catch the "Ctrl-C" keystroke to break out
        # of the program cleanly
        except KeyboardInterrupt:
            break
    
    # Once the program is over, do any sort of cleanup as needed
    print('Program terminated')

# THe following block prevents main() from running when the file
# is imported instead of run as a main program
if __name__ == '__main__':
    main()
```

**Run in your browser:**
https://onlinegdb.com/xsR2ikth3


------------------------------------------------------------------------

# Summary

-   Embedded firmware must be deterministic.
-   ME 405 uses cooperative multitasking implemented with finite state machines.
-   State transition diagrams communicate FSM behavior visually.
-   Keep transition actions instantaneous.
-   Scheduling policy directly affects task latency and determinism.

# Related Topics
* [[lecture_03_lab_hardware_overview_and_toolchain|Lecture 3 - Lab Hardware Overview and Toolchain]]