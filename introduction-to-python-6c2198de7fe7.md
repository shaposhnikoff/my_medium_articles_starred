Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m86[39m, end: [33m105[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m134[39m, end: [33m136[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m21[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m9[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m21[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m167[39m, end: [33m169[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m143[39m, end: [33m147[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m170[39m, end: [33m174[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m264[39m, end: [33m272[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m302[39m, end: [33m311[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m313[39m, end: [33m317[39m }

# Introduction to Python

A walkthrough of the basics with code examples

![[Source](https://www.python.org/)](https://cdn-images-1.medium.com/max/2000/1*PPIp7twJJUknfohZqtL8pQ.png)*[Source](https://www.python.org/)*

## Introduction

Python is an interpreted, high-level language created by Guido van Rossum and released in 1991. It is dynamically typed and garbage collected.

Python programs have the extension .py and can be run from the command line by typing python file_name.py.

Probably its most noticeable characteristic is its use of significant white space to delimit code blocks, instead of the more popular {} symbols.

End-of-line semicolons (;) are optional and usually not used in Python.

The latest version is 3.7.4 (when this article was written), which is what I will use in the examples.

## Installation

Download the latest version [here](https://www.python.org/downloads/).

PyCharm is an IDE that makes it easy to write and run Python code. The community edition is free and you can get it from [here](https://www.jetbrains.com/pycharm/download/#section=mac).

## Print to the Console

<iframe src="https://medium.com/media/fdd16236c31422b206ef33570d3cda45" frameborder=0></iframe>

## Comments

<iframe src="https://medium.com/media/adab603ac11289c5cc17ccc49873eb31" frameborder=0></iframe>

## Arithmetic Operators

<iframe src="https://medium.com/media/fb66e73411765e428f3ea8c7a9339e6e" frameborder=0></iframe>

## Variables

Variables do not need to be declared.

<iframe src="https://medium.com/media/b9c6d4b81e2630942fd2fdf2a482b460" frameborder=0></iframe>

Their data types are inferred from their assignment statement, which makes Python a dynamically typed language. This means that different data types can be assigned to the same variable throughout the code.

<iframe src="https://medium.com/media/252f3af47049a581d332dc7c0214cb3b" frameborder=0></iframe>

## Comparison, Logical, and Conditional Operators

Comparison: ==, !=, <, >, <=, >=

Logical: and, or, not

Conditionals: if, else, elif

<iframe src="https://medium.com/media/43d67d3d7241938469950d792c6c51f1" frameborder=0></iframe>

<iframe src="https://medium.com/media/2ed49b4bb1ed4155ce3d13196f12b100" frameborder=0></iframe>

## Data Types

### **Strings**

<iframe src="https://medium.com/media/643f4b19caa8efbf0c02a096753b821a" frameborder=0></iframe>

<iframe src="https://medium.com/media/bfd3ccc90020a65f59180a96ee56a6c2" frameborder=0></iframe>

<iframe src="https://medium.com/media/7ff9617f3cadbe1b8965cfd876827446" frameborder=0></iframe>

<iframe src="https://medium.com/media/c466cdb176e13fd0b30b6b0b65b85455" frameborder=0></iframe>

<iframe src="https://medium.com/media/2b8de7f1045353a544d4ec10cec71b2a" frameborder=0></iframe>

<iframe src="https://medium.com/media/10b1cb939f10f7f656d42285f8b60f44" frameborder=0></iframe>

### **Numbers**

Python supports three numeric data types: int, float, and complex. They are inferred, so need not be specified.

<iframe src="https://medium.com/media/2a21b028372f04d019a5800aab57ed15" frameborder=0></iframe>

### **Booleans**

<iframe src="https://medium.com/media/a5b481770d6ddae0bf69a7da60717d49" frameborder=0></iframe>

### **Lists**

A list in Python is what other programming languages call an array. They use Zero-based indexing, and the items can contain any type of data. List items are nested in [].

<iframe src="https://medium.com/media/743c854e27ad8c1aa0a21be24b0acfd7" frameborder=0></iframe>

### **Tuples**

Tuples are just like lists, but immutable (cannot be modified). They are surrounded by ().

<iframe src="https://medium.com/media/7e9e1710b05dc7677ca72a0d3f194101" frameborder=0></iframe>

### **Dictionaries**

Dictionaries are key-value pairs. They are surrounded by {} and are similar to objects in JavaScript. The values can have any data type.

<iframe src="https://medium.com/media/696eb9e26379f42670bc85c6095d3fac" frameborder=0></iframe>

## Loops

### For loops

<iframe src="https://medium.com/media/6fc138adfa15b017298257ba29541844" frameborder=0></iframe>

### While loops

<iframe src="https://medium.com/media/c0862ddb75de7f90e6fa21f86bd631b4" frameborder=0></iframe>

## Functions

Functions are defined using the def keyword.

<iframe src="https://medium.com/media/d5d8239af09a1f6cf4656f6514be2665" frameborder=0></iframe>

Default values for arguments may also be defined.

<iframe src="https://medium.com/media/13323af6bbcce75908558d9665365718" frameborder=0></iframe>

## Classes

Classes are collections of variables and functions that work with those variables.

Classes in Python are defined with the class keyword. They are similar to classes in other languages like Java and C#, but differences include self being used instead of this to refer to the object resulted from the class. Also, the constructor function’s name is __init__, instead of the more popular classname. self must be used every time a class variable is referenced and must be the first argument in each function’s argument list, including the constructor.

Python does not support method overloading, but it can be achieved to some extent by using default values for arguments (shown in the Functions section).

<iframe src="https://medium.com/media/d7e010e792e4c870598df6429cbec5f5" frameborder=0></iframe>

Python also supports inheritance, which means that classes can be set to inherit methods and variables from another class or multiple classes (multiple inheritance).

<iframe src="https://medium.com/media/3a51b93eb8ea0c4269417ab6b06f4e14" frameborder=0></iframe>

## File I/O

<iframe src="https://medium.com/media/17852cb2c4b371ddae7405320c74af78" frameborder=0></iframe>

## References

[Python 3 documentation](https://docs.python.org/3/)

[Python basics](https://www.astro.ufl.edu/~warner/prog/python.html)
