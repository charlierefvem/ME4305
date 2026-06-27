---
title: Lecture 1 - Philosophy of Mechatronics and Python Fundamentals
type: lecture
topics:
- mechatronics
- Python
- whitespace
- objects
- lists
- strings
- loops
tags:
- mechatronics
- python
- programming-fundamentals
---

# Motivation

This course is about mechatronic systems, not just mechanical systems with electronics attached.

A useful working definition is:

> We put the smarts in mechanical systems.

The "smarts" come from the overlap between several areas:

- mechanics
- electronics
- programming
- control theory

A successful mechatronic system requires all of these pieces to work together. In this course, Python is one of the main tools we will use to make that happen.

> TODO: Consider adding a clean version of the four-area sketch from lecture:
> mechanics, electronics, programming, and control theory.

# Concepts

## Python is an interpreted language

Python is an interpreted language, as opposed to a compiled language.

That is, Python code is not compiled directly before going to the hardware. The Python interpreter, at runtime, interprets Python code line by line and executes it.

For the lab hardware, the rough idea is:

```text
Python code that we write
        ↓
MicroPython interpreter
        ↓
compiled code running on the microcontroller
        ↓
hardware behavior
```

The MicroPython interpreter that runs on our lab hardware is itself a compiled program written in C and Assembly. Its job is to run the Python code that we write.

## Trivia

- The Python interpreter that runs on your computer was originally written in C, though it now also includes some Python code.
- The Python programming language is named after Monty Python, not the animal.

## Big picture Python basics

In Python, whitespace matters.

Indentation is used for structuring code, including loops and conditionals. Use four spaces per indentation level, not tabs.

In some cases, spaces and newlines have specific meaning as well.

Python is also fundamentally object oriented. Every variable is an object of some kind. We will return to this idea later in the course.

# Examples

## Example 1: `if` statements

Python `if` statements do not use parentheses around the condition by default. The colon starts the indented block.

```python
x = 3

if x == 3:
    print("x is equal to 3")
```

Important syntax details:

- `==` checks whether two values are equal.
- `:` starts the indented block.
- The indented code is the part that runs only if the condition is true.
- The indentation is not optional.

## Example 2: waiting for user input

The `input()` function pauses the program and waits for the user to type something.

```python
my_input = input("Enter a number")
my_num = float(my_input)

print(my_num)
```

The value returned by `input()` is a string. If you want to treat the result as a number, convert it explicitly. In this example, `float()` converts the input text to a floating-point number.

## Example 3: working with lists

A list can store multiple values.

```python
my_list = []                  # create empty list
my_list = [1, 2, "three"]     # create list
```

A list is an object, so it has methods that operate on the list.

```python
my_list = [1, 2, "three"]

my_list.append("four")        # append to end
print(my_list)

last_item = my_list.pop()     # remove and return last item
print(last_item)
print(my_list)
```

The method call

```python
my_list.append("four")
```

can be read as:

```text
object.method(argument)
```

That is, call the `append()` method on the object `my_list`, and pass in the thing that should be appended.

## Example 4: string concatenation

The `+` operator can be used with numbers, but it can also be used with strings. In that case, it concatenates the strings.

```python
str1 = "cat"
str2 = "dog"

print(str1 + str2)
```

Output:

```text
catdog
```

This is an example of an overloaded operator: the meaning of `+` depends on the types of the objects it is applied to.

## Example 5: string joining

The `join()` method is often a cleaner way to combine strings from a list.

```python
my_list = ["cat", "dog"]

print("-".join(my_list))
```

Output:

```text
cat-dog
```

In this example, the string `"-"` is inserted between each item in `my_list`.

## Example 6: iterating through a list

A Python `for` loop can iterate directly through the items in a list.

```python
my_list = [1, 2, 4, 8, 16]

for my_num in my_list:
    print(my_num)
```

Output:

```text
1
2
4
8
16
```

In this example:

- `my_num` is the name we give each item while iterating.
- `my_list` is the iterable, or list-like object, being iterated through.

Lists can also be indexed. For the example above:

```text
index:    0   1   2   3   4
value:    1   2   4   8   16
reverse: -5  -4  -3  -2  -1
```

## Example 7: iterating with indices

Sometimes we need both the index and the value. In that case, use `enumerate()`.

```python
my_list = [1, 2, 4, 8, 16]

for idx, my_num in enumerate(my_list):
    print(idx, my_num)
```

Output:

```text
0 1
1 2
2 4
3 8
4 16
```

The `enumerate()` function converts a list into index-value pairs.

For example:

```python
my_list = [1, 2, 4]

print(list(enumerate(my_list)))
```

Output:

```text
[(0, 1), (1, 2), (2, 4)]
```

## Example 8: iterating through a list in reverse

The `reversed()` function lets us iterate through a list in reverse order.

```python
my_list = [1, 2, 4, 8, 16]

for my_num in reversed(my_list):
    print(my_num)
```

Output:

```text
16
8
4
2
1
```

## Example 9: using `range()`

The `range()` function is often used when an integer sequence is needed.

```python
my_list = [1, 2, 4, 8, 16]

for my_idx in range(len(my_list)):
    print(my_list[my_idx])
```

The expression `range(len(my_list))` produces indices from `0` through `N-1`, where `N` is the length of the list.

This pattern is generally avoided when directly iterating through the list would work, but it can be useful when list items need to be modified inside the loop.

## Example 10: using `break`

The `break` statement quits the entire loop early.

```python
for my_num in range(5):
    print(my_num)

    if my_num == 1:
        break
```

Output:

```text
0
1
```

The `break` statement exits the most local loop if loops are nested.

## Example 11: using `continue`

The `continue` statement quits the present iteration and resumes looping with the next item.

```python
for my_num in range(3):
    if my_num == 1:
        continue

    print(my_num)
```

Output:

```text
0
2
```

The loop still runs through the full range, but the `print()` statement is skipped when `my_num` is equal to `1`.

# Insights

## Indentation is syntax

In Python, indentation is not just style. It changes the meaning of the program.

Use four spaces per indentation level. Avoid mixing tabs and spaces.

## `==` is not `=`

Use `=` for assignment.

```python
x = 3
```

Use `==` for comparison.

```python
x == 3
```

## Prefer direct iteration when possible

This is usually preferred:

```python
for my_num in my_list:
    print(my_num)
```

This is sometimes necessary, but should not be the first choice:

```python
for my_idx in range(len(my_list)):
    print(my_list[my_idx])
```

Use the index-based form when the index itself matters or when list items need to be modified inside the loop.

## Methods belong to objects

The syntax

```python
my_list.append("four")
```

is an early example of object-oriented programming. The object is `my_list`; the method is `append()`.

# Summary

Mechatronics combines mechanics, electronics, programming, and control theory to put the "smarts" into mechanical systems.

Python is the primary programming language used early in this course. It is interpreted, indentation-sensitive, and object-oriented.

The most important Python ideas introduced in this lecture were:

- indentation and whitespace
- `if` statements
- user input with `input()`
- type conversion with `float()`
- lists
- list methods such as `append()` and `pop()`
- string concatenation and joining
- iteration with `for`
- `enumerate()`
- `reversed()`
- `range()`
- `break`
- `continue`

# Related Topics

- [[Object-Oriented Programming]]
- [[Finite State Machines]]
- [[MicroPython]]
- [[Real-Time Systems]]

# Candidate Static Notes

These are concepts from this lecture that may eventually deserve short reusable reference notes.

- [[What is Mechatronics]]
- [[Interpreted vs Compiled Languages]]
- [[Python Whitespace]]
- [[Python Lists]]
- [[Python Loops]]
- [[Python Objects]]
