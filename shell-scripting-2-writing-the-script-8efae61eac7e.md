Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m13[39m, end: [33m73[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m126[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m83[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m100[39m, end: [33m130[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m138[39m, end: [33m155[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m22[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m90[39m }

# Shell Scripting 2: Writing the Script

Standard Streams, Redirection, Script inputs, execution and substitution

In part one of this series, I talked about the basics of Linux system. I could have simply given what I follow to write the shell script but that would not be useful. If you are writing it for enterprises and are managing some serious applications generating big revenues, you code deliverable should be good. The first part was necessary because if you are already good with Linux basics/administration, honestly you wouldn’t be reading this blog as the admins are already way smarter on the scripting part. These two blogs are intended for application developers who, I believe, must know about the Linux basics and scripting because as you write the core application, you will also be responsible to write some testing automation scripts, application failover and recovery scripts, Dockerfile, Jenkins pipeline, application upgrades and disaster recovery workflows.

Here also I would not be covering basic syntax for the code. There are tonnes of write-ups you will find on the internet. I will focus on how you can use the basic blocks along with some crucial items to write an enterprise-level and production-ready shell script.

## Basic Concepts

I hope from the previous blog, you are already familiar with the terms — terminal and shell, what are the different kinds of shell available out there.

The first basic shell script would be like this —

    **>> cd /home/innovationchef
    >> vi script.sh
    .. echo "Hello World"
    :wq
    >> chmod 775 script.sh
    >> ./script.sh
    Hello World**

Once you get the above script running. We are good to start writing the basic scripts. Here I would enlist the items that you need to go and learn about separately —

### Variable and function declaration

You need to know about the local/global variables, reading a var from stdin using the ***read ***command, variable concatenations and substitutions, using ***declare ***to create read-only and integer variables and ***unset ***a variable. Give special focus on arrays and associative arrays. Always do a explicit declaration of the variable along with assignment and don’t leave it up to the shell to decide.

    # Declaration**
    >> declare -a DEPLOYMENT_NAME=(order basket, login)
    >> declare -a DEPLOYMENT_NAME=([1]=order [2]=basket [3]=login)
    >> declare -A DEPLOYMENT_REPLICAS_MAP=([order]=4 [login]=10 [basket=7])**

    # To access the values 
    **>> for val in "${DEPLOYMENT_REPLICAS_MAP[@]}"; do 
    >>   echo "${val}"
    >> done
    4 10 7**

    # To access the keys
    **>> for key in "${!DEPLOYMENT_REPLICAS_MAP[@]}"; do 
    >>   echo "${key}"
    >> done
    order login basket**

    # Size of the array 
    **>> ${#DEPLOYMENT_REPLICAS_MAP[@]}**

    # Adding an item
    **>> DEPLOYMENT_NAME+=(payment categories)
    >> DEPLOYMENT_NAME+=([4]=payment [5]=categories)
    >> DEPLOYMENT_REPLICAS_MAP+=([payment]=3 [categories]=2)**

    # Deleting an item
    **>> unset DEPLOYMENT_REPLICAS_MAP[1]
    >> unset DEPLOYMENT_REPLICAS_MAP[payment]**

    # Command Substitution
    **>>** **arr=( $(ls) )**

    # Split arrays 
    **>>** **${arr[@]:1:4}**

For function declaration, you should keep in mind is that it is like any other functional programming ways of going it. The way you can pass arguments to a function in shell script is same as the way you pass arguments to the shell script itself. We will look at the latter below.

### Environment Variables

Some important Internal variables —

Note that I already gave a list of around 35 basic commands in the previous blog. These are global env variables available to your user with which you are logging in. These are the commonly user ones —

    **>> $BASH             **-- the path to the *Bash* binary itself
    **>> $BASH_VERSINFO[0]** -- Major version no. of installed bash
    **>> $FUNCNAME **        -- name of the current function
    **>> $GROUPS  **         -- groups current user belongs to
    **>> $HOME    **         -- home directory of the user
    **>> $HOSTNAME**
    **>> $IFS     **         -- This var determines how Bash recognizes fields, or word boundaries when it interprets character strings.
    **>> $LINENO  **         -- line number of the shell script in which this variable appears
    **>> $PATH    **         -- path to binaries, a list of directories, separated by colons
    **>> $PPID    **         -- The process ID of parent process
    **>> $PWD**
    **>> $REPLY            **-- default value when a variable is not supplied to [read](https://www.linuxtopia.org/online_books/advanced_bash_scripting_guide/internal.html#READREF)
    **>> $SECONDS          **-- The number of seconds the script has been running.
    **>> $UID              **-- user id of the current user

Note that a script can **export** variables only to child [processes](https://tldp.org/LDP/abs/html/special-chars.html#PROCESSREF), that is, only to commands or processes which that particular script initiates. A script invoked from the command-line *cannot* export variables back to the command-line environment. *Child processes cannot export variables back to the parent processes that spawned them.*

### Basic code flow

There are logical operators defined like —

    # for arithmetic manipulations**
    -eq, -nq, -le, -gt, -ge, -lt**

    # for string manipulation
    **==, =, !=, \>, \<
    -z **is string empty
    **-n** for not empty

    # for file checking 
    **-f filename   -> ** true if it is a regular file
    **-d directory  -> **true if it is a dir 
    **-s filename   ->** if it is non-zero size.

    # **&& and ||** are supported like in any other language

We can use this in the basic code flow just like any other language. You can look on the internet about — **if-else **block, **for **and **while **loops with **break **and **continue **statements and the switch **case — esac **block.

## Substitution

### Command Substitution

Command substitution allows the output of a command to replace the command itself. Bash performs the expansion by executing command and replacing the command substitution with the standard output of the command, **with any trailing newlines deleted. Embedded newlines are not deleted, but they may be removed during word splitting.**

    **>> declare -a ARR=$(**seq 1 5**)
    >> echo $ARR**
    1 2 3 4 5

### Process Substitution

Process substitution allows a process’s input or output to be referred to using a filename.

    **>> <(command)
    >> >(command)**

You will find a good example here —
[**Bash Process Substitution**
*In addition to the fairly common forms of input/output redirection the shell recognizes something called process…*www.linuxjournal.com](https://www.linuxjournal.com/content/shell-process-redirection)

### Variable Substitution

    **${var}            -- **Substitute the value of *var*.
    **${var:-word}      -- **value of word is var is null or unset
    **${var:=word}      -- I**f *var* is null or unset, *var* is set to the value of **word**.
    **${var:?message}   -- **If *var* is null or unset, *message* is printed to standard error. This checks that variables are set correctly.
    **${var:+word}      -- **If *var* is set, *word* is substituted for var. The value of *var* does not change.

    ## Examples 
    **>> foo=${bar:-something}
    >> echo $foo**
    something
    **>> echo $bar**

    **>> foo=${bar:=something}**
    **>> echo $foo 
    **something
    **>> echo $bar    **<-- something too, as there's an assignement to bar**
    **something

## Expansion

***Brace Expansion: ***Provide a comma-separated list of values with curly brackets. There are two optional parts — **preamble** and **postscript**. **Preamble**: to add text at the front of each generated string and **Postscript: **to append text at the end of** the generated string.**

    { String1, String2,... ,StringN }
    { <start> . . <end> }
    <preamble> { string or range } <postscript>

    **>> echo {"I like ","Learn "}{"PHP","Programming"}
    **I like PHP I like Programming Learn PHP Learn Programming

    **>> echo {01..03}{A..C}
    **01A 01B 01C 02A 02B 02C 03A 03B 03C** **

    **>> echo "Hi "{Ankit, Lohani, Innovationchef}
    **Hi Ankit Hi Lohani Hi Innovationchef

***Tilde expansion:***

    **~ **  -- replaced with /home
    **~+**  -- replaced with current working dir

***Path expansion:***

    >> cd Desktop
    **>> echo D*
    **DawnFolder DCIM.jpg**
    >> echo /usr/*/share
    /usr/local/share /usr/mongo/share**

**Arithmetic expansion: **Arithmetic expansion and evaluation is done by placing an integer expression using the following format and the result is substituted.

    **$((**expression**))**

    **>> if (( VARIABLE == 0 )); then echo "true"; fi
    **

The expression can contain anything from usual mathematical operators like +- to bitwise operators like > < to logical operators like == and +=.

Note that there are other ways of arithmetic manipulation, however, I prefer double parenthesis over ***let ***and ***expr***

## Grouping Commands

    ##** **command2 is executed if, and only if, command1 returns an exit status of zero (success).**
    >> command1 && command2**

    ## command2 is executed if, and only if, command1 returns a non-zero exit status.
    **>> command1 || command2**

    ## In pipes - Each command in a pipeline is executed in its own subshell, which is a separate process. It is a stream and all actions happen together unlike the last example. 
    **>> ps -ef | grep java | wc -l**

    ## In the below groupings, the redirection is applied on the complete block.

    ## below causes a subshell environment to be created and all cmds are executed there.
    **>> ( ls ps )**

    ## below causes the list to be executed in the current shell context.
    **>> { mkdir log.log, chmod 775 log.log, cat log.log }**

## Standard Streams & Redirection

Streams can be seen as a channel or a pipe through which data is going to flow and reach the shell. It is up to you where you connect these streams or pipes. There are three standard streams — one for input, another for output and finally for error. You can, if you wish, connect the input stream to a keyboard or a microphone, your output stream to your terminal on the screen and your error to your printer. It is just a pipe whose one end is connected to the shell and other other is free to be connected to anything you like! Note that input stream takes in the data and output/error spits it out. So, connecting input stream to a printer is useless!

These 3 streams are represented in linux as numeric values and are also called file descriptors. STDIN=0, STDOUT=1, STDERR=2. By default, 0 is connected to your keyboard and 1 and 2 are connected to the terminal. Hence you see everything on the screen and you never realize the difference.

How do we use it? Well, when your command runs in the screen, you see everything on the terminal and you will never need to understand all of this. However, when you go for certain automation and background running jobs, the out is not channel’ed out to the terminal. You will probably direct it to a file and you need to understand how that works. Let’s look at redirection.

As noted earlier, everything in unix is a file. It means that the devices attached to the system appears as files in the local file system, even the keyboard and the mouse! These devices can be enlisted here — **/dev. **Now, say you do a ssh to your terminal from two different places, there will be two devices enlisted as files in the /dev dir, namely — **/dev/pts/1 **and **/dev/pts/2. **All the three streams 0, 1 and 2 are connected to the same terminal where you are going to write commands and read output/error.

    ## If you do terminal 1
    **>> ls -l**

    ## Output will show up in terminal 1
    ## To redirect it, you can use **> 
    **## We will redirect it to a file /dev/2

    **>> ls -l 1> /dev/pts/2**

    ## This file descriptor is associated to your 2nd terminal and the 
    ## moment something is written to it, the terminal 2 will display it 
    ## Voila, you just printed the output of command in T1 to T2 term

Now that we know what streams and redirection is, there are 2 basic items in the checklist you should know

### **Output & Error Redirection**

**File redirection: **You can redirect to a file using the*** > filename.log ***symbol. This will override the existing file — also k/a clobbering. If you want to append rather than over-write use ***>>.***

    ## Clobbering
    **>> ls -l > logfile.log
    >> ls -l 1> logfile.log**

    ## Appedning
    **>> ls -l >> logfile.log**

    ## Redirecting both STDOUT and STDERR together
    **>> ls -l &> logfile.log**

**Stream redirection: **You can redirect the output to other stream instead of a file. You will use a ampersand here.

    ## Redirect STDOUT to STDERR and STDERR to STDOUT
    **>> ls -l 1>&2 
    >> ls -l 2>&1**

    ## Redirect both in the same command
    **>> ls -1 1> stdout.log 2> stderr.log
    >> ls -l > out.log 2>&1
    **## Note, in the second way, we first redirect to file and then 
    ## provide the stream redirection logic

Read another interesting topic on **/dev/null **here —
[**Step by step breakdown of /dev/null**
*Newcomers to Bash programming will sooner or later come across /dev/null and another obscure jargon: > /dev/null 2>&1…*medium.com](https://medium.com/@codenameyau/step-by-step-breakdown-of-dev-null-a0f516f53158)

I will leave two open questions for you to google and figure out —

1. Why do we have STDOUT and STDERR different? Eventually we will need error to be presented like any other output, right?

1. What would happen if I redirect both STDOUT and STDERR to the same file but separately? Like this — **>> ls -1 1> out.log 2> out.log**

### **Input Redirection**

Till now, I only talked about the output redirection. When we come to redirection in STDIN, there are two thing we need to talk about

**piping: **In my previous blog, I already talked about the pipes and out they stream output of one command to input of other command. That is nothing but a redirection. In the below example, the output of ps aux is redirected to grep as its input stream

    **>> ps aux | grep java
    >> ps aux |& grep java**

Note that the connections are provided before any redirection. In that sense it is asynchronous or lazily loaded in some sense. By default, only STDOUT is piped as input to the following command. If you want the STDERR to go as well, you may use |& instead of a single |.

**reading from files: **Using a < symbol we can take in the input stream from another file

    **>> wc < /home/innotionchef/myBlog.txt**

    ## this is same as below pipe redirection

    **>> cat /home/innotionchef/myBlog.txt | wc **

### **Creating New File Descriptors**

I found this block [here ](https://www.linuxtopia.org/online_books/advanced_bash_scripting_guide/io-redirection.html)and it was enough to understand the concept

    [j]<>filename
          # Open file "filename" for reading and writing, and assign file descriptor "j" to it.
          # If "filename" does not exist, create it.
          # If file descriptor "j" is not specified, default to fd 0, stdin.
          #
          # An application of this is writing at a specified place in a file. 
          echo 1234567890 > File    # Write string to "File".
          exec 3<> File             # Open "File" and assign fd 3 to it.
          read -n 4 <&3             # Read only 4 characters.
          echo -n . >&3             # Write a decimal point there.
          exec 3>&-                 # Close fd 3.
          cat File                  # ==> 1234.67890
          # Random access, by golly.

    ----------------------------------------------------------------

    exec 6<&0          # Link file descriptor #6 with stdin.
                       # Saves stdin.
    
    exec < data-file   # stdin replaced by file "data-file"
    
    read a1            # Reads first line of file "data-file".
    read a2            # Reads second line of file "data-file."

    exec 0<&6 6<&-
    #  Now restore stdin from fd #6, where it had been saved,
    #+ and close fd #6 ( 6<&- ) to free it for other processes to use.
    #
    # <&6 6<&-    also works.

Redirection can be applied to a block of code as well

    {
        # part of script 
    } > file1 2>file2 

I will show two important use cases where these concepts come in handy. After reading this, you would definitely want to use it in your application somewhere.

### **Use case 1: Create a shell script that writes the output to the logfile as well as the console**

I have seen scripts doing this —

    **>> $LOGFILE=/home/innovationchef/log.txt
    >> echo "File not found"
    >> echo "File not found" >> $LOGFILE**

That’s Bad! There will be so much boilerplate code hanging around in your script. We will use 2 things that we learnt so far — process substitution and redirection and along with a new **exec **and **tee **command to do this.

First we will understand the tee command because we are not aware of it. **tee command** reads the STDIN and writes it to both the STDOUT and one or more files

    **>> ps -ef | grep java** | tee /home/innovationchef/logs.log
    ## The output of the blocked pat goes to a file and is also displayed on the screen

The exec command comes from the family of core process theory — fork(), exec(), wait(), exit(). As discussed in the previous blog, it replaces the current process with new process. This replacement also happens for the redirection of the original process it replaced. In bash, the exec can be followed with a command, its arguments and any redirection. If you don’t provide any redirection, the original processes redirection will be applied. In our use case, we will over-write it.

Coming back to the use case —

    # The below would be creating a subshell**
    >> (command)**

    # The below would create a process file**
    >> (tee -"${LOGFILE}")**

    # The below will give us a file representing its STDOUT
    **>> >(tee -"${LOGFILE}")**

    ## the exec command with the redirection you want
    ## Below redirects the current shell to whatever yhou provide in the right
    **>> exec &> **>(tee -"${LOGFILE}")

    ## And we are done
    **>> echo "This will be logged to the file and to the screen"**

    ## As I discussed in redirection, you can also use the below
    **>> exec > >(tee -"${LOGFILE}") 
    >> 2>&1**

Of course there are multiple other ways, but I find the above one the most elegant way of doing it.

### Use case 2: Managing multiple cluster nodes from 1 server

**HERE Document**

It is a special form of I/O redirection that tell bash shell to read input from the current source until a line containing only the delimiter is seen.

    interactive-command <<unique-delimiter-string
    .. line 1
    .. line 2
    ..
    .. line n
    unique-delimiter-string

This SO answer already shows a the usages —
[**How can I write a heredoc to a file in Bash script?**
*Read the Advanced Bash-Scripting Guide Chapter 19. Here Documents. Here's an example which will write the contents to a…*stackoverflow.com](https://stackoverflow.com/a/25903579/7345493)

The use case you normally find on the internet is the *cat* one. I will show you another cool use case.

Suppose you are managing kafka cluster spread over 5 nodes and you wrote a script to do all cool stuff easily — like check status of the node, start and stop a node. Then you will have to it for both zookeeper and broker. The script would be big and you will have to execute it on each node. That means, you have to keep the same script on 5 different nodes and then go to each node and execute manually.

One day you will get irritated and will want to control the 5 nodes from 1 server. You would want to ssh to the 5 node from one server and execute script on that server. But, you would still need to keep the master script on each node.

Then you decided that you would keep the master script on the root server and do a scp every time you want to operate on the node. So, you would be basically doing a scp, then a ssh and finally you wild also want to delete the copy of the master script from the kafka node. That is still clumsy.

Then you found out that you can do this —

    **>> sshpass -p password scp -o StrictHostKeyChecking=no /localpath/kafka-manager.sh username@ip-address:/remotepath**

    **>> sshpass -p password ssh -o StrictHostKeyChecking=no username@ip-address "cd /remotepath; ./kafka-manager.sh; rm kafka-manager.sh; exit"**

Great right? I myself used this solution for sometime until I found the better way of doing it! Why create a script and transfer it? It was because the kafka-manager.sh script was too big and I didn’t want to write in the inverted commas after the ssh command. That was ugly and my IDE did not provide a great formatting of my script file.

Then I used the here document concept —

    **>> sshpass -p password ssh -o StrictHostKeyChecking=no username@ip-address <<MYUNIQUESTRING
    **## Here I worte the contents of the kafka-manager.sh file
    ## All 200 lines of code here
    **MYUNIQUESTRING**

Now this is what we call an elegant solution! No SCP and everything in 1 script at the controller server node.

## Important Components of a Shell Script

### Shebang

This fancy term is sitting every where on google. Go and check it out.

When you login to a shell, you get a default interpreter for your session which can be updated by your server administrator or you can do it yourself using the bashrc files. I would still mention the most common one that is being used and is usually available to you by default. Still having it in your script is a recommended activity. You can also provide a */usr/bin/env. *It makes your code portable in the sense that whatever bash executable appears first in the running user’s $PATH variable will be used.

    *#!/bin/sh*

### Shell Parameters

Note that there are two kind of parameters — the positional parameters that are provided as argument to you shell script and special parameters that are denoted by some special character.

    **>> ./my-script.sh foo bar**

    ## Within the script foo bar can be accessed as
    ## ${1}, ${2}.... ${9}, ${10} etc
    ## ${0} is the name of the script itself
    ## $* and $@ represent all args as one
    ## $# is the number of args passed
    ## $$ is the PID of the script run instance
    ## $? return code of the last executed command 

    ## Shift can manually move the pointer to an argument 
    while (( $# )); do
        echo $1
        **shift**
    done 

### getopts

getopts is the very important and nice feature that you will need in almost every script that you write. You will have to provide common arguments like script logging type — console or file or both, environment — prod or non-prod, and probably at least I command to tell what the script has to do in this execution instance (unless your script is meant to do the exact thing always).

Kevin here has covered all the important aspects to get you started —
[**Parsing bash script options with getopts**
*A common task in shell scripting is to parse command line arguments to your script. Bash provides the getopts built-in…*sookocheff.com](https://sookocheff.com/post/bash/parsing-bash-script-arguments-with-shopts/)

### Options

Options are built-in settings from the shell that would change it’s behavior. There are many but I would point out only the few that you would require.

Use (-)dash to activate and + to deactivate them

    ## Exit as soon as any command returns a non-0 exit code
    set -o errexit

    ## Exit the script if there are any un-initialized variable
    set -o nounset

    ## If the pipe redirection fails after any pipe - fail fast
    set -o pipefail

    ## Print each command to stdout before executing it
    set -o verbose

    ## Read commands in script, but do not execute them (syntax check)
    set -o noexec

    ## Prevent overwriting of files by redirection
    set -o noclobber

### Double dash

Develop a habit of using (- -) without spaces in most of your commands. It is used in most bash built-in commands to signify the end of command options, after which only positional parameters are accepted. Now, eve though something looks like an option, it will be treated as a argument. This is to be safe when you don’t know what can be on the right end of the command.

This one is quite useful with the **printf **command when it is wrapped in another function. [Here](https://unix.stackexchange.com/a/11382) is an example.

### Traps and Exit

I have already discussed the kernel signals in the previous blog. Here is how to write a handler for them in your shell script and take decisions. Using traps is important because if a SIG comes and you are in the middle of an arithematic operation or you have blocked some system resources during your run, it can result into unpredictable outcomes from your shell.

    >> function handle() {
        echo "handled"
    }
    trap handle SIGTERM SIGINT SIGHUP

Also, you can define what you want to do for each signal and it differs from case to case basis. In most examples you will see clean up of any temp directory you created as a part of your execution, you should also consider logging out from any other session that you might have logged in during the execution.

Here is a good article on it —
[**SignalTrap**
*Signals are a basic tool for asynchronous interprocess communication. What that means is one process (A) can tell…*mywiki.wooledge.org](https://mywiki.wooledge.org/SignalTrap)

Now, when you decided to exit from you application, do provide an exit code. It is very helpful for the parent process to decide what happened. This is because, a lot of times you will have some Ansible instance calling your script and you might not be invoking it yourself from the terminal. If becomes meaningful for the client party to deduce the execution summary from the exit code. Important ones are —

    1   -- General script errors
    127 -- command not found
    130 -- Script terminated by CTRL+C
    126 -- command cannot execute, when you are trying to read a file you don't have access to

You may define you own error codes and communicate it to the parent process what he should expect, just beware that you are not over-riding something already well defined in Linux.

### Some other useful commands

    **>> dirs    --** lists the contents of the directory stack
    **>> eval    -- **convert string into command**
    >> exec    -- **This shell builtin replaces the current process with a specified command - remember fork(), wait(), exec()??
    **>> find    -- **help you do a lookup over the file directories**
    >> basename -- **extract filename from the **
    >> dirname  -- **extract directory name from the

Finally, here is a sample shell script that includes almost everything we have covered in this blog. In this script, the production support team would be able to scale up the pods if all were initially down.

<iframe src="https://medium.com/media/336a24414dca0f4a9322d1912fa8c31d" frameborder=0></iframe>
