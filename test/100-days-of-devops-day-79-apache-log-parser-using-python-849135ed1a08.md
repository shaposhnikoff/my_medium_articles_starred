
# 100 Days of DevOps — Day 79-Apache Log Parser Using Python

Welcome to Day 79 of 100 Days of DevOps, Focus for today is Apache Log Parser Using Python

*The aim of this tutorial is to create Apache log parser which is really helpful in determine offending IP addresses during the DDoS attack on your website. This is what we are going to do*

* *Read Apache log file(access.log)*

* *Count quantity of requests to your website from each IP address*

*If you look at the content of access.log this is how it looks*

    *192.168.0.1 — — [23/Apr/2017:05:54:36 -0400] “GET / HTTP/1.1” 403 3985 “-” “Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36”*

*Let’s break down these fields*

    *192.168.0.1 --> IP address*

    *[23/Apr/2017:05:54:36 -0400] --> Date,time and timezone*

    *GET / HTTP/1.1 --> HTTP get request to read the page*

    *403 --> Server response code*

    *3985 --> Number of byte transferred*

    *Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36” --> Finally there are data of user's hardware,OS and browser*

*But the piece we are interested in is IP address*

*So as mentioned above our first step is*

* ***Read the Apache log file***

    ***def apache_log_reader(**logfile**):***

    ***# We are saying opened file to the f variable, where f is a reference to the file object 
        with **open**(**logfile**) as **f**:
            **log **= **f.read**()
        **print**(**log**)
    ***

    *# Create entry point of our code
    **if **__name__ **== '__main__':
        **apache_log_reader**("access_log")***

*Now let’s go to the second part*

* ***Count IP address***

    ***import **re
    **from **collections **import **Counter*

    ***def apache_log_reader(**logfile**):
        **myregex **= r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}'***

    ***    with **open**(**logfile**) as **f**:
            **log **= **f.read**()
            **my_iplist **= **re.findall**(**myregex,log**)
            **ipcount **= **Counter**(**my_iplist**)
            for **k, v **in **ipcount.items**():
                **print**("IP Address " + "=> " + **str**(**k**) + " " + "Count "  + "=> " + **str**(**v**))***

    *# Create entry point of our code
    **if **__name__ **== '__main__':
        **apache_log_reader**("access_log")***

*Let’s break this code*

* *As we don’t need the entire entry we need to do some pattern search and that we can do with the help of regular expression*

* *We imported the re module and then write the regular expression matching pattern, where*

    ***\d:** Any numeric digit[0–9]*

*For more info please refer to*
[**Python Regular Expression**
*Regular expressions can be think like a mini-language for specifying text pattern*medium.com](https://medium.com/@devopslearning/python-regular-expression-8ee28d35f3a7)

    ***r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}'***

* *r stands for raw string*

* *Then we are using the collection module*
[**Python Collections Module**
*This module implements specialized container datatypes providing alternatives to Python’s general purpose built-in…*medium.com](https://medium.com/devops-challenge/python-collections-module-2b1129052d62)

Output

    IP Address => 192.168.0.2 Count => 1
    IP Address => 192.168.0.3 Count => 1
    IP Address => 192.168.0.4 Count => 3
    IP Address => 192.168.0.5 Count => 1
    IP Address => 192.168.0.6 Count => 1

*This is not the only way to write this code, there is a much better way to write the same piece of code, so stay tuned ;-)*

Now the better way to write the apache log parser

<iframe src="https://medium.com/media/382edee01a8a24b07bd4db0d2855c5c6" frameborder=0></iframe>

GitHub Link
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/blob/master/apache_log_parser.py)

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
[**100 Days of DevOps — Day 78- Python OS/Subprocess Module**
*Welcome to Day 78 of 100 Days of DevOps, Focus for today is Python OS/Subprocess Module*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-78-python-os-subprocess-module-95ae25bc686d)
