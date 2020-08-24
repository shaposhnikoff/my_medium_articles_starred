Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m73[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m273[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m95[39m }

# Decorating functions in Python

What is a high-order function?

![](https://cdn-images-1.medium.com/max/3840/1*9NiBn43iYhow3wDTTu7JQg.jpeg)

In this post, I’ll be explaining what Python decorators are and how you can implement one. Let’s get started!

## What is a decorator?

A decorator is no more than a comfortable way to call a high-order function. You’ve probably seen it many many times already, it’s about those “@dostuff” strings that you find from time to time above a function’s signature:

    @dostuff
    def foo():
      pass

Ok, but now, what is a high-order function? A high-order function is just any function that takes one (or more) function as an argument and/or returns a function. For instance, in the above example, “@dostuff” would be the decorator, “dostuff” would be the name of the high-order function and “foo” would be the decorated function and the parameter of the high-order function.

All clear? Great, let’s start implementing our first decorator!

## Building our first decorator

In order to start implementing our decorator, I’ll introduce you to [functools](https://docs.python.org/3/library/functools.html): Python’s module for high-order functions. Specifically, we’re going to be using the [wraps](https://docs.python.org/3/library/functools.html#functools.wraps) function.

Let’s create a high-order function that will print the execution time of the decorated function: we’ll call it “timeme”. In this way, every time we want to compute the execution time of a function, we’ll just need to add the decorator “@timeme” above the signature of the target method. Let’s start defining the signature of “timeme”:

    def timeme(func):
      pass

As mentioned before, a high-order function takes another function (the decorated one) as its argument, so we’ve included “func” in its signature. Now, we need to add a wrapper function that will contain the timing logic. For this purpose, we’ll be creating a “wrapper” function that will be wrapped by functools’ [wraps](https://docs.python.org/3/library/functools.html#functools.wraps) function:

    from functools import wraps

    def timeme(func):
      @wraps(func)
      def wrapper(*args, **kwargs):
        pass

      return wrapper

Note that “timeme” is returning the function “wrapper”, which in turn, besides printing the execution time, will return the result of the decorated function.

Now, let’s complete the wrapper by implementing the timing logic:

    from functools import wraps
    import time

    def timeme(func):
      @wraps(func)
      def wrapper(*args, **kwargs):
        print("Let's call our decorated function")
        start = time.time()
        result = func(*args, **kwargs)
        print('Execution time: {} seconds'.format(time.time() - start))
        return result
      return wrapper

Note that the decorated function “func” is being executed with its positional and keyword arguments. I’ve added some print messages for you to observe the execution order. Alright! Let’s try it out, I’ll create a simple function with a print message, decorated by “timeme”:

    @timeme
    def decorated_func():
      print("Decorated func!")

If you run it, you will see something like the following:

    Let's call our decorated function
    Decorated func!
    Execution time: 4.792213439941406e-05 seconds

As you can see, the first print message was placed in the high-order function. Then, we are calling the decorated function and printing its own message. Finally, we compute the execution time and print it.
