---
title: Lecture 2 - Collections and File I/O
type: lecture
topics:
- Python
- Collections
- File I/O
tags:
- python
- collections
- file-io
---

# Motivation

This lecture introduces Python's built-in collection types and basic file I/O. These tools form the foundation for later labs involving data logging, CSV processing, and communication with embedded hardware.

> **Candidate static notes**
>
> -   \[\[Python File I/O\]\]
> -   \[\[Python Lists\]\]
> -   \[\[Python Tuples\]\]
> -   \[\[Python Dictionaries\]\]
> -   \[\[Python Sets\]\]
> -   \[\[Python Bytes\]\]
> -   \[[ASCII](#ascii)\]
> -   \[\[Python Arrays\]\]

## The `with` Statement

Python files should almost always be opened using a `with` block. This guarantees that the file is automatically closed when execution leaves the block, even if an exception occurs.

### Traditional approach

``` python
filename = "data.csv"

file = open(filename, "r")

for line in file:
    # Process one line at a time
    pass

file.close()
```

### Preferred approach

``` python
filename = "data.csv"

with open(filename, "r") as file:
    for line in file:
        # Process one line at a time
        pass
```

### Other common file methods

``` python
with open(filename, "r") as file:
    file.read(n)       # Read n characters
    file.readline()    # Read one line
    file.readlines()   # Read all lines
```

**Note**

-   `\n` represents a newline.
-   `\r` represents a carriage return.

------------------------------------------------------------------------

# Python Collection Types

A collection is an aggregate object that stores multiple pieces of data in a single variable.

  Type          | Mutable | Ordered | Typical Contents
  --------------|---------|---------| ----------------------
  `list`        |      ✓  |   ✓     |   Mixed data
  `tuple`       |      ✗  |     ✓   |   Mixed data
  `dict`        |      ✓  |    ✓\*  |  Key/value pairs
  `set`         |      ✓  |   ✗     |  Unique elements
  `bytes`       |      ✗  |    ✓    |   8-bit binary data
  `bytearray`   |      ✓  |    ✓     |    Mutable binary data
  `array.array` |      ✓  |      ✓   |    Uniform numeric data
\*Insertion ordered in modern Python (3.7+).

## Lists

Lists are:

-   ordered
-   indexable
-   mutable
-   able to store mixed data types
-   commonly nested to represent 2D and 3D data

Common methods:

-   `append()`
-   `pop()`
-   `insert()`
-   `remove()`
-   `reversed()`

### Examples

``` python
my_list = ["a", "b", "c", "d"]

print(my_list[2])
# C
```

``` python
my_list.append("e")
print(my_list)
```

``` python
zeros = 100 * [0]
zeros = [0 for _ in range(100)]
```

``` python
numbers = list(range(100))
```

## Tuples

Tuples are:

-   ordered
-   indexable
-   immutable
-   frequently used for returning multiple values from functions

``` python
def getAccelXYZ():
    return (x, y, z)

accel = getAccelXYZ()

x, y, z = getAccelXYZ()
```

Python also allows swapping values without a temporary variable.

``` python
x, y = y, x
```

## Dictionaries

Dictionaries:

-   are indexed by key
-   require unique keys
-   require immutable keys
-   associate every key with a value

``` python
my_dict = {
    "one": 1,
    "two": 2,
    "three": 3,
}

my_dict["four"] = 4
```

## Sets

Sets are:

-   mutable
-   unordered
-   not indexable
-   intended primarily for membership testing
-   automatically eliminate duplicates

``` python
char_in = input()

if char_in in {"Y", "y", "yes"}:
    pass
```

## Bytes and Bytearrays

`bytes` objects:

-   store compact binary data
-   are immutable
-   commonly represent ASCII characters

``` python
with serial.Serial("COM4", baudrate=115200) as ser:
    print(ser.read(1))
```

Typical output:

``` python
b"y"
```

The leading `b` indicates a bytes object.

`bytearray` behaves similarly but is mutable.

### ASCII

ASCII represents characters as integer byte values.

Examples:

``` python
char = ser.read(1)

if char.decode() == "G":
    ...

if char[0] == 71:
    ...

if char == b"G":
    ...
```

All three expressions test for the same character.

----------------------------------------------------------------------

## `array.array`

Unlike lists, arrays store only a single numeric type.

Advantages:

-   compact storage
-   predictable memory usage
-   efficient numeric operations

Typecodes:

| Unsigned            | Signed                        | Data size       |
| ------------------- | ----------------------------- | --------------- |
| B (0 to 255)        | b (-128 to 127)               | byte (8 bits)   |
| H (0 to 65535)      | h (-32768 to 32767)           | short (16 bits) |
| I (0 to 65535)      | i (-32768 to 32767)           | int (16 bits)   |
| L (0 to 4e9)        | l (-2e9 to 2e9)               | long (32 bits)  |
| Q (0 to 1.8e19)     | q (-9e18 to 9e18)             | quad (64 bits)  |
|                     | f (-inf to inf)               | float (32 bits) |
|                     | d (-inf to inf)               | double\* (64 bits) |
\* On most STM32 family microcontrollers there is no hardware support for double precision floating point numbers. The STM32L476RG used in ME 4305 does not support double precision floating point arithmetic. 

Example:

``` python
import array

my_array = array.array("H", 1000 * [0])
```

Type code `"H"` creates an unsigned 16-bit integer array.


------------------------------------------------------------------------

# Summary

-   Prefer `with` when opening files.
-   Python provides several built-in collection types, each optimized
    for different use cases.
-   Choose the collection based on mutability, ordering, indexing, and
    storage efficiency.
-   Binary communication in embedded systems frequently uses `bytes` and
    `bytearray`.
-   `array.array` is preferable to `list` when storing large homogeneous
    numeric datasets.

------------------------------------------------------------------------

# Related Topics

- [[lecture_01_philosophy_of_mechatronics_and_python_fundamentals|Lecture 1 - Philosophy of Mechatronics and Python Fundamentals]]
- [[Object-Oriented Programming]]
- [[Finite State Machines]]
- [[MicroPython]]
- [[Real-Time Systems]]