
# 100 Days of DevOps — Day 81-Debugging Python Code

Welcome to Day 81 of 100 Days of DevOps, Focus for today is Debugging Python Code

*Logging is a great way to debug our code, normal way to do that is to add a print() in your code but Python provides logging module which help us to debug what happens in our code and in which order. In order to enable logging to add the below entry in your code*

    ***import **logging*

    *# This is setup code to enable logging in Python*

    *logging.basicConfig**(**level**=**logging.DEBUG, format**='%(asctime)s -%(levelname)s - %(message)s')***

*To illustrate logging let’s take a simple example where I am going to add the first five number*

    *>>> 1+2+3+4+5*

    *15*

    *Now let’s do it with the help of Python Code*

    *sum = 1
    for i in range(1,6):
        sum = sum + i*

    *print(sum)*

*Output is 16 which is wrong it should be 15 so there is some sort of bug in this code*

    *16*

*After enable logging*

    ***import **logging
    logging.basicConfig**(**level**=**logging.DEBUG, format**='%(asctime)s -%(levelname)s - %(message)s')
    **logging.debug**("Start of Program")
    **sum **= **1
    **for **i **in **range**(**1,6**):
        **logging.debug**("Value of i is (%s)" %(**i**))
        **sum **= **sum **+ **i
        logging.debug**("Value of i is (%s), Value of sum is (%s)" % (**i,sum**))***

    *print**(**sum**)
    **logging.debug**("End of Program")***

*The output looks like this, as you can see here even at first iteration for i = 1 sum =2 which is wrong so sum declaration is wrong and it should be set to zero(sum=0)*

    *2017–04–17 16:51:20,810 -DEBUG — Start of Program
    2017–04–17 16:51:20,810 -DEBUG — Value of i is (1)
    2017–04–17 16:51:20,810 -DEBUG — Value of i is (1), Value of sum is (2)
    2017–04–17 16:51:20,811 -DEBUG — Value of i is (2)
    2017–04–17 16:51:20,811 -DEBUG — Value of i is (2), Value of sum is (4)
    2017–04–17 16:51:20,811 -DEBUG — Value of i is (3)
    2017–04–17 16:51:20,811 -DEBUG — Value of i is (3), Value of sum is (7)
    2017–04–17 16:51:20,811 -DEBUG — Value of i is (4)
    2017–04–17 16:51:20,811 -DEBUG — Value of i is (4), Value of sum is (11)
    2017–04–17 16:51:20,811 -DEBUG — Value of i is (5)
    2017–04–17 16:51:20,811 -DEBUG — Value of i is (5), Value of sum is (16)
    2017–04–17 16:51:20,811 -DEBUG — End of Program*

*So the program should look like*

    *sum **= **0
    **for **i **in **range**(**1,6**):
        **sum **= **sum **+ i***

    *print**(**sum**)***

*Output*

    *15*

*In order to **disable** logging, we need to add this at the start of the program*

    *logging.**disable(**logging.CRITICAL**)***

*Doing this is much easier when comparing with adding a print statement and the reason for that it’s difficult to remove print when we are done with debugging(by mistake we might remove the print which outputting the program output)*

*There are five log levels going from lowest to highest*

![](https://cdn-images-1.medium.com/max/2000/1*HLuX2_L1nqlwyMqo-UBzUQ.png)

    *#Different logging level*

    *>>> logging.DEBUG*

    *10*

    *>>> logging.INFO*

    *20*

    *>>> logging.WARNING*

    *30*

    *>>> logging.ERROR*

    *40*

    *>>> logging.CRITICAL*

    *50*

*Instead of logging all these messages to console we can redirect it to a file using **filename** directive*

    *logging.basicConfig**(filename='debugfile',**level**=**logging.DEBUG, format**='%(asctime)s -%(levelname)s - %(message)s')***

    ***cat debugfile***

    *2017–04–17 21:57:20,101 -DEBUG — Start of Program*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (1)*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (1), Value of sum is (1)*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (2)*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (2), Value of sum is (3)*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (3)*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (3), Value of sum is (6)*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (4)*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (4), Value of sum is (10)*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (5)*

    *2017–04–17 21:57:20,101 -DEBUG — Value of i is (5), Value of sum is (15)*

    *2017–04–17 21:57:20,101 -DEBUG — End of Program*

***Python Debugger(pdb)***

*Advantage of pdb is it allows a program to pause execution and give user a chance to run the code and inspect the variable and step through code*

*Let look at this simple program, it’s similar to what illustrated with logging example*

    ***import **pdb
    val **= [**1,2,3,4,5**]
    for **i **in **val**:
        **sum **= **0
        sum **= **sum **+ **i*

    *print**(**sum**)***

*The output of this program is wrong it's supposed to be 15*

    *5*

*So let’s try to debug it via pdb*

    ***import **pdb*

    *val **= [**1,2,3,4,5**]
    ***

    ***for **i **in **val**:
        **sum **= **0
        sum **= **sum **+ **i
        **pdb.set_trace()***

    *print**(**sum**)***

*Under pdb*

*To get the list of command supported by pdb(type h → help)*

![](https://cdn-images-1.medium.com/max/2240/1*tBYPtQK3rICu0lDDekpD-g.png)

*Coming back to our problem*

    *-> for i in val:
    (Pdb) i # Checking the value of the i
    1
    (Pdb) sum # Checking the value of the sum
    1
    (Pdb) c #c is the abbreviation of continue
    -> for i in val:
    (Pdb) i
    2
    (Pdb) sum
    2
    (Pdb)*

*So here we found out that the value of sum is not incrementing*

*To fix this problem we declared sum outside the loop*

    ***import **pdb*

    *val **= [**1,2,3,4,5**]***

    *sum **= **0
    **for **i **in **val**:
        **sum **= **sum **+ **i*

    *print**(**sum**)***

*Now the output is equal to*

    *15*

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
[**100 Days of DevOps — Day 80-Python Unit Testing(Pytest)**
*Welcome to Day 80 of 100 Days of DevOps, Focus for today is Python Unit Testing(Pytest)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-80-python-unit-testing-pytest-67168a91ea06)
