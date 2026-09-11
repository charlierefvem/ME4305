---
title: Mutability
type: reference
status: dirty
---

In Python, **every variable is actually a pointer** under the hood. When you pass an argument into a function, Python **passes that pointer by value** (copying the memory address into a local scope, just like moving an address into a register).

Because the function gets a copy of the pointer, the behavior depends entirely on the object's mutability:

- **Mutable objects (like lists):** The local pointer still points to the original memory block. If you use a method to modify the object, you are dereferencing that pointer and altering the original data in place.
- **Immutable objects (like ints or strings):** The memory block is read-only. Any operation that seems to 'change' it actually allocates a brand-new memory block and updates your local pointer to point to the new address, leaving the caller's original pointer untouched.