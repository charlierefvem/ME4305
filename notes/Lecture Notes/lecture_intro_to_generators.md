---
title: Introduction to Generator Functions
type: lecture
tags:
  - python
  - micropython
topics:
  - Generators
source: ME405-2262 Lecture 8
status: draft
---

# Motivation

In ME 4305 we use cooperative multitasking to approximate concurrency between multiple tasks. To do this we need a way to write code that is easily divided so that the code can stop, allow other code to run, and resume operation seamlessly. This functionality can be implemented using functions and classes, but Python has a built-in tool that handles this elegantly: the generator function.

# Generator Functions

Standard functions, as you are already familiar with, take in one set of input parameters, run operations on those parameters, and then return one set of output parameters. That is, the functions run "all at once".

Generator functions are similar, but allow the operations to be split apart into multiple runs so that the function may sequentially generate output. That is, once a generator object is created, each time it is invoked (iterated upon) it will only produce, or `yield`, a single element of a sequence. These sequences can be finite or infinite in length. If the sequence ends, the generator can optionally `return` a final value or a result from running an algorithm or process.

Since generator functions are to be re-entered, they maintain internal state automatically, differing from a standard function. That is, once a generator object is created, it will maintain all of its internal variables until it is done generating output.

When the function is done yielding and ends or hits a `return` statement, a `StopIteration` exception is raised by the interpreter. This exception is raised to inform the caller that the sequence has ended; this is usually handled automatically but it will depend on how the generator is utilized.

## Example 1

The function below shows the basic structure of a generator function.

``` python
def my_generator_function(param_1, param_2)
    
    # Code to run the first time the generator is iterated upon
    yield
    
    # Code to run the second time the generator is iterated upon
    yield
    
    # Code to run the third (and final) time the generator is iterated upon
    return
```

In the preceding example it should be noted that the values of `param_1` and `param_2` persist until the function is iterated upon three times and `return` is reached.

## Example 2

In this example, we will contrast two techniques you might use to wait for character input. The first technique will use blocking code that prevents other code from executing, and the second approach will refactor the code using a generator function to make the approach cooperative.

**Approach 1**:
``` python
# This function will block until a specific character is entered in PuTTY and
# then echo that character once it is received.
def wait_for_character_blocking(token, ser):
    while True:
        # Read a character when one is ready (this line may block on its own)
        char_in = ser.read(1).decode()
        # If the token matches, echo the character back to the user and 
        # exit the generator
        if char_in == token:
            print(f"You pressed {token}!")
            return

if __name__ == "__main__":
    ser = pyb.USB_VCP()
    print("Press g to continue: ")
    wait_for_character_blocking("g", ser)
    # Code here runs after "g" is pressed
```

With this approach, the MCU will be unable to run any other code after `wait_for_character_blocking()` is called until the requested token is received. If this occurred in a real-time system, such as one performing motor control, the control loop would stop running which may cause serious problems.

**Approach 2**:
``` python
# This function will cooperatively wait for a specific character to be entered
# in PuTTY and then echo that character once it is received.
def wait_for_character_coop(token, ser):
    while True:
        # Check for pending characters and only once one is ready read the
        # character
        if ser.any():
            char_in = ser.read(1).decode()
            # If the token matches, echo the character back to the user and 
            # exit the generator
            if char_in == token:
                print(f"You pressed {token}!")
                return
        yield
        
if __name__ == "__main__":
    ser = pyb.USB_VCP()
    print("Press g to continue: ")
    for _ in wait_for_character_coop("g", ser):
        # Run other code while waiting for input
        pass
    # Code here runs after "g" is pressed
```

The `for` loop is repeatedly calling `next()` on the generator object created by `wait_for_character_coop("g", ser)` behind the scenes. On each iteration, the generator function continues until it reaches a `yield` statement. The `yield` statement marks a point where execution pauses, giving the body of the `for` loop (represented by `pass` in the example) opportunity to run. When the generator is iterated again, execution resumes immediately after the most recent `yield` statement with all local variables preserved.

## Example 3

For another example consider the Collatz sequence which is produced by iteratively plugging the output of the following function back into itself.

$$
f(n) = 
\begin{cases}
\frac{n}{2} & \text{if } n \text{ is even} \\
3n+1 & \text{if } n \text{ is odd}
\end{cases}
$$

We can implement this sequence very elegantly using a generator function.

``` python
# A generator function for producing elements in the Collatz sequence
def collatz(n):
    # Check for invalid seeds first
    if type(n) is not int:
        raise TypeError('Input must be an integer')
    if n < 1:
        raise ValueError('Input must be positive')
    
    # Start by yielding the initial seed value
    yield n
    
    # Apply the collatz "function" until hitting the stopping condition
    while n != 1:

        # Even values of n
        if n % 2 == 0:
            n //= 2
        # Odd Values of n
        else:
            n = 3*n + 1      
        
        # Yield the general case
        yield n

# The following block of code runs when this file is executed as a script, but not when the file is imported as a module in another file
if __name__ == "__main__":
    # This first example shows how to convert the generator to a list
    # which will run the generator as many times as needed until the generator
    # function exists (the while loop ends)
    print(list(collatz(39)))
    
    # This next example shows how to loop through the numbers in the generator
    # using a for loop similar to how we iterate through lists
    for num in collatz(39):
        print("The num is", num)
    
    # You can also call next() manually, but the above methods are generally
    # more useful
    seq = collatz(8)
    print(next(seq))
    print(next(seq))
    print(next(seq))
    print(next(seq))
    print(next(seq))  #<---- notice that this line reaches the end of the
                      #      while loop causing a StopIteration exception
```

The preceding example includes a few nuances you should pay attention to:
* The generator function does input validation and may raise either a `TypeError` or a `ValueError` because the sequence is only well defined for integers greater than or equal to 1.
* The input validation does not run when the generator function is used to build a generator object. That is, `seq = collatz(8)`, as used in the example, only creates a generator object, but does not run any iterations. It is only once the first iteration occurs that the input validation takes effect. Neglecting this often causes false positives to manifest in test code.
* Two operators are used that may be unfamiliar to beginner programmers:
    * The `%` symbol implements the modulus or remainder operator. The expression `a % b` means divide `a` by `b` and return the remainder.
    * The `//` symbol implements the integer divide operator. Python's standard divide operator, `/`, always returns a `float` object, but `a // b` will return an `int` object (for integers `a` and `b`) .
* The generator function ends without hitting a `return` statement; Python interprets this the same as hitting `return` with no value or hitting `return None`.

# Summary
* Generators are convenient for implementing algorithms that must start and stop to allow other code to run.
* A generator will run line by line until hitting `yield`.
* Subsequent iterations of a generator will resume running line by line starting immediately after the most recently crossed `yield`.
* In lab you will use generator functions to implement cooperative tasks.

# Candidate Static References
* \[\[Generators\]\]
* \[\[Coroutines\]\]
* \[\[yield from\]\] 
* \[\[send\]\]
* \[\[Lazy Programming\]\]