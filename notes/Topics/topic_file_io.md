---
title: File Input and Output
type: topic
tags:
  - python
  - file-io
source:
  course: ME405
  term: 2262
  lecture: 2
status: draft
---

## Motivation

This lecture introduces Python's basic file I/O. These tools form the foundation for later labs involving data logging.

> **Candidate static notes**
>
> -   \[\[Python File I/O\]\]
> -   \[\[Exception Handling\]\]

## The `with` Statement

Python files should almost always be opened using a `with` block. This guarantees that the file is automatically closed when execution leaves the block, even if an exception occurs.

### Example 1

The traditional approach to reading a file is to open the file, keep the file stored in an object, and then close the file after processing. This approach may be unsafe as the file will remain open if an exception (an error) occurs before the file is closed.

``` python
filename = "data.csv"

file = open(filename, "r")

for line in file:
    # Process one line at a time
    pass

file.close()
```

### Example 2

The modern, preferred approach to reading a file is to use the `with` construct. Using the `with` statement means that Python is responsible for managing the file, not your code. Therefore when things go wrong, like when an exception is raised, Python can automatically clean up safely by closing the file.

``` python
filename = "data.csv"

with open(filename, "r") as file:
    for line in file:
        # Process one line at a time
        pass
```

### Example 3

The preceding two examples show that a file is an iterable. That is, it is possible to iterate through the file directly, line by line, using a for loop or other iterator.

It is also possible to interact with the file without iteration using the family of read methods, as shown in this example.

``` python
with open(filename, "r") as file:
    file.read(n)       # Read n characters
    file.readline()    # Read one line
    file.readlines()   # Read all lines
```

**Note:** the iterator method and both `file.readline()` and `file.readlines()` check for line endings in the form of either newline or carriage return characters to know where each line ends.
* `\n` (10 in ASCII) represents a newline character
* `\r` (15 in ASCII) represents a carriage return character
Both of these characters are legacy artifacts from the transition from typewriters to computers in the 1940s. The newline character, `\n`, is often called the "line feed" character and is carryover from advancing the paper in a typewriter. Likewise, the carriage return, `\r`, is carryover from returning the typewriter carriage back to the left column.

------------------------------------------------------------------------

## Reading and Processing a File

You may have noticed in the preceding examples that the files were opened with `"r"` as a second input parameter, following the filename. Python can open files in a variety of modes, but the modes you are likely to use in ME 4305 include `"r"`, for read, `"w"`, for write, and `"a"` for append. In this section we will look at some common methods for parsing text in a file. You will get practice with this in homework and in lab.

**Note:** Python has a robust `csv` module that should be used when reading CSV files in practice on a computer running CPython. In this example the `CSV` module is not used for many reasons:
1) The standard `csv` module available in CPython is not available in MicroPython.
2) The purpose of this example is to demonstrate direct file I/O and basic text parsing.
3) You will have to read a CSV in homework for practice without using the `CSV` module either.

### Example 4

In this example, a CSV (comma separated value) file will be opened and the contents will be converted from text to lists. A CSV file is a plaintext file in which data is stored in a tabular format with commas separating columns and line-endings separating rows. A CSV file can be thought of as an extremely simple version of a spreadsheet.

An example CSV could look like:
``` text
Time,Distance
0.0,0.0000
0.1,0.0998
0.2,0.1987
0.3,0.2955
0.4,0.3894
0.5,0.4794
0.6,0.5646
0.7,0.6442
0.8,0.7174
0.9,0.7833
1.0,0.8415
```

To convert each column of text into a list of numbers, each line must be stripped of special characters, split on the comma acting as a delimiter (separator), and then converted to a pair numeric values.

``` python
filename = "data.csv"
time = []
distance = []

with open(filename, "r") as file:
    # Extract header, strip special characters, split on comma
    # (This advances the file pointer by one row)
    headers = file.readline().strip().split(",")
    
    # Extract rows, strip, split, convert values, then append
    for line in file:
        t, d = line.strip().split(",")
        time.append(float(t))
        distance.append(float(d))
```

**Note:** the preceding example assumes a well-formed CSV file and is fragile to inconsistencies such as missing data, missing delimiters, or data incompatible with conversion to a Python ``float``. In your homework assignment you will be asked to improve this example by adding exception handling. It should also be noticed that the example grows the size of the `time` and `distance` objects on each iteration; this is fine for small sets of data but preallocation of these output lists or use of `array.array` objects should be considered for large sets of data.

------------------------------------------------------------------------

## Writing to a File

In ME 4305 you will likely not be producing files actively while your code runs on hardware, for many reasons. First, file IO is slow, as it involves writing to flash instead of RAM, and second, the filesystem management can become opaque when writing to a file from MicroPython firmware while the same file is visible to your operating system (like Windows).

### Example 5

This example is the inverse of Example 4; that is, instead of reading data from a CSV file and producing Python list objects, data from Python list objects will be written to a CSV file.

``` python
filename = "data.csv"
time = [0.0, 0.1, 0.2, 0.3, 0.4, 0.5]
distance = [0.0, 0.0998, 0.1987, 0.2955, 0.3894, 0.4794]

with open(filename, "w") as file:
    # Write formatted header
    file.write("Time,Distance\n")
    
    # Write rows with line endings and with columns separated by commas
    for t, d in zip(time, distance):
        file.write("{},{}\n".format(t, d))
```

This example uses two techniques that we may have not covered yet:
* `zip` is a useful function that pairs off items from multiple collections. In this example, it pairs off individual time and distance values at each iteration in a tuple, `(t, d)`. This makes it simple to write both time and distance to the file in the same line. Use of `zip` here instead of indexing with range is a simple but classic example of good Pythonic style.
* `"{},{}\n".format(t, d)` is an example of one method of string formatting in Python:
	* The curly brackets in `"{},{}\n"` designate *where* to place the formatted values in the produced string; the `"\n"` after the curly brackets adds a newline to each row so that the CSV file has one pair of datapoints per line.
	* `.format(t,d)` designates *what* data to place in the formatted string.

------------------------------------------------------------------------
## Summary

* Prefer `with` when opening files.

## See Also
* [[topic_collections|Collections]]