---
title: Hardware and Software Toolchain
type: topic
tags:
  - stm32
  - nucleo
  - micropython
source:
  course: ME405
  term: 2262
  lecture: 3
status: draft
---

## Motivation

Before writing meaningful firmware, it is important to understand the development hardware and software tools that will be used throughout the course.

## Hardware Tools

All the firmware that you write this quarter will run on a development board called a Nucleo (pronounced new-klee-oh). The Nucleo is a low cost, high performing microcontroller development platform that is especially appealing due to the ST-Link debugger module that comes for free with each development board.

### STM32 Nucleo-L476 Development Board

The primary development board for this course is the **STM32 Nucleo-L476RG**.
![A top-down view of the Nucleo L476RG development board from ST Microelectronics.](images/hardware_toolchain/nucleo.png)

Important components include:
* **U5:** Main STM32L476RG microcontroller
* **U2:** ST-LINK microcontroller (used for programming/debugging)
* **CN1**: ST-LINK USB connector (firmware programming)
* **CN5**, **CN6**, **CN8**, **CN9**:  Arduino-compatible headers (the narrow black strips on the left and right sides)
* **CN7**, **CN10** Morpho expansion headers (the numerous pin headers on the left and right sides)
* **B1:** User button (connected to PC13)
* **LD2:** User LED (connected to PA5)
* Reset button
See more: [Nucleo 64 User Manual](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf)
#### Notes
- The ST-LINK section and the application MCU are separate devices. Signals run between the devices through the narrow webs connecting the two halves of the PCB.
* The reset button should generally only be used when necessary during development. If the reset button is pressed at the wrong moment it can nonpermanently corrupt the file system; while this is recoverable it can be a waste of time while you are working in the lab.
* The particular Nucleo variant, the L476RG, was selected due to its large Flash and suitable RAM size. Many faster MCUs are available on other Nucleo variants but few offer as much space for program size and runtime.

------------------------------------------------------------------------

### "Shoe of Brian"
![A top-down view of the Shoe of Brian accessory board that plugs into the bottom of the Nucleo L476RG.](images/hardware_toolchain/shoe_of_brian.png)

The Shoe of Brian expansion board provides a native USB connection for MicroPython which unlocks several quality of life features that improve the workflow considerably.
* Flashing code is done by saving to the enumerated USB storage drive `PYBFLASH`.
* USB is much faster and more reliable than simpler protocols like UART.
The Shoe of Brian also exposes some of the most frequently used pins through screw terminals, which may provide a more robust or long term solution when the hardware is embedded within a machine or enclosure.

------------------------------------------------------------------------

### Alternative Hardware

Although this course uses the Nucleo platform, MicroPython also supports many other STM32 boards. The original and official PyBoard, from the creators of MicroPython, uses a faster and more capable STM32 variant. The formfactor and cost of the PyBoard make it a poor choice for ME 4305 and, unfortunately, the same MCU variant is not available on a Nucleo.

Examples shown in lecture include:
-   Official **PyBoard** (STM32F405)
    ![A top-down view of the pyboard, from the creators of Micropython.](images/hardware_toolchain/pyboard.png)
    -   Native USB
    -   SD card slot
-   STM32 **Black Pill** (STM32F411)
    ![A top-down view of the STM32 Blackpill.](images/hardware_toolchain/blackpill.png)
    * The "lightest weight" MicroPython compatible development board.
    * Available for only a few dollars per board when bought from overseas.

------------------------------------------------------------------------

## Software Tools

Several editors and development environments are suitable for use in ME 4305, so you are free to choose what you like best or have used before. Throughout the course, examples may show different editors. The specific editor is far less important than understanding the firmware itself.

### Full IDEs

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

### Thonny

A lightweight IDE designed specifically for Python and MicroPython.

Provides:
-   local interpreter
-   built-in serial monitor
-   simple interface

### Text Editors

Examples:
-   Notepad++
-   other code editors

Useful for editing source files, but they generally lack debugging and interpreter support.

### Jupyter Notebook / JupyterLab

Useful for homework and analysis.

Advantages:
-   executable cells
-   embedded equations
-   plots
-   formatted documentation

Not recommended for firmware development.

### Serial Terminals

Examples:
-   PuTTY
-   `screen`
-   HyperTerminal
-   many others

For this ME 4305 you will need:
1.  A Python editor or IDE (Spyder, PyCharm, or VS Code recommended)
2.  A serial terminal (PuTTY recommended)
3.  Jupyter Notebook/Lab for homework assignments

------------------------------------------------------------------------

## Summary

* Become familiar with the physical layout of the Nucleo board.
* Understand the role of the Shoe of Brian expansion board.
* Install appropriate development tools before beginning the labs.
