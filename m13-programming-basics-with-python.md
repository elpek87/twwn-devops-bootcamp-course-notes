# PROGRAMMING BASICS WITH PYTHON

## INTRODUCTION TO PYTHON

Python is a high-level, interpreted, general-purpose programming language created by Guido van Rossum.

It is designed to be:
- easy to read
- easy to write
- easy to maintain
- flexible

Python is widely used in:
- Web development
- Data science & machine learning
- Automation & scripting
- Game development
- DevOps & cloud engineering
- Machine learning
- AI

It's basically a very powerful language with many, many extensions provided by community.

As DevOps Engineer your main job is to automate everything - and python is one of the best tools for automation in modern infrastructure.

It can be used in:
1. Infrastructure automation - most cloud providers have python SDKs
2. CI/CD Pipeline scripting
3. Cloud & API Integration
4. Cleanup, backups, visualization etc.

## INSTALLATION AND LOCAL SETUP

There's no Pycharm CE as for now - then I just used:
`$ brew install pycharm`

## OUR FIRST PYTHON PROGRAM
Just a prtint statement:
```
print(200)
```
```
print("200 is a grat number")
```

## PYTHON IDE VS SIMPLE FILE EDITOR
The difference between coding in a Python IDE and a simple file editor comes down to features, productivity, and automation.

## STRINGS AND NUMBER DATA TYPES
Data types in python:
- string (str) - text type, double or single quoted
- integer (int) - whole number, positive, negative without decimals
- float - numbers with decimals

Operators are used to perform operations.

String concatenation - combining multiple strings - plus sign.

This approach requires spaces and use of str on a number:
```
print ("20 days are " + str(20 * 24 * 60) + " minutes")
```

This approach with f is nicer - evaluates expression inside {} and inserts their value:
```
print(f"20 days are {20 * 24 * 60} minutes")
```

## VARIABLES
A variable in Python is simply a name that refers to a value stored in memory.

Defining a var in python is easy because it's dynamically typed.

One of the ways of naming the vars is using underscores and non capital letters such as:

```
calculation_to_seconds = 20 * 24 * 60
```
Avoid reserved words to name variables.

Variable can have multiple scopes - global scope and local scope.

## FUNCTIONS
A function in Python is a reusable block of code that performs a specific task. It runs only when it's called.

To define a function:
```
def name_of_func():
```
To call a function:
```
name_of_funct()
```
Functions take parameters.
Changes inside function do not affect how function is used.
Inside function body you can create variables.
To make function return value we use `return` keyword.
Functions can be nested.

## ACCEPTING USER INPUT

To ask user for input simply use:
```
input()
```
It's built-in function.
Python waits for input - doesn't execute further instructions.

When calling a function that returns value just assign it to a variable and print that variable:

```
calculated_value = days_to_units(user_input)
print(calculated_value)
```

Input is always treated as a string - not a number to make it a number use:

```
user_input_number = int(user_input)
```

It's called casting.

User input should always be validated.

## CONDITIONALS (IF/ELSE) AND BOOLEAN DATA TYPE

A conditional in Python is a way to make your program decide what to do based on a condition. If some expression is TRUE do something otherwise (when FALSE) do something else.

TRUE / FALSE is boolean data type

When using elif - it has condition, final else doesn't - it's a fallback.

If and else statements can be nested - but it's not recommended to overuse them!

## ERROR HANDLING WITH TRY-EXCEPT

A try–except in Python is used for error handling. Try to run some code and if an error happen do not crash - handle it.

When handling errors with try / except you can either specify type of error to handle or handle any error - but it's not the best.

## WHILE LOOPS

A while loop in Python is used to repeat code as long as a condition is True. It keeps running again and again until the condition becomes False.

## LISTS AND FOR LOOPS

A list in Python is a collection of multiple values stored in a single variable. Defined inside [].

To access elements of the list you use index nubers of the elements.

You can also do multiple things with the lists such as appending values with .append() etc.

A for loop in Python is used to repeat code for each item in a sequence.

Split function by default takes spaces as separators between list elements.

## COMMENTS

Comments can be used to explain code as well as preventing the interpreter from executing commented-out code.

Commenting out larger portions of code can be done as follows:
```
"""
Some python code
More python code
"""
```

## SETS

A set in Python is a built-in data type that stores an unordered collection of unique elements. In comparison to lists sets use { }. In set items do not have defined order. They also can't be referred to by index and changed - only added and removed.

## BUILT-IN FUNCTIONS

A set in Python is a built-in data type that stores an unordered collection of unique elements. They are part of Python itself.

These are for example:
```
print("some text")
input("enter value")
set([1,2,3])
int("20")
```

This is the example of data-type specific built-in function:
```
"2, 3".split()
```

## DICTIONARY DATA TYPE

A dictionary in Python is a built-in data type used to store data in key–value pairs.
Unlike in lists - items can be accessed by key.

## MODULES

A module is a file with reusable code.

Let's say we have a file helper.py with functions - we need to reference to it from main.py and we'll be able to use the functions:

```
import helper

helper.function()
```
Often there's no need to import whole module when we need a single function - then we can do:
```
from helper import validate_and_execute
```
Then you don't need to reference function using module name prefix. You can also import variables or even all "*".

Contents of module file are called definitions.

You can rename module if you want:
```
import helper as h
```
In big projects with multiple files you can cross import contents between files.

There are many modules already written (both built-in and third-party) - often  no need to create your own from scratch.

## PROJECT: COUNTDOWN APP

Modules usually come with extensive documentation - read it.

## PACKAGES, PYPI AND PIP

A package is a collection of multiple Python modules. PAckage must include an __init__.py file.

PyPi (Python Package Index) is a repository (storage) for third-party Python packages.

pip is Python’s package manager that installs and manages packages from PyPI (Python Package Index).

To install package using pip:
`$ pip install package`

To uninstall package using pip:
`$ pip uninstall package`

## PROJECT: AUTOMATION WITH PYTHON (SPREADSHEET)

To handle a file (in general) in python we use io module but since we're working with spreadsheets we're going to use something more suitable for the task.

By default range would start with 0 so we have to explicitly make it start from 2:
```
for product_row in range(2, product_list.max_row)
```
Another thing is that range excludes the last row - to include it:
```
for product_row in range(2, product_list.max_row + 1)
```

## OOP: CLASSES AND OBJECTS

Object-Oriented Programming (OOP) is a way of structuring code around objects instead of just functions. In OOP you create objects that contain data (variables) and behavior (functions).

A class is blueprint for creating objects. It helps you define what data an object will have and what actions it can perform.

In python files should be started with non capital letter and classes with the capital letter.

- class is like an object constructor
- all classes have __init__() function
- __init__() is executed automatically when objects from the class are created
- values are passed to the constructor as parameters
- to create an object we need to call class constructor
- functions that belong to a class are called methods
- self is passed to a method automatically
- classes usually are in separate files
- in python almost everything is an object

## PROJECT: API REQUEST TO GITLAB

When using string inside the string double quotes on both of them are not an option - we should go with single quotes on one of the string.
```
for project in my_projects:
    print(f"Project Name: {project['name']}")
```
