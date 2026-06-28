---
title: Collections
type: lecture
topics:
  - Python
  - Collections
tags:
  - python
  - collections
source: ME405-2262 Lecture 2
status: draft
---

# Motivation

This lecture introduces Python's built-in collection types. These tools form the foundation for future work.

> **Candidate static notes**
> -   \[\[Python Lists\]\]
> -   \[\[Python Tuples\]\]
> -   \[\[Python Dictionaries\]\]
> -   \[\[Python Sets\]\]
> -   \[\[Python Bytes\]\]
> -   \[\[Python Arrays\]\]
> -   \[\[Mutability\]\]
> -  \[\[Comprehensions\]\]

# Python Collection Types

A collection is any Python object that stores multiple pieces of data in a single variable.

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

**Note:** mutability is a nuanced concept in Python we will return to several times in the term. For this early point in the term, you should think of *immutable* variables as read-only and *mutable* variables as modifiable. A more precise definition will come shortly in future notes.

## Lists (`list` Objects)

Lists are the most common or standard type of collection due to their rich feature set and convenience, however they are not always the best choice for reasons outlined in these notes below.

Lists are:
* ordered
* indexable
* mutable
* able to store mixed data types
* commonly nested to represent 2D and 3D data

Common methods:
* `append()`
* `pop()`
* `insert()`
* `remove()`
* `reversed()`

### Example 1

``` python
my_list = ["a", "b", "c", "d"]
print(my_list[2])
```
Output:
```text
c
```

### Example 2

``` python
my_list = ["a", "b", "c", "d"]
my_list.append("e")
print(my_list)
```
Output:
```text
['a', 'b', 'c', 'd', 'e']
```

### Example 3

This example shows two ways to create a list of 100 zeros. Both produce equivalent lists, but the second approach using a list comprehension generalizes more naturally to computed values. Comprehensions are good examples of high-level Python features that should be used strategically. As with other advanced topics, we will revisit comprehensions later in the quarter.

``` python
zeros = 100 * [0]
zeros = [0 for _ in range(100)]
```

### Example 4

``` python
numbers = list(range(100))
```

## Tuples (`tuple` Objects)

Python tuples are immutable lists. That means once they are created the individual items of the tuple cannot be modified. Tuples are less versatile than lists, but often immutability is the intended goal when using a tuple so the rich features of lists are unwanted anyway.

Tuples are
* ordered
* indexable
* immutable

### Example 5

A common use-case for a tuple is returning multiple values from a function. In this example a dummy function returns the x-, y-, and z-components of an acceleration vector measured from an accelerometer.

``` python
def getAccelXYZ():
	# Query sensor
    return (x, y, z)

# Assign output to a new tuple
accel = getAccelXYZ()

# Unpack the tuple into separate variables
x, y, z = getAccelXYZ()
```

### Example 6

Python's tuple unpacking also allows swapping values without needing a temporary variable.

``` python
x, y = y, x
```

## Dictionaries (`dict` Objects)

In high-level languages like Python, more advanced collection types are built directly into the language. A dictionary is a mapping between a set of keys and a set of values optimized for quick lookup by key rather than by numerical index as would be done with a list or tuple.

Dictionaries:
* are indexed by key
* require unique and immutable keys

### Example 7

``` python
my_dict = {
    "one": 1,
    "two": 2,
    "three": 3,
}

# Append to the dictionary
my_dict["four"] = 4
```

## Sets (`set` Objects)

The set is one of the less used collection types in Python, but it has an important niche: checking for membership. While any type of collection can be checked for membership, it may take a considerable amount of time to check for membership in unoptimized collection types. For example, to check membership in a list or tuple, Python must examine and check each object in the list or tuple, which means lookup time scales with the length of the collection. Sets are optimized for membership checks such that lookup time doesn't scale with the size of the set.

Sets only work with unique values, so redundant entries are automatically eliminated.

Sets are:
* mutable
* unordered (not indexable)

### Example 8

``` python
char_in = input()

if char_in in {"Y", "y", "yes"}:
    pass
```

## `bytes` and `bytearray` Objects

The preceding collections can store a variety of Python object types, making them flexible, but they are not optimized for memory efficiency. For example, a list requires at least 4 or 8 additional bytes of overhead *per item* to store metadata about the list, depending on whether the system uses 32-bit or 64-bit pointers.

`bytes` objects:
* store compact binary data
* are immutable
* commonly represent ASCII characters

### Example 9

``` python
with serial.Serial("COM4", baudrate=115200) as ser:
    print(ser.read(1))
```
Output:

``` python
b"y"
```

The leading `b` indicates a bytes object.

`bytearray` behaves similarly but is mutable.

### ASCII

ASCII (American Standard Code for Information Interchange) is a character encoding that defines standard characters, like those on a keyboard, as integer byte values. That is, ASCII is simply a convention for representing characters using numbers.

### Example 10

The following example reads a single character from a serial port and then shows three methods for checking if the received character is an upper case "G", which has an ASCII equivalent decimal value of 71.

``` python
char = ser.read(1)

# Check by decoding bytes object to str object
if char.decode() == "G":
    ...

# Check by indexing bytes object to retrieve numerical value of the character
if char[0] == 71:
    ...

# Check by comparing directly to another bytes object
if char == b"G":
    ...
```

All three expressions test for the same character.

## Arrays (`array.array` Objects)

Unlike lists, `array.array` objects, commonly referred to as arrays, store only uniform numeric data; that is, all items must be of the same numeric type. At the time of creation, a typecode must be specified to define the specific numeric type that the array holds.

Advantages:
* compact storage
* predictable memory usage
* efficient numeric operations

### Typecodes:

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

### Example 11

``` python
import array

my_array = array.array("H", 1000 * [0])
```

Type code `"H"` creates an unsigned 16-bit integer array.


------------------------------------------------------------------------

# Summary

* Python provides several built-in collection types, each optimized for different use cases. Always select the most appropriate type.
* Choose the collection based on mutability, ordering, indexing, and storage efficiency.
* Binary communication in embedded systems frequently uses `bytes` and `bytearray` objects.
* Arrays (`array.array` objects) are preferable to lists when storing large homogeneous numeric datasets.