---
title: Review of Signed Numbers
type: topic
tags:
  - binary
  - hexadecimal
  - twos-complement
  - sign-extension
  - bit-manipulation
source:
  course: ME405
  term: 2262
  lecture: 15
status: draft
---

# Motivation

When working with sensors and embedded hardware, the data we read is usually stored as raw bytes. The hardware does not send us "the number -77" in decimal notation. Instead, it sends a sequence of bits, and our code must interpret those bits correctly.

This lecture reviews several tools that matter when reading binary sensor data:
* Two's Complement representation for signed integers
* Sign Extension when converting between integer widths
* Bit shifting and bit masking for extracting packed fields
* `memoryview` objects for working with mutable buffers
* The `struct` module for packing and unpacking binary data
These topics show up naturally when working with- I2C sensors such as the BNO055 IMU used in lab.

# Signed Numbers in Binary

In decimal notation, we usually write negative numbers using a **signed magnitude** format. For example, in the number `-23`, the minus sign represents the sign and the digits `23` represent the magnitude.

In binary notation, signed integers are usually stored using **two's complement** format. In this format, the most significant bit (MSB) encodes the sign, but it is also part of the value of the number.

For an 8-bit signed number:

* `MSB = 0` means the number is positive.
* `MSB = 1` means the number is negative.

## The Two's Complement Operation

The two's complement of a binary number can be found in two steps:

1. Find the one's complement by flipping every bit.
2. Add `1` to the result.

For example, to negate an 8-bit value, first flip each bit and then add one. This is not just a visual trick; it is the operation that makes addition and subtraction work consistently for signed binary integers.

## Interpreting Two's Complement Values

To interpret a two's complement number, first look at the MSB. If the MSB is `0`, the value is positive and can be interpreted like an ordinary unsigned binary number. If the MSB is `1`, the value is negative.

For example, consider the 8-bit value

```text
0b10110011
```

Since the MSB is `1`, the number is negative. To find the magnitude, take the two's complement:

```text
original:  10110011
1's comp:  01001100
2's comp:  01001101
```

The magnitude is

$$
64 + 8 + 4 + 1 = 77
$$

Therefore,

$$
0b10110011 = -77
$$
when interpreted as an 8-bit signed integer.

## Two's Complement Ranges

The range of a two's complement signed integer depends on the number of bits used.

For an 8-bit signed integer:

| Hex value | Signed value |
|---:|---:|
| `0x00` | `0` |
| `0x7F` | `127` |
| `0x80` | `-128` |
| `0xFF` | `-1` |

For a 16-bit signed integer:

| Hex value | Signed value |
|---:|---:|
| `0x0000` | `0` |
| `0x7FFF` | `32767` |
| `0x8000` | `-32768` |
| `0xFFFF` | `-1` |

This is why the number of bits matters. The same pattern of bits can represent a different number depending on whether it is treated as an 8-bit, 16-bit, or 32-bit signed integer.

# Sign Extension

The hardware has no idea whether a collection of bits represents an 8-bit, 16-bit, or 32-bit number. That interpretation is entirely determined by the software. The application of two's complement depends on the number of bits used to represent an integer because the MSB is the sign bit. The operation of extending the number of bits used to represent a number without changing its value is called **sign extension**.

Suppose a signed 16-bit number is assigned to a 32-bit number. The 16-bit integer must be sign-extended so that the 32-bit number has the correct value.

For example, suppose the 16-bit value is

```text
0b1100 1001 0010 1110
```

This value is negative because bit 15, the MSB of a 16-bit value, is `1`.

Before sign extension, a 32-bit container might hold the value as

```text
0b0000 0000 0000 0000 1100 1001 0010 1110
```

This is just the original 16-bit value with 16 extra zeros in front. However, that does **not** preserve the signed value. To sign-extend the number correctly, the sign bit must be copied into bits 16 through 31:

```text
0b1111 1111 1111 1111 1100 1001 0010 1110
```

## Method One: Logic

In Python, one direct way to sign-extend a 16-bit value is to check whether it is in the range that represents negative numbers before sign extension:

```python
if value > 32767:
    value -= 65536
```

Before sign extension, negative 16-bit values will appear in the unsigned range from `32768` to `65535`. To correct these values to the signed range from `-32768` to `-1`, subtract

$$
2^{16} = 65536
$$

## Method Two: Bit Manipulation

A more C-style approach uses bit manipulation:

```python
mask = 1 << 15    # Create a mask with 1 in b15, the sign bit for a 16-bit int
value = (value ^ mask) - mask
```

The XOR of the value and the mask toggles bit 15.
* If the number was originally positive, bit 15 toggles from `0` to `1`; subtracting the mask then toggles the bit back to `0`, having no overall effect.
* If the number was originally negative, bit 15 toggles from `1` to `0`; subtracting the mask then causes a borrow from bit 16. Since bit 16 is `0`, the borrow propagates upward through the higher bits, giving the correct negative value.

In a previous topic on I2C, `struct.unpack("<hhh", buf)` converted six bytes into three signed 16-bit integers. This lecture topic explains why those integers have the correct positive and negative values and why sign extension preserves those values when larger integer types are used.
# Summary

In this lecture, we reviewed signed numbers in binary and focused on how two's complement encodes negative values. We then used that idea to motivate sign extension, where the sign bit must be copied when a signed integer is widened from one bit-width to another.

Two's complement is not just a way of writing negative numbers. It is the convention that lets the same hardware addition circuits work for both positive and negative integers.

Sign extension is the operation that preserves the value of a signed integer when increasing the number of bits used to represent it. For a positive value, this means extending with zeros. For a negative value, this means extending with ones.

# See Also:
* [[topic_collections|Collections]]
* [[topic_inertial_measurement_units|Introduction to IMUs]]
* [[reference_memoryviews|Buffers and memoryview Objects]]