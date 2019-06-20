
# Shell Scripting Tips

echo by default add a new line at the end of output text

    *# echo "hello world"*

    *hello world*

* *In some cases or when using inside shell script we want to avoid that, this can be done using -n*

    *[root@docker bash]# echo -n "hello world"*

    *hello world[root@docker bash]#*

*As per man page*

    ***-n**     do not output the trailing newline*

* *echo also accept escape sequences*

    *# echo "a\tb\t"*

    *a\tb\t*

* *To add tab*

    *# echo -e "a\tb\t"*

    *a b*

* *where -e*

    ***-e**     enable interpretation of backslash escapes*

* *To find the environment variable of any process but get its process id*

    *[root@docker bash]# pgrep docker*

    *983*

    *1008*

* *Then look inside the /proc filesystem*

    *# cat /proc/983/environ*

    *LANG=en_US.UTF-8PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/binNOTIFY_SOCKET=/run/systemd/notify*

* *This look like a jumbled output, everything separated by key-value pairs, to parse the output we can use utility like tr(We are substituting null characters with new line)*

    *# cat /proc/983/environ |tr -s '\0' '\n'*

    *LANG=en_US.UTF-8*

    *PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin*

    *NOTIFY_SOCKET=/run/systemd/notify*

* *Finding the length of a string*

    *[root@docker bash]# var=abc*

    *[root@docker bash]# echo ${#var}*

    *3*

* *Performing an *arithmetic* operation*

    *[root@docker bash]# x=1*

    *[root@docker bash]# y=2*

    *[root@docker bash]# let z=x+y*

    *[root@docker bash]# echo $z*

    *3*

* *Square bracket can be used as an alternate to let command*

    *[root@docker bash]# a=$[ x + y ]*

    *[root@docker bash]# echo $a*

    *3*

* *We can also use expr to perform arithmetic operation*

    *[root@docker bash]# b=`expr 1 + 2`*

    *[root@docker bash]# echo $b*

    *3*

*P.S: All the above methods don’t support floating point number and *operate* only on integers*

* *bc is used to solve this purpose*

    *[root@docker bash]# echo "4 * 1.2" |bc*

    *4.8*

*OR*

    *[root@docker bash]# x=4*

    *[root@docker bash]# y=5.2*

    *[root@docker bash]# echo "$x * $y"|bc*

    *20.8*

* *If we want decimal precision*

    *[root@docker bash]# echo "scale=2;$y / $x"|bc*

    *1.30*

* *File Descriptors*

    *0 STDIN
    1 STDOUT
    2 STDERR*

* *Let’s look at standard errors*

    *[root@docker bash]# ls nofile*

    *ls: cannot access nofile: No such file or directory*

* *If we want to re-direct this output to a file*

    *[root@docker bash]# ls nofile > testfile*

    *ls: cannot access nofile: No such file or directory*

    *[root@docker bash]# cat testfile*

    *[root@docker bash]#*

* *To re-direct this output*

    *[root@docker bash]# ls nofile 2> testfile*

    *[root@docker bash]# cat testfile*

    *ls: cannot access nofile: No such file or directory*

* *To re-direct both stdout and stderr to a file using*

    *2&>1 OR &>*

* *Sometimes we are not interested in output, in those cases, we can re-direct the output to /dev/null(any data is discarded/blackhole)*

    *[root@docker bash]# ping -c 1 www.google.com*

    *PING www.google.com (172.217.6.68) 56(84) bytes of data.*

    *64 bytes from test.example.net (172.217.6.68): icmp_seq=1 ttl=54 time=28.7 ms*

    *--- www.google.com ping statistics ---*

    *1 packets transmitted, 1 received, 0% packet loss, time 0ms*

    *rtt min/avg/max/mdev = 28.777/28.777/28.777/0.000 ms*

    *[root@docker bash]# ping -c 1 www.google.com > /dev/null*

* *Someone we need to re-direct multiple line of text as standard input*

    *#!/bin/bash*

    *cat<<**EOF**>>test.txt*

    *This is a test file*

    *testing the redirection operator*

    ***EOF***

*NOTE: Line appears between EOF act as a standard input*
