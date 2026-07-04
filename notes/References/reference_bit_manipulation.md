---
title: Bit Manipulation Techniques
type: reference
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

# Bit Manipulation

While relatively infrequent in Python code it is sometimes necessary to interact directly with binary values and manipulate individual bits with a byte or larger integer.

## Bit Masks

A **bit mask** is a binary value used to examine or modify individual bits within an integer. Bit masks are commonly used in communication protocols, hardware registers, and digital I/O.

A mask is usually written in **binary** (`0b...`), in **hexadecimal** (`0x...`), or built through **bit shifting** (`1 << 2 | 1 << 4`).

| Operation     | Operator                    | Purpose                             |
| ------------- | --------------------------- | ----------------------------------- |
| Set bit(s)    | <code>value \|= mask</code> | Turn selected bits **on**           |
| Clear bit(s)  | `value &= ~mask`            | Turn selected bits **off**          |
| Test bit(s)   | `value & mask`              | Check whether selected bits are set |
| Toggle bit(s) | `value ^= mask`             | Flip selected bits                  |

### Example 1

```python
value = 0b0101

BIT0 = 0b0001
BIT1 = 0x02
BIT2 = 1 << 2

value |= BIT1      # Set bit 1
# value == 0b0111

value &= ~BIT2     # Clear bit 2
# value == 0b0011
```

A mask has **1s** wherever you want an operation to occur and **0s** everywhere else. The `|=` operator leaves existing 1s unchanged while turning the masked bits on. To clear bits, the `~` operator inverts the mask so that `&=` preserves every bit except those selected by the original mask.

### Example 2

In this example the BNO055 calibration status byte will be decoded.

Some sensor registers pack several small fields into a single byte. For example, the BNO055 calibration status register stores four 2-bit status values in one byte:

```text
 b7                   b0
+-----+-----+-----+-----+
| SYS | GYR | ACC | MAG |
+-----+-----+-----+-----+
```

Each field is two bits wide. The least significant two bits store the magnetometer status, the next two bits store the accelerometer status, the next two bits store the gyroscope status, and the most significant two bits store the system status.

The following MicroPython implementation splits reads from the register and divides it into four "crumbs" (2-bit numbers).

```python
buf = bytearray(1)

# Read one byte from the BNO055 calibration status register.
my_i2c.mem_read(buf, dev_addr, mem_addr)

mag_stat = buf[0] & 0b11
acc_stat = (buf[0] >> 2) & 0b11
gyr_stat = (buf[0] >> 4) & 0b11
sys_stat = (buf[0] >> 6) & 0b11
```

The pattern is:
1. Shift the desired bitfield to the right until it is aligned with the least significant bits.
2. Mask with `0b11` to clear every other bit.
The mask `0b11` is useful here because each status field is exactly two bits wide.

# Summary

Bit manipulation is often about two steps: move the desired field into position, then mask away everything else. In the BNO055 calibration example, each field is shifted until it is right-aligned and then masked with `0b11`.