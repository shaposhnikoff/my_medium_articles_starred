Unknown markup type 10 { type: [33m10[39m, start: [33m72[39m, end: [33m84[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m91[39m }

# 100 Days of DevOps — Day 53-Introduction to Regular Expression — Part 1

Welcome to Day 53 of 100 Days of DevOps, Focus for today is Introduction to Regular Expression
> *What is a Regular Expression?*

*It’s a pattern matching language*

*OR*

*Regular expressions are specially encoded text strings used as patterns for matching sets of strings*

*OR*

*Is a sequence of characters that define a search pattern*

*This is what Regular Expression looks like*

    *\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}*

* *To break down this*

    ** \d : Matches a digit 0-9
    * {1,3}: Repeat the prior pattern 1-3 times
    * . : is a wildcard, so we need to escape it*

*The above expression match any IP address*

*eg:*

    *192.168.0.1
    127.0.0.1*
> *The concept we have just learned let’s try to use it with grep command*

* *I have a file called myipaddress*

    *# cat myipaddress*

    *192.168.0.1*

    *172.16.0.2*

    *10.0.0.3*

* *to grep ip address from this file*

    *# grep -P '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' myipaddress*

    ***192.168.0.1***

    ***172.16.0.2***

    ***10.0.0.3***

*Where -P*

    ***-P**, **--perl-regexp***

    *Interpret PATTERN as a Perl regular expression.  This is highly experimental and **grep** **-P** may warn of unimplemented features.*

* *If I try to use -E*

    ***-E**, **--extended-regexp***

    *Interpret PATTERN as an extended regular expression (ERE, see below).  (**-E** is specified by POSIX.)*

    *# grep -E ‘\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}’ myipaddress
    #*

* *Now why this doesn’t return anything because extended regular expression doesn’t use \d to refer digits it uses :digit:*

    *# grep -E '[[:digit:]]{1,3}\.[[:digit:]]{1,3}\.[[:digit:]]{1,3}\.[[:digit:]]{1,3}' myipaddress*

    ***192.168.0.1***

    ***172.16.0.2***

    ***10.0.0.3***

* *So far I have shown you all the digit example but how to match word, to match words you need to use \w OR you can use ranges*

    *\w, \W: ANY ONE word/non-word character. For ASCII, word characters are [a-zA-Z0-9_]*

    *Ranges will look like this [A-Z] or [a-z]*

* *Let’s take a simple example where I need to match word, special character, and digit*

    ***A-1***

    ***B-2***

    ***E-3***

* *To do that*

    *# grep -P '\w\-\d' test*

    ***A-1***

    ***B-2***

    ***E-3***
> *WhiteSpaces*

* *\s, \S: ANY ONE space/non-space character. For ASCII, whitespace characters are [ \n\r\t\f]*
> **Position Anchors**: does not match character, but position such as start-of-line or end-of-word

* *Let say I want to search for root user in /etc/passwd file*

    *# grep root /etc/passwd*

    ***root**:x:0:0:**root**:/**root**:/bin/bash*

    *operator:x:11:0:operator:/**root**:/sbin/nologin*

* *This doesn’t seem to be correct*

    *# grep ^root /etc/passwd*

    ***root**:x:0:0:root:/root:/bin/bash*

* ***^: start-of-line***

* *Similar way we can use $ which denotes the end of line*

    *# grep "bash$" /etc/passwd*

    *root:x:0:0:root:/root:/bin/**bash***

    *centos:x:1000:1000:Cloud User:/home/centos:/bin/**bash***

    *testuser:x:1001:1001::/home/testuser:/bin/**bash***

* *Let’s look at one of the common problems*
> Scenario 1: Delete all the blank line in the file

    *# cat error_log*

    *[Fri Apr 05 04:21:33.481424 2019] [suexec:notice] [pid 7698] AH01232: suEXEC mechanism enabled (wrapper: /usr/sbin/suexec)*

    *[Fri Apr 05 04:21:33.491890 2019] [auth_digest:notice] [pid 7698] AH01757: generating secret for digest authentication ...*

    *[Fri Apr 05 04:21:33.492419 2019] [lbmethod_heartbeat:notice] [pid 7698] AH02282: No slotmem from mod_heartmonitor*

* *As you can see we have white space after each line*

    *# sed -i -r ‘/^\s*$/d’ /var/log/httpd/error_log*

* *To fix this we can use sed in a combination of what we have learned today*

* *The way sed generally works*

    ***sed ’s/find/replace/g’ <filename>***

    ** sed is a Unix utility that parses and transforms text
    * -i : edit files in place
    * **-r : ** use extended regular expressions in the script
    * d : signify we want to delete these lines*
> Scenario 2: Look for the specific word in the file

    *# grep -i error error_log*

    *[Fri Apr 05 04:21:33.494401 2019] [core:notice] [pid 7698] AH00094: Command line: '/usr/sbin/httpd -D FOREGROUND' my**error***

    *[Fri Apr 05 04:22:05.502364 2019] [autoindex:**error**] [pid 7702] [client 204.14.239.17:65129] AH01276: Cannot serve directory /var/www/html/: No matching DirectoryIndex (index.html) found, and server-generated directory index forbidden by Options directive **error**log*

    *[Fri Apr 05 04:22:29.747570 2019] [autoindex:**error**] [pid 7701] [client 70.42.131.189:15077] AH01276: Cannot serve directory /var/www/html/: No matching DirectoryIndex (index.html) found, and server-generated directory index forbidden by Options directive*

* *Here as you can see I am looking for word error but grep is returning all the lines which include error(i.e errorlog and myerror)*

    *# grep -P '\berror\b' error_log*

    *[Fri Apr 05 04:22:05.502364 2019] [autoindex:**error**] [pid 7702] [client 204.14.239.17:65129] AH01276: Cannot serve directory /var/www/html/: No matching DirectoryIndex (index.html) found, and server-generated directory index forbidden by Options directive errorlog*

    *[Fri Apr 05 04:22:29.747570 2019] [autoindex:**error**] [pid 7701] [client 70.42.131.189:15077] AH01276: Cannot serve directory /var/www/html/: No matching DirectoryIndex (index.html) found, and server-generated directory index forbidden by Options directive*

    *You have new mail in /var/spool/mail/root*

* *\b: the boundary of a word, i.e., start-of-word or end-of-word*

*NOTE: Word Boundary is not the same in every case, in case of vim you need to use angle brackets,so expression will be like this :/\<error\>*

![](https://cdn-images-1.medium.com/max/5760/1*nibHA0v0PyoyIF5fjNs7RQ.png)

*Some of my favorite website to make your regular expression life easy*
[**RegExr: Learn, Build, & Test RegEx**
*Regular expression tester with syntax highlighting, PHP / PCRE & JS Support, contextual help, cheat sheet, reference…*regexr.com](https://regexr.com/)
[**Online regex tester and debugger: PHP, PCRE, Python, Golang and JavaScript**
*Regex101 allows you to create, debug, test and have your expressions explained for PHP, PCRE, Python, Golang and…*regex101.com](https://regex101.com/)
[**Regex Tester - Javascript, PCRE, PHP**
*Test your Javascript and PCRE regular expressions online.*www.regexpal.com](https://www.regexpal.com/)

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
