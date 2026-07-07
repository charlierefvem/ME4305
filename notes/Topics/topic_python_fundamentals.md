---
title: Python Fundamentals
type: topic
tags:
  - python
  - programming-fundamentals
source:
  course: ME405
  term: 2262
  lecture: 1
status: draft
---

## Motivation

Python is an extremely versatile high-level language. It is often a good choice for beginner programmers due to the ubiquity of the language; that is, skills learned in the context of firmware development will still lead to valuable and generally applicable growth.

## About Python

### Python is an Interpreted Language

Python is an interpreted language, as opposed to a compiled language. That is, Python code is not compiled before going to the hardware. Instead, a secondary program called the Python interpreter, at execution, interprets Python code line by line. During execution, the interpreter internally translates portions of the program into forms that can be executed efficiently. These implementation details are handled automatically and are not something programmers normally need to think about.

For our lab hardware in ME 4305, the rough workflow is:

```text
Python code that we write and store on the microcontroller as .py files
        ↓
MicroPython interpreter reads and compiles code at startup
        ↓
Compiled code runs on the microcontroller
        ↓
The hardware (hopefully) responds with the intended output behavior
```

The Micropython interpreter that runs on our lab hardware is itself a compiled program written mostly in C. Its job is to run the Python code that we write.

### Trivia

- The Python interpreter that runs on your computer, often referred to as CPython, was originally written in C, though it now also includes Python code.
- The Python programming language is named after Monty Python, not the animal.

### Big picture Python basics

In Python, whitespace matters. Indentation is used for structuring code, including loops and conditionals. Use four spaces per indentation level, not tabs. In some cases, spaces and newlines have specific meaning as well.

Python is fundamentally an object-oriented language. *Every* variable is an object of some kind. We will return to this idea later in the course when you are asked to write your own classes.

## Python Coding Examples
The best way to learn programming of any kind is to program! The following examples illustrate many of the fundamentals of the Python language.

In ME 4305 we will focus on two things while writing code:
1) What is the Pythonic (most idiomatic) approach? That is, how can we write code that uses Python as it was designed and intended to be used.
2) When should we deliberately violate or bend the idiomatic approach? Often in embedded programming the idiomatic approach, while easier to read an maintain, leads to poor performance on real hardware due to limited resources.
While these two foci compete with one another, we will aim to develop intuition about the resources available so that we may choose the right approach for a given task.

In the short-term, learning the absolute basics must come first.

### Example 1: `if` statements

Python `if` statements do not use parentheses around the condition because `if` is a keyword, not a function. Only add parentheses for clarification (grouping).

For an `if` statement, the colon starts an indented block; all code within the statement must exist within this indented block; unindenting tells the interpreter that the block of code has ended.

```python
x = 3

if x == 3:
    print("x is equal to 3")
```

Important syntax details:
* '=', the assignment operator, assigns values to an object.
* `==`, the equality operator, checks whether two values are equal.
* `:` starts the indented block.
* The indented code is the part that runs only if the condition is true.
* The indentation is *not* optional.

### Example 2: waiting for user input

The `input()` function pauses the program and waits for the user to type something. In most lab work you will need to write non-blocking code, however this function will remain useful in homework assignments.

```python
my_input = input("Enter a number: ")
my_num = float(my_input)
print(my_num)
```

Output:
```text
Enter a number: 42
42.0
```

The value returned by `input()` is a string. If you want to treat the result as a number, convert it explicitly. In this example, `float()` converts the input text to a floating-point number.

### Example 3: working with lists

A list can store multiple items and each item may be any kind of Python object. In most cases lists should be used to store data of a uniform type, but not always.

```python
my_list = []                  # create empty list
my_list = [1, 2, "three"]     # create list
```

A list is an object, so it has methods that operate on the list.

```python
my_list = [1, 2, "three"]

my_list.append("four")        # append a new item to the end of the list
print(my_list)

last_item = my_list.pop()     # remove and return the last item
print(last_item)
print(my_list)
```

Output:
```text
[1, 2, 'three', 'four']
four
[1, 2, 'three']
```

The method call

```python
my_list.append("four")
```

can be read as:

```text
object.method(argument)
```

That is, call the `append()` method on the object `my_list`, and pass in the item that should be appended.

### Example 4: string concatenation

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

### Example 5: string joining

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

### Example 6: iterating through a list

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
* `my_num` is the name we give each item while iterating.
* `my_list` is the iterable, or list-like object, being iterated through.

Lists can also be indexed both forward, in the traditional sense, and in reverse, using negative indices. For the example above:

```text
index:    0  →  1  →  2  →  3  →  4
value:    1     2     4     8     16
reverse: -5  ← -4  ← -3  ← -2  ← -1
```

### Example 7: iterating with indices

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

### Example 8: iterating through a list in reverse

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

### Example 9: using `range()`

The `range()` function is often used when an integer sequence is needed.

```python
my_list = [1, 2, 4, 8, 16]

for my_idx in range(len(my_list)):
    print(my_list[my_idx])
```

The expression `range(len(my_list))` produces indices from `0` through `N-1`, where `N` is the length of the list.

This pattern is generally avoided when directly iterating through the list would work, but it can be useful when list items need to be modified inside the loop. Standard iteration provides read-only access to list items, so iterating through a range and indexing the list allows the list items to be mutated.

### Example 10: using `break`

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

### Example 11: using `continue`

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

## Insights

### Indentation is syntax

In Python, indentation is not just style. It changes the meaning of the program.

Use four spaces per indentation level. Avoid mixing tabs and spaces.

### `==` is not `=`

Use `=` for assignment.

```python
x = 3
```

Use `==` for comparison.

```python
x == 3
```

### Prefer direct iteration when possible

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

### Methods belong to objects

The syntax

```python
my_list.append("four")
```

is an early example of object-oriented programming. The object is `my_list`; the method is `append()`.

## Summary

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

## See Also:
* [[topic_collections|Collections]]
* [[topic_classes_and_objects|Classes and Objects]]
* [[reference_mutability|Mutability]]