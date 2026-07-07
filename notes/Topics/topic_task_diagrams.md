---
title: Task Diagrams
type: topic
tags:
  - tasks
  - multitasking
source:
  course: ME405
  term: 2262
  lecture: 9
status: draft
---

## Motivation

In many past lectures we have motivated deliberately splitting the design phase from the implementation phase. In this lecture you will learn a powerful design and documentation tool called a Task Diagram that should be used as a high level (zoomed out) plan for your entire application.

## Task Diagrams

Task diagrams show task periods, priorities, and perhaps most importantly, the shared data exchanged by tasks. Shared data can come in many forms
* Standard Python Objects
* Thread-safe `share` objects
    * Hold one value.
    * Similar to variables.
    * Can safely share numeric data between ISRs and tasks.
* Thread-safe `queue` objects
    * FIFO buffers.
    * Fixed maximum size.
    * Stream data without loss.
    * Can safely share numeric data between ISRs and tasks.

### Diagram Syntax

A task diagram is a graph that represents individual tasks with nodes and data shared between tasks as edges.

**Tasks:**
* Tasks are drawn as a square or rectangle with the task name, period, and priority plainly indicated.
* The location of the tasks on the diagram do not convey any information; only the connectivity of the diagram is meaningful.
* All tasks should be shown on the diagram even if some tasks run within interrupt callbacks. This information can be annotated on each task in the diagram.
**Shared Data**:
* The edges (arrows) represent communication between tasks. The annotations attached to each edge describe the shared data used for that communication.
* On each arrow the following annotations are drawn.
    * The name of the variable or object being shared.
    * A type such as `bool`, `int`, `float`, etc.
    * If the shared data is not a primitive type, but instead a container of primitive types, such as a `list`, `array`, `share`, or `queue`, a more detailed annotation. This annotation should describe the storage container type, the stored object type, and the number of objects stored in the container. For example, to annotate a shared `tuple` of three `float` values, you would use `tuple<float[3]>`.
* The direction of each arrow shows the flow of information from producer (at the tail) to consumer (at the tip).
* Most transitions use single-ended arrows (one producer and one consumer) but double-ended arrows are permitted in the case of two tasks communicating back and forth (both are producers and consumers). 
* Arrows that split are permitted in rare circumstances with multiple consumers reading from one producer. The inverse case, of multiple producers, is not permitted on a task diagram; that is, you cannot have any merging arrows or arrows with multiple tails.
* Each piece of shared data should have a clear owner responsible for creating and maintaining it, even if other tasks may later read or modify its contents.

**Note**: while task diagrams are extremely valuable documentation and design tools, they provide abstraction like anything else. Therefore it is often useful to include a pair of tables along with your diagram that outline what each task does and what each piece of shared data represents.
### Example

In this example a task diagram is presented for a simple data collection scheme. The application is meant to wait for input from a button and then collection data from an ADC for 10 seconds at 100Hz before finally printing the data back to the user.

![Task diagram showing an ADC task, user interface task, and interrupt button task communicating through shares.](images/multitasking/task_diagram.svg)

The diagram depicts three tasks and three shared data. The tables below describe some of the design choices encoded in the task diagram.

| Task        | Period | Priority | Behavior                                                                                                                                                                                                                                                    |
| ----------- | ------ | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ADC Task    | 10ms   | 1        | The ADC Task is given the highest priority because it has strict latency requirements. By making it the highest priority it is less likely to run late and will have reduced jitter.                                                                        |
| Button Task | N.A.   | Inf.     | The Button Task is not a traditional scheduled task, but an ISR. Therefore the notion of period and priority do not make much sense. The ISR will run close to immediately after the button is pressed so it is both asynchronous and of infinite priority. |
| User Task   | 0ms    | 0        | The User Task is an example of an idle task. It's job is to fill up any unused time to print data as quickly as possible. That way it can run very quickly while not interfering with any other tasks. An application can only have one idle task.          |

| Shared Data   | Type     | Length | Purpose                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------- | -------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `button_flag` | `bool`   | 1      | The button flag is a boolean value used to communicate bidirectionally between the ADC Task and the Button Task ISR. When a button is pushed and the ISR runs, the Button Task will set the flag high to inform the ADC Task that a button was pressed. The Button Task will then ignore future button presses until the flag is lowered.<br><br>Once the ADC Task has finished collecting data it will lower the flag, informing the Button Task that it should resume looking for input.<br><br>The button flag must be a thread-safe object to prevent corruption from an ISR triggering while another task accesses the variable. |
| `time_values` | `uint32` | 1000   | To collect timestamps it is typical to use units of milliseconds or micro seconds. Depending on the units selected larger or smaller data types may be used. If microseconds are used and the time may span from zero to ten seconds, then a 32-bit integer is required to avoid overflow. Time values are typically positive, so an unsigned integer is most appropriate.                                                                                                                                                                                                                                                            |
| `adc_values`  | `uint16` | 1000   | A typical ADC has a resolution between 10 and 16 bits, with most being 12 bits. Any of these resolutions fit within an unsigned 16 bit integer.<br><br>As the ADC Task reads data it will fill up the buffer until it is full and then stop collecting data.<br><br>The User Task can check when the ADC Task has filled the buffer and start emptying it one item per iteration to remain cooperative.<br><br>Once the buffer is empty again the ADC Task may finally lower the button flag mentioned above to prep for the next cycle.                                                                                              |
**Bonus Insight**: Several of the shared data serve multiple purposes. The buffer used to send ADC also acts as a pair of flags, because the buffer provides information about whether it is full, empty, or somewhere in between.

## Summary
* Use task diagrams during design.
* Share standard Python objects when you don't need thread safety.
* If you need thread safety use `share` objects for single values and `queue` objects for FIFO communication.

## See Also:
* [[topic_fsms_and_state_transition_diagrams|Finite State Machines and State Transition Diagrams]]
* [[topic_priority_schedulers|Cooperative Multitasking and the Scheduler]]
* [[topic_scheduling_tasks|Using the Scheduler]]