---
title: Software Timing
type: topic
tags:
  - python
  - micropython
source:
  course: ME405
  term: 2262
  lecture: 8
status: draft
---

## Motivation

In ME 4305 we primarily focus on embedded firmware, which almost always means code running in real-time interacting with the world through sensors and actuators. Consequently, the code you write needs a way of keeping track of time accurately. Some of this is handled by the scheduler, but you will inevitably need to do some timekeeping on your own to timestamp data collection or interface with hardware.

## Using the `utime` Module

The MicroPython `utime` module aims to reproduce the core functionality of CPython's `time` module. Most of the useful timekeeping functions are provided in this library. The following six functions are ones that all students should be aware of.

| Function       | Purpose                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------- |
| `sleep_ms()`   | A blocking millisecond delay function.                                                            |
| `sleep_us()`   | A blocking microsecond delay function                                                             |
| `ticks_ms()`   | A running millisecond counter. Values must be used used differentially.                           |
| `ticks_us()`   | A running microsecond counter. Values must be used differentially.                                |
| `ticks_diff()` | A function to safely compute differentials between tick values from `ticks_ms()` or `ticks_us()`. |
| `ticks_add()`  | A function to safely add tick values from `ticks_ms()` or `ticks_us()`.                           |
The two sleep functions should be used with caution, as blocking code is generally not compatible with cooperative multitasking. Short delays in the range of microseconds to a couple of milliseconds *may* be OK during real-time operation, but should generally be avoided in favor of cooperative timekeeping as illustrated below.

Cooperative timekeeping utilizes the ticks functions which return time values in either milli- or microseconds. These functions return the amount of elapsed time since some unknown datum; consequently, the output of the functions cannot be used in an absolute sense, but only in a differential sense. Additionally, the internal counters used by the ticks functions may overflow so it is unsafe to do standard arithmetic on the output of the ticks functions. Instead, you must use either `ticks_diff()` for subtraction or `ticks_add()` for addition to avoid errors.

### Example 1

The following example shows how you might handle timing inside a task if you were not using a scheduler. Similar techniques can be used within states of scheduled tasks to measure time asynchronously with respect to the task period.

``` python
interval = 100_000 # time interval or period [us]
start = ticks_us() # timestamp of 1st run
deadline = start # first run deadline is the start time

while True:

    now = ticks_us() # present time [us]

    if ticks_diff(deadline, now) <= 0: # deadline elapsed
    
        # Run looping code here; can reference "start" and "now"
        # variables using ticks_diff() for timestamping actions
        # of code (like data collection)
        
        deadline = ticks_add(deadline, interval)# prep next deadline
```

Notice that the next deadline is computed from the previous deadline rather than the current time. This prevents the execution duration from adding to the interval, causing drift in the timing.

**Note**: if even you use the microsecond ticks function you should *not* expect microsecond precision with this timing method. For lower latency timing at the cost of additional overhead and additional coding restrictions you may consider timer callbacks instead.

## Summary
* The MicroPython `utime` module implements some of the features of CPython's `time` module.
* To delay, use the functions `sleep_ms()` and `sleep_us()`.
* Avoid long delays in cooperative tasks.
* Perform cooperative timekeeping using the ticks functions.
    * Get ticks with `ticks_ms()` or `ticks_us()`
    * Perform arithmetic with `ticks_diff()` and `ticks_add()`, not `-` or `+`.