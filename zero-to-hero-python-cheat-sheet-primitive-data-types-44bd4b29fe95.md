Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m93[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m85[39m, end: [33m86[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m110[39m, end: [33m112[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m60[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m8[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m174[39m, end: [33m175[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m45[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m78[39m, end: [33m83[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m119[39m, end: [33m123[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m68[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m4[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m23[39m }

# Zero to Hero Python Cheat Sheet: Primitive Data Types

Photo by Hitesh Choudhary on Unsplash

Whether you are a data scientist or an engineer or just starting with programming, working with Python can be really fun.

However, learning and remembering syntax is hard and sometimes doesn’t come instinctively. Having a lookup for the most common use cases can come in handy. It lets us focus on the task at hand and not waste time looking up syntaxes.

This is the first post of the Zero to Hero Python series. The next posts cover

1. [working with the console, as well as lists and tuples](https://medium.com/p/67baefbf5f12).

1. [using sets and dictionaries](https://medium.com/post/4af7aaa17598).

*Let’s get started!*

## Working With Numbers

### Basic arithmetic

For basic arithmetic operations, use a basic-math syntax:

    >>> 100
    100
    >>> 10 + 20
    30
    >>> 20 - 10
    10
    >>> 10 * 20
    200
    >>> 20 / 10
    2.0

**Note:** The output of the division operation is always float.

### Integer division

Integer division removes decimal places and provides an integer response. It is denoted by //.

    >>> 100 // 11
    9
    >>> 100.0 // 11
    9.0

### Modulus

A modulus operator provides the remainder from an integer division. It is denoted by %.

    >>> 100 % 11
    1
    >>> 100.0 % 11
    1.0

### Exponents

An exponent operator is used to calculate the power of a number raised to some other number. It is denoted by **.

    >>> 4 ** 2
    16
    >>> 11 ** 3
    1331

## Working With Strings

A string can be created using both double quotation marks (") or single quotes (').

    >>> "I love Python!"
    'I love Python!'
    >>> 'I love Python!'
    'I love Python!'

### Concatenation

As with numbers, use the + operator.

    >>> x = "Strings" + "Together"
    >>> x
    'StringsTogether'

### String array

A string is also an array of characters and can be treated as such. Use indices to access characters.

    >>> str = "I love Python!"
    >>> ch = str[0]
    >>> ch
    'I'

### Length

Use the len function to find the length of a string:

    >>> str = "I love Python!"
    >>> len(str)
    14

### String formatting

Use curly braces {} and the format function to create strings from formats and templates.

    >>> template = "{} love {}"
    >>> template.format("I", "Python")
    'I love Python'

To reuse values in multiple places in the template, use indices.

    >>> template = "{0} love {1}. {2} also loves {1}."
    >>> template.format("I", "Python", "John")
    'I love Python. John also loves Python.'

If you don’t want to play with too many indices, you can use named parameters as well.

    >>> template = "{person1} loves {thing}. {person2} also loves {thing}."
    >>> template.format(person1="John", person2="Harry", thing="Python")
    'John loves Python. Harry also loves Python.'

If you already have variables in your code, you can reference them directly in templates using formatted string literals, also called f-strings. Just start the template with f.

    >>> name = "John"
    >>> str = f"{name} loves Python"
    >>> str
    'John loves Python'

## Working with Booleans

Boolean values are available as the literals True and False in Python.

    >>> b = True
    >>> b
    True
    >>> b = False
    >>> b
    False

### Negation

Boolean variables can be flipped using the not keyword.

    >>> not True
    False
    >>> not False
    True
    >>> b = True
    >>> not b
    False
    >>> b = False
    >>> not b
    True

### Boolean operations And and Or

Boolean operations are available using keywords and and or for the operations respectively.

    >>> True and False
    False
    >>> True or False
    True
    >>> a = True
    >>> b = False
    >>> a and b
    False
    >>> a or b
    True

### Arithmetic operations

In arithmetic operators, True is treated as 1, and False is treated as 0.

    >>> True + False
    1
    >>> True + True
    2
    >>> False + True
    1
    >>> True + 2
    3
    >>> False - 2
    -2
    >>> True * 10
    10
    >>> False * 7
    0

### Typecasting

Numbers can be typecasted to Booleans using the bool function. 0 evaluates to False, and all other numbers evaluate to True.

    >>> bool(0)
    False
    >>> bool(1)
    True
    >>> bool(3.14)
    True
    >>> bool(-3.14)
    True
    >>> bool(0.0)
    False

Using Boolean operators and and or automatically typecasts numbers to Boolean.

    >>> 5 or 0 #equivalent to True or False
    5
    >>> 5 and 0 #equivalent to True and False
    0

Empty string typecasts to False, while nonempty string typecasts to True.

    >>> bool("")
    False
    >>> bool("I love Python")
    True

### Evaluating Booleans

Equality can be checked by using the == operator.

    >>> 5 == 5
    True
    >>> 5 == -5
    False

All inequality checks have syntax corresponding to their meanings.

    >>> 5 < 4
    False
    >>> 5 > 4
    True
    >>> 5 <= 4
    False
    >>> 5 >= 4
    True

Simple inequality (not equals) can checked using !=.

    >>> 5 != 5
    False
    >>> 5 != 4
    True

## **Working With None**

The None keyword is used to denote an empty object.

    >>> x = None #Variable defined and set as None
    >>> x # No error but no result either as x is None
    >>> y # We see error as y is undefined, which is different from None
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    NameError: name 'y' is not defined

None evaluates to False when converted to a Boolean.

    >>> bool(None)
    False

This article is part of a series of Python Cheat Sheets. The next installments cover [variables and collections](https://medium.com/p/67baefbf5f12), and [using dictionaries and sets](https://medium.com/post/4af7aaa17598).
