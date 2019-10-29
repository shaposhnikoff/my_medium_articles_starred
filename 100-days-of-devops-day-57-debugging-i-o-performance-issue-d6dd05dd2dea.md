
# 100 Days of DevOps — Day 57-Debugging I/O Performance Issue

Welcome to Day 57 of 100 Days of DevOps, Focus for today is Debugging I/O Performance Issue

*The disk is probably slowest among all the component(CPU/Memory) and causes the performance bottleneck. There are various opensource tools available to narrow down the issue*
> *ps command*

*ps command is primarily used for CPU/Memory stats because it doesn’t have a statistic for disk I/O. While it doesn’t tell you info about disk I/O it does show the process state which can be used to indicate whether or not the process is waiting for I/O*

    *PROCESS STATE CODES*

    *D Uninterruptible sleep (usually IO)*

    *R Running or runnable (on run queue)*

    *S Interruptible sleep (waiting for an event to complete)*

    *T Stopped, either by a job control signal or because it is being*

    *traced.*

    *W paging (not valid since the 2.6.xx kernel)*

    *X dead (should never be seen)*

    *Z Defunct (“zombie”) process, terminated but not reaped by its*

    *parent.*

* *Process waiting for I/O are in D state or Uninterruptible sleep*

    *ps auwx | awk ‘$8 ~ “D”’*

*So any process while waiting for read()/write(), the process will be put up in special kind of sleep known as D state or Disk Sleep.*

*These process cannot be killed as we can’t send a signal to these process(SIGTERM/SIGKILL). The signal can only be delivered to process in run queue and these processes are still in I/O queue*
> *vmstat command*

    *# vmstat 1 3
    procs — — — — — -memory — — — — — — -swap — — — -io — — — system — — — -cpu — — -*

    *r b swpd free buff cache si so bi bo in cs us sy id wa st*

    *0 0 0 72112 3176 1129872 0 0 187 9967 2078 1340 11 19 66 4 0*

    *1 0 0 72484 3176 1129688 0 0 0 0 2772 1765 17 26 57 0 0*

    *0 0 0 72484 3184 1129612 0 0 0 20 2771 1795 18 25 56 0 0*

* *So important column to look at*

    *IO*

    *bi: Blocks received from a block device (blocks/s).*

    *bo: Blocks sent to a block device (blocks/s).*

* *Please always ignore the first line of the report which contains the average value since the last time the computer was rebooted.*

*To test it we can run a simple test*

    *# dd if=CentOS-6.5-x86_64-bin-DVD1.iso of=/dev/null bs=1M*

    *# vmstat 1 100
    procs — — — — — -memory — — — — — — -swap — — — -io — — — system — — — -cpu — — -*

    *r b swpd free buff cache si so bi bo in cs us sy id wa st*

    *0 0 0 72676 3128 1117412 0 0 2777 5342 1136 753 5 9 82 3 0*

    *1 1 0 64608 3128 1124436 0 0 54916 0 678 1167 0 1 85 14 0*

    *2 1 0 68824 3128 1120224 0 0 272272 0 3320 5758 0 8 0 92 0*

    *2 1 0 65848 3128 1123296 0 0 267744 0 3260 5666 0 9 0 91 0*

    *1 1 0 61632 3128 1127424 0 0 268856 0 3287 5656 0 7 0 93 0*

    *0 1 0 70436 3128 1118688 0 0 267948 0 3268 5607 0 9 0 91 0*

* *Here you can see when dd is reading that big file, block in(bi) shoots up*
> *top*

    *# top
    top — 14:08:18 up 30 min, 3 users, load average: 0.02, 0.04, 0.01*

    *Tasks: 85 total, 1 running, 84 sleeping, 0 stopped, 0 zombie*

    *Cpu(s): 0.0%us, 0.0%sy, 0.0%ni,100.0%id, **0.0%wa**, 0.0%hi, 0.0%si, 0.0%st ← — — — — —*

    *Mem: 1316280k total, 1243852k used, 72428k free, 3152k buffers*

    *Swap: 511992k total, 0k used, 511992k free, 1117396k cached*

* *Here look for a %wait column, higher the number the more the CPU waiting for I/O access.*

    *wa — iowait*

* *Amount of time the CPU has been waiting for I/O to complete*
> *sar*

* *The same I/O wait you can check using sar*

    *# sar -u 1 3*

    *Linux 2.6.32-431.el6.x86_64 (XXXX[.testing.com](http://centos6.testing.com))       04/08/2019      _x86_64_        (1 CPU)*

    *02:10:23 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle*

    *02:10:24 PM     all      0.00      0.00      0.00      1.00      0.00     99.00*

    *02:10:25 PM     all      0.00      0.00      0.00      0.00      0.00    100.00*

    *02:10:26 PM     all      0.00      0.00      0.00      0.00      0.00    100.00*

    *Average:        all      0.00      0.00      0.00      0.33      0.00     99.67*

*For more info about SAR*
[**100 Days of DevOps — Day 56-Debugging Performance Issue using SAR**
*Welcome to Day 56 of 100 Days of DevOps, Focus for today is Debugging Performance Issue using SAR*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-56-debugging-performance-issue-using-sar-fcb61d6dc641)
> *IOSTAT*

* *All the tools we have discussed above have one limitation it’s telling us the system is struggling with I/O issue but doesn’t tell us which disk is a bottleneck. To find this iostat is the tool for us*

    *# iostat -tkx 1 3*

    *Linux 2.6.32-431.el6.x86_64 ([centos6.testing.com](http://centos6.testing.com))       04/08/2019     _x86_64_        (1 CPU)*

    *04/08/2019 02:12:46 PM*

    *avg-cpu:  %user   %nice %system %iowait  %steal   %idle*

    *0.00    0.00    0.00    0.00    0.00  100.00*

    *Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util*

    *sda               0.00     0.00    1.00    0.00     8.00     0.00    16.00     0.00    1.00   1.00   0.10*

*For system point of view please check*

* *rrqm/wrqm → If the read/write request is merging properly.If there is an issue try to change scheduler in a testing environment and see what is best suited according to your application/environment*

*To check the current scheduler*

    *# cat /sys/block/sda/queue/scheduler
    noop anticipatory deadline [cfq]*

* *To change the scheduler*

    *# echo noop > /sys/block/sda/queue/scheduler
    # cat /sys/block/sda/queue/scheduler*

    *[noop] anticipatory deadline cfq*

* *avgqu-sz → Defines the average number of requests within the io scheduler queue plus, the average number of I/O outstanding to storage(lun queue)*

    *# cat /sys/block/sda/queue/nr_requests (io scheduler)
    128*

    *# cat /sys/block/sda/device/queue_depth (driver)
    31*

*So I/O flow diagram is*

    *Scheduler    ----> Driver -----> Storage
    (nr_request)                    (queue_depth)*

* *The driver is a thin layer they don't queue I/O just repackage it in an acceptable form to the HBA/NIC and then passes it to storage*

* *So as long as avgqu-sz value remains below the LUN queue depth, I/O passes quickly from the scheduler to the driver to storage.*

* *Only when the number of io passed to the driver exceeds the device lun queue value is io held within the scheduler and you will start seeing the performance issue*
> *iotop*

* *Even iostat has one limitation we will find out with the help of iostat where the issue lies but the problem we still don't know which process is responsible for that, to find out we have a tool called iotop.*

    *# iotop*

    *Total DISK READ: 0.00 B/s | Total DISK WRITE: 0.00 B/s*

    *TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND*

    *1 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % init*

    *2 be/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [kthreadd]*

    *3 rt/4 root        0.00 B/s    0.00 B/s  0.00 %  0.00 % [migration/0]*

* *The only issue with IOTOP it included from RHEL5.8*

* *(The basic requirement for per-process I/O monitoring is the file /proc/self/io and kernel compiled with CONFIG_TASK_IO_ACCOUNTING parameter enabled on the system)*

* *To get more information about the process*

    *# cat /proc/1/io*

    *rchar: 12313659*

    *wchar: 49424975*

    *syscr: 24261*

    *syscw: 37016*

    *read_bytes: 42787840 ← — — — — -*

    *write_bytes: 159744 ← — — — — -*

    *cancelled_write_bytes: 32768*
> *Systemtap*

* *So iotop will be available after 5.8 so what should we need to do in case of the system running version < 5.8, in that case, we can use systemtap.*

* *SystemTap is a framework which allows easy probing instrumentation of any component inside the kernel.*

    *# stap -v iotop.stp*

    *Process           KB Read      KB Written*

    *crond                 5               0*

    *stapio                 0               0*
> *Blktrace*

* *With the help of blktrace or blocktrace, we can investigate I/O pattern at the block level*

* *Mount the debugfs filesystem*

    *mount -t debugfs nodev /sys/kernel/debug*

    *# blktrace /dev/sda*

* *We need to use control+C to terminate the program, it will create few files like sda.blktrace.X in the current directory*

    *# blkparse sda.blktrace.0*

    *8,0    0      397     3.312975484   219  U   N [jbd2/sda2-8] 1*

    *8,0    0      398     3.312976689   219  D  WS 26024032 + 56 [jbd2/sda2-8]*

    *8,0    0      399     3.313174047     0  C  WS 26024032 + 56 [0]*

    *8,0    0      400     3.313200457   219  A FWFS 26024088 + 8 <- (8,2) 25612440*

    *8,0    0      401     3.313200916   219  Q FWFS 26024088 + 8 [jbd2/sda2-8]*

    *8,0    0      402     3.313201448   219  G FWFS 26024088 + 8 [jbd2/sda2-8]*

    *8,0    0      403     3.313202010   219  P   N [jbd2/sda2-8]*

    *8,0    0      404     3.313202322   219  I FWFS 26024088 + 8 [jbd2/sda2-8]*

    *8,0    0      405     3.313210286   219  U   N [jbd2/sda2-8] 1*

    *8,0    0      406     3.318909890     0  D  WS 26024088 + 8 [swapper]*

    *8,0    0      407     3.319005492     0  C  WS 26024088 + 8 [0]*

    *8,0    0      408     3.319416254     0  C  WS 26024088 [0]*

    *Q  IO handled by request queue code*

    *I  IO inserted onto request queue*

    *D  IO issued to driver*

    *C  IO completion*

    *G  Get request*

    *P  Plug request*

    *U  Unplug request*

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
