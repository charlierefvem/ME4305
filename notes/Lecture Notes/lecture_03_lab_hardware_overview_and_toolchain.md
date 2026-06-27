---
title: Lecture 3 - Lab Hardware Overview and Toolchain
type: lecture
topics:
- STM32 Nucleo
- MicroPython
- Development Tools
- UART
- Virtual COM Ports
tags:
- me405
- stm32
- micropython
---

# Motivation

Before writing meaningful firmware, it is important to understand the development hardware, the supporting accessories, and the software tools that will be used throughout the course.

> **Candidate static notes**
>
> -   \[\[STM32 Nucleo\]\]
> -   \[\[Shoe of Brian\]\]
> -   \[[UART](#uart)\]
> -   \[\[Virtual COM Port\]\]
> -   \[\[MicroPython Development Environment\]\]

## STM32 Nucleo-L476 Development Board

The primary development board for this course is the **STM32 Nucleo-L476RG**.
![A top-down view of the Nucleo L476RG development board from ST Microelectronics.](nucleo.png)

Important components include:

-   **U5:** Main STM32L476RG microcontroller
-   **U2:** ST-LINK microcontroller (used for programming/debugging)
-   ST-LINK USB connector (firmware programming)
-   Arduino-compatible headers
-   Morpho expansion headers
-   **B1:** User button (PC13)
-   **LD2:** User LED (PA5)
-   Reset button

### Notes

-   The ST-LINK section and the application MCU are separate devices.
-   The reset button should generally only be used when necessary during development.

------------------------------------------------------------------------

## "Shoe of Brian"
![A top-down view of the Shoe of Brian accessory board that plugs into the bottom of the Nucleo L476RG.](shoe_of_brian.png)

The Shoe of Brian expansion board provides:

-   Native USB connector for MicroPython
-   Access to otherwise inaccessible Morpho pins
-   Convenient breakout connections for laboratory hardware

The intent is to simplify laboratory wiring while exposing additional I/O.

------------------------------------------------------------------------

## Alternative Hardware

Although this course uses the Nucleo platform, MicroPython also supports many other STM32 boards.

Examples shown in lecture include:

-   Official **PyBoard** (STM32F405)
    ![A top-down view of the pyboard, from the creators of Micropython.](pyboard.png)
    -   Native USB
    -   SD card slot
-   STM32 **Black Pill** (STM32F411)
    ![A top-down view of the STM32 Blackpill.](blackpill.png)

These illustrate that MicroPython is portable across many STM32 devices.

------------------------------------------------------------------------

# Software Tools

Several editors and development environments are suitable.

## Full IDEs

Examples include:

-   Spyder
-   PyCharm
-   Visual Studio Code

Advantages:

-   syntax highlighting
-   linting
-   local interpreter
-   debugger
-   optional serial monitor support

## Thonny

A lightweight IDE designed specifically for Python and MicroPython.

Provides:

-   local interpreter
-   built-in serial monitor
-   simple interface

## Text Editors

Examples:

-   Notepad++
-   other code editors

Useful for editing source files, but they generally lack debugging and interpreter support.

## Jupyter Notebook / JupyterLab

Useful for homework and analysis.

Advantages:

-   executable cells
-   embedded equations
-   plots
-   formatted documentation

Not recommended for firmware development.

## Serial Terminals

Examples:

-   PuTTY
-   `screen`
-   HyperTerminal
-   many others

For this course you will need:

1.  A Python editor or IDE (Spyder, PyCharm, or VS Code recommended)
2.  A serial terminal (PuTTY recommended)
3.  Jupyter Notebook/Lab for homework assignments

------------------------------------------------------------------------

# Serial Communication

The course hardware exposes two independent USB connections:
![A sketch showing that the two USB ports on the Shoe and Nucleo can be used independently.](shoe_coms.png)
``` text
PC
 ├── USB #1
 │      │
 │   ST-LINK USB
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
   Native USB
        │
    Main MCU
```

The ST-LINK interface communicates with the application MCU over UART, while the native USB port connects directly to the application MCU.

## Important Terms

### UART

**Universal Asynchronous Receiver Transmitter**

-   A true serial communication peripheral.
-   Transfers binary data over TX and RX lines.
-   Operates at a specified baud rate (commonly 115200 baud).

    TX  --->  RX
    RX  <---  TX

### COM Port

A communication endpoint presented by the operating system for serial devices.

Historically associated with RS-232 hardware, but the abstraction remains in modern operating systems.

### Virtual COM Port (VCP)

A USB device that emulates a traditional serial COM port.

Many USB devices, including the ST-LINK and many Bluetooth devices, present themselves as virtual COM ports.

------------------------------------------------------------------------

## Summary

-   Become familiar with the physical layout of the Nucleo board.
-   Understand the role of the Shoe of Brian expansion board.
-   Install appropriate development tools before beginning the labs.
-   Distinguish between UART, COM ports, and Virtual COM Ports.
-   Recognize that the board exposes two different USB communication paths.

> **Editorial Note**
>
> The annotated board photographs are an important part of the lecture. If possible, preserve them alongside the Markdown rather than replacing them with text-only descriptions.

## Related Topics
* [[lecture_02_collections_and_file_io|Lecture 2 - Collections and File I/O]]