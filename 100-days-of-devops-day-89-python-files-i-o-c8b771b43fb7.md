
# 100 Days of DevOps — Day 89-Python Files I/O

Welcome to Day 89 of 100 Days of DevOps, Focus for today is Python Files I/O

*To open a file in Python we use a built-in open() function which accepts a number of arguments*

    ***open(file,mode,encoding)***

* ***file**: the path to file(required)*

* ***mode**: read/write/append,binary/text*

* ***encoding: **text encoding to use(it depend upon system to system)*

    *>>> import sys*

    *>>> sys.getdefaultencoding()*

    *‘utf-8’*

*Python File Mode*

    *r for reading*

    *w for writing*

    *r+ opens for reading and writing (cannot truncate a file)*

    *w+ for writing and reading (can truncate a file)*

    *rb+ reading or writing a binary file*

    *wb+ writing a binary file*

    *a+ opens for appending*

*Writing to the first file*

    *>>> **f = open(“testfile”,”w”)***

    *>>>** f.write(“This is our first test file”)***

    *27*

    *>>> **f.close()***

*Output*

    *ls -l testfile*

    *-rw-rw-r — 1 plakhera wheel 27 Apr 20 17:17 testfile*

    *cat testfile*

    *This is our first test file*

*To read a file*

    *>>> **a = open(‘testfile’,’r’)***

    *>>> **a.read()***

    *‘This is our first test file’*

    *# To read the first 5 characters
    >>> **a.read(5)***

    *'This '*

    *# Now if we try to read the rest of file  
    >>>** a.read()***

    *'is our first test file'*

*Now if the file has multiple lines and we want to read line by line*

    *>>> **x.readline()***

    *‘My new file \n’*

    *>>> **x.readline()***

    *‘Ok one more line’*

*Now if we want to append to an existing file use **append(a) mode***

    *>>> **f = open(“myfile”,”a”)***

    *>>>** f.write(“Testing append mode”)***

    *19*

    *>>> **f.close()***

    *>>>*

    *>>>*

    *>>>*

    *>>> **g = open(“myfile”,”r”)***

    *>>> **g.read()***

    *‘My new file \nOk one more lineTesting append mode’*

*Now let's take a look at one more example*

    ***import **sys
    **def readfile(**filename**):
        **f **= **open**(**filename,**'rt')
        for **i **in **f**:
            **print**(**i**)
        **f.close**()
    ***

    ***if **__name__ **== '__main__':
        **readfile**(**sys.argv**[**1**])***

*Output*

    *python3 exception.py testfile*

    *this is a test file*

    *One more line to read*

*As you can see we have a problem here, each line is already terminated by a **newline** and then print adds it own*

*To fix that we can strip the newline*

    ***import **sys
    **def readfile(**filename**):
        **f **= **open**(**filename,**'rt')
        for **i **in **f**:
            i = i.strip()  <---
            **print**(**i**)
        **f.close**()
    ***

    ***if **__name__ **== '__main__':
        **readfile**(**sys.argv**[**1**])***

*OR we can use the write method of **standard out stream***

    ***import **sys
    **def readfile(**filename**):
        **f **= **open**(**filename,**'rt')
        for **i **in **f**:
            sys.stdout.write(i) <--
        **f.close**()
    ***

    ***if **__name__ **== '__main__':
        **readfile**(**sys.argv**[**1**])***

*As you see in all the above examples we are using close to close the file. close is important as we are telling the underlying the OS that we are done with it, if we are not closing the file then there is a possibility that we might lose the data. There may be pending writes that buffered up which might not get written completely. Also if we are opening lots of files we might run out of **system resources.***

*So might need some way that we always remember to close a file. Python implements a resource cleanup called **with-block.***

*So now our code will look like this*

    ***import **sys
    **def readfile(**filename**):
    **    **with open(filename,'rt') as f: <--**
    **        for **i **in **f**:
                **sys.stdout.write**(**i**)***

    ***if **__name__ **== '__main__':
        **readfile**(**sys.argv**[**1**])***

*We don’t need to call **close** explicitly, as with will take care of it whenever execution exits the block.*

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
[**100 Days of DevOps — Day 88-Lists in Python**
*Welcome to Day 88 of 100 Days of DevOps, Focus for today is Lists in Python*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-88-lists-in-python-a6eb7fdb6cee)
