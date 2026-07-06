---
title: Cooperative User Input and Output
type: topic
tags:
  - 
source:
  course: ME405
  term: 2262
  lecture: 7
status: draft
---

# Motivation

A core principle of cooperative multitasking is to avoid blocking code. Printing long strings and waiting for user input are both examples of blocking code, and must be handled appropriately.

Serial communication, and printing in general, should be implemented cooperatively within a single task run by the scheduler. Likewise, for character input, rather than blocking while waiting for characters, periodically check for new data and process complete commands when available.

# Cooperative Character I/O

## Printing Characters

Printing or writing to a serial port is relatively slow, so large blocks of output should often be subdivided and printed incrementally, either one line at a time or a few lines at a time. A rule of thumb that tends to work well in MicroPython is to print no more than ~512 characters in one batch, as the outgoing serial buffer can usually keep up with 512 byte chunks.

### Example 1

In this example it will be assumed that data has been collected in a pair of lists, `times`, and `values`, that are of equal length. The contents of the example would likely be placed within a state of a "user interface" task.

```python
...
elif state == DUMP_DATA_STATE:
    if len(times) > 0:
        t = times.pop(0)
        v = values.pop(0)
        ser.write(f"{t},{v}\r\n")
    else:
        state = WAIT_FOR_CMD_STATE
...
yield
```

## Reading Characters

To cooperatively read characters, try to observe the following workflow:
1. Check whether data is available.
2. Read available characters.
3. Accumulate characters into a buffer.
4. Detect end-of-line.
5. Process the completed command.
6. Return immediately to allow other scheduled tasks to execute.

### Example 2

In this example a task will wait for a single character input command (cooperatively) and only retrieve the command once a character is available, as indicated by the `.any()` method belonging to the serial port.

```python
...
elif state == WAIT_FOR_CMD_STATE:
    # Check if at least one character is pending
    if ser.any():
        # Read the character when ready and decode into a string
        char_in = ser.read(1).decode()
        # Process each character appropriately after a state transition
        if char_in == "h":
            state = PRINT_HELP_MENU
        elif char_in in {"L", "l}:
            state = LEFT_MOTOR_PROMPT
        elif ...
...
yield
```

### Example 3

The preceding example shows a simple way to read single character commands in a user interface, however you will almost certainly also need to take in multicharacter entry for numeric values in your UI. The following algorithm, presented as a flowchart, is one approach that will run cooperatively. The algorithm accumulates characters until a carriage return or newline is received and then processes the batch of characters.

A flowchart has different rules compared to a FSM: the entirety of the flowchart would exist within a single state of a FSM, running one full pass of the flowchart per iteration of the FSM. The algorithm could also be ported to a reusable coroutine if it is to be used in multiple FSM states.

It will be left as an exercise for the reader to convert the flowchart into working firmware.

![A detailed flowchart outlining an algorithm for multicharacter numeric data entry.](images/coop_io/multichar_flowchart.svg)

**Insight**: later in the course, this style of user interface will be used to automate a tuning and data collection interface that will eventually interact with a Python script running on a PC.
# Summary

Serial port interaction can be slow and therefore result in blocking code. Efforts should be made to write cooperative code instead of blocking code by polling for input before reading and by subdividing large blocks of output into multiple smaller chunks before printing.

# See Also:
* [[reference_formatting_strings|Formatting Strings]]