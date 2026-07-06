---
title: Serial Communication
type: topic
tags:
  - micropython
source:
  course: ME405
  term: 2262
  lecture: 3
status: draft
---

# Motivation

At first the interfacing between your computer and the Nucleo can feel foreign and in some rare moments redundant. As a result, it is important to develop a solid understanding of the communication protocols and signal routing  on the Nucleo hardware stack.

------------------------------------------------------------------------

# Serial Communication

The combination of the Nucleo L476RG and the Shoe of Brian exposes two independent USB connections:
![A sketch showing that the two USB ports on the Shoe and Nucleo can be used independently.](images/hardware_toolchain/hardware_toolchain.svg)

``` text
PC
 ├── USB #1
 │      │
 │  ST-LINK USB
 │      │
 │  ST-LINK MCU
 │      │
 │     UART
 │      │
 │   Main MCU
 │
 │
 └── USB #2
        │
   Native USB (on Shoe of Brian)
        │
    Main MCU
```

The ST-LINK interface communicates with the application MCU over UART, while the native USB port connects directly to the application MCU.

## Important Terms

### UART

**Universal Asynchronous Receiver Transmitter**
* A true serial communication peripheral.
* Operates at a specified baud rate (commonly 115200 baud).
* Transfers binary data over TX (transmit) and RX (receive) lines in full duplex mode.
	* TX  --->  RX
	* RX  <---  TX

### COM Port

A communication endpoint presented by the operating system for serial devices.

Historically associated with RS-232 hardware, but the abstraction remains in modern operating systems.

### Virtual COM Port (VCP)

A USB device that emulates a traditional serial COM port.

Many USB devices, including the ST-LINK and many Bluetooth devices, present themselves as virtual COM ports.

------------------------------------------------------------------------

# Summary

* Distinguish between UART, COM ports, and Virtual COM Ports.
* Recognize that the board exposes two different USB communication paths.

# See Also:
* [[topic_hardware_overview|Hardware and Software Toolchain]]
* [[topic_virtual_com_ports|Virtual Communication Ports]]
* [[reference_memoryviews|Buffers and memoryview Objects]]