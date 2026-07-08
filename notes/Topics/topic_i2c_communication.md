---
title: I2C Communication
type: topic
tags:
  - micropython
  - i2c
  - sensors
source:
  course: ME 405
  term: 2262
  lecture: 14
status: draft
---

## Motivation

Many sensors used in embedded systems are not connected through a single analog voltage or a simple digital signal. Instead, they communicate with the microcontroller using a digital communication protocol. I2C is one of the most common protocols for this kind of sensor communication because it allows many peripheral devices to share the same two communication lines.

This note introduces I2C as a bus protocol, explains how data moves across the bus, and connects the protocol-level ideas to the `pyb.I2C` interface used on the STM32-based lab hardware.

## Names for I2C

The protocol is commonly referred to by a few closely related names:
* IIC, I$^2$C, or I2C: Inter-Integrated Circuit
* TWI: Two-Wire Interface

The name Two-Wire Interface comes from the physical layer: each bus uses two communication lines, plus power and ground.

## Basic Features of I2C

I2C is a synchronous serial communication protocol. Data is sent in binary, bit by bit, in a sequential pattern. The bus uses two communication signals:
* `SDA`: serial data
* `SCL`: serial clock

I2C uses a controller/peripheral hierarchy. One controller communicates with one or more peripheral devices on the same bus. Older documentation may describe this as a master/slave hierarchy, but the preferred terminology here is controller/peripheral.

Several features define the protocol:
* It is a bus protocol, so many devices can share the same bus.
* A 7-bit device address creates an address space of 128, however several of these addresses are reserved and therefore unavailable for use as device addresses.
* Data can move in either direction, but not at the same time, so I2C is half-duplex.
* Communication is initiated by the controller. Peripherals respond when addressed.
* The physical layer uses open-drain outputs in a wired-or configuration so that many devices can cooperatively share the same `SDA` and `SCL` signals.
* Typical pull-up values are in the range of 2.2k to 4.7k and vary depending on timing requirements.

![I2C bus overview. Device 1 is the controller and connects to two shared communication lines labeled SCL and SDA. Device 2 and Device 3 are peripherals connected to the same SCL and SDA bus. Both lines are pulled up through resistors, illustrating the open-drain wired-or physical layer used by I2C.](images/iic/physical_layer.svg)

### Comparison to SPI

The most comparable protocol is SPI, the Serial Peripheral Interface. SPI typically uses four or more communication lines per bus instead of two. It can also be full-duplex, depending on how it is configured.

SPI is usually simpler to use and supports much faster clock speeds. The speed advantage comes partly from the use of actively driven tri-state outputs rather than open-drain outputs. I2C gives up some speed in exchange for a smaller number of communication wires and an addressable shared bus.

## How I2C Sends Data

I2C transfers data using the `SDA` line, while the `SCL` line synchronizes the sending and receiving devices. The clock line acts like a drum beat: it sets the pace of communication and helps guarantee that the receiving device samples the data line after the sending device has updated it.

The controller normally generates the clock signal. Some peripherals can perform clock stretching, where the peripheral holds the clock line low to slow the controller down. Clock stretching is supported by the I2C standard, but some controllers do not handle it reliably.

All data transfers begin when the controller performs a start condition on the bus.

![Open-drain I2C physical layer. Two devices share the SDA and SCL lines. Each device contains switches or transistor-like outputs that can pull the line down to ground, while external pull-up resistors return the lines to 3.3 V when no device is pulling them low. This illustrates why no device actively drives a logic high on an I2C bus.](images/iic/physical_layer_wired_or.svg)

## Device Addresses and the Read/Write Bit

To distinguish between devices on a shared bus, each I2C peripheral must have a unique address. In the common 7-bit addressing mode, the device address is written as

```text
a6 a5 a4 a3 a2 a1 a0
```

An additional bit, called the `R/W` bit, is concatenated with the 7-bit address to form the transmitted address byte:

```text
a6 a5 a4 a3 a2 a1 a0 R/W
```

## Read and Write Transactions

A read transaction begins with a start condition, followed by the address byte (the 7-bit address concatenated with the read bit). The addressed peripheral then returns one or more data bytes. After each byte, the receiver sends an acknowledge (ACK) bit until the last byte is received then the receiver sends a negative acknowledge (NACK) bit to tell the peripheral to stop sending. The transaction ends with a stop condition.

![I2C read timing diagram from Analog Devices. The controller starts the transfer, sends the 7-bit peripheral address with the read bit set to 1, and receives data bytes from the peripheral.|700](images/i2c_read_timing.png)

A write transaction also begins with a start condition, followed by the 7-bit address and a write bit. The controller then sends one or more data bytes to the peripheral. During the high portions of the SCL pulses, the data on SDA stays stable so the receiver can sample it.

![I2C write timing diagram from Analog Devices. The controller starts the transfer, sends the 7-bit peripheral address with the write bit set to 0, then sends a data byte to the peripheral.|700](images/i2c_write_timing.png)

External figure source for timing diagrams: Analog Devices, I2C primer, https://www.analog.com/en/technical-articles/i2c-primer-what-is-i2c-part-1.html

## Register-Based Devices

Almost all I2C devices use a register-style interface. A register is a specific address inside the peripheral device. The controller uses the register address to choose which internal value it wants to read or write. Extremely simple sensors may omit this kind of register map, but most useful sensors have many different values and configuration options, so a register map is the normal interface.

A useful analogy is to think of an I2C peripheral as a bank or a post office. The **device address** is like the street address of the building—it tells the controller which building should receive the request. Once inside the building, the **register address** is like the number on an individual lock box. The controller must first reach the correct device, and then specify which internal register it wishes to access.

To read data from a register, the communication is normally broken into two steps, each handled by a separate I2C transfer:
1. The controller writes one byte, or sometimes more than one byte, to specify the selected register address.
2. The controller reads one or more bytes from the peripheral. The peripheral should respond with data starting at the specified register address.

To write data to a register, the process still has two logical steps, but it can often be completed in a single I2C transfer:
1. The controller writes one byte, or sometimes more than one byte, to specify the selected register address.
2. Before ending the transfer, the controller sends one or more data bytes. Those bytes should be written into the peripheral registers starting at the specified address.

Notice that the I2C peripheral driver forms another abstraction layer. User tasks should request measurements from the driver rather than directly implementing I2C transactions throughout the application.
### Example 1

The BNO055 IMU is an example of a register-based I2C device that we will use in ME 4305. The portion of the register map that holds accelerometer data is shown below.

| Register address | Register contents |
| --- | --- |
| `0x08` | `ACC_DATA_X_LSB` |
| `0x09` | `ACC_DATA_X_MSB` |
| `0x0A` | `ACC_DATA_Y_LSB` |
| `0x0B` | `ACC_DATA_Y_MSB` |
| `0x0C` | `ACC_DATA_Z_LSB` |
| `0x0D` | `ACC_DATA_Z_MSB` |
The important pattern is that each accelerometer axis is stored as two bytes: a least-significant byte and a most-significant byte. Reading six bytes beginning at address `0x08` gives the raw X, Y, and Z accelerometer values.

## `pyb.I2C` Transfer Functions

The STM32 hardware has built-in support for simple I2C read and write transfers. The MicroPython API provides the `pyb.I2C` class with functions for both simple transfers and register-based interaction.

| Transfer type | Read: peripheral to controller | Write: controller to peripheral |
| ------------- | ------------------------------ | ------------------------------- |
| Simple        | `pyb.I2C.recv()`               | `pyb.I2C.send()`                |
| Register      | `pyb.I2C.mem_read()`           | `pyb.I2C.mem_write()`           |

### Example 2

This example reads six bytes from the BNO055 IMU starting at the first accelerometer data register as indicated by the register map snippet in Example 1.

```python
import pyb

# Device and memory/register addresses from the IMU register map.
dev_addr = 0x28  # IMU address
mem_addr = 0x08  # ACC_DATA_X_LSB

# Create a six-byte mutable buffer to hold the returned data.
buf = bytearray(0 for _ in range(6))

# used for the specific STM32 board and MicroPython version in lab.
my_i2c = pyb.I2C(1, pyb.I2C.MASTER)

# Read six bytes from the peripheral, starting at the selected register.
my_i2c.mem_read(buf, dev_addr, mem_addr)
```

In the call to `mem_read()`, the arguments play three different roles:
- `buf` is the mutable buffer where the returned data will be stored. A `bytearray` is useful here because it can be modified in place.
- `dev_addr` is the I2C address of the device, which determines which peripheral responds.
- `mem_addr` is the memory or register address inside the peripheral, which determines what data is read.

**Insight**: returning to the bank or post office analogy, one could say that `mem_read()` first drives to the correct building using the device address, then walks inside and opens the requested lock box using the register address. However, each lock box contains only **one byte** of information. In this example, a complete accelerometer measurement spans six consecutive lock boxes, so `mem_read()` opens six adjacent lock boxes and returns all six bytes together.

### Example 3:

After reading the six accelerometer bytes, the byte data still needs to be interpreted. The `struct` module can unpack those six bytes into three signed 16-bit integer values. This example is assumed to run after Example 2.

**Note**: in the example the data is converted into three signed 16-bit integer values, but in practice these values will also need to be scaled by a gain to convert the values into engineering units. The end result for each axis will likely be a floating point value.

```python
import struct

acc_x, acc_y, acc_z = struct.unpack("<hhh", buf)
```

The format string `"<hhh"` has two parts:
- `<` specifies little-endian byte order, meaning the least-significant byte comes first. This matches the register map which includes LSB values before MSB values.
- `hhh` specifies three signed 16-bit integers.

This matches the register layout where each axis is stored as a low byte followed by a high byte:
```text
< h h h
| | | |
| | | +-- signed 16-bit integer for Z acceleration
| | +---- signed 16-bit integer for Y acceleration
| +------ signed 16-bit integer for X acceleration
+-------- little-endian
```

The first character in the format string is the endianness specifier, or byte-order specifier:
- `>` means big-endian, with the most-significant byte first.
- `<` means little-endian, with the least-significant byte first.

**Note:** it is also allowed to use format strings like `'<3h'` to refer to three signed 16-bit integers in little-endian format. This style can be convenient in scenarios that require production of format strings programmatically.

## Insights

I2C is easy to describe at a high level but has several layers that matter in practice:
* At the physical layer, open-drain outputs and pull-up resistors allow many devices to share the same lines without fighting each other.
* At the protocol layer, the controller controls when communication starts and which peripheral is being addressed.
* At the device layer, most useful peripherals expose many internal registers, so the controller must specify both the peripheral address and the internal register address.
* At the software layer, byte-oriented data often needs to be converted into meaningful integer values using the device's register map and byte order.

The most important engineering habit is to keep these layers separate. The device address tells the bus which chip should respond. The register address tells that chip which internal data should be accessed. The buffer contains the raw bytes returned by the chip. The unpacking step converts those raw bytes into useful variables.

## Summary

I2C is a synchronous, half-duplex serial bus protocol that uses two communication lines: `SDA` for data and `SCL` for the clock. The bus uses open-drain outputs with pull-up resistors, allowing multiple devices to share the same wires.

The controller initiates all communication. Each peripheral has an address, and the transmitted address byte includes both the 7-bit device address and a read/write bit. Most I2C devices use register maps, so reading sensor data usually means selecting a register address and then reading one or more bytes from that starting location.

On the lab hardware, the `pyb.I2C` class provides functions for both simple transfers and memory/register transfers. For sensors such as an IMU, a common workflow is to read a block of bytes with `mem_read()` and then use `struct.unpack()` to interpret those bytes as signed integers with the correct endianness.