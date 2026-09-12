---
title: Mutability
type: reference
tags:
  - python
  - programming
  - data-structures
source:
  course: ME4305
status: draft
---

## Objects, Names, and Mutability

Python represents data as **objects**. An object has a type, an identity, and a value. A variable name does not contain an object; instead, the name is **bound** to an object. Assignment changes that binding:

```python
speed = 2.0
speed = 2.5
```

The second line binds `speed` to a different floating-point object. It does not modify the original `2.0` object.

**Mutability** describes whether an object's contents can change after the object is created:

- A **mutable** object can be changed in place while keeping the same identity.
- An **immutable** object cannot be changed in place. An operation that produces a different value must return another object.

> [!table]
> Common mutable and immutable Python types.
>
> | Usually mutable | Immutable |
> | --- | --- |
> | `list` | `int`, `float`, `bool` |
> | `dict` | `str`, `bytes` |
> | `set` | `tuple`, `frozenset` |
> | `bytearray` | `NoneType` (the type of `None`) |
> | Most class instances |  |

Mutability is a property of an object's **type**, not of the variable name used to access it. The same name can be rebound to objects of different types, and multiple names can refer to the same object.

## Mutation Versus Rebinding

The distinction between mutation and rebinding explains most behavior that initially seems surprising.

```python
samples = [10, 20, 30]
samples.append(40)          # Mutates the existing list
samples[0] = 5              # Also mutates the existing list

label = "left"
label = label.upper()       # Creates a string, then rebinds label
```

List methods such as `append()` and item assignment change the existing list. Strings are immutable, so `upper()` returns another string. The assignment then binds `label` to that result.

Rebinding one name never changes what another name refers to. Mutating a shared object, however, makes the change visible through every name bound to that object.

## Aliasing: Multiple Names for One Object

Assignment does not automatically copy an object:

```python
command = [0.2, 0.2]
saved_command = command

command[0] = 0.5

# command       == [0.5, 0.2]
# saved_command == [0.5, 0.2]
```

Both names refer to the same list. This situation is called **aliasing**. It is useful when different parts of a program should share state, but it can cause bugs when a programmer expects an independent snapshot.

The `is` operator tests whether two names refer to the same object. The `==` operator tests whether their values compare as equal:

```python
a = [1, 2]
b = a
c = [1, 2]

a == c    # True: equal contents
a is c    # False: different list objects
a is b    # True: the same list object
```

Use `is` mainly for identity checks such as `value is None`. Use `==` when comparing values.

## Function Arguments

Calling a function binds each parameter name to the object supplied by the caller. Python does not copy the object merely because it was passed to a function.

```python
def add_measurement(data, value):
    data.append(value)       # Mutates the caller's list


measurements = [1.1, 1.2]
add_measurement(measurements, 1.3)

# measurements == [1.1, 1.2, 1.3]
```

Rebinding a parameter affects only that local name:

```python
def replace_measurements(data):
    data = [0.0, 0.0]        # Rebinds the local name only


measurements = [1.1, 1.2]
replace_measurements(measurements)

# measurements is still [1.1, 1.2]
```

This model is sometimes called **call by object reference** or **call by sharing**. It is more accurate than saying that Python variables are pointers or that Python simply “passes by reference.” Python implementations manage object storage, and normal Python code should reason about names and objects rather than memory addresses.

> [!insight]
> A function can mutate an object received from its caller, but rebinding a parameter does not rebind the caller's variable name. The function should make intentional mutation clear through its name, documentation, or return value.

## Copying Mutable Objects

Create a copy when subsequent changes should be independent. Common mutable containers provide a `copy()` method, and a full list slice also creates a new list:

```python
original = [10, 20, 30]
snapshot = original.copy()   # The same as original[:]

original[0] = 5

# original == [5, 20, 30]
# snapshot == [10, 20, 30]
```

These are **shallow copies**. A shallow copy creates a new outer container but continues to share the objects inside it. That matters for nested mutable containers:

```python
original = [[1, 2], [3, 4]]
shallow = original.copy()

original[0].append(9)

# original == [[1, 2, 9], [3, 4]]
# shallow  == [[1, 2, 9], [3, 4]]
```

Use `copy.deepcopy()` when the nested mutable objects must also be independent:

```python
from copy import deepcopy

independent = deepcopy(original)
```

Deep copying can be expensive and may copy objects that were intended to be shared. Prefer constructing only the independent data that the program actually needs.

## Common Consequences

### Mutable Default Arguments

Default argument values are evaluated once when a function is defined, not each time it is called. A mutable default is therefore shared by every call that uses it:

```python
def record(value, history=[]):       # Avoid this pattern
    history.append(value)
    return history
```

Use `None` as a sentinel when each call should receive a new object:

```python
def record(value, history=None):
    if history is None:
        history = []
    history.append(value)
    return history
```

### Augmented Assignment

An augmented assignment such as `+=` may mutate an object or produce a new object, depending on its type:

```python
values = [1, 2]
alias = values
values += [3]              # The list is mutated in place
# alias == [1, 2, 3]

count = 2
count += 1                 # A new int is produced and count is rebound
```

Do not assume that `x += y` is always equivalent in effect to `x = x + y` when aliases may exist.

### Immutable Containers Can Hold Mutable Objects

A tuple cannot be changed to refer to different items, but an item inside the tuple may itself be mutable:

```python
channels = ([1.2, 1.3], "volts")
channels[0].append(1.4)    # Valid: mutates the contained list

# channels[0] = []         # TypeError: would modify the tuple
```

The tuple remains immutable because its direct references do not change. “Immutable” does not necessarily mean that every object reachable from a container is also immutable.

### Dictionary Keys and Set Elements

Dictionary keys and set elements must be **hashable**, which requires their hash value to remain stable. Immutable built-in types such as numbers, strings, and tuples of hashable items are common choices. Mutable lists, dictionaries, and sets cannot be used directly as dictionary keys or set elements.

## Why Mutability Matters in Engineering Code

Mutating an existing list, `bytearray`, or other buffer can avoid repeated allocation and can be appropriate in a fast control loop or communication driver. Shared mutable state can also let several parts of a program observe current measurements or commands.

The same sharing can create stale-snapshot bugs, unintended changes, and code whose behavior depends on which function ran most recently. Make the ownership of mutable state clear, copy data when a true snapshot is needed, and avoid allowing unrelated parts of a program to modify the same object without a deliberate interface.

For buffer-specific behavior, including views that intentionally share storage, see [[reference_memoryviews|Buffers and memoryview Objects]].

## Practical Guidelines

- Treat assignment as binding a name, not copying an object.
- Ask whether an operation mutates an existing object or returns a new one.
- Document functions that intentionally modify their arguments.
- Copy mutable inputs when the function must preserve a snapshot or work independently.
- Remember that `copy()` is shallow when containers are nested.
- Avoid mutable default arguments unless persistent shared state is explicitly intended.
- Use `is None` for a sentinel check and `==` for ordinary value comparison.
- Prefer immutable data when a value should not change after construction.

## Summary

Mutability is the ability of an object to change in place. Python names are bound to objects, so assigning one name to another does not automatically copy the object. If several names share a mutable object, a mutation is visible through all of them; rebinding one name is not. This distinction explains the behavior of function arguments, copies, mutable defaults, augmented assignment, and shared buffers.
