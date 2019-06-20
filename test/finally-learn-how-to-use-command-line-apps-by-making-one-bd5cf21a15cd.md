Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m95[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m119[39m, end: [33m140[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m60[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m88[39m, end: [33m109[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m146[39m, end: [33m167[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m20[39m }

# Finally learn how to use command line apps… by making one!

At the risk of alienating a lot of readers… I grew up with GUI’s, so I never needed to learn the ways of the terminal! This is socially acceptable in todays society of friendly user interfaces, except maybe if you’re in software engineering… whoops!

![Manager — “Oh this is easy, just use this command line app”, Me — ”Yes…the command line… I’ll use that…”](https://cdn-images-1.medium.com/max/2000/1*cyhNkd0jd7Zd3zBDLaKpSg.png)*Manager — “Oh this is easy, just use this command line app”, Me — ”Yes…the command line… I’ll use that…”*

## Getting back my software engineering street cred’

I’ve gotten pretty far just through copy and pasting full command-line operations from Stack-Overflow but have never become comfortable enough to use a command line app “properly”. What finally caused it to click for me was when I stumbled across a very handy python library called [**argparse](https://docs.python.org/3/library/argparse.html) **that allows you to build a nice robust command line interface for your python scripts.

[This tutorial](https://docs.python.org/3.5/howto/argparse.html) is great in-depth explanation about how to use **argparse, **but I’ll go over the key eye openers**:**
> # [argparse](https://docs.python.org/3/library/argparse.html) is part of the standard python library so pop open your code editor and follow along (you don’t need to install anything)!

## **HELP**

The most useful part of every command line app!

    # inside a file called my_app.py

    **import** **argparse**
    parser = argparse.ArgumentParser(description="*Nice little CL app!*")
    parser.parse_args()

The code above will do nothing, except by default you have the **help **flag!

In the command line you can run:

    python my_app.py --help

or

    python my_app.py -h

and you’ll get an output like this:

    usage: my_app.py [-h]
    

    **Nice little CL app!**
    

    optional arguments:

    -h, --help  **show this help message and exit**

Seems pretty cool right? but wait, when you use **argparse **to** **add more functions (see below) this **help** output will automatically fill up with all of the instructions of how you can use your app!

## REQUIRED ARGUMENTS

Lets say you want to have your app take in some variable, we just use the parser.add_argument() function and give our argument some label (in this case “name”):

    import argparse
    parser = argparse.ArgumentParser(**description**="*Nice little CL app!*")
    

    parser.add_argument("***name***", **help**="*Just your name, nothing special*")
    

    args = parser.parse_args()
    print("*Your name is what?* " + args**.name**)

Notice how we added the **help **text! Now when we run python my_app.y -h we get all of the app details:

    usage: my_app.py [-h] name
    

    **Nice little CL app!**
    

    positional arguments:

    name        **Just your name, nothing special**
    

    optional arguments:

    -h, --help  **show this help message and exit**

Pretty cool, but let’s run our app with python my_app.py

    usage: my_app.py [-h] name

    my_app.py: error: too few arguments

That’s right! Automatic input checking!

Now lets run python my_app.py "Slim Shady"

    Your name is what? Slim Shady

Pretty slick!

## OPTIONAL ARGUMENTS

Maybe you want to give someone the option of telling you little more about themselves? Adding a double dash when using parser.add_argument() function will make that argument optional!

    import argparse
    parser = argparse.ArgumentParser(**description**=”*Nice little CL app!*”)
    parser.add_argument(“*name*”, **help**=”*Just your name, nothing special*”)
    

    parser.add_argument("***--**profession*”, **help**=”Y*our nobel profession*”)
    

    args = parser.parse_args()
    print(“*Your name is what? *“ + args.name)

    if args.profession:
        print(“*What is your profession!? a *“ + args.profession)

If you want to pass in a variable for that argument you just need to specify the double-dashed argument name before the variable you want to pass:

    python my_app.py "Slim Shady" **--profession "gift wrapper"**

which gives you :

    Your name is what? Slim Shady

    What is your profession!? a gift wrapper

Or you don’t have to, it is optional after all!

    python my_app.py "Slim Shady"

still gives you:

    Your name is what? Slim Shady

and the magic once again when running python my_app.py -h :

    usage: my_app.py [-h] [--profession PROFESSION] name
    

    **Nice little CL app!
    **

    positional arguments:

    name                      **Just your name, nothing special**

    optional arguments:

    -h, --help                **show this help message and exit**

    --profession PROFESSION   **Your nobel profession**

## FLAGS

Maybe you just want to enable something cool to happen. Add action="*store_true*" to your parser.add_argument() function and you have yourself a **flag** argument :

    import argparse

    parser = argparse.ArgumentParser(**description**="*Nice little CL app!*")
    parser.add_argument("*name*", **help**="*Just your name, nothing special*")
    parser.add_argument("*--profession*", **help**="*Your nobel profession*")
    

    parser.add_argument("*--cool*", **action**="*store_true*", **help**="*Add a little cool*")
    

    args = parser.parse_args()
    print("*Your name is what? *" + args.name)

    cool_addition = "* and dragon tamer*" if args.cool else ""

    if args.profession:
        print("*What is your profession!? a *" + args.profession + cool_addition)

It’s pretty neat, you just plop the **flag** name into your command like so:

    python my_app.py "Slim Shady" --profession "gift wrapper" **--cool**

and presto!

    Your name is what? Slim Shady

    What is your profession!? a gift wrapper and dragon tamer

and remember, you don’t have to use it, it’s just a flag:

    python my_app.py "Slim Shady" --profession "gift wrapper"

will still give:

    Your name is what? Slim Shady

    What is your profession!? a gift wrapper

look though at the **help **command python my_app.py -h :

    usage: my_app.py [-h] [--profession PROFESSION] [--cool] name
    

    **Nice little CL app!
    **

    positional arguments:

    name                      **Just your name, nothing special**

    optional arguments:

    -h, --help                **show this help message and exit**

    --profession PROFESSION   **Your nobel profession**

    --cool                    **Add a little cool**

I’m just going to assume you’re as satisfied with that as I am from now on.

## SHORT FORM

Unveiling the mysterious one character arguments that confused me for so long. Just by adding a single letter prefixed with a single dash to your parser.add_argument() functions you have super short versions of the same arguments:

    import argparse

    parser = argparse.ArgumentParser(**description**="*Nice little CL app!*")
    parser.add_argument("*name*", **help**="*Just your name, nothing special*")

    parser.add_argument("-p", "*--profession*", **help**="*Your nobel profession*")

    parser.add_argument("-c", "*--cool*", **action**="*store_true*", **help**="*Add a little cool*")

    args = parser.parse_args()
    print("*Your name is what? *" + args.name)
    cool_addition = "* and dragon tamer*" if args.cool else ""
    if args.profession:
        print("*What is your profession!? a *" + args.profession + cool_addition)

So that instead of typing in :

    python my_app.py "Slim Shady" --profession "gift wrapper" --cool

you can just type:

    python my_app.py "Slim Shady" -p "gift wrapper" -c

and you’ll get the same output:

    Your name is what? Slim Shady

    What is your profession!? a gift wrapper and dragon tamer

And this is reflected in the help text (python my_app.py -h ):

    usage: my_app.py [-h] [--profession PROFESSION] [--cool] name

    **Nice little CL app!**

    positional arguments:

    name                           **Just your name, nothing special**

    optional arguments:

    -h, --help                      **show this help message and exit**

    -p PROFESSION, --profession PROFESSION   **Your nobel profession**

    -c, --cool                      **Add a little cool**

![a perfect little command line app!](https://cdn-images-1.medium.com/max/2000/1*OYyJ-v1Ao09QEhv71MW_Gg.jpeg)*a perfect little command line app!*

You now have a perfect little command line app and are hopefully more comfortable with finding your way around command line apps!

Just remember --help !
[**ZackAkil/super-simple-command-line-app**
*super-simple-command-line-app - Code for "Finally learn how to use command line apps... by making one!"*github.com](https://github.com/ZackAkil/super-simple-command-line-app)
