---
title: Scheduling Tasks
type: lecture
tags:
  - tasks
  - multitasking
  - scheduler
  - generators
source:
  course: ME405
  term: 2262
  lecture: 7
status: draft
---

# Motivation

This lecture introduces the software architecture used for the remainder of the course. Tasks are organized into layers, communicate through shares and queues, and are typically implemented as generator-based finite state machines.

# Recommended Code Organization

This lecture provides many of the final building blocks that you will use in lab the remainder of the quarter. To stay organized you should follow the file hierarchy below, which outlines the best way to structure your code fore maximum reusability. Notice that each layer is an abstraction of the layer beneath it.

| Level  | Example Files                                           | Purpose                                             |
| ------ | ------------------------------------------------------- | --------------------------------------------------- |
| High   | `main.py`                                               | Create objects, create tasks, run scheduler.        |
| Middle | `task1.py`<br>`task2.py`<br>`task3.py`                  | Application logic using generators and shared data. |
| Low    | `pid.py`<br>`motor.py`<br>`encoder.py`<br>`irsensor.py` | Hardware drivers and reusable libraries.            |

You will also take advantage of several libraries as your progress in lab, the most important being the scheduler, implemented in `cotask.py`.

# Better Tasks with Generators

Generator functions are ideal for cooperative multitasking as they are built to yield CPU time to run other code cooperatively.
* By default, you should plan each task to be implemented as a finite state machine. Simple tasks can be implemented using a FSM with only one state if needed.
* Generators should run at most one iteration of the FSM and then `yield`. In future lectures we may discuss techniques for subdividing a state into multiple cooperative slices.
* A task can be built from a single generator function but it is often useful to use a generator function implemented as a method so that the generator may call other methods with shared state.

## Example 1

This preliminary example shows how to write a FSM based task using a generator. The example is meant to be run using CPython, so it does not utilize the scheduler, which only runs in MicroPython.

The task implements the same trivial finite state machine covered in a previous lecture, reproduced below.

![A state transition diagram with three states. State 0, the initialization state, always transitions to state 1, the run state. State 1 always transitions to state 2, the run thrice state. State 2 transitions to state 1 after it self-transitions enough times to increment count to 2.](images/state_transition_diagram.png)

``` python
import time

# The states of the FSM
S0_INIT = 0
S1_RUN = 1
S2_RUN_THRICE = 2

# Task implemented as its own generator function
def task_gen_fcn():
    
    # A variable to indicate what state the FSM is about to run
    state = S0_INIT
    
    # A counter variable used to track runs through state 2
    count = 0
    
    # Attempt to run infinite iterations of the state machine
    while True:
    
        if (state == S0_INIT):
            # Run state zero code
            print("The state is ", state)
            state = S1_RUN
            
        elif (state == S1_RUN):
            # Run state one code
            print("The state is ", state)
            state = S2_RUN_THRICE
            
        elif (state == S2_RUN_THRICE):
            # Run state two code
            print("The state is ", state)
            if (count == 2):
                state = S1_RUN
                count = 0
            else:
                count += 1 # Increment count
            
        else:
            # If the state isnt 0, 1, or 2 we have an invalid state
            raise ValueError('Invalid state')
    
        # Yield the value of the next state to run
        yield state

def main():
    # Create object of the generator function
    task_obj = task_gen_fcn()
    
    while(True):
        try:
            # Try to run one iteration of the task
            # Note: this won't be needed when using the scheduler. Instead,
            # cotask.task_list.pri_sched() will run the task for you.
            next(task_obj)
            
            # Sleeping (delaying) for 1/2 second slows down the printing. Remove
            # this when porting to MicroPython
            time.sleep(0.5)
        
        # Catch a "Ctrl-C" keystroke to break out of the program cleanly
        except KeyboardInterrupt:
            # In lab you should do cleanup here, like turning motors off
            print('Program terminated')
            break

if __name__ == '__main__':
    main()
```

**Run in your browser**: https://onlinegdb.com/111Py9AVi

## Example 2

This next example extends Example 1 to create multiple tasks from the same generator function.

``` python
import time

S0_INIT = 0
S1_RUN = 1
S2_RUN_THRICE = 2

def task(taskName):
    
    # A variable to indicate what state the FSM is about to run
    state = S0_INIT
    
    # A counter variable used to track runs through state 2
    count = 0
    
    while True:
        # Implement FSM inside while loop
        if (state == S0_INIT):
            # Run state zero code
            print(f"{taskName}: ", end="")
            print("The state is ", state)
            state = S1_RUN
            
        elif (state == S1_RUN):
            # Run state one code
            print(f"{taskName}: ", end="")
            print("The state is ", state)
            state = S2_RUN_THRICE
            
        elif (state == S2_RUN_THRICE):
            # Run state zero code
            print(f"{taskName}: ", end="")
            print("The state is ", state)
            if (count == 2):
                state = S1_RUN
                count = 0
            else:
                count += 1 # Increment count
            
        else:
            # If the state isnt 0, 1, or 2 we have an invalid state
            raise ValueError('Invalid state')
        
        yield state

def main():
    # Create a genator object to use for running the task 1
    task1Obj = task("Task 1")
    # Create a second genator object to use for running the task 2
    task2Obj = task("Task 2")
    
    try:
        while(True):
            # The following code manually calls next() to invoke states in the
            # task. When running on hardware, you will instead add the generator 
            # to the task list to be run by the scheduler
            next(task1Obj)
            next(task2Obj)
            
            # Sleeping (delaying) for 1/2 second slows down the printing. Remove
            # this when porting to MicroPython
            time.sleep(0.5)
        
    # Catch a "Ctrl-C" keystroke to break out of the program cleanly
    except KeyboardInterrupt:
        # In lab you should do cleanup here, like turning motors off
        print('Program terminated')

if __name__ == '__main__':
    main()
```

**Run in your browser**: https://onlinegdb.com/mCsBP54UZ

## Example 3

In this third example the code is refactored into two files. In `taskexample.py` a class is defined that implements the finite state machine and in `main.py` an object of the class is instantiated and the task is run.

`taskexample.py`
``` python
# The states of the FSM
S0_INIT = 0
S1_RUN = 1
S2_RUN_THRICE = 2

# Task implemented as a method of a task class
class TaskExample:
        
    def __init__(self, task_label):
        # A variable to indicate what state the FSM
        # is about to run
        self.state = S0_INIT
        
        # A counter variable used to track runs through
        # state 2
        self.count = 0
        
        # A label for the task to distinguish it's print statements
        self.task_label = task_label
        
    def run(self): 
        # Attempt to run infinite iterations of the state machine
        while True:
            # Implement FSM inside while loop
            if (self.state == self.S0_INIT):
                # Run state zero code
                print(self.task_label, ":")
                print("\tThe state is ", self.state)
                self.state = self.S1_RUN
                
            elif (self.state == self.S1_RUN):
                # Run state one code
                print(self.task_label, ":")
                print("\tThe state is ", self.state)
                self.state = self.S2_RUN_THRICE
                
            elif (self.state == self.S2_RUN_THRICE):
                # Run state two code
                print(self.task_label, ":")
                print("\tThe state is ", self.state)
                if (self.count == 2):
                    self.state = self.S1_RUN
                    self.count = 0
                else:
                    self.count += 1
                
            else:
                # If the state isnt 0, 1, or 2 we have an invalid state
                raise ValueError('Invalid state')
        
            # Yield the value of the next state to run
            yield self.state
```

`main.py`
``` python
import time
import taskexample

def main():
    
    # Create object of the task class first
    task_class_obj_1 = taskexample.TaskExample("TASK 1")
    task_class_obj_2 = taskexample.TaskExample("TASK 2")
    
    # Then create the generator objects
    task_gen_obj_1 = task_class_obj_1.run()
    task_gen_obj_2 = task_class_obj_2.run()
    
    try:
        while(True):
            # The following code manually calls next() to invoke states in the
            # task. When running on hardware, you will instead add the generator 
            # to the task list to be run by the scheduler
            next(task_gen_obj_1)
            next(task_gen_obj_2)
            
            # Sleeping (delaying) for 1/2 second slows down the printing. Remove
            # this when porting to MicroPython
            time.sleep(0.5)
        
    # Catch a "Ctrl-C" keystroke to break out of the program cleanly
    except KeyboardInterrupt:
        # In lab you should do cleanup here, like turning motors off
        print('Program terminated')

if __name__ == '__main__':
    main()
```

**Run in your browser**: https://onlinegdb.com/ZxapaDZ-D

## Example 4

In this final example you will see how the code changes when using the scheduler; that is, Example 3 is adapted here to run within the scheduler. Note that the implementation of the task class is unchanged for this example.

**Note**: The following code will only run in MicroPython.

`main.py`
``` python
import cotask
import taskexample

def main():
    # Create object of the task class
    t_example = taskexample.TaskExample("Example Task")
    
    # Add the task to the scheduler's task list so that it can be iterated.
    cotask.task_list.append(cotask.Task(t_example.run(), name="Example Task",
                                        priority=1, profile=True))
    
    try:
        while(True):
            # Run the scheduler which will call `next()` on each task when it
            # needs to be dispatched.
            cotask.task_list.pri_sched()
        
    # Catch a "Ctrl-C" keystroke to break out of the program cleanly
    except KeyboardInterrupt:
        # In lab you should do cleanup here, like turning motors off
        print('Program terminated')

if __name__ == '__main__':
    main()
```

## Summary

* Separate application logic from hardware drivers.
* Use task diagrams during design.
