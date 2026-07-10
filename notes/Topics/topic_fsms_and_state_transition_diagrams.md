---
title: Finite State Machines and State Transition Diagrams
type: topic
tags:
  - fsm
  - state
  - transition
source:
  course: ME405
  term: 2262
  lecture: 4
status: draft
---
## Motivation

Embedded firmware must be deterministic and robust. One way to promote determinism and robustness is to *design* firmware before implementing. Separating the design phase from the implementation phase, or even better iterating between the two, allows the designer to focus on each part with full attention and limited distraction.

One of the primary design tools you will use in ME 4305 is the finite state machine.

## Tasks and Finite State Machines

In ME 4305 every cooperative task should be designed as a finite state machine (FSM).

### Definitions

**Finite State Machine**: A mathematical model or abstraction of a deterministic system.
* A finite state machine can be viewed as a discrete abstraction of the broader class of state-determined systems encountered in dynamics and mathematics.

**Task**: A portion of code that executes concurrently and asynchronously with respect to other tasks. In ME 4305, most cooperative tasks are implemented as a FSM.

**State**: The operating mode or internal condition of a task.
* Only one state is active at a time; that is, all states in a common task must be mutually exclusive. In parlance, we say "a task is in a state".

**Transition**: A change from one state to another. Transitions are effectively instantaneous, or in programming vocabulary, transitions are *atomic*.
- Every transition has:
	-  A trigger or condition
	-  An optional transition action
	-  A destination state
- All transitions must be mutually exclusive; that is, there can be no ambiguity in which transition to follow. For exactly one transition to occur the must be no overlap in trigger or condition between transitions out of a common state.

Programmers should remember that the finite state machine is not the code itself, but rather an abstraction or concept implemented by the code. To aid in development of the finite state machine designers use state transition diagrams. Again, the diagram is not the FSM, but a depiction or projection of the FSM to make the abstract concept concretely defined. It is then the programmer's choice how the code implements the logic depicted on the state transition diagram.

### State Transition Diagrams

A state transition diagram is a visual depiction of the abstract finite state machine. Each state should be drawn as an ellipse with the state's name and/or number labelling the ellipse.

![A state represented by an ellipse. In the center is the label S0 followed by the state name INIT.](images/multitasking/state_ellipse.svg)

Transitions between states are shown as arrows. Each arrow must begin and end in exactly one state; branching arrows are not allowed. The transitions are labelled by placing the condition and action inside square brackets separated by a forward slash.
* In most cases the transition should include one condition and one action: `[CONDITION / ACTION]`.
* If there is no action then the transition label should only show the condition: `[CONDITION]`. 
* If multiple conditions trigger the same transition or multiple  the condition may be written as a logical expression: `[CONDITION_1 or CONDITION_2 / ACTION]`
* If multiple conditions must simultaneously apply to trigger the transition a logical expression can also be used: `[CONDITION_1 and CONDITION_2 / ACTION]`.
* If multiple actions are to be performed they may be separated by semicolons: `[CONDITION / ACTION_1; ACTION_2]`.
Each of these cases may be mixed and matched as long as the conventions are followed.

![A transition is labeled with its triggering condition and optional action.](images/multitasking/state_transition.svg)

Some students incorrectly assume that the conditions and actions must be valid executable lines of code. While in some cases this may be appealing, it is not a requirement that your state transition diagram matches the exact syntax used to implement the logic encoded by the diagram. For example, it is common to use the condition `ALWAYS` to represent immediate transitions that occur every time the state runs.

A state transition diagram is the graphical representation of a finite state machine and is made by stitching together many states with transitions. Ever state transition diagram must have an explicit start transition, shown with an unlabeled arrow going from a `START` box. Without a start condition it can be ambiguous which state a task is in immediately upon startup.

![State transition diagram showing states drawn as ellipses connected by directed transitions. Each transition is labeled with its triggering condition and optional action, and the diagram includes a designated start transition.](images/multitasking/start_transition.svg)

In ME4305 you will likely use state transition diagrams to design your finite state machines and therefore the architecture for your Micropython code. However, tools do exist that let you program graphically by designing the finite state machine directly. Stateflow (part of Simulink) is an example of software capable of implementing programs directly from state transition diagrams.

### States versus Transitions

One of the most common design questions is deciding whether behavior belongs inside a **state** or on a **transition**.

Recall:

> Transitions are effectively instantaneous.

Consequently, transition actions should be "one-shot" operations. In other words, the system should not spend any significant amount of time on the transition. The definition of significant greatly depends on the timing characteristics of the system being modeled or controlled.

#### Door Example
A simple model might contain only:
* Open
* Closed

with *Opening* and *Closing* represented as transition actions.

A higher-fidelity model might instead define four states:
* Open
* Opening
* Closing
* Closed

The appropriate level of detail depends on the goals of the design.

### Self-transitions

Self-transitions are transitions from one state back to the same state and are optional to show on the diagram. It is implied by convention that, when no state transitions have applicable conditions, the state does not transition. Include self-transitions only when they improve clarity; omit them when they merely clutter the diagram.

------------------------------------------------------------------------

## Programming Example

The following example shows both the transition diagram and a Python implementation for an extremely simple FSM with three states.

![A state transition diagram with three states. State 0, the initialization state, always transitions to state 1, the run state. State 1 always transitions to state 2, the run thrice state. State 2 transitions to state 1 after it self-transitions enough times to increment count to 2.](images/multitasking/example_transition_diagram.svg)

The following Python script implements the preceding example in executable code.

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
                count = 0
                state = S2_RUN_THRICE
                
            elif (state == S2_RUN_THRICE):
                # Run state zero code
                print("The state is ", state)
                if (count == 2):
                    state = S1_RUN
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

**Run in your browser:** https://onlinegdb.com/xsR2ikth3


------------------------------------------------------------------------

## Summary

-   Embedded firmware must be deterministic.
-   ME 4305 uses finite state machines to design and/or implement tasks.
-   State transition diagrams communicate FSM behavior visually.
-   Keep transition actions instantaneous.

## See Also
* [[topic_task_diagrams|Task Diagrams]]
* [[topic_priority_schedulers|Cooperative Multitasking and the Scheduler]]
* [[topic_scheduling_tasks|Using the Scheduler]]