---
title: Formatting Strings
type: reference
tags:
  - f-strings
source:
  course: ME405
  lecture: 12
  term: 2262
status: draft
---

# String Formatting

Python provides two common ways to create formatted strings:

-   **f-strings** (recommended for most new code)
-   **the `.format()` method** (common in older code)

Both use the same **format specifiers**, so once you learn the specifiers you can use either approach.

Formatting strings is especially useful for telemetry, debugging, logging sensor data, generating CSV output, displaying values with units, and creating clear user interfaces.

## Format Specifiers

A **format specifier** appears after a colon (`:`) inside the braces and controls **how a value is displayed** without changing the value itself. You can think of a specifier as a small set of building blocks. A useful beginner pattern is:

``` text
:[[fill]align][width][,][.precision][type]
```

Each part (enclosed by square brackets in the string above) is optional, so most specifiers use only the pieces they need.

| Component    | Position in the specifier | Meaning                                       | Common values                   |
| ------------ | ------------------------- | --------------------------------------------- | ------------------------------- |
| `fill`       | Before `align`            | Character used to pad extra space             | space, `0`, `.`                 |
| `align`      | Near the beginning        | How the value is positioned in the field      | `<` left, `>` right, `^` center |
| `width`      | Before precision and type | Minimum number of characters to display       | `8`, `10`, `12`                 |
| `,`          | Before precision and type | Add thousands separators                      | `,`                             |
| `.precision` | Before type               | Number of digits after the decimal for floats | `.2`, `.3`, `.4`                |
| `type`       | Last                      | The display style for the value               | `f`, `e`, `d`, `%`, `s`         |

The **width** before the dot is the minimum field width, while the **precision** after the dot represents the number of digits to include after the decimal point for floating-point values. If the formatted value is wider than the requested width, Python automatically expands the field.

## Example 1

For this example, `{:8.2f}` combines width `8`, precision `.2`, and type `f`.

``` python
my_pi = 3.14159

print(repr(f"{my_pi:8.2f}"))
```
`Output`:
``` text
'    3.14'
```
The output includes four leading spaces to pad the string to a width of eight, as specified.

Note that the function `repr()` used in the example makes the output display as a string representation (with apostrophes shown) rather than as text.

## Example 2

This next example will show a few common string formats using the `.format()` method.

``` python
x = 3.14159

print(repr("{:.2f}".format(x)))
print(repr("{:,}".format(12345)))
print(repr("{:.1%}".format(0.875)))
```
`Output`:
```text
'3.14'
'12,345'
'87.5%'
```

It is possible to mix fixed string data with formatted values.

``` python
voltage = 12.37
current = 1.52

print(repr("V = {:.2f} V".format(voltage)))
print(repr("I = {:.2f} A".format(current)))
```
`Output`:
```text
'V = 12.37 V'
'I = 1.52 A'
```

## Example 3

This example is the same as Example 2, but uses f-strings instead of `.format`.

``` python
x = 3.14159

print(repr(f"{x:.2f}"))
print(repr(f"{12345:,}"))
print(repr(f"{0.875:.1%}"))
```
`Output`:
```text
'3.14'
'12,345'
'87.5%'
```

```python
voltage = 12.37
current = 1.52

print(repr(f"V = {voltage:.2f} V"))
print(repr(f"I = {current:.2f} A"))
```
`Output`:
```text
'V = 12.37 V'
'I = 1.52 A'
```

### Example 4

This final example is more practical and shows how you might create a set of data and print that data in a comma separated format. Note the use of list comprehensions to compactly create time and data values.

``` python
from math import pi, sin

times = [t * 0.1 for t in range(11)]
vals = [sin(2*pi * t) for t in times]

for time, val in zip(times, vals):
    print(f"{time:.2f},{val:.4f}")
```
`Output`:
``` text
0.00,0.0000
0.10,0.5878
0.20,0.9511
0.30,0.9511
0.40,0.5878
0.50,0.0000
0.60,-0.5878
0.70,-0.9511
0.80,-0.9511
0.90,-0.5878
1.00,-0.0000
```

# Summary

Both **f-strings** and **`.format()`** use the same format specifiers. In modern Python, f-strings are generally preferred because they are concise and easy to read, but `.format()` is still common in existing code. Sometimes `.format()` is still preferred because it allows you to create reusable format strings treated as constants that can be formatted during runtime.

Start by learning a few common specifiers such as `.2f`, `.3e`, and `d`; as your programs become more sophisticated, you can explore the complete format specification mini-language in the official Python documentation.
