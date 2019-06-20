
# 100 Days of DevOps — Day 87-While/For Loop Python

Welcome to Day 87 of 100 Days of DevOps, Focus for today is While/For Loop Python

*With a while loop, we can make a block of code executed as long as while statement is True*

    *test **= **0
    **while ( **test **< **5**):
        **print**("hello")
        **test **= **test **+ **1*

    *hello
    hello
    hello
    hello
    hello*

*Here as soon as the execution reaches the end of a while statement’s block, it jumps back to the start to re-check the condition. We are initializing test=0 and as long as the value of a test is less than 5, While checking the condition and prints the hello.*

*When execution runs through the loop we call it iteration we say this while loop iterates 5 times.*

*But let’s take a look at the other two cases*

* *If forget to initialize the value, I will receive an error*

    ***while **n **< **5**:
        **print**(**n**)
        **n **= **n**+**1*

*Output*

    *Traceback (most recent call last):
     File “/Users/plakhera/Documents/python_testing/test.py”, line 1, in <module>
     while n < 5:
    **NameError: name ’n’ is not defined***

    *Process finished with exit code 1*

* *If I forget to increment the value, then it will become an infinite loop*

    *n **= **0
    **while **n **< **5**:
        print(**n**)***

***Infinite Loop***

    *>>> while True:*

    *… print(“hello”)*

    *…*

    *hello*

    *hello*

    *hello*

*Since this condition is always True this will cause the loop to be executed forever. To come out of infinite loop use Ctrl+c*

***Break:** Cause statement to jump out of the loop, without re-checking the condition*

    *test **= **0
    **while (**test **< **10**):
        **print**(**test**)
        **test **= **test **+ **1
        **if **test **== **5**:
            break***

*Output*

    *0
    1
    2
    3
    4*

***Continue***

*In the case of continue statement when the program execution reaches the continue statement, program execution immediately jump back to the start off while loop and re-check the condition, and that is the reason test==5 never executed*

    *test **= **0
    **while (**test **< 6):
        **test **= **test **+ **1
        **if **test **== **5**:
            continue
        **print**("Current Value of test: " + **str**(**test**))***

*Output*

    *Current Value of test: 1
    Current Value of test: 2
    Current Value of test: 3
    Current Value of test: 4
    Current Value of test: 6*

***For Loop***

*For loop is used when we want to iterates a specific number of times.*

    ***for **i **in **range**(**5**):
        **print**(**i**)***

*Output*

    *0
    1
    2
    3
    4*

*Sum of first 100 number*

    *total **= **0
    #Because it goes one less
    **for **i **in **range**(**101**):
        **total **= **total **+ **i
    print**(**total**)***

*Output*

    *5050*

*OR one more favorite interview question, print all the even number between 0 to n number*

    ***for **i **in **range**(**0,11,2**):
        print(**i**)***

*Output*

    *0
    2
    4
    6
    8
    10*

*Let summarize*

![](https://cdn-images-1.medium.com/max/2536/1*jiMMKcQRXqqnv03LVWC2bQ.png)

*Let’s try to solve a few more problem*

*In this problem, we need to count the number of vowels in a string*

    *s **= 'welcome to the world of python'
    **count **= **0
    **for **i **in **s**:
        if (**i **== "a" or **i **== "e" or **i **== "i" or **i **== "o" or **i **== "u"):
            **count **+= **1*

    ***print(**count**)***

*Is there is any better way to write the same code?*

    *s **= 'welcome to the world of python'
    **count **= **0
    vowels **= ["a"**,**"e"**,**"i"**,**"o"**,**"u"]
    for **i **in **s**:
        if **i **in **vowels**:
            **count **+= **1*

    ***print(**count**)***

*But this code will fail if we have Vowels like AEIOU*

    *s **= 'welcOmE to the world of python'***

*It will return only 6 vowels*

*There are multiple ways to deal with this problem but the easiest way right now is the use of lower()*

    *s **= 'welcOmE to the world of python'
    s = s.lower()
    **count **= **0
    vowels **= ["a"**,**"e"**,**"i"**,**"o"**,**"u"]
    for **i **in **s**:
        if **i **in **vowels**:
            **count **+= **1*

    ***print(**count**)***

*Later on, when we jump to the regular expression we will see we can solve the same problem using regular expression*

    *>>> import re*

    *>>> s = ‘welcOmE to the world of python’*

    *>>> vowels = re.findall(‘[aeiou]’,s,re.IGNORECASE)*

    *>>> print(len(vowels))*

    *8*
[**Python Regular Expression**
*Regular expressions can be think like a mini-language for specifying text pattern*medium.com](https://medium.com/@devopslearning/python-regular-expression-8ee28d35f3a7)

*There might be a situation where we might need to know how many times the gives element is repeated in a string. There are a number of ways to do that but the easiest way is to use collections module*

    *>>> from collections import Counter*

    *>>> s*

    *‘welcOmE to the world of python’*

    *>>> a = Counter(s)*

    *>>> print(a)*

    *Counter({‘ ‘: 5, ‘o’: 4, ‘t’: 3, ‘w’: 2, ‘e’: 2, ‘l’: 2, ‘h’: 2, ‘c’: 1, ‘O’: 1, ‘m’: 1, ‘E’: 1, ‘r’: 1, ‘d’: 1, ‘f’: 1, ‘p’: 1, ‘y’: 1, ’n’: 1})*

*Let’s take one more case in which I need to find out how many times **prash** repeated in this string*

    *s **= 'hello my name is prash and the last name is prash too'
    **count **= **0
    **for **i **in **range**(**len**(**s**)):
        if **s**[**i**:**i**+**5**] == "prash":
            **count **+= **1*

    ***print(**count**)
    2***

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
[**100 Days of DevOps — Day 86-Python Flow Control(if-else statement)**
*Welcome to Day 86 of 100 Days of DevOps, Focus for today is Python Flow Control(if-else statement)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-86-python-flow-control-if-else-statement-a20cf04b4fbe)
