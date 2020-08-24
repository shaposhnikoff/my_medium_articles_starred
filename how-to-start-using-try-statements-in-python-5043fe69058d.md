Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m99[39m, end: [33m105[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m144[39m, end: [33m150[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m180[39m, end: [33m181[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m42[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m53[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m122[39m, end: [33m125[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m134[39m, end: [33m140[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m193[39m, end: [33m197[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m202[39m, end: [33m209[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m84[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m49[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m98[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m85[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m150[39m, end: [33m155[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m100[39m, end: [33m103[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m103[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m183[39m, end: [33m190[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m241[39m, end: [33m244[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m76[39m, end: [33m99[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m33[39m }

# What Are Try/Except Statements in Python?

An introduction to safeguarding your code

![Photo by [Caspar Camille Rubin](https://unsplash.com/@casparrubin?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/free-code?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/10368/1*-NfgBteJ0tzuzx3WSSqW3Q.jpeg)*Photo by [Caspar Camille Rubin](https://unsplash.com/@casparrubin?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/free-code?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

When we start out programming our scripts are simple and straightforward and, most importantly, we tend to assume a perfect operating environment. Every variable will always exist and the data will be properly formed. If only the real world were so reliable!

The truth is that if something can fail, it will. Being prepared for these failures is a major milestone for any programmer.

In Python — along with most other languages — we use try statements to safeguard our code.

Let’s learn about what try statements do and how to begin using them!

## What are Exceptions?

Before we dive into the syntax and implementation of try statements, it’s important to establish what exceptions are.

An exception is the result of code within a try statement that fails. Think of an exception as your code being insulted — “I take exception to that request!”

There are a variety of exception types, we’ll be introduced to a few of them through this tutorial.

## How to Use Try Statements

A try statement, often referred to as a try block, in fact consists of at least two parts: try and except.

The code we want to safeguard goes inside the try portion of the block. Afterwards, we define what happens when something goes wrong within the except.

    try:
       print(x)
    except Exception:
       print("Something broke!")

    # Something broke!

In the example above, we use the catch-all value Exception to route any exception to this portion of the block. We also print a vague message. But what if we want the actual error message from Python?

You can define multiple except blocks within a try, specifying the exact type of exception that should be routed to each. Additionally, you can set the error message to a variable—e is common—so it can be used in your program.

    try:
       print(int(x))
    except NameError:
      print("The variable is undefined")   
    except Exception as e:
       print(e)

    # with x undefined => The variable is undefined
    # with x as 'text' => invalid literal for int() with base 10: 'text'

Now we’re in business! Our code is safeguarded, organized, and easily extended.

Let’s see what this looks like in a while loop tasked with collecting an integer input.

    while True:
      try:
        num = int(input("Enter an int: "))
        break
      except Exception as e:
        print(e)

    print("Your number is",num)

    # Enter an int: a
    # invalid literal for int() with base 10: 'a'
    # Enter an int: 1.1
    # invalid literal for int() with base 10: '1.1'
    # Enter an int: 1
    # Your number is 1

## The Different Try/Except Variations

So far we’ve used a try/except and even a try/except/except, but this is only two-thirds of the story.

There are two other optional segments to a try block: else and finally. Both of these optional blocks will come **after** the try and the except. Also, there’s nothing stopping you from using both else and finally in a single statement — but keep them in that order if you do.

Let’s go through each individually and see how they extend the behavior of a simple try/except.

### Try/Except/Else

When attaching an else statement to the end of a try/except, this code will be executed **after** the try has been completed, but only **if no exceptions occur**.

We can take the previous example of prompting a user for an integer input and use an else block to thank them for valid input and breaking out of the while loop.

    while True:
      try:
        num = int(input("Enter an int: "))
      except Exception as e:
        print(e)
      else:
        print("Thank you for the integer!")
        break

    # Enter an int: a
    # invalid literal for int() with base 10: 'a'
    # Enter an int: 3
    # Thank you for the integer

### Try/Except/Finally

When attaching a finally statement to the end of a try/except, this code will be executed **after** the try has been completed, **regardless of exceptions**.

Again, we’ll use our previous example and add a simple counter to illustrate this behavior.

    count = 0
    while True:
      try:
        num = int(input("Enter an int: "))
        break
      except Exception as e:
        print(e)
      finally:
        count += 1
        print("Attempt #:",count)

    # Enter an int: a
    # invalid literal for int() with base 10: 'a'
    # Attempt #: 1
    # Enter an int: 3
    # Attempt #: 2

This might look a bit odd because the break is still inside the try. It’s reasonable to think that the finally would be cut short upon proper input, however, that’s not the case. The finally section will still execute, regardless of how the try is exited.

*It’s a very small next step to take what we’ve demonstrated here and form a try/except/else/finally statement. I believe you’ll have no problems with that so I leave it to you to experiment and try it out.*

*If you’re accustomed to using try statements, what are some of your most common use cases? Share your experiences below!*
