
# 100 Days of DevOps — Day 74- Introduction to GIT

Welcome to Day 74 of 100 Days of DevOps, Focus for today is Introduction to GIT

*Git is a [free and open source](https://git-scm.com/about/free-and-open-source) distributed version control system designed to handle everything from small to very large projects with speed and efficiency.*

*Installing git*

    *# yum -y install git*

*To check GIT version*

    *# git --version*

    *git version 1.8.3.1*

***Basic GIT Configuration settings***

*Before starting our first GIT Project we need to do setup configuration . For eg: setting up username and email address will give us the credit that this checkin is done by this username/email id.*

***Setting up username***

    *# git config --global user.name “plakhera”*

    *# git config --global user.name*

    *plakhera*

***Setting up email***

    *# git config --global user.email “test@gmail.com”*

    *# git config --global user.email*

    *test@gmail.com*

*All these git config saved in simple plain text file(**.gitconfig**),as this plain text file you can modify this file*

    ***# cat ~/.gitconfig***

    *[user]*

    *name = plakhera*

    *email = test@gmail.com*

*or we can list the content of this file*

    *# **git config --list***

    *user.name=plakhera*

    *user.email=test@gmail.com*

*Now the below setting is just to give cosmetic stuff to git i.e when you run any git command it just give color to the output of those commands*

    ***# git config --global color.ui true***

*To handle auto carriage return line feed(whenever we put any file in git its just going to strip any carriage return and just put line feed at the end of line)*

    ***# git config --global core.autocrlf input***

*NOTE: For **windows** operating system set it to **true***

***Configure Aliases***

*What aliases will do it will let us create our own command*

    *# It will run status command in silent mode
    # **git config — global alias.s “status -s”***

    *# To show log in oneline, with all branches,along with graph and decorate the output of content
    # **git config — global alias.l “log --oneline --all --graph --decorate”***

***First GIT Project***

*With all the basic setting in place let’s start our first GIT project*

    *# **git init myfirstproject***

    *Initialized empty Git repository in /root/myfirstproject/.git/*

*This is what we all need to do to start myfirst project*

    *# **cd myfirstproject/***

    *# **git status***

    *# On branch master*

    *#*

    *# Initial commit*

    *#*

    *nothing to commit (create/copy files and use “git add” to track)*

*Now let start our project and create our first file*

    *# **touch firstfile***

    *#** git status***

    *# On branch master*

    *#*

    *# Initial commit*

    *#*

    *# Untracked files:*

    *# (use “git add <file>…” to include in what will be committed)*

    *#*

    *# firstfile*

    *nothing added to commit but untracked files present (use “git add” to track)*

*Now what git is trying to tell us that we have created one file but so far we didn’t tell git anything about that file so what git is saying what should I do with this file :-)*

*Now lets add this file in staging area*

![](https://cdn-images-1.medium.com/max/2000/1*zVDjNDLDQdu0EIj1ROfCKA.png)

*Now lets commit this file*

    ***# git commit -m “my first project file”***

    *[master (root-commit) 8e3d2a5] my first project file*

    *1 file changed, 0 insertions(+), 0 deletions(-)*

    *create mode 100644 firstfile*

*Lets break down this*

* ***master**: it’s telling us that it’s the master branch*

* ***root-commit**: it’s the first commit we create this project*

* ***8e3d2a5**: sha hash and that is the unique identifier for the commit*

*Anyone comes with subversion world always have this question why GIT doesn’t have mono-incrementing commit like we have with subversion, and the reason for that when we working with GIT it’s a distributed version control system so many developer at the same time making changes to the project without having any knowledge of what other commiter is doing that why this unique hash is helpful so that we are not going to step into someone else foot :-)*

* ***1 file changed:** which is self explanatory as we only made changes to one file*

* ***100644**: which is a standard unix permission set*

*Now check the status, what it’s saying that all the work were done is now saved in history*

    *# **git status***

    *# On branch master*

    *nothing to commit, working directory clean*

*Now what we have done above lets visualize it with the help of this pic what we called **three stages of git**, this seems to be lot of trouble/extra work when we first started with GIT*

![](https://cdn-images-1.medium.com/max/2000/1*65i8h2MafBeaSmmZO5c6MA.png)

*Now let’s check the git commit history*

    *# **git log***

    *commit 8e3d2a58e5c9182f2d646d1b0586a58ba1d5c17b*

    *Author: plakhera <laprashant@gmail.com>*

    *Date: Thu Apr 20 12:02:41 2017 -0400*

    *my first project file*

*It will give all the information*

* ***Commit hash***

* ***Author of the commit***

* ***Date when we commit these changes***

* ***Commit message***

*But if you remember we created that alias*

    *# git l*

    ** 8e3d2a5 (**HEAD**, **master**) my first project file*

***Renaming file in GIT***

*There are two ways to rename file in GIT*

* *GIT way*

    *# **git mv firstfile secondfile***

    ***# It staged the file too**
    # **git status***

    *# On branch master*

    *# Changes to be committed:*

    *# (use “git reset HEAD <file>…” to unstage)*

    *#*

    *# renamed: firstfile -> secondfile*

    *# **git commit -m "rename firstfile to secondfile"***

    *[master de2c659] rename firstfile to secondfile*

    *1 file changed, 0 insertions(+), 0 deletions(-)*

    *rename firstfile => secondfile (100%)*

* *Unix way*

    *# **mv secondfile thirdfile***

    *# **git status***

    *# On branch master*

    *# Changes not staged for commit:*

    *# (use “git add/rm <file>…” to update what will be committed)*

    *# (use “git checkout — <file>…” to discard changes in working directory)*

    *#*

    *# deleted: secondfile*

    *#*

    *# Untracked files:*

    *# (use “git add <file>…” to include in what will be committed)*

    *#*

    *# thirdfile*

    *no changes added to commit (use “git add” and/or “git commit -a”)*

    *# **git add -A .***

    *# **git commit -m “renaming secondfile to thirdfile”***

    *[master 5ada133] renaming secondfile to thirdfile*

    *1 file changed, 0 insertions(+), 0 deletions(-)*

    *rename secondfile => thirdfile (100%)*

*Previous to version to git version 2.0 if we try to move the file which is basically deleting and creating new file we need to use -A*

    ***# git add .***

    *warning: You ran ‘git add’ with neither ‘-A ( — all)’ or ‘ — ignore-removal’,*

    *whose behaviour will change in Git 2.0 with respect to paths you removed.*

    *Paths like ‘secondfile’ that are*

    *removed from your working tree are ignored with this version of Git.*

    ** ‘git add — ignore-removal <pathspec>’, which is the current default,*

    *ignores paths you removed from your working tree.*

    ** ‘git add — all <pathspec>’ will let you also record the removals.*

    *Run ‘git status’ to check the paths you removed from your working tree.*

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
