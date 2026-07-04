---
title: Mechanical Switch Bounce and Methods for Debouncing
type: reference
tags:
  - switches
  - buttons
  - gpio
  - interrupts
  - debounce
source:
  course: ME 405
  term: 2262
  lecture: 23
status: draft
---

# Motivation

Mechanical switches and buttons seem simple, but they create several practical issues in embedded systems. A switch connected to a GPIO pin needs a complete circuit, a known default logic state, and a strategy for handling non-ideal switching behavior.

To interface with an array of switches, the immediate design questions are:
* What additional components are needed to complete the switch circuits?
* How can fewer GPIO pins be used to read the switches?
* How should the firmware avoid treating one physical button press as many separate events?
The last question is the main focus of this reference note: **switch bounce** and **debouncing**.

# Active-Low Button Circuits

Recall that a common button circuit is the **active-low** configuration. In an active-low circuit, the GPIO pin is normally held high through a pull-up resistor. When the switch closes, the pin is connected to ground and the input reads low.

For a single switch:
* The resistor pulls the input up to $V_{DD}$ when the switch is open.
* The switch pulls the input down to ground when pressed or closed.
* The GPIO input reads logic `1` when inactive.
* The GPIO input reads logic `0` when active.

On many microcontrollers, the pull-up resistor can be configured internally, so an external resistor may not be required.

![Active-low switch circuit. The circuit shows an MCU GPIO pin connected to a node with a pull-up resistor to VDD and a switch to ground.|700](images/gpio_active_low_switch.svg)

# Reading Multiple Switches with Fewer GPIO Pins

Reading from an entire array of switches independently could require many GPIO pins. GPIO pins may or may not be a limited resource depending on the number pins available on your MCU and the utilization of the pins for other features. It is often desirable to reduce the number of pins needed to below the number of switches and several techniques exist for this with different benefits and drawbacks.

## Wired-OR

Because the switch circuits are active-low, multiple switches can be combined using a **wired-OR** style connection. In positive logic this looks like an AND of the high-level signals, but in active-low logic any one switch can pull the shared line low.

This can reduce the number of GPIO pins needed, but there is a tradeoff: if switches are combined onto one line, the firmware may only know that **some** switch in the group was hit, not necessarily which specific switch was hit.

Common groupings could include:
* one input for any switch
* one input for each batch of switches
* one input for the entire array of switches

## Switch Matrices

It is possible to build a matrix of switches by arranging them into rows and columns. Consider an $m \times n$ array of switches. To read from each switch individually, $m\cdot n$ GPIO pins would be required. In a matrix only $m+n$ GPIO pins would be required. Depending on the matrix size this could save a considerable number of GPIO pins. The tradeoff for the reduced GPIO usage is more complicated means of reading from the matrix. The MCU must scan rows and columns rather than simply check the state of each pin.

### Example 1

For a $4\times4$ array of switches, like the hexadecimal keypad shown below, the number of required GPIO reduces by half from 16 to 8.

![A schematic snippet for a 4x4 switch matrix. Rows are labeled Y1 through Y4 and columns are labeled X1 through X4.|700](images/switch_matrix.svg)

**Note**: the above schematic is simplified from how these types of matrices are typically constructed. As shown, the matrix will create false readings if multiple buttons are pressed simultaneously. More robust circuits add series diodes to each switch. By allowing current to flow only one way through each switch the false readings can be prevented.

## Shift Registers and Port Expanders

Another technique for reading many switches with few GPIO pins is to use a device that converts parallel data into serial data. A parallel-in-serial-out shift register is the simplest device to do such a conversion. A port expander, connected with I2C or SPI to the MCU, would be a more modern example. Either of these devices can work in complement to the matrix configuration or can work directly with each switch.

# Switch Bounce

An ideal mechanical switch would change cleanly from off to on, and then cleanly from on back to off as shown in the waveform below.

![Ideal switching behavior. The ideal switching plot shows one clean transition from OFF to ON and one clean transition back to OFF.](images/switch_ideal_waveform.png)

A real mechanical switch does not usually behave that way. When the contacts first touch, they can physically bounce off one another mechanically. This produces several rapid transitions before the signal settles. The same thing can happen when the switch opens. An example waveform is shown below.

![Real-world switching behavior. The real-world plot shows several rapid transitions during the bounce time at both the rising and falling edges before the signal settles.](images/switch_bounce_waveform.png)

The practical firmware problem is that one physical switch press may produce multiple rising and falling edges. If the input is connected to an interrupt, each of those edges may trigger an ISR.

Image source: https://forum.digikey.com/t/switch-bounce-in-mechanical-switch-and-debounce-circuit/14231

## Hardware Debouncing

There are hardware-based approaches to switch debounce. The DigiKey article linked above gives several examples.

### Simple RC Filter

A simple RC debounce circuit uses a resistor-capacitor filter to slow the transition seen by the GPIO input.

![Simple RC debounce circuit. The circuit shows a pull-up resistor R1 to plus 5 volts, a switch to ground, and a simple RC filter made from R2 and C1 feeding the port input.](images/switch_rc_debounce.png)

For the simple RC filter shown above, the approximate time constants are
$$
\tau_{rise} = (R_1 + R_2)\,C_1
$$
and
$$
\tau_{fall} = R_2\,C_1.
$$

### Modified RC Filter with Diode

A modified hardware debounce circuit can add a diode so that the rise and fall times can be tuned independently.

![Modified RC debounce circuit. The circuit shows a pull-up resistor R1, a switch to ground, a diode D1, a resistor R2, a capacitor C1 to ground, and a Schmitt trigger before the port input. Handwritten notes identify the rise time constant as R1C1 and the fall time constant as R2C1.](images/switch_rc_diode_debounce.png)

For the modified RC filter shown in the figure above, the approximate time constants are
$$
\tau_{rise} = R_1\,C_1
$$
and
$$
\tau_{fall} = R_2\,C_1.
$$

## Software Debouncing

In many cases, it is cheaper and sometimes simpler to debounce switch inputs in software instead of hardware. The problem is that each switch press can cause multiple rising and falling edges, and each edge may trigger an interrupt service routine.

A software debounce strategy is:
1. Let the first edge trigger the ISR.
2. Record the event.
3. Disable the associated interrupt channel for a debounce period.
4. Re-enable the channel later from a regularly scheduled task.

In this approach, the ISR responds quickly to the first detected edge, but additional edges caused by bounce are ignored until the debounce interval has passed.

The debounce interval should be chosen to be longer than the expected mechanical bounce time but shorter than the shortest intentional switch activation expected by the application. Choosing the debounce interval is an engineering tradeoff. The interval should be long enough to reject mechanical bounce but short enough that intentional switch transitions are not suppressed.
### Example 2

This example will illustrate one method for implementing debounce using a combination of a scheduled task and an interrupt service routine.

**Ingredients for Interrupt-Based Software Debouncing**
1) A `queue` object for button events so that other tasks can be aware of the detected edges.
    1) A queue is particularly well suited for asynchronous events because multiple events can occur before another task has an opportunity to process them.
    2) The queue can be any size, but it may make sense for the queue size to match the number of switches or perhaps double the number of switches if press and release events are to be processed separately. If the queue is small, overwrite behavior may be useful so that the more recent switch events are detected in other tasks.
    3) If the queue is used from an ISR, it must be thread-protected. When calling `.put()` inside an ISR, set `in_ISR=True`.
2) One or More External Interrupts
    1) Use an ISR, or a set of ISRs, attached to `ExtInt` IRQs for the switch lines.
    2) The ISR lines are based on the pin number, not the port. For example, `PA0`, `PB0`, and `PC0` all share ISR line `0`, so they cannot be used for separate external interrupts at the same time. There are 16 total ISR lines, numbered `0` through `15`, and those lines can be accessed from different ports.
    3) The ISR should:
        1) Put something in the queue to indicate that an edge occurred or which switch line received the edge.
        2) Disable IRQs on the appropriate ISR line or lines.
        3) If needed, set a flag so that other code can re-enable IRQs later.
3) A Scheduled Task for Re-Enabling IRQs
    1) A task should run at a regular debounce interval. Its job is to re-enable IRQs after the debounce period expires.
#### Debounce Task

The code block below shows a task that disables an external interrupt after a switch event, then re-enables it after the debounce interval has passed.

```python
from pyb import ExtInt, Pin, enable_irq, disable_irq
from array import array

# A task compatible with cotask.py that handles debounce for up to 16 switches
# connected to different ExtInt ISR lines.
class task_switch:

    # Create a task object using a defined set of switch pins and a queue
    # to store edge events.
    #
    # Params:
    #    event_queue A queue object used to hold switch event flags. Each
    #                flag is represented by a byte. The lower nibble describes
    #                the ISR line and b4 encodes the switch transition:
    #                - b4 clear describes a falling edge (a switch press)
    #                - b4 set describes a rising edge (a switch release)
    #    switches A collection of pyb.Pin objects used to read the switches. All
    #             switches must use unique ISR lines.
    #
    def __init__(self, event_queue, switches):
        # A Queue used to store edge detection events
        self._event_queue = event_queue
        
        # A dictionary used to map pin numbers (ISR lines) to Pin objects
        self._pins = {switch.pin(): switch for switch in switches}
        
        # A dictionary used to map pin numbers (ISR lines) to ExtInt objects
        # All ISRs use the same callback function
        self._callbacks = {
            switch.pin(): ExtInt(
                switch,
                ExtInt.IRQ_RISING_FALLING,
                Pin.PULL_UP,
                self._callback,
            )
            for switch in switches
        }

        # An array of two 16-bit integers used to store current and previous
        # debounce states. A one in any position indicates that switch has
        # recently been pressed or released. The previous state must be retained
        # to guarantee that the task iterates twice before reenabling callbacks
        # so that the debounce time is greater than the task period instead of 
        # less than the task period.
        #
        #     self._db_mask[0] = current debounce state
        #     self._db_mask[1] = previous debounce state
        #
        self._db_mask = array("H", [0x0000, 0x0000])

    # This callback runs on the first rising or falling edge associated with a
    # press or release on any of the switches. When the callback runs the ISR
    # line is passed in as an integer.
    def _callback(self, ISR_src):

        # Set the debounce state to include the channel which triggered this
        # ISR cycle.
        self._db_mask[0] |= 1 << ISR_src

        # Disable the callback on this channel so that no more interrupts
        # occur until after the debounce period.
        self._callbacks[ISR_src].disable()

        # Put the event into the event queue so that other tasks can know
        # a press or release occurred.
        self._event_queue.put(ISR_src | (self._pins[ISR_src].value() << 4),
                              in_ISR=True)

    # This task should be scheduled with a period equal to or greater than
    # the expected debounce period for the switches. A period of at least 30
    # ms is recommended.
    def run(self):
        # This task has only one state
        while True:
            # Begin critical section
            irq_state = disable_irq()

            # Remember the mask so that appropriate lines can be reenabled
            reenable_mask = self._db_mask[1]  

            # Shift the current debounce state to previous state and reset
            # the current state to zero.
            self._db_mask[1], self._db_mask[0] = self._db_mask[0], 0x0000

            # End critical section 
            enable_irq(irq_state)  
              
            # Check which channels have pending debounce by examining the
            # remembered mask representing debounce states. Reenable any
            # channels that are due.
            for isr_src in range(16):  
                if reenable_mask & (1 << isr_src):  
                    self._callbacks[isr_src].enable()
            
            yield
```

##### How the Debounce Mask Works

The array
```python
self._db_mask = array("H", [0x0000, 0x0000])
```
stores two 16-bit bitmasks:
* `self._db_mask[0]`: the current debounce state
* `self._db_mask[1]`: the previous debounce state

When an ISR triggers, the callback sets the corresponding bit in the current debounce mask:
```python
self._db_mask[0] |= 1 << ISR_src
```

The callback then disables the interrupt source so that bounce edges on the same line do not keep generating interrupts. 

**Insight**: notice that the callback performs only a few simple operations before returning. This keeps the ISR execution time short while deferring less time-critical work to the scheduled task.

Each time the task runs, the first thing it does is start a critical section, that is, a section in which ISR callbacks are temporarily disabled. During this critical section, a copy of the previous mask is stored to remember which callbacks must be reenabled. The current mask is then shifted into the previous mask and the current mask is cleared:
```python
irq_state = disable_irq()

reenable_mask = self._db_mask[1]
self._db_mask[1], self._db_mask[0] = self._db_mask[0], 0x0000

enable_irq(irq_state)
```
 
 After the critical section, the task checks the remembered previous debounce mask. Any line that was in the current mask during the previous task cycle is now due to be re-enabled:
```python
for ISR_src in range(16):
    if reenable_mask & (1 << ISR_src):
        self._callbacks[ISR_src].enable()
```

The critical section prevents an ISR from modifying the mask at the same time the task is shifting and clearing it.

**Note**: the code assumes that every `ISR_src` bit that becomes set corresponds to a key in `self._callbacks`. This is consistent with the callback only setting bits from configured pins, but the loop still scans all 16 possible ISR lines.

# Insights

Mechanical switches produce digital-looking signals, but their transitions are not ideal. Without debouncing, one physical press can look like several presses to the firmware.

Hardware debouncing can clean the signal before it reaches the microcontroller, but it costs components and board space. Software debouncing uses firmware timing to ignore bounce after the first detected edge.

Interrupt-based debouncing is a good fit when the system should react quickly to the first edge but ignore the rapid transitions that follow. Disabling only the affected interrupt line avoids blocking unrelated interrupts.

For active-low switches, internal pull-ups and wired-OR grouping can reduce wiring and GPIO usage, but combining switches also reduces how much information the firmware receives about which exact switch was hit.

# Candidate Static Notes
* \[\[Interrupts\]\]
* \[\[External Interrupts\]\]
* \[\[Queues\]\]
* \[\[Critical Sections\]\]
* \[\[Romi Robot Platform\]\]