
# 100 Days of DevOps — Day 86-Python Flow Control(if-else statement)

Welcome to Day 86 of 100 Days of DevOps, Focus for today is Python Flow Control(if-else statement)

***if Statements***

*To understand if statement, let see one example*

    *name **= "Prashant"
    if **name **== "Prashant":
        **print **("Hello " + **name**)***

    *Hello Prashant*

*Here as you can see, if statement starts with a condition, if the condition evaluates to true the indented code block is executed else it just skip the statement.*

***Indentation** is just a Python way to tell which code block is inside if statement and which is not.*

*So if statement can be used to conditionally execute code depending on whether or not the if statement’s condition is True or False*

*Now if we change the code, in this case as the condition evaluates to False, it just skips the print block.*

    *name **= "Prashant1"
    if **name **== "Prashant":
        **print **("Hello " + **name**)***

*Let’s take a look at if-else statement*

    *name **= "Prashant1"
    if **name **== "Prashant":
        **print**("Hello " + **name**)
    else:
        **print**("Hello "+ **name**)***

    *Hello Prashant1*

*As you can see, if the statement is not True so else block is executed. Remember only one block is going to be executed.*

*Let’s add elif to this list which provides many blocks to be executed. In the case of elif order of elif statement matter, as the execution enters the block which is True, it just going to skip rest of the conditions.*

*An else statement comes at the end. Its block is executed if all of the previous conditions have been false*

    *name **= "Prashant1"
    if **name **== "Prashant":
        **print**("Hello " + **name**)
    elif **name **== "Lak":
        **print**("Hello "+ **name**)
    else:
        **print**("Hello "+ **name**)***

    *Hello Prashant1*

*Let’s take a look at one more code*

    *print**("Please enter your name")
    **name **= **input**()
    if **name**:
        **print**("Welcome Back")
    else:
        **print**("Who are you")***

*This is something weird and the reason it will work because of the **Truthy/Falsey** value. Blank string means falsey and others are truthy.*

*So if we don’t enter anything*

    *Please enter your name*

    *Who are you*

*In case if I enter a value(Prashant)*

    *Please enter your name
    Prashant
    Welcome Back*

*In case of integers*

* *0 is falsey and all others are truthy*

* *0.0 is falsey, all others are truthy*

*To check these let's pass value to **bool()** function.*

    *>>> bool(0)*

    *False*

    *>>> bool(42)*

    *True*

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
[**100 Days of DevOps — Day 85- Shell Script to find the failed login**
*Welcome to Day 85 of 100 Days of DevOps, Focus for today is Shell Script to find the failed login*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-85-shell-script-to-find-the-failed-login-a87975b9e21f)
