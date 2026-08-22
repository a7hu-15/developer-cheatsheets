# 🐍 Python Cheatsheet

A practical Python reference covering syntax, patterns, and best practices.

---

## 📦 Data Types

```python
# Integers & Floats
x = 42
y = 3.14
z = 1_000_000      # underscore separator for readability

# Strings
name = "Alice"
multi = """Multi
line string"""
f_string = f"Hello, {name}!"       # f-strings (Python 3.6+)
raw = r"no \n escape"              # raw string

# Boolean
is_active = True
is_empty = False

# None
result = None

# Type checking
type(x)           # <class 'int'>
isinstance(x, int)  # True
```

---

## 📋 Collections

### Lists (mutable, ordered)
```python
nums = [1, 2, 3, 4, 5]
nums.append(6)          # Add to end
nums.insert(0, 0)       # Insert at index
nums.extend([7, 8])     # Add multiple
nums.pop()              # Remove & return last
nums.pop(0)             # Remove & return at index
nums.remove(3)          # Remove first occurrence
nums.sort()             # Sort in-place
nums.reverse()          # Reverse in-place
nums.index(4)           # Find index of value
len(nums)               # Length
```

### Tuples (immutable, ordered)
```python
point = (3, 4)
x, y = point              # Unpacking
single = (42,)             # Single-element tuple (note the comma!)
```

### Dictionaries (key-value pairs)
```python
user = {"name": "Alice", "age": 30}
user["email"] = "alice@example.com"   # Add/update
user.get("phone", "N/A")              # Get with default
user.pop("age")                        # Remove key
user.keys()                            # All keys
user.values()                          # All values
user.items()                           # All (key, value) pairs
"name" in user                         # Check key exists

# Dictionary comprehension
squares = {x: x**2 for x in range(6)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

### Sets (unique, unordered)
```python
fruits = {"apple", "banana", "cherry"}
fruits.add("date")
fruits.discard("banana")      # Remove (no error if missing)
fruits.remove("apple")        # Remove (KeyError if missing)

# Set operations
a = {1, 2, 3}
b = {2, 3, 4}
a | b    # Union: {1, 2, 3, 4}
a & b    # Intersection: {2, 3}
a - b    # Difference: {1}
a ^ b    # Symmetric difference: {1, 4}
```

---

## 🔄 Control Flow

```python
# If / Elif / Else
if x > 10:
    print("big")
elif x > 5:
    print("medium")
else:
    print("small")

# Ternary operator
status = "even" if x % 2 == 0 else "odd"

# Match statement (Python 3.10+)
match command:
    case "start":
        start()
    case "stop":
        stop()
    case _:
        print("Unknown command")
```

---

## 🔁 Loops

```python
# For loop
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 10, 2):   # 2, 4, 6, 8
    print(i)

# Enumerate (index + value)
for i, item in enumerate(["a", "b", "c"]):
    print(f"{i}: {item}")

# Zip (parallel iteration)
names = ["Alice", "Bob"]
ages = [30, 25]
for name, age in zip(names, ages):
    print(f"{name} is {age}")

# While loop
while condition:
    do_something()

# Loop with else (runs if loop completes without break)
for item in items:
    if item == target:
        break
else:
    print("Not found")

# List comprehension
squares = [x**2 for x in range(10)]
evens = [x for x in range(20) if x % 2 == 0]
```

---

## 🔧 Functions

```python
# Basic function
def greet(name: str) -> str:
    """Return a greeting message."""
    return f"Hello, {name}!"

# Default arguments
def power(base, exp=2):
    return base ** exp

# *args and **kwargs
def func(*args, **kwargs):
    print(args)     # Tuple of positional args
    print(kwargs)   # Dict of keyword args

# Lambda functions
square = lambda x: x ** 2
add = lambda a, b: a + b

# Decorators
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    import time
    time.sleep(1)
```

---

## 📦 Classes

```python
class Dog:
    """A simple Dog class."""

    species = "Canis familiaris"  # Class variable

    def __init__(self, name: str, age: int) -> None:
        self.name = name          # Instance variable
        self.age = age

    def __str__(self) -> str:
        return f"{self.name} ({self.age} years old)"

    def __repr__(self) -> str:
        return f"Dog(name='{self.name}', age={self.age})"

    def bark(self) -> str:
        return f"{self.name} says Woof!"

    @property
    def human_years(self) -> int:
        return self.age * 7

    @classmethod
    def from_birth_year(cls, name: str, birth_year: int) -> "Dog":
        from datetime import datetime
        age = datetime.now().year - birth_year
        return cls(name, age)

    @staticmethod
    def is_adult(age: int) -> bool:
        return age >= 2


# Inheritance
class GuideDog(Dog):
    def __init__(self, name: str, age: int, handler: str) -> None:
        super().__init__(name, age)
        self.handler = handler

# Dataclasses (Python 3.7+)
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float
    z: float = 0.0

    def distance_from_origin(self) -> float:
        return (self.x**2 + self.y**2 + self.z**2) ** 0.5
```

---

## 📁 File I/O

```python
# Read entire file
with open("file.txt", "r") as f:
    content = f.read()

# Read line by line
with open("file.txt", "r") as f:
    for line in f:
        print(line.strip())

# Write to file
with open("output.txt", "w") as f:
    f.write("Hello, World!\n")

# Append to file
with open("log.txt", "a") as f:
    f.write("New log entry\n")

# Read/write JSON
import json

with open("data.json", "r") as f:
    data = json.load(f)

with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# Read/write CSV
import csv

with open("data.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)
```

---

## ⚠️ Error Handling

```python
# Try / Except / Finally
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")
except (ValueError, TypeError):
    print("Value or type error")
except Exception as e:
    print(f"Unexpected: {e}")
else:
    print("No errors!")       # Runs if no exception
finally:
    print("Always runs")      # Cleanup code

# Raise exceptions
def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# Custom exceptions
class NotFoundError(Exception):
    def __init__(self, item: str):
        super().__init__(f"'{item}' was not found")
        self.item = item

# Context managers
from contextlib import contextmanager

@contextmanager
def managed_resource():
    print("Setup")
    yield "resource"
    print("Cleanup")
```

---

## 🧰 Useful Built-in Functions

```python
# Sorting
sorted([3, 1, 2])                        # [1, 2, 3]
sorted(users, key=lambda u: u["age"])     # Sort by key
sorted(items, reverse=True)               # Descending

# Mapping & Filtering
list(map(str.upper, ["a", "b", "c"]))     # ['A', 'B', 'C']
list(filter(lambda x: x > 0, [-1, 2, -3, 4]))  # [2, 4]

# Aggregation
sum([1, 2, 3])          # 6
min([1, 2, 3])          # 1
max([1, 2, 3])          # 3
all([True, True])       # True
any([False, True])      # True

# Misc
abs(-5)                 # 5
round(3.14159, 2)       # 3.14
len("hello")            # 5
range(0, 10, 2)         # 0, 2, 4, 6, 8
reversed([1, 2, 3])     # 3, 2, 1
```

---

## 📦 Common Standard Library Modules

```python
import os               # OS interface (files, paths, env vars)
import sys              # System-specific parameters
import json             # JSON encoding/decoding
import re               # Regular expressions
import math             # Mathematical functions
import random           # Random number generation
import datetime         # Date and time
import pathlib          # Object-oriented file paths
import collections      # Specialized container types
import itertools        # Iterator building blocks
import functools        # Higher-order functions (lru_cache, reduce)
import typing           # Type hints
import unittest         # Unit testing
import logging          # Logging
import argparse         # CLI argument parsing
import hashlib          # Secure hashes (SHA, MD5)
import secrets          # Cryptographically strong random
```

---

## 🏗️ Virtual Environments

```bash
# Create a virtual environment
python -m venv .venv

# Activate (macOS / Linux)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Deactivate
deactivate

# Install packages
pip install requests

# Save dependencies
pip freeze > requirements.txt

# Install from requirements
pip install -r requirements.txt
```

---

## 🧪 Testing

```python
# unittest
import unittest

class TestMath(unittest.TestCase):
    def test_addition(self):
        self.assertEqual(1 + 1, 2)

    def test_division_by_zero(self):
        with self.assertRaises(ZeroDivisionError):
            1 / 0

# pytest (simpler syntax)
def test_addition():
    assert 1 + 1 == 2

def test_string():
    assert "hello".upper() == "HELLO"

# Run tests
# python -m unittest discover
# pytest
```

---

## 💡 Python Tips & Tricks

```python
# Swap variables
a, b = b, a

# Multiple assignment
x = y = z = 0

# Walrus operator (Python 3.8+)
if (n := len(items)) > 10:
    print(f"Too many items: {n}")

# Chained comparisons
if 0 < x < 100:
    print("In range")

# Merge dictionaries (Python 3.9+)
merged = dict1 | dict2

# String multiplication
line = "-" * 40

# Flatten a list
flat = [x for sublist in nested for x in sublist]

# Remove duplicates while preserving order
unique = list(dict.fromkeys(items))

# Get environment variables
import os
db_url = os.environ.get("DATABASE_URL", "sqlite:///default.db")
```

---

*Made with 🐍 for Pythonistas everywhere.*
