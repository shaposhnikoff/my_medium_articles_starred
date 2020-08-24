Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m60[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m93[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m36[39m }
Unknown markup type 10 {
  type: [33m10[39m,
  start: [33m107[39m,
  end: [33m110[39m,
  href: [32m''[39m,
  title: [32m''[39m,
  rel: [32m''[39m,
  name: [32m''[39m,
  anchorType: [33m0[39m,
  creatorIds: [],
  userId: [32m''[39m
}
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m83[39m }
Unknown markup type 10 {
  type: [33m10[39m,
  start: [33m132[39m,
  end: [33m142[39m,
  href: [32m''[39m,
  title: [32m''[39m,
  rel: [32m''[39m,
  name: [32m''[39m,
  anchorType: [33m0[39m,
  creatorIds: [],
  userId: [32m''[39m
}
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m148[39m, end: [33m153[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m208[39m, end: [33m222[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m69[39m, end: [33m89[39m }

# Elements of Functional Programming in Python

Learn how to use the lambda, map, filter, and reduce functions in Python to transform data structures.

![Photo by [Markus Spiske](https://unsplash.com/@markusspiske?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/11520/0*zmC8C1YFNx_QStAQ)*Photo by [Markus Spiske](https://unsplash.com/@markusspiske?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*
> # “Object-oriented programming makes code understandable by encapsulating moving parts. Functional programming makes code understandable by minimizing moving parts.” : [Michael Feathers](https://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052/ref=as_li_ss_tl?ie=UTF8&qid=1470702619&sr=8-1&keywords=Working+with+Legacy+Code&linkCode=sl1&tag=devdaily-20&linkId=2fee99425a0027b3ac69953055fb9e9e)

There are multiple programming languages in the world, and so are the categories in which they can be classified. A **programming paradigm **is one such** **way which tries to classify programming languages based on their features or coding style. A programming paradigm is essentially a style or a way of programming.

Most of the times, we understand Python as an **object-oriented **language, where we **model our data in the form of classes, objects, and methods. **However, there also exist several alternatives to OOP, and **functional programming is one of them.**

Here are some of the conventional programming paradigms prevalent in the industry:

![Conventional programming paradigms: Content Source [Wikipedia](https://en.wikipedia.org/wiki/Programming_paradigm)](https://cdn-images-1.medium.com/max/2000/1*TOsk9zG2gpYmf5d2JmIcUA.png)*Conventional programming paradigms: Content Source [Wikipedia](https://en.wikipedia.org/wiki/Programming_paradigm)*

## Functional Programming(FP)

![The Functional Programming Paradigm](https://cdn-images-1.medium.com/max/2000/1*1yVFdiXsp3u40OZfCrG14A.png)*The Functional Programming Paradigm*

As per Wikipedia, [*Functional programming](https://en.wikipedia.org/wiki/Functional_programming) is a programming paradigm, a style of building the structure and elements of computer programs, that treats computation as the evaluation of mathematical **functions **and **avoids** changing **state** and **mutable** data.*

The above definition might sound confusing at first, but it essentially tries to put forward the following aspects:

* FP relies on functions, and everything is done using functions. Moreover, FP focuses on **defining** what to do, instead of performing some action. The functions of this paradigm are treated as **first-class** functions. This means functions are treated like any other objects, and we can assign them to variables or pass them into other functions.

* The data used in functional programming must be **immutable, **i.e. should never change. This means if we need to modify a data in a list, we need to create a new list with updated values rather than manipulating the existing one.

* The programs written in FP should be **stateless**. A stateless function has no knowledge about its past. Functional programs should carry out every task as if they are performing it for the first time. Simply put, the functions are only dependent on the data passed to them as arguments and never on the outside data.

* **Laziness** is another property of FP wherein we don’t compute things that we don’t have to. Work is only done on demand.

If this makes sense now, here is a nice comparison chart between the OOP and FP which will make things even more apparent.

![**Original Image: [www.educba.com](https://www.educba.com/)**](https://cdn-images-1.medium.com/max/2000/1*-gu_BRWWhEKJrIbF-_Olmg.png)***Original Image: [www.educba.com](https://www.educba.com/)***

Python provides features like lambda, filter, map, and reduce that can easily demonstrate the concept of Functional Programming. All the codes used in this article can be accessed from the associated [Github Repository](https://github.com/parulnith/Elements-of-Functional-Programming-in-Python) or can be viewed on my_binder by clicking the image below.

![](https://cdn-images-1.medium.com/max/2000/1*diXEGUjFEsKvBrHdiB2htA.png)

## The Lambda Expression

**Lambda expressions** — also known as “anonymous functions” — allow us to create and use a function in a single line. They are useful when we need a short function that will be used only once. They are mostly used in conjunction with the map, filter and the sort methods, which we will see later in the article.

Let’s write a function in Python, that computes the value of 5x + 2. The **standard approach** would be to define a function.

![](https://cdn-images-1.medium.com/max/2720/1*rQNizMFN-o99geo15sRnZQ.png)

Now let’s compute the same expression using **Lambda** functions. To create a lambda expression, we type in the keyword lambda, followed by the inputs. Next, we enter a colon, followed by the expression that will be the return value.

![](https://cdn-images-1.medium.com/max/2720/1*nZLkWix5FpI46Lpefb9Ffg.png)

This lambda function will take the input x and return 5x + 2, just like the earlier function f. There is a problem, however. Lambda is not the name of the function. It is a Python keyword that says that what follows is an anonymous function. So how do we use it? One way is to give it a name.

Let us call this lambda expression g. Now, we can use this like any other function.

![](https://cdn-images-1.medium.com/max/2720/1*PP1rynAEmxkUSXD5nyIxjg.png)

### Lambda expression with multiple inputs.

The examples below show how lambda functions can be used with or without input values.

![](https://cdn-images-1.medium.com/max/2666/1*SoD-pLkczzQhrRLbfYs3Ow.png)

### Lambda expression without inputs.

Now, let’s look at a common use of Lambda function where we do not assign it a name. Let’s say we have a list of the first seven U.S Presidents and we’d like to sort this list by their last name. We shall create a Lambda function that extracts the last name, and uses that as the sorting value.

![](https://cdn-images-1.medium.com/max/5440/1*kE_LifRrWOwgBKbR-pY_Qg.png)
> The map, filter, and reduce functions simplify the job of working with lists. When used along with lambda expressions they help to make our lives easier by accomplishing a lot in a single line of code.

## The Map Function

The [map](https://docs.python.org/3/library/functions.html#map) function applies a function to every item of *iterable*, yielding the results. When used with lists, Map transforms a given list into a new list by applying the function to all the items in an input_list.

### Syntax

    map(function_to_apply, iterables)

### Usage

Suppose we have a function that computes the volume of a cube, given the value of its edge(a).

    def volume(a):
        """volumne of a cube with edge 'a'"""
        return a**3

What if we need to compute the volumes for many different cubes with different edge lengths?

    # Edge length in cm
    edges = [1,2,3,4,5]

There are two ways to do this — one by using the **direct** method and the other by using the **map** function.

![](https://cdn-images-1.medium.com/max/5440/1*NwIEY5quwhTuPbVF2789vw.png)

Now let’s see how we can accomplish this task using a single line of code with the map function.

![](https://cdn-images-1.medium.com/max/5440/1*2lRO7Z6e4jhmgbQ0ftbLpA.png)

The map function takes in two arguments. The first is a function, and the second is our list, tuple, or any other iterable object. Here, the map function applies the volume function to each element in the list.

An important thing to note here is that the output of the map function is not a list but a map object, which is an iterator over the results. We can, however, turn this into a list by passing the map to the list constructor.

### Example

Let’s now see an example which demonstrates the use of lambda function with the map function. We have a list of tuples containing name and heights for five people. Each of the height is in centimeters, and we need to convert it into feet.

We will first write a converter function using a lambda expression which will accept a tuple as the input and will also return a tuple.

![](https://cdn-images-1.medium.com/max/6240/1*Zd9gS-wPVhD-s5PbUfBoGA.png)

## The Filter Function

The ‘[filter’ ](https://docs.python.org/3/library/functions.html#filter)function constructs an iterator from those elements of *iterable* for which *function* returns true. This means filter function is used to select certain pieces of data from a list, tuple, or other collection of data, hence the name.

### Syntax

    filter(function, iterable)

### Usage

Let’s see an example where we want to get the list of all numbers that are greater than 5, from a given input list.

![](https://cdn-images-1.medium.com/max/5440/1*eomHvuNhT0PUbSI7aJsoXg.png)

We first create a lambda function that tests the input to see if it is above 5 or not. Next, we pass in the list of data. The filter function will only return the data for which the function is true. Once again, the return value is not a list, but a filter object. This object has to be passed to a list constructor to get the output.

### Example

An interesting use case of the ‘filter’ function arises when the data consists of missing values. Here is a list containing some of the countries in Asia. Notice numerous strings are empty. We’ll use the filter function to remove these missing values. We’ll pass none as the first argument, and the second argument is the list of data as before.

![](https://cdn-images-1.medium.com/max/5704/1*5hAHj0aVBLiIhC4gvHtWiA.png)

This filters out all values that are treated as false in a boolean setting.

## The Reduce Function

The ‘r[educe](https://docs.python.org/3/library/functools.html?highlight=functools%20reduce#functools.reduce)’ function is a bit unusual, and, as of Python 3, it is no longer a built-in function. Instead, it has been moved to the functools module. The ‘reduce’ function transforms a given list into a single value by applying a function cumulatively to the items of *sequence*, from left to right,

### Syntax

    reduce(func, seq)

where reduce continually applies the function func() to the sequence seq and returns a single value.

### Usage

Let’s illustrate the working of the reduce function with the help of a simple example that computes the product of a list of integers.

![](https://cdn-images-1.medium.com/max/5440/1*JDm0Q_6_eFLyBgadxgRzlA.png)

The following diagram shows the intermediate steps of the calculation:

![Source: [Working of ‘Reduce’ function in Python](https://www.python-course.eu/lambda.php)](https://cdn-images-1.medium.com/max/2000/1*tFi8CEmD3eAPwP3_nHWaTg.png)*Source: [Working of ‘Reduce’ function in Python](https://www.python-course.eu/lambda.php)*

However, Guido van Rossum, the creator of Python, had to say this about the ‘reduce’ function:
> Use functools.reduce if you really need it; however, 99% of the time an explicit for loop is more readable.

The above program can also be written with an explicit for loop:

![](https://cdn-images-1.medium.com/max/5440/1*cJeB7yx0SN20ctCGuRmYlQ.png)

### Example

The ‘reduce’ function can determine the maximum of a list containing integers in a single line of code. There does exist a built-in function called max() in Python which is generally used for this purpose as max(list_name).

![](https://cdn-images-1.medium.com/max/5440/1*JB2XGZ84BZWiW_EQrigVvQ.png)

## List Comprehensions: Alternative to map, filter and reduce

[List comprehension](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions) is a way to define and create lists in Python. In most cases, list comprehensions let us create lists in a single line of code without worrying about initializing lists or setting up loops.

It is also a substitute for the lambda function as well as the functions map(), filter() and reduce(). Some people find it a more *pythonic* way of writing functions and find it easier to understand.

### Syntax

![](https://cdn-images-1.medium.com/max/2810/1*FdeynwA-yejIoCW81tvaCQ.png)

![](https://cdn-images-1.medium.com/max/3668/1*XOTv6_ghdQU10Zs4X31vdA.png)

### Usage

Let’s try and replicate the examples used in the above sections with list comprehensions.

* **List Comprehensions vs. Map function**

We used map function in conjunction with the lambda function to convert a list of heights from cm to feet. Let’s use list comprehensions to achieve the same results.

![](https://cdn-images-1.medium.com/max/6240/1*l-zGbFAEx52zrRrFHtwYWg.png)

* **List Comprehensions vs. Filter function**

We used the filter function to remove the missing values from a list of Asian countries. Let’s use list comprehensions to get the same results.

![](https://cdn-images-1.medium.com/max/6104/1*0mgelgA-g3Lb6_YG70kwRQ.png)

* **List Comprehensions vs. Reduce function**

Similarly, we can determine the maximum of a list containing integers quickly with list comprehension instead of using lambda and reduce.

![](https://cdn-images-1.medium.com/max/5440/1*hmkZrAAkn18cvRKaY0Dn0Q.png)

We have used a generator expression above which is similar to list comprehension but with round brackets instead of the square one.

List comprehensions are a diverse topic and require an article of their own. Keeping this in mind, here is an article that I wrote that covers not only list comprehensions but even dictionary, set, and generator comprehensions in python.
[**Comprehending the ‘Comprehensions’ in Python**
*Understanding and implementing the list, dictionary, set, and generator comprehensions in python.*towardsdatascience.com](https://towardsdatascience.com/comprehending-the-concept-of-comprehensions-in-python-c9dafce5111)

## Conclusion

The map, filter, and reduce functions significantly simplify the process of working with lists and other iterable collections of data. Some people have reservations about using them, especially since list comprehensions appear to be more friendly, yet their usefulness cannot be ignored.

## References

* [Don’t Be Scared Of Functional Programming](https://www.smashingmagazine.com/2014/07/dont-be-scared-of-functional-programming/)

* [Functional Programming HOWTO](https://docs.python.org/3.7/howto/functional.html)

* [Lambda, filter, reduce and map](https://www.python-course.eu/lambda.php)
