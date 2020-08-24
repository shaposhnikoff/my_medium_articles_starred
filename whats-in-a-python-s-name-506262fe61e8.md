Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m63[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m88[39m, end: [33m92[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m97[39m, end: [33m107[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m264[39m, end: [33m272[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m97[39m, end: [33m110[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m72[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m132[39m, end: [33m135[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m63[39m }

# What’s in a (Python’s) __name__?

An introduction to the _ _name_ _ variable and its usage in Python

![Photo by [Marius Masalar](https://unsplash.com/@marius?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/7000/0*zv8ZOTI8T_hmrYXG)*Photo by [Marius Masalar](https://unsplash.com/@marius?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

You’ve most likely seen the __name__ variable when you’ve gone through Python code. Below you see an example code snippet of how it may look:

    if __name__ == '__main__':
        main()

In this article, I want to show you how you can make use of this variable to create modules in Python.

### Why is the _ _name_ _ variable used?

The __name__ variable (two underscores before and after) is a special Python variable. It gets its value depending on how we execute the containing script.

Sometimes you write a script with functions that might be useful in other scripts as well. In Python, you can import that script as a module in another script.

Thanks to this special variable, you can decide whether you want to run the script. Or that you want to import the functions defined in the script.

### What values can the __name__ variable contain?

When you run your script, the __name__ variable equals __main__. When you import the containing script, it will contain the name of the script.

Let us take a look at these two use cases and describe the process with two illustrations.

### Scenario 1 - Run the script

Suppose we wrote the script*** ***nameScript.py as follows:

    def myFunction():
        print 'The value of __name__ is ' + __name__

    def main():
        myFunction()

    if __name__ == '__main__':
        main()

If you run nameScript.py, the process below is followed.

![](https://cdn-images-1.medium.com/max/2764/1*208zyrMaQhkQK_mw8cE_Lg.png)

Before all other code is run, the __name__ variable is set to __main__. After that, the main*** ***and myFunction def statements are run. Because the condition evaluates to true, the main function is called. This, in turn, calls myFunction. This prints out the value of __main__.

### Scenario 2 - Import the script in another script

If we want to re-use myFunction in another script, for example importingScript.py, we can import nameScript.py as a module.

The code in importingScript.py could be as follows:

    import nameScript as ns

    ns.myFunction()

We then have two scopes: one of importingScript and the second scope of nameScript. In the illustration, you’ll see how it differs from the first use case.

![](https://cdn-images-1.medium.com/max/2780/1*Xs311ryc0Wfv-cYl6TzNLg.png)

In importingScript.py the __name__ variable is set to __main__. By importing nameScript, Python starts looking for a file by adding .py to the module name. It then runs the code contained in the imported file.

But this time it*** ***is set to nameScript. Again the def statements for main and myFunction are run. But, now the condition evaluates to false and main is not called.

In importingScript.py we call myFunction which outputs nameScript. NameScript is known to myFunction when that function was defined.

If you would print __name__ in the importingScript, this would output __main__. The reason for this is that Python uses the value known in the scope of importingScript.

### Conclusion

In this short article, I explained how you can use the __name__ variable to write modules. You can also run these modules on their own. This can be done by making use of how the values of these variables change depending on where they occur.
