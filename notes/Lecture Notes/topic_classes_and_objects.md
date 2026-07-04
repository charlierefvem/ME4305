---
title: Classes and Objects
type: topic
tags:
  - python
  - classes
source:
  course: ME405
  term: 2262
  lecture: 7
status: draft
---

# Motivation

As with all tasks in engineering, programming involves many balancing tradeoffs. One such tradeoff arises when deciding whether or not to use a language feature that benefits the programmer instead of the application. In every language, perhaps Python more than others, there are certain language features that are less efficient in implementation, but extremely valuable for the programmer while other features may have such a large performance cost, that they should be avoided entirely.

A strong example of a language feature *well* worth the cost is the use of classes and objects to simplify programming through abstraction.

## What is abstraction?

In programming, abstraction is the process of layering an interface upon a body of code with the intention to simplify use of that code body. Abstraction allows the caller of a function to worry only about the input and output of the function without needing full awareness of the internal implementation. Abstraction also leads to be better code reuse because it forces the author to deliberately choose the correct set of parameters that pass through the abstraction layer.

In this lecture the focus will be on the basics of Python classes and objects. You will see an initial draft of a motor driver class that abstracts away the internal details of driving a motor, allowing the user to simply command effort without worrying about details like PWM.

# Review of Objects

Recall from a past lecture the following claim:
>Python is fundamentally an object-oriented language. *Every* variable is an object of some kind. 

Before you learn to write your own classes you should fully understand the context in which you've already used objects.
## Example 1

The Python `str` class is something you've used often so far in lab and in homework. Consider the simple example below:

``` python
my_string = "Hello, world!"
my_string.split(",")
```

In this example the **object** `my_string` of **class** `str` is created on line 1. The string type is one of many built-in types in Python, and is automatically created when the programmer uses quotation marks, `"`, or apostrophes, `'`, to enclose a sequence of characters.

A function that belongs to an object is called a **method**. In line 2 of the example, the string method called `split` is applied to the string object using the parameter `","` (another string object).

You may think of line 2, above, as shorthand for the following function call:

``` python
str.split(my_string, ",")
```

In this snippet the "string split function", `str.split`(), is applied to the pair of parameters `my_string` and `","`. This type of syntax is usually avoided in favor of the shorthand presented in the example above. However, understanding the long form version will help you better understand how to design your own classes.

## Example 2

To create objects that aren't of one of the built-in types, you must call the initializer for the class defining the object. Consider the example below, similar to code you've used in lab, that defines a GPIO pin, **PB6**, specifically, in output mode.

``` python
import pyb
my_pin = pyb.Pin(pyb.Pin.cpu.B6, mode=pyb.Pin.OUT_PP)
```

There are several details to consider in this short example.

| Instance           | Description                                                                                                                                                                     |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pyb`              | An external module (library) that has been imported. Modules are containers for artifacts defined in other files.                                                               |
| `pyb.Pin` or `Pin` | A class belonging to the module `pyb`. Think of the class like a blueprint or recipe to use when creating an object. Classes will be covered below in more detail.              |
| `my_pin`           | An object (instance) of class `Pin`. This object matches the "recipe" defined by the class and retains access to all methods defined in the class as well.                      |
| `pyb.Pin.cpu.B6`   | Within the `Pin` class there exists an inner (nested) class called `cpu` that is used as a container for constants representing each port pin, such as `B6`.                    |
| `pyb.Pin.OUT_PP`   | Similar to `pyb.Pin.cpu.B6`, `OUT_PP` is a constant belonging to the `Pin` class that represents output mode for a pin. Other options include `Pin.ANALOG`, `Pin.IN`, and more. |
Now that an object has to be created it is simple to interact with the pin. To set it high, simply call `my_pin.high()` and to set it low, use `my_pin.low()`. In this example, the abstraction layer has provided the convenience to temporarily forget about things like GPIO and **PB6** and instead focus on what you need to do with `my_pin` to achieve your objective.

## Example 3

Another class you've used in lab is the class that abstracts usage of hardware timers. Without this class you would be forced to write potentially dozens of lines of code, directly interacting with special function registers, to do something as simple as generate PWM. The abstraction layer allows you to only worry about the minimal set of parameters needed for you to tell the class what timer to use and how you'd like it to be configured.

``` python
from pyb import Timer
my_tim = Timer(tim_num, freq=...)
my_chan = my_tim.channel(chan_num, mode=...)
```

This example is slightly more subtle than for the pin, because two objects are created.

| Instance           | Description                                                                                                                                                                                                                                                                                |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Timer`            | A specific class imported from the `pyb` module so that unused classes do not get imported.                                                                                                                                                                                                |
| `Timer()`          | The initializer for the `Timer` class used to create and initialize instances of the class.                                                                                                                                                                                                |
| `my_tim`           | An object of class `Timer`.                                                                                                                                                                                                                                                                |
| `my_tim.channel()` | A method belonging to class `Timer` that creates objects of class `TimerChannel`. Functions that create objects of another class are often called factories.                                                                                                                               |
| `my_chan`          | An object of class `TimerChannel`. Note that it is impossible to directly instantiate objects from class `TimerChannel`. Due to the inherent relationship between timers and timer channels it just doesn't make sense to have a channel that is not associated, or created from, a timer. |

# Defining Your Own Classes

While not "day one" fundamental, being able to write your own Python class is a valuable skill and will be required in almost every assignment you do the remainder of this quarter, whether on homework or in lab.

As with all things in engineering, it is best practice to *design* before implementing.

## Designing Classes

There are many techniques for laying out a class before jumping in to the implementation. One of the simplest is to draw an informal class diagram. A class diagram consists of three things, each drawn in one of three stacked boxes:
1) The name of the class you are designing.
    * Idiomatic Python convention is to use `PascalCase` when naming classes. That is, the first letter of each word should be uppercase and there should be no underscores between words.
2) The set of **attributes** that define the internal state of the object.
    * Attributes names should be in a lower-case `underscore_separated` format.
    * Private attributes are invisible outside of the class definition and are meant for variables that the user of your class should not interact with; these variables must start with a single underscore, `'_'`, prefixing the variable name.
    * Each attribute should have its type listed afterwards with a colon separating the variable and the type, e.g. `variable_name: type_name`.
3) The set of **methods** that can act on the object.
    * Function names follow the same name convention as attributes, lower-case underscore separated names are preferred.
    * Private methods are defined using an underscore prefix and are intended to never be called outside of the class implementation. That is, private methods should only be called from within other methods of the same class.
    * Any functions that return an output should have the output type indicated after the function, separated with a colon, e.g.  `get_string(): str`.
    * Any function that takes input parameters should follow the same type convention as covered for attributes and function outputs.

### Example 4

In this example a class diagram is presented for an H-bridge motor driver with a simple phase and direction interface. The class, `MotorDriver`, will abstract away details like GPIO and PWM so that the caller may run simple commands like `left_motor.enable()` and `right_motor.set_effort(42)`.

![Motor driver class diagram listing attributes and methods.|500](images/class_diagram.svg)
**Note:** when designing a class spend a reasonable amount of time deliberating on a correct set of initializer parameters, attributes, and methods as these choices indirectly define the core structure of your implementation. In this example, exactly five parameters are needed to fully define the hardware interface between the microcontroller and the motor driver, but all five parameters don't need to be stored as attributes for the lifetime of the object. Instead, three of the parameters defining the PWM pin, timer, and channel, are combined into one timer channel attribute.

# Implementing a Class

Implementation of a class in Python is quite simple as it is a core feature of the language. Use the `class` keyword similar to how you would use the `def` keyword when defining a function.

``` python
class MyClass:
# Class defined here
```

Inside the indented block defining the class you can write your methods. Every class must start with an initializer method that runs when a user creates an instance of your class and the method *must* be named exactly `__init__()`, including two underscores as a prefix and again as a suffix. 

For functions defined within your class to have access to attributes the first parameter must be the special parameter called `self`. The variable `self` is how you refer to objects that will be created from this class while still implementing the class. In other words, `self` is like a placeholder for the specific object names that will eventually get used when the object is defined.

Recall from earlier in the lecture the long-form way to call a class method:

```python
str.split(my_string, ",")
```

You can imagine how this method might be defined to better understand how `self` works.

``` python
class str:

    def __init__(self):
        pass
    
    def split(self, delimiter):
        pass
```

To call the split method using the long form syntax you are passing in the name of the object, `my_string`, to the parameter `self` so that when the method runs it is able to interact with `my_string`.

**Reminder:** the mentioned long-form syntax is not often utilized so you should prefer the standard `object.method()` style of syntax in your own code.

## Example 5

In this example, a partially complete class definition will be presented that corresponds to the class diagram in Example 4. You may use this in lab as a starter template when the time comes but know that it is not complete. You will also be expected to go through and annotate the file with comments which will help you familiarize yourself with the code.

``` python
from pyb import Pin, Timer

    class motor_driver:

        def __init__(self, pwm_pin: Pin, dir_pin: Pin,
                     nslp_Pin: Pin, tim: Timer, chan: int):

            # Store a copy of each input parameter as an attribute
            self._dir_pin = Pin(dir_pin, mode=Pin.OUT_PP)
            self._nslp_pin = Pin(nslp_pin, mode=Pin.OUT_PP)
            self._pwm_chan = tim.channel(chan,
                                         pin=pwm_pin,
                                         mode=Timer.PWM,
                                         pulse_width_percent=0)
                                          
        def enable(self):
            pass
            
        def disable(self):
            self._nslp_pin.low()
            
        def set_effort(self, effort: float):
            # This function has bugs that you must fix
            self._pwm_chan.pulse_width_percent(effort)

# Code in the following block will run when this file is run as a script, but not when used as a module in other files.
if __name__ == "__main__":
    # Both motors can use the same timer but different channels
    pwm_tim = Timer(4, freq=20_000)
    left_motor = MotorDriver(Pin.cpu.B6, Pin.cpu.C0, Pin.cpu.C1, pwm_tim, 1)
    right_motor = MotorDriver(Pin.cpu.B7, Pin.cpu.C2, Pin.cpu.C3, pwm_tim, 2)
    
    left_motor.enable()
    right_motor.enable()
    
    left_motor.set_effort(42)
    right_motor.set_effort(-42)
```

**Note**: if you ever want to create an indented block, but you don't want to write code inside of it yet use `pass`. This keyword tells the interpreter that you've deliberately left an indented block empty. Pass should not appear anywhere else in your code.
## Example 6
Many programmers (including your instructor) consider the use of global variables to be poor coding practice. In some cases, they seem necessary, like in interrupt callbacks, but almost always they can be avoided.

One way to eliminate global variables in a callback is to use a class method, instead of a standard function, as the callback function. With this approach class attributes can be accessed in the callback instead of global variables.

Consider the following two examples for collecting data inside a timer callback. The first uses global variables.

``` python
# Example 1 - Using global variables
from array import array
from time import ticks_ms
from encoder import Encoder
from pyb import Timer

t_buf = array('L',(0 for n in range(1000)))
p_buf = array('L',(0 for n in range(1000)))
d_buf = array('L',(0 for n in range(1000)))
idx = 0

enc_A = Encoder()

def tim_cb(cb_src):
    global t_buf, p_buf, d_buf, idx
    
    enc_A.update()
    
    t_buf[idx] = ticks_ms()
    p_buf[idx] = enc_A.get_position()
    d_buf[idx] = enc_A.get_delta()
    
    idx += 1
    
tim = Timer(7, freq=100, callback=tim_cb)
```

The second uses a lightweight class to encapsulate the variables that would otherwise be global.

``` python
# Example 2 - Using a class
from array import array
from time import ticks_ms
from encoder import Encoder
from pyb import Timer


class Collector:

    def __init__(self, tim, enc):
        self.tim = tim
        self.enc = enc
        
        self.t_buf = array('L',(0 for n in range(1000)))
        self.p_buf = array('L',(0 for n in range(1000)))
        self.d_buf = array('L',(0 for n in range(1000)))
        self.idx = 0
        
        self.tim.callback(self.tim_cb)
        
    def tim_cb(self, cb_src):
        self.enc.update()
        self.t_buf[self.idx] = ticks_ms()
        self.p_buf[self.idx] = self.enc.get_position()
        self.d_buf[self.idx] = self.enc.get_delta()
        self.idx += 1

col = Collector(Timer(7, freq=100), Encoder())
```


# Object-Oriented Programming

This lecture has almost exclusively focused on the practical use of classes and has promoted the benefits of encapsulation and abstraction that result. For completeness it should be mentioned that classes offer a much broader set of features than what has been covered here, two of which are worth mentioning for completeness.

## Inheritance

To provide an additional layer of abstraction and reusability Python allows you to create an inherited class or child class that borrows functionality from an existing class. The child class can then modify or extend the features of the parent class while avoiding reimplementing any of the features that do carry over.

## Polymorphism

To expand the usefulness of inheritance Python allows polymorphism. That is, code written with a specific class in mind should generally function when applied to a descendant of that class. A simple example of this would be Python's set of exceptions which are organized using inheritance. In Python you may `raise` an exception using the base `Exception` class, but are more likely to raise specific exceptions like a `TypeError` which inherits directly from the `Exception` class.

# Final Insights

Just as a free-body diagram, transfer function, electrical schematic, or state machine hides unnecessary detail to emphasize the behavior of a physical system, a class hides implementation details to emphasize the software interface. Use class with deliberate care and you will write better code that is easier to maintain and understand.

# Summary

* Classes define objects.
* Encapsulation is the primary object-oriented technique used throughout this course. 
* Inheritance and polymorphism are important extensions that become more valuable in larger software systems.
* Design first, implement second.
* Prefer encapsulation over global variables.

# Candidate Static Notes
* \[\[Exceptions\]\]