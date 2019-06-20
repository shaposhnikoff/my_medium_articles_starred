
# Introduction to GIT — Part 1

As per official website
> Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.
[**Git**
*Git is easy to learn and has a tiny footprint with lightning fast performance. It outclasses SCM tools like Subversion…*git-scm.com](https://git-scm.com/)

***Installation***

[https://git-scm.com/downloads](https://git-scm.com/downloads)

***MAC***

    *$ brew install git*

***Linux***

    *yum -y install git*

*Once it’s installed to check the version of git*

    *$ git --version*

    *git version 2.17.1*

***Creating a first GIT repository***

*There are two ways you can start with any GIT project*

* *Create it from scratch*

* *Clone an existing repository*

*Let’s look at the first approach, create it from scratch*

    *$ mkdir test_git
    $ cd test_git/
    $ echo "testing git" >> test.txt*

    *#To initialize a git repository*

    *$ git init .*

    *Initialized empty Git repository in /Users/plakhera/test_git/.git/*

*If we look at the content test_git, there is one hidden directory .git which signifies that our directory is a git repository*

    *$ ls -la*

    *total 8*

    *drwxr-xr-x 4 plakhera wheel 128 Jun 12 08:32 .*

    *drwxr-xr-x 294 plakhera wheel 9408 Jun 12 08:31 ..*

    *drwxr-xr-x 9 plakhera wheel 288 Jun 12 08:32 .git*

    *-rw-r — r — 1 plakhera wheel 12 Jun 12 08:31 test.txt*

***NOTE:** If we try to compare between Git and CVS/SVN, CVS and SVN places revision information in CVS and .svn subdirectories with-in each of project directories. In comparison to that GIT places all the information in one top level directory .git.*

***Adding file to a repository***

*We have created an empty git repository, now to manage it’s contents we must need to explicitly deposit a file into the repository and to do that we need to use git add*

    *$ git add test.txt*

*This step is also called Staging a file which is an interim step before final commit. This is often confusing to beginners but we see the benefits of this later as GIT try to separate add and commit steps to avoid volatility.At this point of time we have options to put the code back to the working directory or push it forward to repo*

*At this stage if we check the status of a file, we will see something like this, which is the in-between state of test.txt*

    *$ git status*

    *On branch master*

    *No commits yet*

    *Changes to be committed:*

    *(use "git rm --cached <file>..." to unstage)*

    *new file:   test.txt*

*To commit this file to the repository*

    *$ git commit -m "initial commit to the repository"*

    *[master (root-commit) c9a57f9] initial commit to the repository*

    *1 file changed, 1 insertion(+)*

    *create mode 100644 test.txt*

*Once you are done with the commit, you will see the message like this*

    *$ git status*

    *On branch master*

    *nothing to commit, working tree clean*

**GIT three stage thinking**

![](https://cdn-images-1.medium.com/max/2000/1*k8fuw0YYi7q98lUId2Iyfg.png)

Ref: [https://git-scm.com/book/en/v2/Getting-Started-Git-Basics](https://git-scm.com/book/en/v2/Getting-Started-Git-Basics)

*Before start committing data to any git repository, we just need to set some basic environment configuration and options, at the bare minimum we must need to tell git about username and email address.*

    *$ **git config --global user.name "Prashant Lakhera"**
    $ **git config --global user.email "plakhera@gmail.com"***

*We can also tell GIT about username and email using environment variables*

    *export GIT_AUTHOR_NAME = “Prashant Lakhera”*

    *export GIT_AUTHOR_EMAIL = “plakhera@gmail.com”*

*Once these environment variable set these override any other configurations*

*Similar manner we can set our favorite editor*

    ***export GIT_EDITOR=vim***

    *OR*

    ***$ git config --global core.editor "vim"***

***To list the settings of all variables***

    *$ git config -l*

    *user.name=Prashant Lakhera*

    *user.email=plakhera@gmail.com*

    *core.editor=vim*

*All this information is stored inside simple text file*

    *$ cat .git/config*

    *[core]*

    *repositoryformatversion = 0*

    *filemode = true*

    *bare = false*

    *logallrefupdates = true*

    *ignorecase = true*

    *precomposeunicode = true*

    *[remote "origin"]*

    *url = /Users/plakhera/test_git*

    *fetch = +refs/heads/*:refs/remotes/origin/**

    *[branch "master"]*

    *remote = origin*

    *merge = refs/heads/master*

*If you want to remove any of settings*

    ***git config --unset --global user.email***

*Let’s try to make another commit*

    *$ echo "some more changes to test file" >> test.txt*

    *#Check git status now*

    *$ git status*

    *On branch master*

    *Changes not staged for commit:*

    *(use "git add <file>..." to update what will be committed)*

    *(use "git checkout -- <file>..." to discard changes in working directory)*

    *modified:   test.txt*

    *no changes added to commit (use "git add" and/or "git commit -a")*

*Again add and commit your changes*

    *$ git add .
    $ git commit -m "adding new changes to the git repo"*

    *[master 72db21f] adding new changes to the git repo*

    *1 file changed, 1 insertion(+)*

***Viewing your commits***

*You can use git log to show the sequence of individual commits*

    *$ git log*

    *commit 72db21fe9915460de47f73385221471970257544 (**HEAD -> master**)*

    *Author: Prashant Lakhera <plakhera@gmail.com>*

    *Date:   Tue Jun 12 09:02:00 2018 -0700*

    *adding new changes to git repo*

    *commit c9a57f99dbe04b0a5405e9f412863197b4594461*

    *Author: Prashant Lakhera <plakhera@gmail.com>*

    *Date:   Tue Jun 12 08:49:10 2018 -0700*

    *initial commit to the repository*

*OR*

*If you are looking for precise log output*

    *$ git log --oneline*

    *72db21f (**HEAD -> master**) adding new changes to git repo*

    *c9a57f9 initial commit to the repository*

*To see more detailed information about the particular commit*

    *$ git show 72db21f*

    *commit 72db21fe9915460de47f73385221471970257544 (**HEAD -> master**)*

    *Author: Prashant Lakhera <plakhera@gmail.com>*

    *Date:   Tue Jun 12 09:02:00 2018 -0700*

    *adding new changes to git repo*

    ***diff --git a/test.txt b/test.txt***

    ***index c68bd6a..1dd6298 100644***

    ***--- a/test.txt***

    ***+++ b/test.txt***

    *@@ -1 +1,2 @@*

    *testing git*

    *+some more changes to test file*

*If you want to see difference between two revisions run git diff*

    *$ git diff c9a57f9 72db21f*

    ***diff --git a/test.txt b/test.txt***

    ***index c68bd6a..1dd6298 100644***

    ***--- a/test.txt***

    ***+++ b/test.txt***

    *@@ -1 +1,2 @@*

    *testing git*

    *+some more changes to test file*

*Output look similar to the output of Linux diff command.*

*To remove a file using git, use git rm command*

    *$ git rm test1.txt*

    *rm 'test1.txt'*

    *$ git status*

    *On branch master*

    *Changes to be committed:*

    *(use "git reset HEAD <file>..." to unstage)*

    *deleted:    test1.txt*

    *$ git commit -m "Removing test1 as it's no longer required"*

    *[master 5105192] Removing test1 as it's no longer required*

    *1 file changed, 0 insertions(+), 0 deletions(-)*

    *delete mode 100644 test1.txt*

* *Removing a file from the repository is similar to adding a file except it uses git rm*

* *It’s a two step process, in the first step we are expressing our intent to remove a file(git rm) and stage the changes*

* *git commit pushing that change to the repository*

*Similar to remove we have move command(git mv)*

    *$ git mv test.txt test1.txt*

    *$ git status*

    *On branch master*

    *Changes to be committed:*

    *(use "git reset HEAD <file>..." to unstage)*

    *renamed:    test.txt -> test1.txt*

    *$ git commit -m "renaming test to test1"*

    *[master 53cc019] renaming test to test1*

    *1 file changed, 0 insertions(+), 0 deletions(-)*

    *rename test.txt => test1.txt (100%)*

***Creating Copy of Repository***

* *We can create complete copy or clone of the repository using git clone command*

    *$ git clone test_git test1_git*

    *Cloning into 'test1_git'...*

    *done.*

*git clone operation is similar to cp or rsync command. Although these two repositories are exactly similar(complete repository with full history)but there are some differences which we can check with the help of*

    *diff -r test_git test1_git*

***GIT Alias***

*Alias is really helpful if we type any git command frequently*

    *$ git config --global alias.st "status"*

*Here rather than typing the full git status command, now I only need to type git st*

    *$ git st*

    *On branch master*

    *Your branch is up to date with 'origin/master'.*

    *nothing to commit, working tree clean*
