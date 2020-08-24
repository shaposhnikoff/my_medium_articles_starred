Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m6[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m118[39m, end: [33m120[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m76[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m60[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m88[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m6[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m78[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m56[39m }

# Zero to Hero Python Cheat Sheet: Working with Collections

Learning about interacting with the console, lists and tuples

![Photo by [Hitesh Choudhary](https://unsplash.com/@hiteshchoudhary?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/10944/1*yA1fxkE5WZwA9G-fNmXt1g.jpeg)*Photo by [Hitesh Choudhary](https://unsplash.com/@hiteshchoudhary?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

In the [previous Zero to Hero Python Cheat Sheet](https://medium.com/@arindamroy/zero-to-hero-python-cheat-sheet-primitive-data-types-44bd4b29fe95), we covered dealing with primitive data types like integers, strings and boolean values. In this piece, we will be looking at interacting with the console and learning about lists and tuples.

The next post covers[ working with dictionaries and sets](https://medium.com/post/4af7aaa17598).

*Let’s get started!*

## Working with the Console

### Read data from the console

Use the in-built input function to accept parameters from the console. input accepts a string parameter, which is the message printed to console to ask for input.

    >>> str = input("Input your string : ")
    Input your string : I love Python!
    >>> str
    'I love Python!'

*Note: *input* function reads only string values from console. Use typecasting to convert them to required data type.*

### Write Data to Console

Use the in-built print function to write data to console:

    >>> print("I love Python!")
    I love Python!
    >>> str = "I love Python variables too!"
    >>> print(str)
    I love Python variables too!

The print function prints a newline at the end by default:

    >>> print("Line 1"); print("line 2");
    Line 1
    line 2

To change this behavior, pass end parameter to the print function:

    >>> print("I love Python", end="!"); print("No newline here.")
    I love Python!No newline here.

## Working with Lists

### Initialisation

Lists are used to store sequences of data. Elements of a list are comma-separated and written inside square-brackets ([]).

Here’s a list of the first five integers:

    >>> numbers = [1, 2, 3, 4, 5]
    >>> print(numbers)
    [1, 2, 3, 4, 5]

You can also initialise an empty list:

    >>> emptyList = []
    >>> print(emptyList)
    []

### Adding elements

Add elements to a list with the in-built append function:

    >>> numbers = [1, 2, 3, 4, 5]
    >>> numbers.append(6)
    >>> print(numbers)
    [1, 2, 3, 4, 5, 6]

To insert an element to a specific index, use the in-built insert function. insert takes two parameters, index and element respectively:

    >>> numbers = [1, 2, 3, 4, 5]
    >>> numbers.insert(2, 99)
    >>> print(numbers)
    [1, 2, 99, 3, 4, 5]

### Fetching elements

Elements can be accessed using indices, similar to arrays:

    >>> numbers = [1, 2, 3, 4, 5]
    >>> x = numbers[0]
    >>> y = numbers[4]
    >>> print(x,y)
    1 5

If you try to access an index that does not exist in the list, you will get an IndexError:

    >>> numbers = [1, 2, 3, 4, 5]
    >>> x = numbers[9]
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    IndexError: list index out of range

### Fetching a range of elements

A range of elements can be extracted from a list using the : syntax.

    >>> numbers = [1, 2, 3, 4, 5]
    >>> lessNumbers = numbers[1:3]
    >>> print(lessNumbers)
    [2, 3]

**Note: The *first parameter is inclusive* and the *second parameter is exclusive.***

To if you only want elements up to an index, you may omit the first parameter. Likewise, if you only want elements from a certain index, you may omit the second parameter.

    >>> numbers = [10, 20, 30, 40, 50]
    >>> firstThree = numbers[:3]
    >>> print(firstThree)
    [10, 20, 30]
    >>> lastThree = numbers[2:]
    >>> print(lastThree)
    [30, 40, 50]

To fetch every *nth *element from a list, use ::n:

    >>> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    >>> thirdNumbers = numbers[::3]
    >>> print(thirdNumbers)
    [1, 4, 7, 10]

To fetch every *nth* element starting from the *mth* element, use m::n:

    >>> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    >>> evenNumbers = numbers[1::2]
    >>> print(evenNumbers)
    [2, 4, 6, 8, 10]

### Deleting elements

To remove an element from a list from a specific index, use the in-built del command:

    >>> numbers = [1, 2, 3, 4, 5]
    >>> del numbers[2] #deletes element at index 2
    >>> print(numbers)
    [1, 2, 4, 5]

To delete a particular element from a list whose value you know, use the in-built remove function:

    >>> numbers = [1, 2, 3, 4, 5]
    >>> numbers.remove(3) #deletes the element with value=3
    >>> print(numbers)
    [1, 2, 4, 5]

Note: remove deletes the *first occurence* of the value from the list:

    >>> numbers = [1, 2, 1, 2]
    >>> numbers.remove(2) # removes first occurence of 2 at index=1
    >>> print(numbers)
    [1, 1, 2]

### Reversal

To reverse a list, use ::-1:

    >>> numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    >>> reversed = numbers[::-1]
    >>> print(reversed)
    [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]

### Concatenation

To join two lists into a new list, use the + operator:

    >>> n1 = [1, 2, 3]
    >>> n2 = [4, 5, 6]
    >>> sum = n1 + n2
    >>> print(sum)
    [1, 2, 3, 4, 5, 6]

To append a list to another lists, use the in-built extend function:

    >>> bigList = [1, 2, 3]
    >>> smallList =[4, 5, 6]
    >>> bigList.extend(smallList)
    >>> print(bigList)
    [1, 2, 3, 4, 5, 6]

### Length

To find the length of a list, use the in-built len function:

    >>> numbers = [1, 2, 3, 4, 5]
    >>> len(numbers)
    5

### Existence

To check if an element is present in a list, use the in command:

    >>> numbers = [1, 2, 3, 4, 5]
    >>> 1 in numbers
    True
    >>> 10 in numbers
    False

## Working With Tuples

Tuples hold sequences of data just like lists. The key difference between lists and tuples is that *tuples are immutable.* This means that once created, tuples cannot be modified.

To create a tuple, create a comma-separated list of elements within brackets (()).

    >>> numbers = (1, 2, 3, 4, 5)
    >>> print(numbers)
    (1, 2, 3, 4, 5)

### Fetching elements

Elements can be accessed using indices, similar to arrays.

    >>> numbers = (1, 2, 3, 4, 5)
    >>> numbers[0]
    1

### Fetching a range of elements

All syntaxes for fetching a range of elements from a tuple are *similar to the syntax of lists *above.

### Changing a tuple

Any attempt to change a tuple will raise a TypeError:

    >>> numbers = (1, 2, 3, 4, 5)
    >>> numbers[2] = 99
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: 'tuple' object does not support item assignment

### Length

Use the in-built len function to find the length of a tuple:

    >>> numbers = (1, 2, 3, 4, 5)
    >>> len(numbers)
    5

### Unpacking

Use comma separated variables to unpack a tuple into its elements:

    >>> numbers = (100, 200, 300)
    >>> x, y, z = numbers
    >>> print("x=%d, y=%d, z=%d" %(x, y, z))
    x=100, y=200, z=300

Unpacking can also be done asymmetrically by using the * in front of the variable which will contain more than one element. These variables are stored as lists:

    >>> numbers = (1, 2, 3, 4, 5, 6)
    >>> first, second, *remaining = numbers
    >>> print(first)
    1
    >>> print(second)
    2
    >>> print(remaining)
    [3, 4, 5, 6]

*In the next edition of the Zero to Hero Python series, we’ll be looking at [Dictionaries and Sets, a way of storing key-value mapping in Python](https://medium.com/post/4af7aaa17598). For any questions or suggestions, please reach out to me in the comments.*
