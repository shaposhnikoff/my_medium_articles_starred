
# 100 Days of DevOps — Day 76-How Linux Kernel is organized

Welcome to Day 76 of 100 Days of DevOps, Focus for today is How Linux Kernel is organized

*In Linux we have*

* ***User Space**: User(Processes/Application/Services) need to do something and for that, a typical interface is a **shell***

* ***Kernel Space**: Kernel is the only component that has direct access to **hardware**.*

    *User Space ----> Kernel Space*

    *Signals
                         System Calls*

*If User needs to interact with Kernel there is a limited option, which is provided by the kernel and strictly defined by the kernel what user can do*

* ***Signal***

* ***System Calls***

***System Calls***

* *An essential part of the Linux Operating System*

* *Processes cannot access the kernel directly*

* *System calls are used as an interface for processes to the kernel. **glibc **provides a library interface to use system calls from programs*

* *A common task like **opening, listing, reading, and writing** to files all involves **system calls***

* *The **fork() and exec()** system calls determine how a process start*

* ***fork():** the kernel creates an almost identical copy of the current process and replaces that*

* ***exec():** the kernel starts a program, which replaces the current process*

*There is one more thing involved in this whole process called **libraries** which is just an additional code used by either shell or process to add more functionalities. For eg: the most important one is **glibc** which provides functions and system calls*

    # ldd $(which passwd)

    linux-vdso.so.1 => (0x00007ffe9fff4000)

    libuser.so.1 => /lib64/libuser.so.1 (0x00007f4074149000)

    libgobject-2.0.so.0 => /lib64/libgobject-2.0.so.0 (0x00007f4073ef9000)

    libglib-2.0.so.0 => /lib64/libglib-2.0.so.0 (0x00007f4073bc1000)

    libpopt.so.0 => /lib64/libpopt.so.0 (0x00007f40739b7000)

    libpam.so.0 => /lib64/libpam.so.0 (0x00007f40737a8000)

    libpam_misc.so.0 => /lib64/libpam_misc.so.0 (0x00007f40735a3000)

    libaudit.so.1 => /lib64/libaudit.so.1 (0x00007f407337b000)

    libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f4073154000)

    libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f4072f37000)

    **libc.so.6 => /lib64/libc.so.6 (0x00007f4072b76000)**

    libgmodule-2.0.so.0 => /lib64/libgmodule-2.0.so.0 (0x00007f4072972000)

    libcrypt.so.1 => /lib64/libcrypt.so.1 (0x00007f407273a000)

    libffi.so.6 => /lib64/libffi.so.6 (0x00007f4072532000)

    libdl.so.2 => /lib64/libdl.so.2 (0x00007f407232e000)

    libcap-ng.so.0 => /lib64/libcap-ng.so.0 (0x00007f4072127000)

    libpcre.so.1 => /lib64/libpcre.so.1 (0x00007f4071ec6000)

    /lib64/ld-linux-x86–64.so.2 (0x00007f4074576000)

    libfreebl3.so => /lib64/libfreebl3.so (0x00007f4071cc3000)

*Typically we have two types of libraries*

* ***Static**: eg header files(stdio.h)*

* ***Dynamic**: Stored on disk(eg: libc.so),generally find inside /lib64 or /usr/lib64*

*Generally, the application doesn’t have any idea where to find these libraries, these are helper program **ld.so** which does this task on behalf of an application. **ld.so**programs reads some default directories*

    *# ls -l /etc/ld.so.conf.d/*

    *total 8*

    *-r — r — r — . 1 root root 63 Oct 19 11:29 kernel-3.10.0–514.el7.x86_64.conf*

    *-rw-r — r — . 1 root root 17 Sep 21 08:18 mariadb-x86_64.conf*

So if we have some libraries in some non-standard path we can put inside this directory and then run ldconfig to update the library cache

    # ldconfig -v

*The way Kernel interact with hardware is via **drivers***

    *Kernel → Drivers → Hardware*

*One important thing to note Linux Kernel is **pluggable** i.e drivers are not a part of Kernel and can be plugged when it’s required, another way to define is Linux Kernel is **modular** in nature.*

*Now let’s zoom in more into Kernel*

*The kernel has an interface called **Memory Management** which decides how information is stored/fetch from RAM*

***Scheduling interface** determine which process get CPU attention and which process need to wait*

***Drivers** Determine how Kernel interact with Disk*

    *Kernel --> Memory Management --> RAM*

    *Kernel --> Scheduling --> CPU*

    *Kernel --> Drivers --> Disk*

*Now to check the currently loaded module, run **lsmod** which is just userspace interface for **/proc/modules** and it represents the data in a nice format.*

    *# lsmod*

    *Module Size Used by*

    *iptable_filter 12810 0*

    *isofs 39844 0*

    *intel_powerclamp 14419 0*

    *intel_rapl 19321 0*

    *iosf_mbi 13523 1 intel_rapl*

    *crc32_pclmul 13113 0*

    *ghash_clmulni_intel 13259 0*

    *cirrus 24597 1*

    *ttm 93908 1 cirrus*

    *drm_kms_helper 146456 1 cirrus*

    *ppdev 17671 0*

    *syscopyarea 12529 1 drm_kms_helper*

    *sysfillrect 12701 1 drm_kms_helper*

    *sysimgblt 12640 1 drm_kms_helper*

*Now to get more information about particular module run **modinfo***

    *# modinfo isofs*

    *filename: /lib/modules/3.10.0–514.el7.x86_64/kernel/fs/isofs/isofs.ko*

    *license: GPL*

    *alias: iso9660*

    *alias: fs-iso9660*

    *rhelversion: 7.3*

    *srcversion: 3967035CBA55EF4A7821695*

    *depends:*

    *intree: Y*

    *vermagic: 3.10.0–514.el7.x86_64 SMP mod_unload modversions*

    *signer: Red Hat Enterprise Linux kernel signing key*

    *sig_key: 75:FE:A1:DF:24:5A:CC:D9:7A:17:FE:3A:36:72:61:E6:5F:8A:1E:60*

    *sig_hashalgo: sha256*

***Strace***

*strace — trace system calls and signals*
> # -c — count time, calls, and errors for each syscall and report summary

    *# strace -fc ls*

    *anaconda-ks.cfg original-ks.cfg*

    *% time seconds usecs/call calls errors syscall*

    *— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —*

    *34.48 0.000060 2 28 mmap*

    *22.99 0.000040 4 11 open*

    *19.54 0.000034 2 18 mprotect*

    *8.62 0.000015 2 10 read*

    *5.17 0.000009 1 14 close*

    *5.17 0.000009 1 12 fstat*

    *2.30 0.000004 2 2 1 access*

    *0.57 0.000001 0 3 brk*

    *0.57 0.000001 1 1 execve*

    *0.57 0.000001 1 1 arch_prctl*

    *0.00 0.000000 0 1 write*

    *0.00 0.000000 0 1 1 stat*

    *0.00 0.000000 0 3 munmap*

    *0.00 0.000000 0 2 rt_sigaction*

    *0.00 0.000000 0 1 rt_sigprocmask*

    *0.00 0.000000 0 2 ioctl*

    *0.00 0.000000 0 2 getdents*

    *0.00 0.000000 0 1 getrlimit*

    *0.00 0.000000 0 2 2 statfs*

    *0.00 0.000000 0 1 set_tid_address*

    *0.00 0.000000 0 1 openat*

    *0.00 0.000000 0 1 set_robust_list*

    *— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —*

    *100.00 0.000174 118 4 total*

*To check for a specific system call*
> # -e expr — a qualifying expression: option=[!]all or option=[!]val1[,val2]…

    *# strace -e open ls*

    *open(“/etc/ld.so.cache”, O_RDONLY|O_CLOEXEC) = 3*

    *open(“/lib64/libselinux.so.1”, O_RDONLY|O_CLOEXEC) = 3*

    *open(“/lib64/libcap.so.2”, O_RDONLY|O_CLOEXEC) = 3*

    *open(“/lib64/libacl.so.1”, O_RDONLY|O_CLOEXEC) = 3*

    *open(“/lib64/libc.so.6”, O_RDONLY|O_CLOEXEC) = 3*

    *open(“/lib64/libpcre.so.1”, O_RDONLY|O_CLOEXEC) = 3*

    *open(“/lib64/libdl.so.2”, O_RDONLY|O_CLOEXEC) = 3*

    *open(“/lib64/libattr.so.1”, O_RDONLY|O_CLOEXEC) = 3*

    *open(“/lib64/libpthread.so.0”, O_RDONLY|O_CLOEXEC) = 3*

    *open(“/proc/filesystems”, O_RDONLY) = 3*

    *open(“/usr/lib/locale/locale-archive”, O_RDONLY|O_CLOEXEC) = 3*

    *anaconda-ks.cfg original-ks.cfg*

    *+++ exited with 0 +++*

*To check for library call use ltrace*

***ltrace — A library call tracer***

* *c count time and calls, and report a summary on exit.*

* *-f trace children (fork() and clone()).*

    *# ltrace -fc ls*

    *anaconda-ks.cfg original-ks.cfg*

    *% time seconds usecs/call calls function*

    *— — — — — — — — — — — — — — — — — — — — — — — — — — — — — —*

    *50.80 0.006370 6370 1 __libc_start_main*

    *6.91 0.000866 866 1 exit*

    *6.87 0.000861 50 17 readdir*

    *4.70 0.000589 49 12 __ctype_get_mb_cur_max*

    *4.43 0.000555 55 10 __errno_location*

    *3.55 0.000445 49 9 malloc*

    *3.28 0.000411 51 8 getenv*

    *1.94 0.000243 48 5 memcpy*

    *1.78 0.000223 55 4 free*

    *1.54 0.000193 48 4 __freading*

    *1.51 0.000189 63 3 __overflow*

    *1.24 0.000156 52 3 strlen*

    *1.14 0.000143 71 2 fclose*

    *0.99 0.000124 124 1 setlocale*

    *0.90 0.000113 56 2 fwrite_unlocked*

    *0.79 0.000099 49 2 __fpending*

    *0.78 0.000098 49 2 fileno*

    *0.76 0.000095 47 2 fflush*

    *0.60 0.000075 75 1 closedir*

    *0.57 0.000072 72 1 opendir*

    *0.57 0.000072 72 1 ioctl*

    *0.51 0.000064 64 1 exit_group*

    *0.47 0.000059 59 1 isatty*

    *0.45 0.000056 56 1 strrchr*

    *0.45 0.000056 56 1 bindtextdomain*

    *0.44 0.000055 55 1 __cxa_atexit*

    *0.42 0.000053 53 1 getopt_long*

    *0.42 0.000053 53 1 strcoll*

    *0.41 0.000052 52 1 textdomain*

    *0.40 0.000050 50 1 realloc*

    *0.39 0.000049 49 1 _setjmp*

    *— — — — — — — — — — — — — — — — — — — — — — — — — — — — — —*

    *100.00 0.012539 101 total*

***Signals***

*Signals provide software interrupt, it’s a method to tell a process that it has to do something*

    *# kill -l*

    *1) SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL 5) SIGTRAP*

    *6) SIGABRT 7) SIGBUS 8) SIGFPE 9) SIGKILL 10) SIGUSR1*

    *11) SIGSEGV 12) SIGUSR2 13) SIGPIPE 14) SIGALRM 15) SIGTERM*

    *16) SIGSTKFLT 17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP*

    *21) SIGTTIN 22) SIGTTOU 23) SIGURG 24) SIGXCPU 25) SIGXFSZ*

    *26) SIGVTALRM 27) SIGPROF 28) SIGWINCH 29) SIGIO 30) SIGPWR*

    *31) SIGSYS 34) SIGRTMIN 35) SIGRTMIN+1 36) SIGRTMIN+2 37) SIGRTMIN+3*

    *38) SIGRTMIN+4 39) SIGRTMIN+5 40) SIGRTMIN+6 41) SIGRTMIN+7 42) SIGRTMIN+8*

    *43) SIGRTMIN+9 44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13*

    *48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12*

    *53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9 56) SIGRTMAX-8 57) SIGRTMAX-7*

    *58) SIGRTMAX-6 59) SIGRTMAX-5 60) SIGRTMAX-4 61) SIGRTMAX-3 62) SIGRTMAX-2*

    *63) SIGRTMAX-1 64) SIGRTMAX*

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
[**100 Days of DevOps — Day 75- Introduction to Fabric**
*Welcome to Day 75 of 100 Days of DevOps, Focus for today is Introduction to Fabric*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-75-introduction-to-fabric-2e80f5c3148f)
