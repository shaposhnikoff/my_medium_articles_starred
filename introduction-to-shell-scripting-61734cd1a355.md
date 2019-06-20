
# Introduction to Shell Scripting

Before going into the in-depth of Shell Scripting, the first question we need to ask what is a Shell?
> *Shell is a command interpreter.*

*A Shell script is nothing but a list of commands stored in a file.*

*Example of simple shell script*

    *# cat test.sh*

    *#!/bin/bash*

    *pwd*

    *ls*

*At the very least it saves the effort of typing the similar command again and again.*

*As we can see, there is nothing unusual here, the shell is executing these command, the similar thing we can do by executing the same command one by one from the shell command line. Later on these script will become a tool which we can modify based on our requirements.*

*Shell scripts are well suited for system administrative tasks or other repetitive tasks that don’t require a full-blown programming language.*

*Now at the top of the script you will see #! which is called she bang(two byte magic number)*

    *#!*

*This tells that this file is a set of commands to be fed to command line interpreters indicated.*

*Immediately after she-bang is a path name, which is the path to the programs that interpret the commands in the script(default is bash in most Unix OS)*

*To check the default shell*

    *# echo $SHELL*

    */bin/bash*

*To get more information or Shells supported by your OS*

    *# cat /etc/shells*

    */bin/sh*

    */bin/bash*

    */sbin/nologin*

    */usr/bin/sh*

    */usr/bin/bash*

    */usr/sbin/nologin*

*NOTE: We can omit #! if the script consists of only generic system commands and no internal shell directives.*

*Executing the script*

* *Give execute permission to everyone*

    *# chmod +x test.sh*

* *As we already gave execute permission to everyone, now let’s try to execute this script*

    *# ./test.sh*

    */root/bash_script*

    *test.sh*

*OR*

    *# bash test.sh*

    */root/bash_script*

    *test.sh*

*Once we test and debug we can copy it to any of our Path variable*

    *# bash test.sh*

    */root/bash_script*

    *test.sh*

*To do that*

    *cp test.sh /usr/bin/testing*

    *# testing*

    */root*

    *bash_script*

*Now the question is why we need to copy this script and why can’t it will be directly executed from the path(eg: /root/bash_script) in my case. It’s because current directory is not in $PATH and that’s is because of the security reason*
