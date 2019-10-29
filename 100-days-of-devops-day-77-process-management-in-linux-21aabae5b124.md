Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }

# 100 Days of DevOps — Day 77-Process Management in Linux

Welcome to Day 77 of 100 Days of DevOps, Focus for today is Process Management in Linux

***What is a process?***

*A dynamically executing instance of a program. The process uses any resources that the Linux Kernel can handle to complete its tasks.*

***Lifecycle of Process***

*Now, let's look at the life cycle of process*

***fork() : **When a process creates a new process, Parent process issues a fork() system call. When the child is created, it receives a PID using the getpid() system call.*

***exec():** It copies the new program to the address space of the child process. In the case of exec Kernel assigns the new physical page to the child process. While in case of fork() entire address space is not copied and both processes share the same address space*

***Copy on write(COW):** COW is a kind of deferred operation which avoids unnecessary overhead because copying the entire address space is a very slow and inefficient operation which uses a lot of processor time and resources*

***exit(): **When program execution has completed the child process terminates with the exit() system call. The exit() system call releases most of the data structure of the process and notifies the parent process of the termination sending a signal. At this time, the process is called a **zombie process***

***wait():** Child process will not be completely removed until the parent process knows of the termination of its child process by the wait() system call. As soon as the parent process is notified of the child process termination, it removes all the data structure of the child process and releases the process descriptor.*

***Threads(LWP):** Threads also called Light Weight Process is an execution unit generated in a single process. It runs parallel with other threads in the same process. Threads can share the same resources such as memory, address space, open files and so on. From a performance point of view, thread creation is less expensive as compared to process creation because they don’t need to copy the resources on creation but on the other hand as they share the same resource, it’s the responsibility of application owner to implement mutual exclusion as well locking.*

***Context Switching: **When any process is executing all the information related to the given process stored in a register on the processor and its cache and the data that is loaded into the register for the executing process is called the context. Every time CPU switches to new process the cache need to be rebuilt and that slow down the entire operation, that why it’s always a good idea to pin processes to specific CPU. To check Context Switching*

    *# pidstat -w*

    *Linux 3.10.0–514.el7.x86_64 (ip-172–31–33–6.us-west-2.compute.internal) 03/20/2017 _x86_64_ (1 CPU)*

    *11:13:14 PM UID PID cswch/s nvcswch/s Command*

    *11:13:14 PM 0 1 0.13 0.00 systemd*

    *11:13:14 PM 0 2 0.00 0.00 kthreadd*

    *11:13:14 PM 0 3 0.05 0.00 ksoftirqd/0*

    *11:13:14 PM 0 7 0.00 0.00 migration/0*

***cswch/s***

*Total number of voluntary context switches the task made per second. A voluntary context switch occurs when a task blocks because it requires a resource that is unavailable.*

***nvcswch/s***

*A Total number of non voluntary context switches the task made per second. A involuntary context switch takes place when a task executes for the duration of its time slice and then is forced to relinquish the processor.*

*OR using sar*

    *# sar -w 1 3*

    *03/20/2017 _x86_64_ (1 CPU)*

    *11:15:13 PM proc/s **cswch/s***

    *11:15:14 PM 1.00 105.00*

    *11:15:15 PM 0.00 129.29*

    *11:15:16 PM 0.00 98.00*

    *Average: 0.33 110.70*

*-w Report task creation and system switching activity.*

*To pin process to particular CPU use **taskset** command*

*where*

*-p, — pid operate on existing given pid*

* *c, — cpu-list display and specify cpus in list format*

*Now lets take an example*

*I ran this command*

    *# dd if=/dev/zero of=/dev/null &*

    *[1] 10704*

*Now to find out in which **processor** this process is running*

    *# ps -aF*

    *UID  PID   PPID  C   SZ  RSS PSR STIME TTY TIME CMD*

    *root 10704 10670 99 26294 660 2 03:20 pts/0 00:01:12 dd if=/dev/zero of=/dev/null*

*Based on psr value it shows its running on processor 2*

*Now to bind this dd command to processor 3*

    *# taskset -pc 3 $(pidof dd)*

    *pid 10704’s current affinity list: 0–3*

    *pid 10704’s new affinity list: 3*

*Now let’s try to perform some test in this case I am going to disable CPU 3 and let see how all the process migrate to different CPU*

*Go to*

    *# pwd*

    */sys/devices/system/cpu/cpu3*

    *# echo 0 > online*

    *#Run the ps command again(**As you can see process is now migrated to CPU1**)
    # ps -aF |grep -i dd*

    *root     10704 10670 99 26294   660   1 03:20 pts/0    00:11:22 dd if=/dev/zero of=/dev/null*

***PROCESS** **STATE** **CODES***

*Here are the different values that the **s**, **stat** and **state** output*

    *D uninterruptible sleep (usually IO)*

    *R running or runnable (on run queue)*

    *S interruptible sleep (waiting for an event to complete)*

    *T stopped by job control signal*

    *t stopped by debugger during the tracing*

    *W paging (not valid since the 2.6.xx kernel)*

    *X dead (should never be seen)*

    *Z defunct (“zombie”) process, terminated but not reaped by*

    *its parent*

*How to check zombie process*

    *ps aux | grep 'Z'*

***Removing Zombie Process***

* *Zombie don’t use any system resources*

* *But they use PID and the max number of PID available is limited*

*To check the maximum number of process in Linux*

    *# cat /proc/sys/kernel/pid_max*

    *32768*

*OR*

    *# sysctl -a |grep -i pid_max*

    *kernel.**pid_max** = 32768*

*To kill a zombie process*

    *kill -s SIGCHLD <ppid>*

*If it doesn’t help then last resort is to reboot the system*

***Monitoring process via /proc***

*Proc is a special virtual filesystem which holds information about any currently running process.*

    *# ls*

    *1 1342 15520 19 221 246 253 30402 30678 32 42 461 728 bus diskstats interrupts key-users meminfo partitions stat tty*

    *10 1343 15521 2 223 247 254 30404 30691 335 43 466 749 cgroups dma iomem kmsg misc sched_debug swaps uptime*

    *1073 1344 15522 20 226 248 255 30405 30693 357 446 471 773 cmdline driver ioports kpagecount modules schedstat sys version*

    *1080 13565 15523 21 228 249 28 30424 30715 377 448 496 8 consoles execdomains irq kpageflags mounts scsi sysrq-trigger vmallocinfo*

    *12 14 16 217 23 250 29 30425 30742 40 45 64 9 cpuinfo fb kallsyms loadavg mtrr self sysvipc vmstat*

    *13 15 1615 22 24092 251 3 30467 30744 41 452 7 acpi crypto filesystems kcore locks net slabinfo timer_list xen*

    *131 15519 18 220 24171 252 30 30543 31 411 456 727 buddyinfo devices fs keys mdstat pagetypeinfo softirqs timer_stats zoneinfo*

*For eg: To check complete command line for the process*

    *# cat /proc/1/cmdline*

    */usr/lib/systemd/systemd — switched-root — system — deserialize20*

*To check the files open by a given process*

    *# pwd*

    */proc/1/fd*

    *[root@ip-172–31–33–6 fd]# \ls*

    *0 1 10 11 12 14 15 16 17 18 19 2 20 25 26 27 28 29 3 30 31 32 33 34 35 36 4 5 6 7 8 9*

*To check memory mapping of this process*

    *# cat /proc/1/maps*

    *# cat /proc/1/smaps*

*Checking Out of Memory Score*

    *# cat /proc/1/oom_score*

    *0*

*To check information about given process*

    *# cat /proc/1/status*

    *Name: systemd*

    *State: S (sleeping)*

    *Tgid: 1*

    *Ngid: 0*

    *Pid: 1*

    *PPid: 0*

    *TracerPid: 0*

    *Uid: 0 0 0 0*

    *Gid: 0 0 0 0*

    *FDSize: 64*

    *Groups:*

    *VmPeak: 190476 kB*

    *VmSize: 125044 kB*

    *VmLck: 0 kB*

    *VmPin: 0 kB*

    *VmHWM: 3540 kB*

    *VmRSS: 3540 kB*

    *RssAnon: 1152 kB*

    *RssFile: 2388 kB*

    *RssShmem: 0 kB*

    *VmData: 82852 kB*

    *VmStk: 136 kB*

    *VmExe: 1296 kB*

    *VmLib: 3636 kB*

    *VmPTE: 104 kB*

    *VmSwap: 0 kB*

    *Threads: 1*

    *SigQ: 0/3814*

    *SigPnd: 0000000000000000*

    *ShdPnd: 0000000000000000*

    *SigBlk: 7be3c0fe28014a03*

    *SigIgn: 0000000000001000*

    *SigCgt: 00000001800004ec*

    *CapInh: 0000000000000000*

    *CapPrm: 0000001fffffffff*

    *CapEff: 0000001fffffffff*

    *CapBnd: 0000001fffffffff*

    *CapAmb: 0000000000000000*

    *Seccomp: 0*

    *Cpus_allowed: 7fff*

    *Cpus_allowed_list: 0–14*

    *Mems_allowed: 00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001*

    *Mems_allowed_list: 0*

    *voluntary_ctxt_switches: 472048*

    *nonvoluntary_ctxt_switches: 4014*

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
[**100 Days of DevOps — Day 76-How Linux Kernel is organized**
*Welcome to Day 76 of 100 Days of DevOps, Focus for today is How Linux Kernel is organized*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-76-how-linux-kernel-is-organized-257bafbc31fc)
