
# 100 Days of DevOps — Day 78- Python OS/Subprocess Module

Welcome to Day 78 of 100 Days of DevOps, Focus for today is Python OS/Subprocess Module

*As a DevOps/System Admin everyday, we encounter a task where we need to interact with OS. In this tutorial, I am going to tell about some important Python module which we can use for this purpose*

* *os*

* subprocess

*OS module allows us to interact with the underlying operating system in different ways*

    *#Import the OS module
    **>>> import os***

    *#Let see the methods that we have access to within this module
    **>>> print(dir(os))***

    *[‘CLD_CONTINUED’, ‘CLD_DUMPED’, ‘CLD_EXITED’, ‘CLD_TRAPPED’, ‘DirEntry’, ‘EX_CANTCREAT’, ‘EX_CONFIG’, ‘EX_DATAERR’, ‘EX_IOERR’, ‘EX_NOHOST’, ‘EX_NOINPUT’, ‘EX_NOPERM’, ‘EX_NOUSER’, ‘EX_OK’, ‘EX_OSERR’, ‘EX_OSFILE’, ‘EX_PROTOCOL’, ‘EX_SOFTWARE’, ‘EX_TEMPFAIL’, ‘EX_UNAVAILABLE’, ‘EX_USAGE’, ‘F_LOCK’, ‘F_OK’, ‘F_TEST’, ‘F_TLOCK’, ‘F_ULOCK’, ‘MutableMapping’, ‘NGROUPS_MAX’, ‘O_ACCMODE’, ‘O_APPEND’, ‘O_ASYNC’, ‘O_CREAT’, ‘O_DIRECT’, ‘O_DIRECTORY’, ‘O_DSYNC’, ‘O_EXCL’, ‘O_LARGEFILE’, ‘O_NDELAY’, ‘O_NOATIME’, ‘O_NOCTTY’, ‘O_NOFOLLOW’, ‘O_NONBLOCK’, ‘O_RDONLY’, ‘O_RDWR’, ‘O_RSYNC’, ‘O_SYNC’, ‘O_TRUNC’, ‘O_WRONLY’, ‘POSIX_FADV_DONTNEED’, ‘POSIX_FADV_NOREUSE’, ‘POSIX_FADV_NORMAL’, ‘POSIX_FADV_RANDOM’, ‘POSIX_FADV_SEQUENTIAL’, ‘POSIX_FADV_WILLNEED’, ‘PRIO_PGRP’, ‘PRIO_PROCESS’, ‘PRIO_USER’, ‘P_ALL’, ‘P_NOWAIT’, ‘P_NOWAITO’, ‘P_PGID’, ‘P_PID’, ‘P_WAIT’, ‘PathLike’, ‘RTLD_DEEPBIND’, ‘RTLD_GLOBAL’, ‘RTLD_LAZY’, ‘RTLD_LOCAL’, ‘RTLD_NODELETE’, ‘RTLD_NOLOAD’, ‘RTLD_NOW’, ‘R_OK’, ‘SCHED_BATCH’, ‘SCHED_FIFO’, ‘SCHED_OTHER’, ‘SCHED_RR’, ‘SEEK_CUR’, ‘SEEK_END’, ‘SEEK_SET’, ‘ST_APPEND’, ‘ST_MANDLOCK’, ‘ST_NOATIME’, ‘ST_NODEV’, ‘ST_NODIRATIME’, ‘ST_NOEXEC’, ‘ST_NOSUID’, ‘ST_RDONLY’, ‘ST_SYNCHRONOUS’, ‘ST_WRITE’, ‘TMP_MAX’, ‘WCONTINUED’, ‘WCOREDUMP’, ‘WEXITED’, ‘WEXITSTATUS’, ‘WIFCONTINUED’, ‘WIFEXITED’, ‘WIFSIGNALED’, ‘WIFSTOPPED’, ‘WNOHANG’, ‘WNOWAIT’, ‘WSTOPPED’, ‘WSTOPSIG’, ‘WTERMSIG’, ‘WUNTRACED’, ‘W_OK’, ‘XATTR_CREATE’, ‘XATTR_REPLACE’, ‘XATTR_SIZE_MAX’, ‘X_OK’, ‘_Environ’, ‘__all__’, ‘__builtins__’, ‘__cached__’, ‘__doc__’, ‘__file__’, ‘__loader__’, ‘__name__’, ‘__package__’, ‘__spec__’, ‘_execvpe’, ‘_exists’, ‘_exit’, ‘_fspath’, ‘_fwalk’, ‘_get_exports_list’, ‘_putenv’, ‘_spawnvef’, ‘_unsetenv’, ‘_wrap_close’, ‘abc’, ‘abort’, ‘access’, ‘altsep’, ‘chdir’, ‘chmod’, ‘chown’, ‘chroot’, ‘close’, ‘closerange’, ‘confstr’, ‘confstr_names’, ‘cpu_count’, ‘ctermid’, ‘curdir’, ‘defpath’, ‘device_encoding’, ‘devnull’, ‘dup’, ‘dup2’, ‘environ’, ‘environb’, ‘errno’, ‘error’, ‘execl’, ‘execle’, ‘execlp’, ‘execlpe’, ‘execv’, ‘execve’, ‘execvp’, ‘execvpe’, ‘extsep’, ‘fchdir’, ‘fchmod’, ‘fchown’, ‘fdatasync’, ‘fdopen’, ‘fork’, ‘forkpty’, ‘fpathconf’, ‘fsdecode’, ‘fsencode’, ‘fspath’, ‘fstat’, ‘fstatvfs’, ‘fsync’, ‘ftruncate’, ‘fwalk’, ‘get_blocking’, ‘get_exec_path’, ‘get_inheritable’, ‘get_terminal_size’, ‘getcwd’, ‘getcwdb’, ‘getegid’, ‘getenv’, ‘getenvb’, ‘geteuid’, ‘getgid’, ‘getgrouplist’, ‘getgroups’, ‘getloadavg’, ‘getlogin’, ‘getpgid’, ‘getpgrp’, ‘getpid’, ‘getppid’, ‘getpriority’, ‘getresgid’, ‘getresuid’, ‘getsid’, ‘getuid’, ‘getxattr’, ‘initgroups’, ‘isatty’, ‘kill’, ‘killpg’, ‘lchown’, ‘linesep’, ‘link’, ‘listdir’, ‘listxattr’, ‘lockf’, ‘lseek’, ‘lstat’, ‘major’, ‘makedev’, ‘makedirs’, ‘minor’, ‘mkdir’, ‘mkfifo’, ‘mknod’, ‘name’, ‘nice’, ‘open’, ‘openpty’, ‘pardir’, ‘path’, ‘pathconf’, ‘pathconf_names’, ‘pathsep’, ‘pipe’, ‘popen’, ‘posix_fadvise’, ‘posix_fallocate’, ‘pread’, ‘putenv’, ‘pwrite’, ‘read’, ‘readlink’, ‘readv’, ‘remove’, ‘removedirs’, ‘removexattr’, ‘rename’, ‘renames’, ‘replace’, ‘rmdir’, ‘scandir’, ‘sched_get_priority_max’, ‘sched_get_priority_min’, ‘sched_getparam’, ‘sched_getscheduler’, ‘sched_param’, ‘sched_rr_get_interval’, ‘sched_setparam’, ‘sched_setscheduler’, ‘sched_yield’, ‘sendfile’, ‘sep’, ‘set_blocking’, ‘set_inheritable’, ‘setegid’, ‘seteuid’, ‘setgid’, ‘setgroups’, ‘setpgid’, ‘setpgrp’, ‘setpriority’, ‘setregid’, ‘setresgid’, ‘setresuid’, ‘setreuid’, ‘setsid’, ‘setuid’, ‘setxattr’, ‘spawnl’, ‘spawnle’, ‘spawnlp’, ‘spawnlpe’, ‘spawnv’, ‘spawnve’, ‘spawnvp’, ‘spawnvpe’, ‘st’, ‘stat’, ‘stat_float_times’, ‘stat_result’, ‘statvfs’, ‘statvfs_result’, ‘strerror’, ‘supports_bytes_environ’, ‘supports_dir_fd’, ‘supports_effective_ids’, ‘supports_fd’, ‘supports_follow_symlinks’, ‘symlink’, ‘sync’, ‘sys’, ‘sysconf’, ‘sysconf_names’, ‘system’, ‘tcgetpgrp’, ‘tcsetpgrp’, ‘terminal_size’, ‘times’, ‘times_result’, ‘truncate’, ‘ttyname’, ‘umask’, ‘uname’, ‘uname_result’, ‘unlink’, ‘unsetenv’, ‘urandom’, ‘utime’, ‘wait’, ‘wait3’, ‘wait4’, ‘waitid’, ‘waitid_result’, ‘waitpid’, ‘walk’, ‘write’, ‘writev’]*

***To print the current working directory***

    *>>> print(**os.getcwd**())*

    */root*

*To change path/current working directory*

    *>>>** os.chdir**(“/tmp”)*

    *>>> print(os.getcwd())*

    */tmp*

*To **print/list** files in the current directory(it return a **list**)*

    *>>> print(**os.listdir**())*

    *[‘.bash_logout’, ‘.bash_profile’, ‘.bashrc’, ‘.cshrc’, ‘.tcshrc’, ‘original-ks.cfg’, ‘anaconda-ks.cfg’, ‘.ssh’, ‘.bash_history’, ‘.pki’, ‘.cache’, ‘.aws’, ‘.lesshst’, ‘.viminfo’,‘anaconda3’, ‘.bashrc-anaconda3.bak’, ‘.python_history’, ‘.config’]*

*To create directories*

    *>>> **os.mkdir**(“test”)*

    *>>> print(**os.listdir**())*

    *[‘.bash_logout’, ‘.bash_profile’, ‘.bashrc’, ‘.cshrc’, ‘.tcshrc’, ‘original-ks.cfg’, ‘anaconda-ks.cfg’, ‘.ssh’, ‘.bash_history’, ‘.pki’, ‘.cache’, ‘.aws’, ‘.lesshst’, ‘.viminfo’, ‘pandas_for_everyone’, ‘anaconda3’, ‘.bashrc-anaconda3.bak’, ‘.python_history’, ‘.config’, ‘test’]*

*But now let say we want to go one level deep i.e both test3 and test4 doesn’t exist, if we are trying to make directory it will not work, to do that we need to use **makedirs***

    *>>> os.mkdir(“test3/test4”)*

    *Traceback (most recent call last):*

    *File “<stdin>”, line 1, in <module>*

    *FileNotFoundError: [Errno 2] No such file or directory: ‘test3/test4’*

    *>>> os.**makedirs**(“test1/test2”)*

*Same way if we want to remove a directory(**rmdir**) or directory inside the directory(**removedirs**)*

    *>>> os.**rmdir**(“test5”)
    >>> os.**removedirs**("test1/test2")*

***NOTE**: As removing is a destructive operation please be cautious before using removedirs*

*To rename a file*

    *>>> os.listdir()*

    *[‘.bash_logout’, ‘.bash_profile’, ‘.bashrc’, ‘.cshrc’, ‘.tcshrc’, ‘original-ks.cfg’, ‘anaconda-ks.cfg’, ‘.ssh’, ‘.bash_history’, ‘.pki’, ‘.cache’, ‘.aws’, ‘.lesshst’, ‘.viminfo’, ‘pandas_for_everyone’, ‘anaconda3’, ‘.bashrc-anaconda3.bak’, ‘.python_history’, ‘.config’, ‘**test**’]*

    *>>> **os.rename**(‘**test**’,’**test1**')*

    *>>> os.listdir()*

    *[‘.bash_logout’, ‘.bash_profile’, ‘.bashrc’, ‘.cshrc’, ‘.tcshrc’, ‘original-ks.cfg’, ‘anaconda-ks.cfg’, ‘.ssh’, ‘.bash_history’, ‘.pki’, ‘.cache’, ‘.aws’, ‘.lesshst’, ‘.viminfo’, ‘pandas_for_everyone’, ‘anaconda3’, ‘.bashrc-anaconda3.bak’, ‘.python_history’, ‘.config’, ‘**test1**’]*

*To check if the path exists*

    *>>> **os.path.exists**(“/etc/resolv.conf”)*

    *True*

    *>>> **os.path.exists**(“/etc/resolvvv.conf”)*

    *False*

*The same way we can check the presence of **file/directory***

    *>>> **os.path.isfile**(“/etc/resolv.conf”)*

    *True*

    *>>> **os.path.isdir**(“/etc/resolv.conf”)*

    *False*

*To print stats*

    *>>> os.**stat**(“test1”)*

    *os.stat_result(st_mode=33188, st_ino=4194982, st_dev=51714, st_nlink=1, st_uid=0, st_gid=0, **st_size**=0, **st_atime**=1492269055, **st_mtime**=1492269055, **st_ctime**=1492269074)*

*To print directory tree and all the files within that directory tree we can use **os.walk** which is a **generator** and has a **tuple** of 3 values*

* *directories(**dirpath**)*

* *directories within that path(**dirname**)*

* *files within that path*

    *>>> for dirpath,dirnames,filenames in os.walk("/test1"):*

    *...     print(dirpath)*

    *...     print(dirnames)*

    *...     print(filenames)*

    *...*

    */test1     # First it goes to toplevel directory*

    *['test2']  # Then all the directories inside it*

    *[]         # All the files below it*

    */test1/test2*

    *[]*

    *[]*

OR Better way to write the same program

    >>> for dirpath, dirnames, filenames in os.walk(“/etc”):
    … print(“Files in %s are: “ % dirpath)
    … for file in filenames:
    … print(“\t” + file)

Output

    Files in /etc are: 
     afpovertcp.cfg
     aliases
     aliases.db

    Files in /etc/apache2 are: 
     httpd.conf
     httpd.conf.pre-update
     magic
     mime.types

For Directories

    >>> for dirpath, dirnames, filenames in os.walk(“/etc”):
    … print(“Directories in %s are:” % dirpath)
    … for dir in dirnames:
    … print(“\t” + dir)
    …

Output

    Directories in /etc are:
     apache2
     asl
     cups
     defaults
     emond.d
     mach_init.d
     mach_init_per_login_session.d
     mach_init_per_user.d
     manpaths.d
     newsyslog.d
     openldap
     pam.d
     paths.d
     periodic
     pf.anchors
     postfix
     ppp
     profile.d
     racoon
     security
     snmp
     ssh

    Directories in /etc/apache2 are:
     extra
     original
     other
     users

*To print all the **environment variables***

    *>>> **os.environ***

    *environ({‘HOSTNAME’: ‘ip-172–31–33–6.us-west-2.compute.internal’, ‘SHELL’: ‘/bin/bash’, ‘TERM’: ‘xterm-256color’, ‘HISTSIZE’: ‘1000’, ‘USER’: ‘root’, ‘LS_COLORS’: ‘rs=0:di=38;5;27:ln=38;5;51:mh=44;38;5;15:pi=40;38;5;11:so=38;5;13:do=38;5;5:bd=48;5;232;38;5;11:cd=48;5;232;38;5;3:or=48;5;232;38;5;9:mi=05;48;5;232;38;5;15:su=48;5;196;38;5;15:sg=48;5;11;38;5;16:ca=48;5;196;38;5;226:tw=48;5;10;38;5;16:ow=48;5;10;38;5;21:st=48;5;21;38;5;15:ex=38;5;34:*.tar=38;5;9:*.tgz=38;5;9:*.arc=38;5;9:*.arj=38;5;9:*.taz=38;5;9:*.lha=38;5;9:*.lz4=38;5;9:*.lzh=38;5;9:*.lzma=38;5;9:*.tlz=38;5;9:*.txz=38;5;9:*.tzo=38;5;9:*.t7z=38;5;9:*.zip=38;5;9:*.z=38;5;9:*.Z=38;5;9:*.dz=38;5;9:*.gz=38;5;9:*.lrz=38;5;9:*.lz=38;5;9:*.lzo=38;5;9:*.xz=38;5;9:*.bz2=38;5;9:*.bz=38;5;9:*.tbz=38;5;9:*.tbz2=38;5;9:*.tz=38;5;9:*.deb=38;5;9:*.rpm=38;5;9:*.jar=38;5;9:*.war=38;5;9:*.ear=38;5;9:*.sar=38;5;9:*.rar=38;5;9:*.alz=38;5;9:*.ace=38;5;9:*.zoo=38;5;9:*.cpio=38;5;9:*.7z=38;5;9:*.rz=38;5;9:*.cab=38;5;9:*.jpg=38;5;13:*.jpeg=38;5;13:*.gif=38;5;13:*.bmp=38;5;13:*.pbm=38;5;13:*.pgm=38;5;13:*.ppm=38;5;13:*.tga=38;5;13:*.xbm=38;5;13:*.xpm=38;5;13:*.tif=38;5;13:*.tiff=38;5;13:*.png=38;5;13:*.svg=38;5;13:*.svgz=38;5;13:*.mng=38;5;13:*.pcx=38;5;13:*.mov=38;5;13:*.mpg=38;5;13:*.mpeg=38;5;13:*.m2v=38;5;13:*.mkv=38;5;13:*.webm=38;5;13:*.ogm=38;5;13:*.mp4=38;5;13:*.m4v=38;5;13:*.mp4v=38;5;13:*.vob=38;5;13:*.qt=38;5;13:*.nuv=38;5;13:*.wmv=38;5;13:*.asf=38;5;13:*.rm=38;5;13:*.rmvb=38;5;13:*.flc=38;5;13:*.avi=38;5;13:*.fli=38;5;13:*.flv=38;5;13:*.gl=38;5;13:*.dl=38;5;13:*.xcf=38;5;13:*.xwd=38;5;13:*.yuv=38;5;13:*.cgm=38;5;13:*.emf=38;5;13:*.axv=38;5;13:*.anx=38;5;13:*.ogv=38;5;13:*.ogx=38;5;13:*.aac=38;5;45:*.au=38;5;45:*.flac=38;5;45:*.mid=38;5;45:*.midi=38;5;45:*.mka=38;5;45:*.mp3=38;5;45:*.mpc=38;5;45:*.ogg=38;5;45:*.ra=38;5;45:*.wav=38;5;45:*.axa=38;5;45:*.oga=38;5;45:*.spx=38;5;45:*.xspf=38;5;45:’, ‘SUDO_USER’: ‘ec2-user’, ‘SUDO_UID’: ‘1000’, ‘USERNAME’: ‘root’, ‘PATH’: ‘/root/anaconda3/bin:/usr/local/sbin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin’, ‘MAIL’: ‘/var/spool/mail/root’, ‘PWD’: ‘/root’, ‘LANG’: ‘en_US.UTF-8’, ‘HISTCONTROL’: ‘ignoredups’, ‘SHLVL’: ‘1’, ‘SUDO_COMMAND’: ‘/bin/bash’, ‘HOME’: ‘/root’, ‘LOGNAME’: ‘root’, ‘LESSOPEN’: ‘||/usr/bin/lesspipe.sh %s’, ‘SUDO_GID’: ‘1000’, ‘_’: ‘/root/anaconda3/bin/python3’})*

*If we are looking for any specific value*

    *>>> **os.environ.get(‘LOGNAME’)***

    *‘root*

*To join two paths together*

    *>>> a = **os.path.join**(“/tmp”, “testhome”)*

    *>>> print(a)*

    */tmp/testhome*

Join is especially helpful while reading the content of the file

    >>> import os

    >>> filename = **os.path.join**(“/root”,”test1")

    >>> mode = “r”

    >>> with open(filename,mode) as f:

    … print(f.read())

    …

    reading the content of the file

    >>> f.close()

To get the process id

    >>> **os.getpid**()

    12462

Simple Directory listing Program

    >>> for file in os.listdir(“.”):
    … info = os.stat(file)
    … print(“%-20s : size %d” % (file, info.st_size))

Output

    animals.py : size 195
    batteries.py : size 38
    continue.py : size 128
    deepreverse.py : size 222
    distance.py : size 360
    evenodd.py : size 151
    fib.py : size 147
    findminmax.py : size 441
    firstclass.py : size 598

For more info please refer
[**16.1. os — Miscellaneous operating system interfaces — Python 3.6.1 documentation**
*This module provides a portable way of using operating system dependent functionality. If you just want to read or…*docs.python.org](https://docs.python.org/3/library/os.html)

**Subprocess Module**

Please be careful while executing shell command via Python
[**17.5. subprocess — Subprocess management — Python 3.6.1 documentation**
*The arguments shown above are merely the most common ones, described below in Frequently Used Arguments (hence the use…*docs.python.org](https://docs.python.org/3/library/subprocess.html)

As per doc “the subprocess module allows you to spawn new processes, connect to their input/output/error pipes, and obtain their return codes. This module intends to replace several older modules and functions”

    os.system
    os.spawn*

To list all files and directories

    #Subprocess always accept list as argument

    >>> **subprocess.call**([“ls”,”-l”])

    total 20

    drwxr-xr-x 20 root root 268 Mar 26 14:39 anaconda3

    -rw — — — -. 1 root root 7712 Oct 20 13:05 anaconda-ks.cfg

    -rw — — — -. 1 root root 6927 Oct 20 13:05 original-ks.cfg

    drwxr-xr-x 4 root root 80 Mar 26 14:26 pandas_for_everyone

    -rw-r — r — 1 root root 32 Apr 15 11:57 test1

    0

    #If we try to run subprocess without passing list

    >>> **subprocess.call**("ls -l")

    Traceback (most recent call last):

    File "<stdin>", line 1, in <module>

    File "/root/anaconda3/lib/python3.6/subprocess.py", line 267, in call

    with Popen(*popenargs, **kwargs) as p:

    File "/root/anaconda3/lib/python3.6/subprocess.py", line 707, in __init__

    restore_signals, start_new_session)

    File "/root/anaconda3/lib/python3.6/subprocess.py", line 1326, in _execute_child

    raise child_exception_type(errno_num, err_msg)

    FileNotFoundError: [Errno 2] No such file or directory: 'ls -l'

    #The correct way is

    >>> **subprocess.call**(["df","-h")]

    Filesystem      Size  Used Avail Use% Mounted on

    /dev/xvda2       10G  4.4G  5.7G  44% /

    devtmpfs        477M     0  477M   0% /dev

    tmpfs           496M     0  496M   0% /dev/shm

    tmpfs           496M   63M  434M  13% /run

    tmpfs           496M     0  496M   0% /sys/fs/cgroup

    tmpfs           100M     0  100M   0% /run/user/1000

    0

Now to save the output to some variable use check_output

    >>> x = **subprocess.check_output**([“ls”,”-l”])

    >>> print(x)

    b’total 20\ndrwxr-xr-x 20 root root 268 Mar 26 14:39 anaconda3\n-rw — — — -. 1 root root 7712 Oct 20 13:05 anaconda-ks.cfg\n-rw — — — -. 1 root root 6927 Oct 20 13:05 original-ks.cfg\ndrwxr-xr-x 4 root root 80 Mar 26 14:26 pandas_for_everyone\n-rw-r — r — 1 root root 32 Apr 15 11:57 test1\n’

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
[**100 Days of DevOps — Day 77-Process Management in Linux**
*Welcome to Day 77 of 100 Days of DevOps, Focus for today is Process Management in Linux*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-77-process-management-in-linux-21aabae5b124)
