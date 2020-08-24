Unknown markup type 10 { type: [33m10[39m, start: [33m47[39m, end: [33m60[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m93[39m }

# What is a container, technically?

Containers are Linux Processes with additional configuration added. Let’s take a look at it from technical perspective:

![The Docker container launches a process called redis-server.](https://cdn-images-1.medium.com/max/2000/1*YXC6yWUtF_RGA71OxJkUhA.jpeg)*The Docker container launches a process called redis-server.*

![From the host, here we can view all the processes running, including those started by Docker.](https://cdn-images-1.medium.com/max/2000/1*82e87I1gGsE8Yn0FcTlaDg.jpeg)*From the host, here we can view all the processes running, including those started by Docker.*

![From the host, here we can view all the processes running, including those started by Docker.](https://cdn-images-1.medium.com/max/2000/1*_wmO3reAinCcyQl3nN_eEw.jpeg)*From the host, here we can view all the processes running, including those started by Docker.*

![Here, Docker shows information about the process including PID(Process ID) and PPID (Parent Process ID) via running a command “docker top db”](https://cdn-images-1.medium.com/max/2488/1*agXSUQbAFnVPJanMt35XMw.jpeg)*Here, Docker shows information about the process including PID(Process ID) and PPID (Parent Process ID) via running a command “docker top db”*

![“pstree” command here, lists all of the sub processes](https://cdn-images-1.medium.com/max/2000/1*WkJWMQY7ZAm0hoaYSsmT5g.jpeg)*“pstree” command here, lists all of the sub processes*

As you can see from Linux perspective, these are standard processes and have the same properties as other processes on our system.
