---
title: Buffers and memoryview Objects
type: reference
tags:
source:
  course: ME405
  term: 2262
  lecture: 15
status: draft
---

## Buffers

Various modules, including the `struct` module, have a few extra features that are useful for writing cleaner code when working with buffered data.

* `struct.pack_into()` and `struct.unpack_from()`
    These methods let you pack values into, and unpack values from, `bytearray` or buffer objects that already exist.
* `struct.calcsize()`
    This method calculates the number of bytes needed to store values for a given format string. Knowing the required buffer size lets us use a `memoryview` object to slice a larger buffer down to the number of bytes needed for the current transaction.

Compare the following two snippets of code that achieve the same thing through different means.

```python
buf = my_i2c.mem_read(6, dev_addr, mem_addr)
```

```python
buf = bytearray(6)
my_i2c.mem_read(buf, dev_addr, mem_addr)
```

The first line reads data and returns it in a new `bytes` object. The bottom two lines create a reusable buffer and then read data into it directly. Optimizations like this, while small individually, can really add up, especially for code that is in the **hot path** (code that must run very often, like a control loop). Therefore, in firmware programming methods that **read into** a buffer will be preferred to methods that read and return a bytes object.

## Slices

A **slice** selects a portion of a sequence such as a string, list, tuple, or `bytearray`. Slices use the general form:

```python
sequence[start:stop:step]
```

All three values are optional.

|Component|Meaning|Default|
|---|---|---|
|`start`|Index of the first element to include|Beginning of the sequence|
|`stop`|Index where the slice ends (**not included**)|End of the sequence|
|`step`|Number of elements to skip between values|`1`|

Python uses **zero-based indexing**, so the first element has index `0`. The `stop` index is **exclusive**, meaning it is not included in the result.

#### Example 1

```python
text = "Engineering"

text[0:4]     # 'Engi'
text[4:]      # 'neering'
text[:6]      # 'Engine'
text[::2]     # 'Egneig'
text[::-1]    # 'gnireenignE'
```

Negative indices count backward from the end of the sequence. For example, `text[-1]` is the last character, and `text[:-1]` returns everything except the last character.

## `memoryview` Objects

A `memoryview` object provides mutable access to another buffer. These fit in nicely with the philosophy to prefer "read into" functions in favor of "read" functions. A `memoryview` can also be used to grant access to certain parts of a buffer to another function.

### Example 2

What will be the value of `buf` after the following code runs?

```python
buf = bytearray([0, 1, 2, 3, 4])

def mutate_last(b):
    """Sets last element of b to 5."""
    b[-1] = 5

b = buf[:3]       # Slice first three elements
mutate_last(b)

print(buf)
```
`Output`:
```python
bytearray(b'\x00\x01\x02\x03\x04')
```

If you pass standard slices of `bytearray` objects into functions and then mutate the slices, the changes will not reflect in the original `bytearray`. The slice creates a new object.

### Example 3

A `memoryview` object can be sliced and modified, and the changes will reflect in the original buffer.

```python
buf = bytearray([0, 1, 2, 3, 4])

def mutate_last(b):
    """Sets last element of b to 5."""
    b[-1] = 5

b = memoryview(buf)[:3]
mutate_last(b)

print(buf)
```
`Output`:
```python
bytearray(b'\x00\x01\x05\x03\x04')
```

The slice `memoryview(buf)[:3]` views the first three elements of the original buffer. When the last element of that view is changed to `5`, element `2` of the original `bytearray` changes.

**Insight**: This is *especially* useful in embedded code because a class can allocate one buffer and then create smaller views into that buffer as needed, rather than allocating new `bytearray` objects repeatedly. A `memoryview` object also allows API functions to place data directly into or take data directly from different portions of a buffer. This style of in-place operation can greatly benefit runtime performance by reducing dynamic memory allocation and therefore the burden on the **garbage collector**.

### Example 4

This example shows advanced usage of the `struct` module. 

The following example shows a simple BNO055 sensor class using small data structures to describe registers and their unpacking formats:

```python
from micropython import const
from struct import calcsize, unpack_from

class BNO055:
    DEV_ADDR = const(0x28)

    class reg:
        EUL_HEADING_MSB = (const(0x1B), b"<B")
        EUL_HEADING_LSB = (const(0x1A), b"<B")
        EUL_HEADING = (const(0x1A), b"<h")
        EUL_DATA_ALL = (const(0x1A), b"<hhh")

    def __init__(self, i2c):
        self._i2c = i2c
        self._buf = bytearray((0 for n in range(22)))

    def _read_reg(self, reg):
        # Determine number of bytes to read
        length = calcsize(reg[1])

        # Create a memoryview object of the right size
        buf = memoryview(self._buf)[:length]

        # Read from the I2C bus into the memoryview
        self._i2c.mem_read(buf, self.DEV_ADDR, reg[0])

        # Unpack the bytes into a tuple
        return unpack_from(reg[1], buf)

    def euler(self):
        head, roll, pitch = self._read_reg(BNO055.reg.EUL_DATA_ALL)
        return (head / 16, roll / 16, pitch / 16)
```

## Summary

* Slices allow advanced indexing of collections.
* `memoryview` objects are useful because they allow a function to operate on a selected part of a buffer without copying that part into a new object.