
# 100 Days of DevOps — Day 88-Lists in Python

Welcome to Day 88 of 100 Days of DevOps, Focus for today is Lists in Python

## Lists in Python

* *The list is a value that contains multiple values(mutable sequence)*

    *>>> x = [1,2,’abc’,2.5]*

    *>>> x*

    *[1, 2, 'abc', 2.5]*

![](https://cdn-images-1.medium.com/max/2000/1*fT2L2KMjNuA78J6HvD_eqg.png)

* *We can access the item in the list with an integer **index** that start with 0(not 1)*

    *>>> x[0]*

    *1*

* *We can use the **negative index** to refer to the last item in the list*

    *>>> x[-1]*

    *2.5*

* *To get multiple items from the list use **slice***

    *>>> x[0:2]*

    *[1, 2]*

    ***#Basically grab every value of the list***

    *>>> x[0:]*

    *[1, 2, 'abc', 2.5]*

* *All the **function** that works with strings works in the same way with list eg:len()*

    *>>> len(x)*

    *4*

* *To delete the value from the list use **del***

    *>>> del x[0]*

    *>>> x*

    *[2, ‘abc’, 2.5]*

* *We can convert a value into a list by passing it to the **list()** function*

    *>>> list(‘hello’)*

    *[‘h’, ‘e’, ‘l’, ‘l’, ‘o’]*

* *To find out the value in list use **in** operator*

    *>>> ‘h’ in ‘hello’*

    *True*

    *#Opposite of that is not in operator*

    *>>> 'h' not in 'hello'*

    *False*

* *Use of for loop with a list, so as you can see for loop iterate over the values in a list*

    *>>> for i in x:*

    *… print(i)*

    *…*

    *2*

    *abc*

    *2.5*

* ***range()** function returns a list-like value which we can pass to the **list()** function if we need an actual value*

    *>>> list(range(0,4))*

    *[0, 1, 2, 3]*

*Let’s take one more example*

    *>>> items = [“a”,”b”,”c”,”d”]
    >>> for i in range(len(items)):*

    *...     print("Value at "+ str(i) + " is equal to: " + items[i])*

    *...*

    *Value at 0 is equal to: a*

    *Value at 1 is equal to: b*

    *Value at 2 is equal to: c*

    *Value at 3 is equal to: d*

***Swapping Variables***

    *>>> a,b,c = 1,2,3*

    *>>> a*

    *1*

    *>>> b*

    *2*

    *>>> c*

    *3*

***Lists Methods***

*Methods are functions that are called on values. List has several methods*

* ***index()** list method returns the index of an item in the list*

    *>>> test = [“a”,”b”,”c”,”d”]*

    *>>> test.index(“a”)*

    *0*

    ***# Index method will raise the exception if it doesn't find the value***

    *>>> test.index("e")*

    *Traceback (most recent call last):*

    *File "<stdin>", line 1, in <module>*

    *ValueError: 'e' is not in list*

    ***# Now in case of duplicate list it only return the index of first value***

    *>>> test = ["a",**"b"**,"c","d","a",**"b"**]*

    *>>> test.index("b")*

    *1*

* ***append()** list method adds value to the end of a list*

    *>>> test.append(“z”)*

    *>>> test*

    *[‘a’, ‘b’, ‘c’, ‘d’, ‘a’, ‘b’, ‘z’]*

* ***insert()** list method adds value anywhere inside a list*

    *>>> test.insert(0,”hola”)*

    *>>> test*

    *[‘hola’, ‘a’, ‘b’, ‘c’, ‘d’, ‘a’, ‘b’, ‘z’]*

* ***remove()** list method removes an item, specified by the value from a list*

    *>>> test.remove(“a”)*

    *>>> test  **#Only first incident of that value is removed***

    *[‘hola’, ‘b’, ‘c’, ‘d’, ‘a’, ‘b’, ‘z’]*

* **reverse(): **A list can be reversed by calling its reverse() method

    >>> a = [2,5,1,3,6,4]

    >>> a.reverse()

    >>> a

    [4, 6, 3, 1, 5, 2]

* ***sort()** list method sorts the items in a list*

    *>>> test1 = [8,3,9,4,6,3]*

    *>>> test1.sort()*

    *>>> test1*

    *[3, 3, 4, 6, 8, 9]*

*But if we want to sort a combination of numbers and strings, Python throws an error **as it doesn’t know how to sort it***

    *>>> test2 = [1,2,3,”a”,”b”]*

    *>>> test2 = [1,3,2,6,5,”a”,”b”]*

    *>>> test2.sort()*

    *Traceback (most recent call last):*

    *File “<stdin>”, line 1, in <module>*

    *TypeError: ‘<’ not supported between instances of ‘str’ and ‘int’*

*The way sorting works is using **ASCII-betical** order that is the upper case first*

    *>>> test3 = [‘a’,’b’,’A’,’B’]*

    *>>> test3.sort()*

    *>>> test3*

    *[‘A’, ‘B’, ‘a’, ‘b’]*

*This doesn’t look good as A should be followed by a, so to do that pass **str.lower** which is technically converting everything to lowercase*

    *>>> test3.sort(key=str.lower)*

    *>>> test3*

    *[‘A’, ‘a’, ‘B’, ‘b’]*

    **# The important use case of key, where we use len as a key(here sorting happens based on the length)
    **>>> b

    ['hello', 'how', 'are', 'u', 'Mr', 'Prashant']

    >>> b.sort(key=len)

    >>> b

    ['u', 'Mr', 'how', 'are', 'hello', 'Prashant']

***NOTE: These list methods operate on the list “in place”, rather than returning a new list value***

* **Sorted():** built-in function sorts any iterable series and returns a list

    >>> x = [5,2,3,1]

    >>> sorted(x)

    [1, 2, 3, 5]

    >>> x

    [5, 2, 3, 1]

* **count(): **Count returns the number of matching elements

    >>> a = [1,2,3,4,1,23,5,6]

    >>> a.count(1)

    2

*New few things we need to be **really careful** about lists*

    *>>> test = [1,2,3,4]*

    *>>> test1 = test*

    *>>> test1*

    *[1, 2, 3, 4]*

    *>>> test1[0] = 9*

    *>>> test1*

    *[9, 2, 3, 4]*

    *>>> test*

    *[9, 2, 3, 4]*

*We only change the value of test1 but the value of the test got changed?*

*The Reason for that when we created this list(test), Python created this list in computer memory but it’s assigned a **reference** to test. Now when we run this*

    *test1 = test*

*A reference get copied to test1 and they referencing the same list and that will cause all sort of weirds error,so please be really careful when dealing with lists*

![](https://cdn-images-1.medium.com/max/2000/1*tSJV97Bgg2HrjglK0SzBFQ.png)

*We don’t have this kind of issues with immutable values like **strings/tuples** as we can’t replace it by new values*

    *>>> a = “abc”*

    *>>> b = a*

    *>>> b = “efg”*

    *# Change in b will not impact a
    >>> b*

    *‘efg’*

    *>>> a*

    *‘abc*

*Let’s go much deeper into the same concept, as you see it’s just the reference which is changing here. Now we have no way to reach x = 100 so at some point of time Python garbage collector will take care of it.*

* *id(): returns a unique identifier for an object*

    *>>> x = 100*

    *>>> id(x)*

    *4340331440*

    *>>> x = 50*

    *>>> id(x)*

    *4340329840*

*So how to take care of this kind of issues in which we want a completely separate list, for that we have a module called **copy**(a copy has a module called **deep copy** which creates a brand new list and returns a reference to new list)*

    *>>> x = [1,2,3]*

    *>>> import copy*

    *>>> y = copy.deepcopy(x)*

    *>>> y*

    *[1, 2, 3]*

    *>>> y[0] = 4*

    *>>> y*

    *[4, 2, 3]*

    *>>> x*

    *[1, 2, 3]*

**OR(full slice technique)**

    >>> a = [4,5,6]

    >>> b = a[:]

    >>> b

    [4, 5, 6]

    >>> id(a)

    **4325208200**

    >>> id(b)

    **4325208328**

    >>> b[0] = 10

    >>> b

    [10, 5, 6]

    >>> a

    [4, 5, 6]

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
[**100 Days of DevOps — Day 87-While/For Loop Python**
*Welcome to Day 87 of 100 Days of DevOps, Focus for today is While/For Loop Python*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-87-while-for-loop-python-cf405b6e868f)
