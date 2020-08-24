Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m53[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m6[39m, end: [33m7[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m90[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m83[39m, end: [33m85[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m93[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m108[39m, end: [33m117[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m90[39m, end: [33m93[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m115[39m, end: [33m119[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m155[39m, end: [33m158[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m58[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m68[39m }

# Zero to Hero Python Cheat Sheet: Working With Dictionaries and Sets

Learning to use sets and dictionaries in Python

![Photo by [Hitesh Choudhary](https://unsplash.com/@hiteshchoudhary?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/10944/1*yA1fxkE5WZwA9G-fNmXt1g.jpeg)*Photo by [Hitesh Choudhary](https://unsplash.com/@hiteshchoudhary?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

In the previous installments of the *Zero to Hero Python series*, we covered:

1. [Dealing with primitive data types like integers, strings, and boolean values](https://medium.com/better-programming/zero-to-hero-python-cheat-sheet-primitive-data-types-44bd4b29fe95).

1. [Interacting with the console and learning about lists and tuples.](https://medium.com/better-programming/zero-to-hero-python-cheat-sheet-working-with-collections-67baefbf5f12)

In this piece, we will be looking at Python’s in-built collections for supporting mathematical concepts like sets and key-value pairs.

Let’s get started!

## Working With Sets

A set is used to store a sequence of *unique* values.

### Initialization

A set can be declared by making use of curly braces {}.

    >>> numSet = {1, 2, 3, 4, 5}
    >>> print(numSet)
    {1, 2, 3, 4, 5}

If you put duplicate elements in the initialization of a set, only one instance of the elements will be retained.

    >>> numSet = {1, 2, 2, 3, 3, 3, 4, 4, 4, 4, 5}
    >>> print(numSet)
    {1, 2, 3, 4, 5}

You can also initialize an empty set using the in-built set function.

    >>> emptySet = set()
    >>> print(emptySet)
    set()

Note: Elements of a set must be immutable. Adding mutable objects to a set will raise an error.

    >>> tuple1 = (1, 2, 3)
    >>> tuple2 = (4, 5, 6)
    >>> tupleSet = {tuple1, tuple2} # no error as tuples are immutable
    >>> print(tupleSet)
    {(4, 5, 6), (1, 2, 3)}
    >>> list1 = [1, 2, 3]
    >>> list2 = [4, 5, 6]
    >>> listSet = {list1, list2} #will raise error as lists are mutable
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: unhashable type: 'list'

### Adding elements

Add elements to a set by using the in-built add function.

    >>> numSet = {1, 2, 3, 4, 5}
    >>> numSet.add(6)
    >>> print(numSet)
    {1, 2, 3, 4, 5, 6}

Note: If you attempt to add a duplicate element to a set, there will be no change to the set. No error will be raised by the add function in this case.

    >>> numSet = {1, 2, 3, 4, 5}
    >>> numSet.add(5)
    >>> print(numSet)
    {1, 2, 3, 4, 5}

### Deleting elements

Remove elements from a set by using the in-built remove function.

    >>> numSet = {1, 2, 3, 4, 5}
    >>> numSet.remove(5)
    >>> print(numSet)
    {1, 2, 3, 4}

Note: Trying to remove an element that does not exist will raise an error.

    >>> numSet = {1, 2, 3, 4, 5}
    >>> numSet.remove(99)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    KeyError: 99

### Length

Find the length of a set using the in-built len function.

    >>> numSet = {1, 2, 3, 4, 5}
    >>> len(numSet)
    5

### **Existence**

Check for the existence of an element in a set using the in operator.

    >>> numSet = {1, 2, 3, 4, 5}
    >>> 2 in numSet
    True
    >>> 99 in numSet
    False

Now, let us look at performing *set operation*s.

### Set intersection

Find the intersection of two sets using the & operator.

    >>> setA = {1, 2, 3, 4, 5}
    >>> setB = {3, 4, 5, 6, 7}
    >>> intersection = setA & setB
    >>> print(intersection)
    {3, 4, 5}

### Set union

Find the intersection of two sets using the | operator.

    >>> setA = {1, 2, 3, 4, 5}
    >>> setB = {3, 4, 5, 6, 7}
    >>> union = setA | setB
    >>> print(union)
    {1, 2, 3, 4, 5, 6, 7}

### Set difference

Set difference returns a set of elements present in the first set and not present in the second set.

Find difference of a set from some other set using the - operator.

    >>> setA = {1, 2, 3, 4, 5}
    >>> setB = {3, 4, 5, 6, 7}
    >>> difference = setA - setB
    >>> print(difference)
    {1, 2}
    >>> reverseDifference = setB - setA
    >>> print(reverseDifference)
    {6, 7}

### Set symmetric difference

Symmetric difference returns a set of elements present in exactly one of the two sets, but not both sets.

Find the symmetric difference of two sets using the ^ operator.

    >>> setA = {1, 2, 3, 4, 5}
    >>> setB = {3, 4, 5, 6, 7}
    >>> symmDiff = setA ^ setB
    >>> print(symmDiff)
    {1, 2, 6, 7}

### Check superset

A set A is a superset of a set B if all elements present in set B are also present in set A.

Check if the left-hand side set is a superset of the right-hand side set using the >= operator.

    >>> bigSet = {1, 2, 3, 4, 5}
    >>> smallSet = {3, 4}
    >>> isSuperSet = bigSet >= smallSet
    >>> print(isSuperSet)
    True

To check if the set on the right-hand side set is a superset of the left-hand set, use the <= operator.

    >>> bigSet = {1, 2, 3, 4, 5}
    >>> smallSet = {3, 4}
    >>> isSuperSet = smallSet <= bigSet
    >>> print(isSuperSet)
    True

## Working With Dictionaries

Dictionaries are used to store key-value pairs in Python.

### Initialization

Dictionaries are also initialized using the curly braces {}, and the key-value pairs are declared using the key:value syntax.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> print(nameToNumber)
    {'John': 1, 'Harry': 2, 'Jacob': 3}

You can also initialize an empty dictionary by using the in-built dict function.

    >>> emptyDict = dict()
    >>> print(emptyDict)
    {}

Empty dictionaries can also be initialized by simply using empty curly braces.

    >>> emptyDict = {}
    >>> print(emptyDict)
    {}

Note: The keys in a dictionary must be immutable. Attempts to create a dict with mutable keys will raise an error.

    >>> tupleA = (1, 2, 3) # tuples are immutable
    >>> stringA = "I love Python!" # strings are immutable
    >>> floatA = 3.14 # float values are immutable
    >>> dictA = {tupleA : True, stringA : False, floatA : True} # no error as all keys are immutable
    >>> print(dictA)
    {(1, 2, 3): True, 'I love Python!': False, 3.14: True}
    >>> listB = [1, 2, 3] #list is mutable
    >>> dictB = {listB : True} # raises an error as lists are mutable
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: unhashable type: 'list'

### Fetching data

Get a value from a dictionary using its key using square brackets ([]).

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> JohnsNumber = nameToNumber["John"]
    >>> print(JohnsNumber)
    1

Note: Trying to fetch a key that does not exist will raise an error.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> nameToNumber["Sam"]
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    KeyError: 'Sam'

To avoid this error, use the in-built get function. Fetching a non existent key using the get function will return None, but will not throw any error.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> johnsNumber = nameToNumber.get("John")
    >>> print(johnsNumber)
    1
    >>> samsNumber = nameToNumber.get("Sam")
    >>> print(samsNumber)
    None

If a key is missing in a dictionary, we can use the get function to return a default value. Pass the default value you want as the second parameter in the get function.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> johnsNumber = nameToNumber.get("John", 99)
    >>> print(johnsNumber)
    1
    >>> samsNumber = nameToNumber.get("Sam", 99)
    >>> print(samsNumber)
    99

### Modifying data

Insert data into a dictionary using the in-built setdefault function.

The setdefault function will create a new key-value pair into the dictionary only if the key does not exist in the dictionary. If the key already exists, it will *not* be overridden.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> nameToNumber.setdefault("Sam", 4)
    4
    >>> print(nameToNumber)
    {'John': 1, 'Harry': 2, 'Jacob': 3, 'Sam': 4}
    >>> nameToNumber.setdefault("Sam", 99) # no changes as the key already exists
    4
    >>> print(nameToNumber)
    {'John': 1, 'Harry': 2, 'Jacob': 3, 'Sam': 4}

To modify existing data in a dictionary, use the in-built update function.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> nameToNumber.update({"Sam" : 4}) # creates new entry
    >>> print(nameToNumber)
    {'John': 1, 'Harry': 2, 'Jacob': 3, 'Sam': 4}
    >>> nameToNumber.update({"Sam" : 99}) # updates existing entry
    >>> print(nameToNumber)
    {'John': 1, 'Harry': 2, 'Jacob': 3, 'Sam': 99}

Existing data can also be modified by using the square bracket syntax.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> nameToNumber["Sam"] = 4 # creates new entry
    >>> print(nameToNumber)
    {'John': 1, 'Harry': 2, 'Jacob': 3, 'Sam': 4}
    >>> nameToNumber["Sam"] = 99 # updates existing entry
    >>> print(nameToNumber)
    {'John': 1, 'Harry': 2, 'Jacob': 3, 'Sam': 99}

### Deleting data

Delete keys from a dictionary using the del command.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> del nameToNumber["John"]
    >>> print(nameToNumber)
    {'Harry': 2, 'Jacob': 3}

Note: Trying to delete a key that does not exist will result in an error.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> del nameToNumber["Sam"]
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    KeyError: 'Sam'

### **Iterations**

We can iterate over the keys in a dictionary using the in-built keys function.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> names = list(nameToNumber.keys()) # using list() to store in a list
    >>> print(names)
    ['John', 'Harry', 'Jacob']

We can iterate over values in a dictionary using the in-built values function.

    >>> nameToNumber = {"John" : 1, "Harry" : 2, "Jacob" : 3}
    >>> values = list(nameToNumber.values())
    >>> print(values)
    [1, 2, 3]

In the next edition of the *Zero to Hero Python series*, we’ll be looking at writing functions in Python, as well as using iterables to process data. For any questions or suggestions, please reach out to me in the comments.
