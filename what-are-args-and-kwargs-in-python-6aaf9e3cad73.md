Unknown markup type 10 { type: [33m10[39m, start: [33m139[39m, end: [33m144[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m148[39m, end: [33m156[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m76[39m, end: [33m84[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m42[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m101[39m, end: [33m102[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m147[39m, end: [33m152[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m181[39m, end: [33m189[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m129[39m, end: [33m134[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m103[39m, end: [33m110[39m }

# What Are *args and **kwargs in Python?

Learn to use variable-length argument lists

![Photo by [Headway](https://unsplash.com/@headwayio?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/argument?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)](https://cdn-images-1.medium.com/max/10944/1*OFRGP4jrkR0i9ODcI1Eaow.jpeg)*Photo by [Headway](https://unsplash.com/@headwayio?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/argument?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

Functions are life, right? If you’re new to Python — whether brand new to coding or coming from another language — you learn the number of parameters in a function’s definition match the number of arguments meant to be passed.

This is foundational — it helps make sense of the world. However, it also sets you up for early mental stumbling blocks as soon as you see *args or **kwargs in a function definition.

Don’t let the syntax scare you. These aren’t super special parameters. They’re not even that fancy, and we’re going to learn how to use them.

## Positional vs. Keyword Arguments

There are two concepts we need to separate in order to learn what *args and **kwargs are.

The first is the difference between positional and keyword arguments. In the most basic of functions, we play a matching game — argument 1 goes with parameter 1, argument 2 goes with parameter 2, and so on.

    def printThese(a,b,c):
       print(a, "is stored in a")
       print(b, "is stored in b")
       print(c, "is stored in c")

    printThese(1,2,3)
    """
    1 is stored in a
    2 is stored in b
    3 is stored in c
    """

All three arguments are required. Missing one will cause an error.

    def printThese(a,b,c):
       print(a, "is stored in a")
       print(b, "is stored in b")
       print(c, "is stored in c")

    printThese(1,2)
    """
    TypeError: printThese() missing 1 required positional argument: 'c'
    """

If we give a parameter a default value in the function definition, then it becomes optional.

    def printThese(a,b,c=None):
       print(a, "is stored in a")
       print(b, "is stored in b")
       print(c, "is stored in c")

    printThese(1,2)
    """
    1 is stored in a
    2 is stored in b
    None is stored in c
    """

Additionally, these optional parameters also become keyword-eligible, meaning you can specify the parameter name in the function call to map it accordingly.

Let’s make all three variables default to None and watch how we can still map them irregardless of order.

    def printThese(a=None,b=None,c=None):
       print(a, "is stored in a")
       print(b, "is stored in b")
       print(c, "is stored in c")

    printThese(c=3, a=1)
    """
    def printThese(a=None,b=None,c=None):
       print(a, "is stored in a")
       print(b, "is stored in b")
       print(c, "is stored in c")

    printThese(c=3, a=1)
    """
    1 is stored in a
    None is stored in b
    3 is stored in c
    """

## The Splat Operator

Let me start by saying I love the name of this operator — it’s so … visual. The * is most commonly associated with multiplication, but in Python it doubles as the splat operator.

I think of this operator as a piñata. I’ve described the spread operator — the JavaScript equivalent of splat — as unpacking a series of dominoes to form a single larger sequence, but splat warrants a more forceful analogy.

A simple example will make this more clear.

    a = [1,2,3]
    b = [*a,4,5,6]

    print(b) # [1,2,3,4,5,6]

In the code example, we’re taking the contents of a and splattering (unpacking) it into our new list b.

## How To Use *args and **kwargs

So we know the splat operator unpacks multiple values, and there are two types of function parameters. Well, if you haven’t figured it out by now, *args is short for arguments, and **kwargs is short for keyword arguments.

Each is used to unpack their respective argument type, allowing for function calls with variable-length argument lists. For example, let’s create a function to print a student’s test scores.

    def printScores(student, *scores):
       print(f"Student Name: {student}")
       for score in scores:
          print(score)

    printScores("Jonathan",100, 95, 88, 92, 99)
    """
    Student Name: Jonathan
    100
    95
    88
    92
    99
    """

Whoa, wait. I didn’t name it *args? That’s right, “args” is a standard convention but still just a name. The secret revealed, in *args, the single asterisk is the real player here, creating a list whose contents are positional arguments — after those defined — from the function call.

With that cleared up, **kwargs should be a easy to digest. The name doesn’t matter, but the double asterisks create a dictionary whose contents are keyword arguments — after those defined — from a function call.

To illustrate this, let’s create a function to print the names of a person’s pets.

    def printPetNames(owner, **pets):
       print(f"Owner Name: {owner}")
       for pet,name in pets.items():
          print(f"{pet}: {name}")

    printPetNames("Jonathan", dog="Brock", fish=["Larry", "Curly", "Moe"], turtle="Shelldon")

    """
    Owner Name: Jonathan
    dog: Brock
    fish: ['Larry', 'Curly', 'Moe']
    turtle: Shelldon
    """

## Parting Words

A few words of wisdom to help you avoid common pitfalls and expand your knowledge.

* Use *args, **kwargs as a standard convention to catch positional and keyword arguments

* You cannot place **kwargs before *args, or you’ll receive an error

* Beware of conflicts between keyword parameters and **kwargs where the value is meant to be passed as a **kwarg but is unknowingly the name of a keyword parameter

* You can use the splat operator in function calls as well
