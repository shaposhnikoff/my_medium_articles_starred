
# 100 Days of DevOps — Day 52-Conditional Testing in Shell

Welcome to Day 52 of 100 Days of DevOps, Focus for today is Conditional Testing in Bash.

*Today I am going to discuss test command*

    *test - check file types and compare values*

* *In your script you generally see, it starts with left bracket( [ ) when need to perform some testing.*
> *Exit Status*

* *In bash after running any command, the exit status is returned and stored in variable $?*

    *0 --> good
    1 or some other code --> issue*
> ***Test Integers***

    *# test 1 -eq 1;echo $?*

    *0*

* *Here we are trying to check if 1 is equal to 1 and then perform command chaining to merge two command*

* *As mentioned above exit status 0 mean good*

    *# test 1 -eq 2;echo $?*

    *1*

* *Exit status other than zero, in the above case, means some issue*

* *The same way we can test **less than** or **greater than **or **not equal to***

    ***# less then**
    # test 1 -lt 2;echo $?*

    *0*

    ***OR***

    ***# greater then***

    *# test 1 -gt 2;echo $?*

    *1*

    ***# not equal to***

    ***OR ***

    *# test 1 -ne 2;echo $?*

    *0*
> *Test Strings*

* *To compare string, use a single equal sign(this is different as compare to other programming languages where single equal sign is assignment operator(=))*

    *# test hello = hello; echo $?*

    *0*

* *Same way you can test not equal to by using !*

    *# test hello != hello; echo $?*

    *1*
> *Test Files*

* *Here I have two files(file1 and file2), where you can see file2(created at 05.15) is newer then file1(created at 05:14)*

    *# ls -l file?*

    *-rw-r--r-- 1 root root 0 Apr  3 05:14 file1*

    *-rw-r--r-- 1 root root 0 Apr  3 05:15 file2*

* *To test if file2 is newer is than file1*

    *# test file2 -nt file1;echo $?*

    *0*

* *To check for block and character device*

    *# To check block device
    # test -b /dev/xvda;echo $?*

    *0*

    *# To check character device
    # test -c /dev/tty1;echo $?*

    *0*

* *To check if the file exists*

    *# test -e file1; echo $?*

    *0*

* *To check if the file exists and its a regular file(use -f)*

    *# mkdir file3*

    *# test -e file3; echo $?*

    *0*

    *# test -f file3; echo $?*

    *1*

* *To check if the file exists and it’s non-zero*

    *# test -s file2; echo $?*

    *1*

    *# echo "hello"> file2*

    *# test -s file2; echo $?*

    *0*

* *Let put together everything in the script we use in our day/today life, to check if the file exists*

<iframe src="https://medium.com/media/2fb58ae43706b4b4b9b8752a8f3ae53a" frameborder=0></iframe>

* *We can further improve this script by asking input from a user rather than hardcode the filename using positional parameter*

<iframe src="https://medium.com/media/33f3773f708e56ece2898936e1e83906" frameborder=0></iframe>

* *When we execute this script*

    *# ./fileexist.sh testfile*

    *The file 'testfile' in not found.*

    *# ./fileexist.sh /etc/shadow*

    *The file '/etc/shadow' exists.*

* *Some other functionality you can test*

    ***-b** FILE*

    *FILE exists and is block special*

    ***-c** FILE*

    *FILE exists and is character special*

    ***-d** FILE*

    *FILE exists and is a directory*

    ***-e** FILE*

    *FILE exists*

    ***-f** FILE*

    *FILE exists and is a regular file*

    ***-g** FILE*

    *FILE exists and is set-group-ID*

    ***-G** FILE*

    *FILE exists and is owned by the effective group ID*

    ***-h** FILE*

    *FILE exists and is a symbolic link (same as **-L**)*

    ***-k** FILE*

    *FILE exists and has its sticky bit set*

    ***-L** FILE*

    *FILE exists and is a symbolic link (same as **-h**)*

    ***-O** FILE*

    *FILE exists and is owned by the effective user ID*

    ***-p** FILE*

    *FILE exists and is a named pipe*

    ***-r** FILE*

    *FILE exists and read permission is granted*

    ***-s** FILE*

    *FILE exists and has a size greater than zero*

    ***-S** FILE*

    *FILE exists and is a socket*

    ***-t** FD  file descriptor FD is opened on a terminal*

    ***-u** FILE*

    *FILE exists and its set-user-ID bit is set*

    ***-w** FILE*

    *FILE exists and write permission is granted*

    ***-x** FILE*

    *FILE exists and execute (or search) permission is granted*

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
