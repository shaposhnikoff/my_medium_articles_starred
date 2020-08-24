Unknown markup type 10 { type: [33m10[39m, start: [33m62[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m259[39m, end: [33m265[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m317[39m, end: [33m324[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m56[39m, end: [33m62[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m169[39m, end: [33m175[39m }

# Operating System — How does zombie process happen and how to prevent it

short comment about zombie process management

## Introduction

A process created in Unix system must use the kernel function fork() , and the address space is cloned from parent process. In Unix system , the parent process is responsible for reap the child process status and memory stack. Then , the parent process calls wait() waiting the child process to terminate and reply a SIGCHLD signal , once parent process receive this signal it starts to reap the child process. So , there is a problem that if the parent process decide not to wait the termination of the child , no one is responsible for the reap and there is one zombie process in system.

## Normal scenario

![normal scenario](https://cdn-images-1.medium.com/max/2000/1*dCTcCl8Nc0ekDxkx_izMfQ.png)*normal scenario*

Parent process calls wait() waiting the child process return a SIGCHLD signal and begin reap.

## Zombie process scenario

![zombie process scenario](https://cdn-images-1.medium.com/max/2000/1*tfiuO55nP6YZ9DaEpnvc1Q.png)*zombie process scenario*

Parent process keep doing its own thing instead calling wait() , so it would not know when the child process terminate. And the reap would never happen causing an zombie process left in the system.

## How to prevent it

![](https://cdn-images-1.medium.com/max/2000/1*MoLt-onIxuXjwgCCjHDreg.png)

In Unix , when the parent process terminate , the init process (created by Unix Kernel) would take care all of its child process. Based on this rule , All we need is to fork() twice makes the init process take care the child process , and the father process would be reaped immediately. No more zombie process would be found in the system.

## There are some solution about preventing zombie process…

You may check here
[**Zombie Processes & Prevention**
*When a process is created in UNIX using fork() system call, the address space of the Parent process is replicated. If…*discuss.leetcode.com](https://discuss.leetcode.com/topic/91261/zombie-processes-prevention)

## Reference

I lost the reference about these pictures , but these are created by someone. Thanks a lot for the kindness. And if you are the author about the pictures please contact me and I will append the reference here.
