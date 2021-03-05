
# 100 Days of DevOps — Day 93-Python Functions

Welcome to Day 93 of 100 Days of DevOps, Focus for today is Python Functions

We can think function as a reusable piece/chunk of code

***Built-in functions***

*The modules that come with Python are called the **standard library**.*

*for eg: **print()/len()** is a built-in function*

*But we can install third-party modules using **Python Package Index(pip)***

    *# pip3 install pandas*

    *Collecting pandas*

    *100% |████████████████████████████████| 11.8MB 142kB/s*

    *Requirement already satisfied: python-dateutil>=2 in /usr/local/lib/python3.6/site-packages (from pandas)*

    *Requirement already satisfied: pytz>=2011k in /usr/local/lib/python3.6/site-packages (from pandas)*

    *Requirement already satisfied: numpy>=1.7.0 in /usr/local/lib/python3.6/site-packages (from pandas)*

    *Requirement already satisfied: six>=1.5 in /usr/local/lib/python3.6/site-packages (from python-dateutil>=2->pandas)*

    *Installing collected packages: pandas*

    *Successfully installed pandas-0.19.2*

***Functions***

*We can write our own functions, functions are like mini-program inside the program and their main aim is to get rid of duplicate code.*

    ***def hello():
        **print**("Hello ")
    ***

    *hello**()***

* *the def statement defines a function*

* hello is the name of the function

*Output*

    *Hello*

*Passing Argument to function*

    ***def hello(**name**):
        **print**("Hello " + **name**)
    ***

    *hello**("Prashant")***

*Output*

    *Hello Prashant*

* *Arguments: The value passed in the function call(eg:Prashant)*

* *Parameter: The variable inside the function(name)*

    ***def sum(**num1,num2**):
        return **num1**+ **num2*

    *output **= **sum**(**5,6**)
    **print**(**output**)***

*Every function has a return value. If our function doesn’t have a return statement, the default return value is None.*

*Output*

    *11*

***Global and Local Scope***

*In the case of Python variables inside the function can have the same name as a variable outside the function but they are considered as two separate variables.*

*The scope can be thought of as an area of the source code and as a container of variables*

* ***Global Scope:** is code outside of all functions. Variables assigned here are global variables. Global scope is created when a program starts and is destroyed when it terminated. The code in global scope can’t access a local variable.*

* ***Local Scope:** Function code is in its own local scope. Variables assigned here are local variables. Local scope is created when the function call and all the variables assigned during this function call exist and when the function returns all these variables are forgotten. The Code in local scope can access a global variable.*

* *The code in one function’s local scope cannot use variables in another local scope*

    *x = 1 #local variable*

    *def test():*

    *x = 1 #global variable*

*One example*

    ***def test():
        **x **= **1*

    *test**()
    **print**(**x**)***

*Output*

*As mentioned above x is local variable so as soon as the function returns all the variables are forgotten*

    */usr/local/Cellar/python3/3.6.0/Frameworks/Python.framework/Versions/3.6/bin/python3.6 /Users/plakhera/medium_python/firstprogram.py
    Traceback (most recent call last):
     File “/Users/plakhera/medium_python/firstprogram.py”, line 5, in <module>
     print(x)
    **NameError: name ‘x’ is not defined***

*If we want this code to work, then we need a keyword global*

    ***def test():
       global **x
       x **= **1*

    *test**()
    **print**(**x**)***

***The Cause of scope is to limit the scope so the impact of a bug in the code will be limited.***

![](https://cdn-images-1.medium.com/max/3072/1*eM9HSEK2rce0WrBngEUH1Q.png)

Let’s take one more example to reinforce the concept, here we are trying to access x before defining it

    **def myfun(**y**):
        **x **+= **1

    x **= **4
    myfun**(**x**)
    print(**x**)**

Output

    /usr/local/Cellar/python3/3.6.0/Frameworks/Python.framework/Versions/3.6/bin/python3.6 /Users/plakhera/Documents/python_testing/test.py
    Traceback (most recent call last):
      File "/Users/plakhera/Documents/python_testing/test.py", line 5, in <module>
        myfun(x)
      File "/Users/plakhera/Documents/python_testing/test.py", line 2, in myfun
        x += 1
    UnboundLocalError: local variable 'x' referenced before assignment

    Process finished with exit code 1

But if this will work if?

    **def myfun(**x**):
        **x **+= **1

    x **= **4
    myfun**(**x**)
    print(**x**)**

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
[**100 Days of DevOps — Day 92-Choosing Right EC2 Instance Type**
*Welcome to Day 92 of 100 Days of DevOps, Focus for today is Choosing Right EC2 Instance Type*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-92-choosing-right-ec2-instance-type-2f5d52bd6c85)
