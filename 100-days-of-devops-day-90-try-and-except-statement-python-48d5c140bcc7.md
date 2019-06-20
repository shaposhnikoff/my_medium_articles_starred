
# 100 Days of DevOps — Day 90- Try and Except Statement Python

Welcome to Day 90 of 100 Days of DevOps, Focus for today is Try and Except Statement Python

*Getting an error or exception in python program means the entire program will crash but in real world situation we don’t want this to happen instead we want the program to detect these type of error/exception and continue to run.*

    ***def divideby(**num**):
       return **10**/**num*

    *print**(**divideby**(**10**))
    **print**(**divideby**(**0**))***

*Output*

*The computer doesn’t know how to handle divide by zero and Python throws the error/program crash*

    *1.0
     print(divideby(0))
     File “/Users/plakhera/medium_python/firstprogram.py”, line 2, in divideby
     return 10/num
    ZeroDivisionError: division by zero*

*Now let’s use try and except*

    ***def divideby(**num**):
       try:
          return **10 **/ **num
       **except:
          **print**("Divide by zero error")***

    *print**(**divideby**(**10**))
    **print**(**divideby**(**0**))***

*So the moment it hits Divide by zero error, code moves to except block and that prevent the program from crashing*

*Output*

    *1.0
    Divide by zero error
    None*

Now let’s take a look at other pieces of code

    def convert(num):
     x = int(num)
     return x

    from loggingcode import convert

    convert(1)
     1

    convert(‘1’)
     1

    # Now if we try to convert something which int can't convert we received a traceback as int dont know how to convert string

    convert(‘hello’)
    Traceback (most recent call last):
     File “/usr/local/lib/python3.6/site-packages/IPython/core/interactiveshell.py”, line 2881, in run_code
     exec(code_obj, self.user_global_ns, self.user_ns)
     File “<ipython-input-6–891f5d73f0fd>”, line 1, in <module>
     convert(‘hello’)
     File “/Users/plakhera/Downloads/salesforce-spring-2017/loggingcode.py”, line 3, in convert
     x = int(num)
    **ValueError:** invalid literal for int() with base 10: ‘hello’

NOTE: Here ValueError is the type of exception object

Now let’s try to re-write this code and use try/except block so that we can handle

    >>> def convert(num):

    … try:

    … x = int(num)

    … return x

    … except ValueError:

    … print(“Please enter a valid number”)

    …

    >>> convert(3)

    3

    >>> convert(“3”)

    3

    >>> convert(“three”)

    Please enter a valid number

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
[**100 Days of DevOps — Day 89-Python Files I/O**
*Welcome to Day 89 of 100 Days of DevOps, Focus for today is Python Files I/O*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-89-python-files-i-o-c8b771b43fb7)
