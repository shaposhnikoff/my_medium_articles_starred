
# Handling Zombie Processes in Linux

Learn what a zombie process in the Linux system is, how it arises and how to eliminate them.

A Linux process is an instance of a program that is executing in the OS with all the required resources (CPU, memory, etc). Every process has an associated number called Process Identifier (PID) that uniquely identifies the process in the Linux’s processes table.

We say that a process becomes a zombie when it has completed its task, but the parent process hasn’t collected its execution status. This process is already dead, so most of its resources (CPU, memory, etc) have already been released and it’s just waiting to be reaped by its parent.

The critical issue related to a zombie is its occupation of a PID, so preventing the OS from assigning this PID to other processes. Few zombies normally aren’t harmful, but as long as the number grows rapidly, it may surpass the maximum allowed number of PIDs and thus, we incur the risk of being out of available PIDs. Therefore, preventing the OS from starting other processes.

### How to “Kill” a Zombie Process?

Actually: **you can’t**, because the zombie is already dead! It’s just waiting to be reaped (give back its PID), the problem lies in its parent that hasn’t collected its child’s execution status. Hence, the issue is: How can we enforce that the parent collects its child execution status?

There are basically two options:

1. Send a signal of type SIGCHLD to the parent process asking it to do so.
2. Kill the parent process.

The option 1 is a request and it may or may not be honored, depending on the implementation of the program.

Whereas the option 2 is more severe, but also more assured to work. But how exactly it works? When you kill the parent process, its child becomes an orphan process that is adopted by the *init* process (PID = 1).

The crucial point is: *init* periodically collects the execution status from its children, hence the zombie can finally “rest in peace” and give away their PID to the processes table.

### Example

The following example was compiled under Linux 4.4.0–104-generic and GCC 5.4.0, and to more easily distribute the example, I’ll use a [Docker container](https://medium.com/@varago.rafael/a-quick-introduction-to-docker-47fb914e3105).

It consists of a program that will create a child process that will exit, whereas the parent process runs in an infinite loop and doesn’t collect its child’s execution status and hence let it becomes a zombie.

Thus, we can test option 2 that will consist in killing the parent process and wait for the *init* process to adopt the, now, orphan child process and then collecting its status.

Traditionally, a process creates its child by calling the syscall *fork* that will duplicate its memory content into two separated memory spaces and, besides other things, the child will have its own PID and a Parent PID (PPID) that will be equal the PID of its parent process.

The following C code does what we need (**note**: it’s just a toy program that hasn’t any real utility and just aims to illustrate our main goal, and you can download the full example [here](https://github.com/rvarago/handling-zombie-processes-linux)):

<iframe src="https://medium.com/media/e898dab8fb58e89ba5ca6a1a8fa91df7" frameborder=0></iframe>

The *fork* syscall creates a child process and both (parent and child) processes will execute the line following *fork*. To distinguish between the parent and child you can check the *fork* return value:

* < 0: failure

* == 0: child process

* > 0: parent process (returns the child’s PID)

First, let’s inspect the absence of zombie processes by issuing the *top* command:

![](https://cdn-images-1.medium.com/max/2000/1*avVbad8dGjBUB4dd9VSNuQ.png)

Now, compile and run the above program:

    gcc -o zombie-test create-zombie.c && ./zombie-test &

This output:

![](https://cdn-images-1.medium.com/max/2000/1*LblZ6vU_l8akwLec3keoqQ.png)

And it can be analyzed by the *ps* utility:

    ps aux | grep test-zombie

That outputs:

![](https://cdn-images-1.medium.com/max/2000/1*yo-Dn_nu6XT_-28MK_vbOg.png)

The output shows that there are two processes that match the searching pattern, note that the child process (PID = 14) has a *Z* status and it’s also marked as *defunct*, which means that the child has become a zombie.

You can double check the fact that this process has become a zombie by running *top* again:

![](https://cdn-images-1.medium.com/max/2000/1*e6kNuVNPwre9DX0d_u9ulg.png)

Looking at the zombie’s stat you can see that you have 1 zombie here.

Try to kill the child process by (SIGTERM is the default signal, thus it can be omitted):

    kill <CPID>

Where <CPID> is the child’s PID (14 in my case).

You’ll note that nothing has happened (you can check by running *ps* again).

To send a SIGCHLD to the parent process:

    kill -SIGCHLD <PPID>

Where *<PPID>* is the parent’s PID (13 in my case).

Note again that the zombie persists, it’s because we’re not treating the SIGCHLD signal.

Therefore the way to reap the child is to kill its parent by:

    kill <PPID>

Now, by killing the parent process, as we’ve said, the *init* process will adopt the orphaned child process, periodically collect its execution status and hence reap the child that give back its PID to the Linux process table.

Running *top* again and you’ll see that the zombie has been reaped (zombie == 0):

![](https://cdn-images-1.medium.com/max/2000/1*MDOGKPeo6laqZpK0K1MSGA.png)

Looking at the code, specifically at the comment (1), the proper way to avoid a zombie is to uncomment the call to the* wait* syscall at the process so it can collect the child’s execution status.

The parameter that *wait* expects is a pointer to an integer that will be filled with the child’s return value to the parent that can analyze it and take the relevant actions. In our example we’re simply passing *NULL*, so we’re ignoring the child’s return value.

By this explanation, we know more closely what *init* is doing when adopting the zombie orphans: it’s continuously calling *wait *on its children.

### Conclusion

I hope that with this article you can have a better understanding about what a zombie process in Linux is, its origin, and how to get away from them.

Although it’s not the main point, I’ve also introduced a very brief example of multiprocessing programming for Linux using the *glibc* and the C programming language by the means of an example using the *fork* syscall.

I really encourage you to search more about how and when to write concurrent/parallel programs (by using processes or threads), its pros and cons, and how to use the C API for Linux development.

### References

[1] [https://kerneltalks.com/howto/everything-need-know-zombie-process/](https://kerneltalks.com/howto/everything-need-know-zombie-process/)

[2] [http://man7.org/linux/man-pages/man2/fork.2.html](http://man7.org/linux/man-pages/man2/fork.2.html)

[3] [http://man7.org/linux/man-pages/man2/waitpid.2.html](http://man7.org/linux/man-pages/man2/waitpid.2.html)

[4] [http://man7.org/linux/man-pages/man2/syscalls.2.html](http://man7.org/linux/man-pages/man2/syscalls.2.html)
