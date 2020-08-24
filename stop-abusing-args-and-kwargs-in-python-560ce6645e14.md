Unknown markup type 10 { type: [33m10[39m, start: [33m65[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m75[39m, end: [33m83[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m93[39m, end: [33m99[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m113[39m, end: [33m116[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m129[39m, end: [33m132[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m5[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m100[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m156[39m, end: [33m162[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m303[39m, end: [33m308[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m313[39m, end: [33m321[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m113[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m119[39m, end: [33m121[39m }

# Stop Abusing *args and **kwargs in Python

Stop Abusing *args and **kwargs in Python

### They’ll come back to haunt you…

![Photo by [Matthew Henry](https://unsplash.com/@matthewhenry?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/10184/0*02aJnOluIRck75OS)*Photo by [Matthew Henry](https://unsplash.com/@matthewhenry?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

There are many tutorials online that aim to teach you how to use *args and **kwargs when defining a function in Python. Perhaps you’ve already spent hours trying to figure out how you could unleash their potential. Maybe, after all that study, you now feel confident about them.

*Don’t!*

Powerful tools are dangerous. You may have made your day easier but mark my words, it will come back and haunt you later.

*But why?*

## Some Basics

Parameters in a Python function can accept two types of arguments:

* Positional arguments that are passed positionally.

* Keyworded arguments that are supplied by keywords.

    def foo(start, end):
        print(start, end)

For example, foo('Hi', end='Bye!') feeds a positional argument, 'Hi', and a keyword argument 'Bye!' with keyword end to function foo. Parameters of a function are pre-defined; the number of parameters accepted in a function is fixed. However, that’s not always the case.

*args allows you to pass an arbitrary number of positional arguments to your function. The asterisk * is an unpacking parameter. They are packed as an iterable tuple inside the function.

<iframe src="https://medium.com/media/3e595e739e74a02daeb1ab0bb00f3216" frameborder=0></iframe>

On the other hand, **kwargs allows you to pass a varying number of keyworded arguments to your function. Since each keyworded argument has a keyword and a value, it’s grouped as an iterable dictionary inside the function.

<iframe src="https://medium.com/media/d0174f54deab24a1688dc81d577742b1" frameborder=0></iframe>

![Photo by [Susan Holt Simpson](https://unsplash.com/@shs521?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/9216/0*rZJ1TJ5MbzBec2_G)*Photo by [Susan Holt Simpson](https://unsplash.com/@shs521?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

## The Problem

You do not really need *args and **kwargs in most cases. How often do you not know how many arguments a pre-defined function should receive?

Code is much harder to debug if you abuse them because you’re letting an arbitrary number of arguments be passed to the function, and the function might have unpredictable behaviour.
> Explicit is better than implicit. — The Zen of Python

## When to Use It?

In short: Use them when you *really* need them. For instance, a function with a lot of optional fields and some of them are used only in some situations. Say, a function plots a graph and you can pass various optional arguments to modify its colour, style, size etc.

Every time you use *args and/or **kwargs, make sure you make very clear documentation to avoid confusion.

There is one scenario where their use might be inevitable. If you’re creating a wrapper for a function with unknown arguments, you would then have to accept an arbitrary number of positional and keyword arguments, then pass them to the function.

For example, decorators in Python work as wrappers that change the behaviour of the code, without changing the function code itself, thus augmenting extra functionalities.

![Photo by [Alexander Schimmeck](https://unsplash.com/@alschim?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/9162/0*afCQEjeUvg4bzGI4)*Photo by [Alexander Schimmeck](https://unsplash.com/@alschim?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

In the following example, we build trace that prints out the name of the executing function as a sanity check. The decorator is applied to a function using @trace on top of the function, as shown below. Since we want to apply this decorator to any functions with any number of arguments, we need to use *args and **kwargs.

<iframe src="https://medium.com/media/fcc30f4c9ec24a39e277965fe07bbe20" frameborder=0></iframe>

## Takeaways

***Avoid them if possible**.*

Note that args and kwargs are just named by convention. You can name them whatever you like. It is the asterisks * and ** that make them powerful.

Thanks for reading! You can [sign up for my newsletter](http://edenau.mailchimpsites.com/) to receive updates on my new articles. If you’re interested in improving your Python skills further, the following articles might be useful:
[**5 Python features I wish I had known earlier**
*Python tricks beyond lambda, map, and filter*towardsdatascience.com](https://towardsdatascience.com/5-python-features-i-wish-i-had-known-earlier-bc16e4a13bf4)
[**4 Common Mistakes Python Beginners should Avoid**
*I learned them the hard way, but you don’t need to*towardsdatascience.com](https://towardsdatascience.com/4-common-mistakes-python-beginners-should-avoid-89bcebd2c628)
[**4 Hidden Python Features that Beginners should Know**
*How to Supercharge Your Python Codes with Ease*towardsdatascience.com](https://towardsdatascience.com/4-hidden-python-features-that-beginners-should-know-ec9de65ff9f8)
