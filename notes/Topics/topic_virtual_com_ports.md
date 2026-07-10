---
title: Virtual Communication Ports
type: topic
tags:
  - micropython
  - serial
  - USB_VCP
  - UART
  - pyserial
source:
  course: ME405
  term: 2262
  lecture: 12
status: draft
---

## Motivation

Embedded systems frequently communicate with external computers for debugging, user interfaces, logging, and data collection. This lecture introduces the serial communication tools available on the STM32 platform and the corresponding tools available on a PC.

## USB Virtual COM Port vs UART

On the STM32 platform two primary serial interfaces are available:
* `pyb.USB_VCP`
* `pyb.UART`

Recall from the earlier hardware discussion:
* **Native USB** connects to the USB Virtual COM Port (VCP).
* **ST-Link USB** connects to UART2.

By default, the REPL is mirrored onto UART2. Consequently:
* `print()` output appears on both interfaces.
* UART2 should generally not be repurposed unless this behavior is intentionally changed by calling `pyb.repl_uart(None)`

Both interfaces support the `write()` method. Unlike `print()`, `write()` does **not** automatically append line endings. Remember:
* `\r` → carriage return
* `\n` → newline

### Example 1:

In this example a virtual COM port, or VCP, will be created and used to transmit a line of text.

```python
from pyb import USB_VCP

ser = USB_VCP()
ser.write("Hello\r\n")
```

### Example 2:

In this example a UART (universal asynchronous receiver-transmitter) will be used to transmit the same line of text. Notice that for the UART, a peripheral number and a baud rate, or speed, must be specified. The following example uses the industry standard of 115200 for the baud rate and will use **UART1**

```python
from pyb import UART

ser = UART(1, baudrate=115200)
ser.write("Hello\r\n")
```

### Configuring UART

There are several UART ports on the STM32, each with a default set of pins that will automatically be configured when the UART is instantiated. Other pins can be used but they must be configured with additional lines of code.

| UART | TX    | RX   | RTS   | CTS   |
| ---- | ----- | ---- | ----- | ----- |
| 1    | `B6`  | `B7` | `B4`  | -     |
| 2    | `A2`  | `A3` | `A1`  | `A0`  |
| 3    | `C4`  | `C5` | `B14` | `B13` |
| 4    | `A0`  | `A1` | -     | -     |
| 5    | `C12` | `D2` | -     | -     |

If you need to use UART on pins that are not default pins you must both configure a new set of pins and unconfigure the default set. To know which pins can access a given UART you must consult the **Alternate Function Table** in the STM32 datasheet.

#### Example 3

This example will create an object for UART 3 using non-default pins.

```python
from pyb import UART, Pin

ser = UART(3, baudrate=115200)
Pin(Pin.cpu.C4, mode=Pin.ANALOG)  # Unconfigure default TX pin
Pin(Pin.cpu.C5, mode=Pin.ANALOG)  # Unconfigure default RX pin
Pin(Pin.cpu.B10, mode=Pin.ALT, alt=7)  # Configure new TX pin
Pin(Pin.cpu.B11, mode=Pin.ALT, alt=7)  # Configure new RX pin
```

The alternate function table shows that `B10` and `B11` can access UART3 through alternate function 7.

![An excerpt from the STM34L476 datasheet from the alternate function table. ](images/vcp/af_table_page_2.png)

## Using pyserial on the PC

On a computer the standard serial package is **pyserial**.

Installation:
```bash
pip install pyserial
```

Although the package is named **pyserial**, it is imported as `serial`. The use is very similar to that of `USB_VCP` or `UART`.
```python
import serial

ser = serial.Serial("COM1", baudrate=115200)
ser.write("Hello\r\n")
```

These serial objects behave similarly to Python file streams and support methods such as:
* `read()`
* `readline()`
* `readlines()`
* iteration with `for`

## Keeping things straight

The table below summarizes the three types of serial communication port covered above. It is important to understand which one to use in the right context.

| Tool     | Platform    | Class Name      | Usage                                                                                                                                                                                                                                 |
| -------- | ----------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| VCP      | MicroPython | `pyb.USB_VCP`   | Primary tool for accessing serial port on the STM32 microcontroller. Should be used for access to the Python REPL, a UI, or for basic data transfer.                                                                                  |
| UART     | MicroPython | `pyb.UART`      | Secondary tool for accessing serial ports on the microcontroller to use for data transfer, debugging, etc. Should likely not be used unless the VCP is already in use or you need UART specifically for another piece of<br>hardware. |
| pyserial | CPython     | `serial.Serial` | Primary tool for accessing serial ports on any computer running Python. Any serial interaction done on a computer, through USB, Bluetooth, or a true serial port, can be done using this module.                                      |

## Bytes versus Strings

Something to keep in mind while working with serial ports is that they return data in the form of bytes objects, not strings. That is, instead of returning a fully featured object of Python's strclass, simpler objects of the bytesclass are returned. In many cases it is possible (and even faster) to work with these objects as returned, but in other cases it is necessary to convert back and forth between data types.

Convert bytes to strings:

```python
text = data.decode()
```

Convert strings to bytes:

```python
data = text.encode()
```

UTF-8 is used by default.

Many serial APIs also accept normal Python strings directly when writing.

## Summary

* Use `pyb.USB_VCP` for most user communication.
* UART is available for additional hardware interfaces.
* Serial communication exchanges bytes rather than strings.
* `encode()` and `decode()` convert between strings and bytes.

## See Also
* [[topic_hardware_overview|Hardware and Software Toolchain]]
* [[topic_serial_communication|Serial Communication]]
* [[reference_formatting_strings|Formatting Strings]]
* [[topic_coop_user_io|Cooperative User Input and Output]]
* [[topic_collections|Collections]]