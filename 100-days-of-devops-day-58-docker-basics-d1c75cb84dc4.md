
# 100 Days of DevOps — Day 58-Docker Basics

Welcome to Day 58 of 100 Days of DevOps, Focus for today is Docker Basics
> What is a docker?

*As per official documentation “Docker is a platform for developers and sysadmins to **develop, deploy, and run** applications with containers”*

*Containers are different than Virtual Machines because*

* *We can start with a small base image instead of the whole operating system*

* *After that, we can just add an application(and its dependencies)that we need*

* *Because containers carry its dependencies, the underlying operating system doesn’t need to supply libraries, executables and other components it needs*

* *Containers are more portable and efficient then Virtual Machine because they eat less memory and storage*
> **Installing Docker on Centos7**

* *First, update the package database*

    *# yum check-update*

* *Now run this command, this will install the latest version of official docker repository*

    *# curl -fsSL [https://get.docker.com/](https://get.docker.com/) | sh*

* *Start the docker daemon*

    *# systemctl start docker*

* *Make sure it start after every reboot*

    *# systemctl enable docker*

    *Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.*

* *Check the status of docker daemon*

    *# systemctl status docker*

    ***●** docker.service - Docker Application Container Engine*

    *Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)*

    *Active: **active (running)** since Sat 2018-04-07 12:41:19 EDT; 1min 22s ago*

    *Docs: [https://docs.docker.com](https://docs.docker.com)*

    *Main PID: 3692 (dockerd)*

    *CGroup: /system.slice/docker.service*

    *├─3692 /usr/bin/dockerd*

    *└─3696 docker-containerd --config /var/run/docker/containerd/containerd.toml*

    *Apr 07 12:41:18 docker.example.com dockerd[3692]: time="2018-04-07T12:41:18-04:00" level=info msg=serving... address="/var/run/docker/containerd/docker-containerd.sock" module="containerd/grpc"*

    *Apr 07 12:41:18 docker.example.com dockerd[3692]: time="2018-04-07T12:41:18-04:00" level=info msg="containerd successfully booted in 0.012498s" module=containerd*

    *Apr 07 12:41:19 docker.example.com dockerd[3692]: time="2018-04-07T12:41:19.026811762-04:00" level=info msg="Graph migration to content-addressability took 0.00 seconds"*

    *Apr 07 12:41:19 docker.example.com dockerd[3692]: time="2018-04-07T12:41:19.027821331-04:00" level=info msg="Loading containers: start."*

    *Apr 07 12:41:19 docker.example.com dockerd[3692]: time="2018-04-07T12:41:19.228479307-04:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon o...d IP address"*

    *Apr 07 12:41:19 docker.example.com dockerd[3692]: time="2018-04-07T12:41:19.397018661-04:00" level=info msg="Loading containers: done."*

    *Apr 07 12:41:19 docker.example.com dockerd[3692]: time="2018-04-07T12:41:19.414815278-04:00" level=info msg="Docker daemon" commit=3d479c0 graphdriver(s)=overlay2 version=18.04.0-ce*

    *Apr 07 12:41:19 docker.example.com dockerd[3692]: time="2018-04-07T12:41:19.415008276-04:00" level=info msg="Daemon has completed initialization"*

    *Apr 07 12:41:19 docker.example.com dockerd[3692]: time="2018-04-07T12:41:19.426610984-04:00" level=info msg="API listen on /var/run/docker.sock"*

    *Apr 07 12:41:19 docker.example.com systemd[1]: Started Docker Application Container Engine.*

    *Hint: Some lines were ellipsized, use -l to show in full.*
> **Docker Image**

* *When docker container is stored in a registry or in our local system its referred to as an image*

* *DockerHub [https://hub.docker.com/](https://hub.docker.com/)*

* *To search for a particular image on the command line*

    *# docker search centos*

    *NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED*

    *centos                             The official build of CentOS.                   4185                [OK]*

    *ansible/centos7-ansible            Ansible on Centos7                              108                                     [OK]*

    *jdeathe/centos-ssh                 CentOS-6 6.9 x86_64 / CentOS-7 7.4.1708 x86_…   94                                      [OK]*

    *consol/centos-xfce-vnc             Centos container with "headless" VNC session…   52                                      [OK]*

    *imagine10255/centos6-lnmp-php56    centos6-lnmp-php56                              40                                      [OK]*

    *tutum/centos                       Simple CentOS docker image with SSH access      37*

    *gluster/gluster-centos             Official GlusterFS Image [ CentOS-7 +  Glust…   26                                      [OK]*

    *centos/mysql-57-centos7            MySQL 5.7 SQL database server                   23*

    *openshift/base-centos7             A Centos7 derived base image for Source-To-I…   22*

    *kinogmt/centos-ssh                 CentOS with SSH                                 19                                      [OK]*

    *centos/python-35-centos7           Platform for building and running Python 3.5…   19*

    *centos/postgresql-96-centos7       PostgreSQL is an advanced Object-Relational …   12*

    *openshift/jenkins-2-centos7        A Centos7 based Jenkins v2.x image for use w…   11*

    *openshift/mysql-55-centos7         DEPRECATED: A Centos7 based MySQL v5.5 image…   6*

    *pivotaldata/centos-gpdb-dev        CentOS image for GPDB development. Tag names…   3*

    *openshift/jenkins-1-centos7        DEPRECATED: A Centos7 based Jenkins v1.x ima…   3*

    *openshift/wildfly-101-centos7      A Centos7 based WildFly v10.1 image for use …   3*

    *openshift/php-55-centos7           DEPRECATED: A Centos7 based PHP v5.5 image f…   1*

    *blacklabelops/centos               CentOS Base Image! Built and Updates Daily!     1                                       [OK]*

    *pivotaldata/centos                 Base centos, freshened up a little with a Do…   1*

    *pivotaldata/centos-mingw           Using the mingw toolchain to cross-compile t…   1*

    *openshift/wildfly-100-centos7      A Centos7 based WildFly v10.0 image for use …   1*

    *pivotaldata/centos-gcc-toolchain   CentOS with a toolchain, but unaffiliated wi…   0*

    *smartentry/centos                  centos with smartentry                          0                                       [OK]*

    *jameseckersall/sonarr-centos       Sonarr on CentOS 7                              0                                       [OK]*

* *We can also limit our search using limit option*

    *# docker search centos --limit 5*

    *NAME                          DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED*

    *centos                        The official build of CentOS.                   4194                [OK]*

    *jdeathe/centos-ssh            CentOS-6 6.9 x86_64 / CentOS-7 7.4.1708 x86_…   94                                      [OK]*

    *openshift/base-centos7        A Centos7 derived base image for Source-To-I…   22*

    *pivotaldata/centos-gpdb-dev   CentOS image for GPDB development. Tag names…   3*

    *pivotaldata/centos            Base centos, freshened up a little with a Do…   1*

* *We can search based on stars*

    *# docker search --filter stars=50 centos*

    *NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED*

    *centos                    The official build of CentOS.                   4194                [OK]*

    *ansible/centos7-ansible   Ansible on Centos7                              108                                     [OK]*

    *jdeathe/centos-ssh        CentOS-6 6.9 x86_64 / CentOS-7 7.4.1708 x86_…   94                                      [OK]*

    *consol/centos-xfce-vnc    Centos container with "headless" VNC session…   52                                      [OK]*

* *We can also search based on the official release*

    *# docker search --filter is-official=true centos*

    *NAME                DESCRIPTION                     STARS               OFFICIAL            AUTOMATED*

    *centos              The official build of CentOS.   4194                [OK]*

* *Similarly, you can search it via UI on dockerhub([https://hub.docker.com/](https://hub.docker.com/))*

![](https://cdn-images-1.medium.com/max/5756/1*ZQPVLXSvc2jW64H72OjSEg.png)

* *To pull a specific image, just follow the instruction on the official page*

![](https://cdn-images-1.medium.com/max/5760/1*i60ai2bTOYEJWVu0W2PLig.png)

    *# docker pull centos*

    *Using default tag: latest*

    *latest: Pulling from library/centos*

    *469cfcc7a4b3: Pull complete*

    *Digest: sha256:283e71ecddb63cf9a76d92cf053f3a8dce613f32ab55b15997d4f4a05b187778*

    *Status: Downloaded newer image for centos:latest*

*NOTE: By default, it pulls the latest tag*

* *To verify it*

    *# docker images*

    *REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE*

    *centos              latest              e934aafc2206        20 hours ago        199MB*

* *To pull all the centos images(use -a options)*

    *$ docker pull -a centos*

    *5.11: Pulling from library/centos*

    *2068b24f564b: Pull complete*

    *Digest: sha256:c40041f5894293d0df8f5c6c2049b92a82c53f1718ecdd73cbf3c1826a08ba4a*

    *5: Pulling from library/centos*

    *38892065247a: Pull complete*

    *Digest: sha256:70fffd687ff9545662c30f9043108489c698662861cd5f76070f7e2cd350564f*

    *6.6: Pulling from library/centos*

    *f9f73d801f05: Pull complete*

    *Digest: sha256:ba9fbbcf6e957b480c6721f0e2abced5082b690d87342a7efd95df6f662c2c2d*

    *6.7: Pulling from library/centos*

    *cbddbc0189a0: Downloading [===========================>                       ]  37.69MB/67.81MB*
> *-a, — all-tags Download all tagged images in the repository*

*Now let’s take a look some more options available with docker images(eg: if we are looking for long image id)*

    *# docker images --no-trunc*

    *REPOSITORY          TAG                 IMAGE ID                                                                  CREATED             SIZE*

    *hello-world         latest              sha256:e38bc07ac18ee64e6d59cf2eafcdddf9cec2364dfe129fe0af75f1b0194e0c96   5 days ago          1.85kB*

    *hello-world         linux               sha256:e38bc07ac18ee64e6d59cf2eafcdddf9cec2364dfe129fe0af75f1b0194e0c96   5 days ago          1.85kB*

    *centos              7                   sha256:e934aafc22064b7322c0250f1e32e5ce93b2d19b356f4537f5864bd102e8531f   10 days ago         199MB*

    *httpd               latest              sha256:805130e51ae9e737e056b75bfd797c4400a012181f1ba10fad71d69e78457f49   3 weeks ago         178MB*
> *— no-trunc Don’t truncate output*

* *We can also use filter option(to filter all the images before/since the particular image is created)*

    *# docker images --filter "before=hello-world"*

    *REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE*

    *centos              7                   e934aafc2206        10 days ago         199MB*

    *httpd               latest              805130e51ae9        3 weeks ago         178MB*

* *The same way we can use since filter(All based on a CREATED column)*

    *# docker images --filter "since=centos:7"*

    *REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE*

    *hello-world         latest              e38bc07ac18e        5 days ago          1.85kB*

    *hello-world         linux               e38bc07ac18e        5 days ago          1.85kB*

* *If we are only looking for image id*

    *# docker images -a -q*

    *e38bc07ac18e*

    *e38bc07ac18e*

    *e934aafc2206*

    *805130e51ae9*

* *Docker Hub Registry is free to use as a public repository and it also allows us to set up one private repository [https://hub.docker.com/billing-plans/](https://hub.docker.com/billing-plans/)*

* *What would be the case where we just want to share our work with your team or we have more than one private repository. Docker provides package called docker-distribution.*

    *# yum -y install docker-distribution*

* *Start the docker-distribution daemon*

    *# systemctl start docker-distribution*

* *Enable the docker-distribution daemon*

    *# systemctl enable docker-distribution*

    *Created symlink from /etc/systemd/system/multi-user.target.wants/docker-distribution.service to /usr/lib/systemd/system/docker-distribution.service.*

* *To check the status*

    *# systemctl status docker-distribution*

    ***●** docker-distribution.service - v2 Registry server for Docker*

    *Loaded: loaded (/usr/lib/systemd/system/docker-distribution.service; enabled; vendor preset: disabled)*

    *Active: **active (running)** since Sat 2018-04-07 13:01:32 EDT; 9s ago*

    *Main PID: 3900 (registry)*

    *CGroup: /system.slice/docker-distribution.service*

    *└─3900 /usr/bin/registry serve /etc/docker-distribution/registry/config.yml*

    *Apr 07 13:01:32 docker.example.com systemd[1]: **[/usr/lib/systemd/system/docker-distribution.service:11] Unknown lvalue 'After' in section 'Install'***

    *Apr 07 13:01:32 docker.example.com systemd[1]: Started v2 Registry server for Docker.*

    *Apr 07 13:01:32 docker.example.com systemd[1]: Starting v2 Registry server for Docker...*

    *Apr 07 13:01:32 docker.example.com registry[3900]: time="2018-04-07T13:01:32-04:00" level=warning msg="No HTTP secret provided - generated random secret. This may cause problems with uploads if multipl...*

    *Apr 07 13:01:32 docker.example.com registry[3900]: time="2018-04-07T13:01:32-04:00" level=info msg="redis not configured" go.version=go1.8.3 instance.id=4645dba2-af9d-4fb3-96f1-6cc4835f977....6.2+unknown"*

    *Apr 07 13:01:32 docker.example.com registry[3900]: time="2018-04-07T13:01:32-04:00" level=info msg="Starting upload purge in 43m0s" go.version=go1.8.3 instance.id=4645dba2-af9d-4fb3-96f1-6....6.2+unknown"*

    *Apr 07 13:01:32 docker.example.com registry[3900]: time="2018-04-07T13:01:32-04:00" level=info msg="using inmemory blob descriptor cache" go.version=go1.8.3 instance.id=4645dba2-af9d-4fb3-....6.2+unknown"*

    *Apr 07 13:01:32 docker.example.com registry[3900]: time="2018-04-07T13:01:32-04:00" level=info msg="listening on [::]:5000" go.version=go1.8.3 instance.id=4645dba2-af9d-4fb3-96f1-6cc4835f9....6.2+unknown"*

    *Apr 07 13:01:39 docker.example.com systemd[1]: **[/usr/lib/systemd/system/docker-distribution.service:11] Unknown lvalue 'After' in section 'Install'***

    *Hint: Some lines were ellipsized, use -l to show in full.*

* *To push an image to the local registry we first need to tag it*

    *# docker tag centos:latest localhost:5000/plakhera/centos*

* *As you can see in the below output they both are referring to same image id*

    *# docker images*

    *REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE*

    *centos                           latest              e934aafc2206        20 hours ago        199MB*

    *localhost:5000/plakhera/centos   latest              e934aafc2206        20 hours ago        199MB*

* *To push an image to a local registry*

    *# docker push localhost:5000/plakhera/centos*

    *The push refers to repository [localhost:5000/plakhera/centos]*

    *43e653f84b79: Pushed*

    *latest: digest: sha256:191c883e479a7da2362b2d54c0840b2e8981e5ab62e11ab925abf8808d3d5d44 size: 529*

* *Now to remove this image locally*

    *# docker rmi localhost:5000/plakhera/centos*

    *Untagged: localhost:5000/plakhera/centos:latest*

    *Untagged: localhost:5000/plakhera/centos@sha256:191c883e479a7da2362b2d54c0840b2e8981e5ab62e11ab925abf8808d3d5d4*

* *But as we pushed this image to local repository we can get it back*

    *# docker pull localhost:5000/plakhera/centos*

    *Using default tag: latest*

    *latest: Pulling from plakhera/centos*

    *Digest: sha256:191c883e479a7da2362b2d54c0840b2e8981e5ab62e11ab925abf8808d3d5d44*

    *Status: Downloaded newer image for localhost:5000/plakhera/centos:latest*

* *To verify it*

    *# docker images*

    *REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE*

    *centos                           latest              e934aafc2206        20 hours ago        199MB*

    *localhost:5000/plakhera/centos   latest              e934aafc2206        20 hours ago        199MB*
> Command to run the container

    *docker run [OPTS] image [COMMAND] [ARGS]*

* *docker run command starts up a process from within a new container*

    *[root@docker ~]# docker run -i -t centos /bin/bash*

    *[root@bed3f29d8652 /]#*
> *-i, — interactive Keep STDIN open even if not attached*
> *-t, — tty Allocate a pseudo-TTY*

* *Inside the container, processes see only*

    ** File system
    * Process table
    * Network Interfaces*

* *To check the status of the container*

    *# docker ps*

    *CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES*

    *bed3f29d8652        centos              "/bin/bash"         4 minutes ago       Up 4 minutes                            romantic_banach*

* *To see all the container(stop/running)*

    *# docker ps -a*

    *CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES*

    *bed3f29d8652        centos              "/bin/bash"         5 minutes ago       Exited (0) 6 seconds ago                       romantic_banach*

* To view information about the image

    # docker inspect hello-world:v1

    [

    {

    "Id": "sha256:e38bc07ac18ee64e6d59cf2eafcdddf9cec2364dfe129fe0af75f1b0194e0c96",

    "RepoTags": [

    "hello-world:v1"

    ],

    "RepoDigests": [

    "hello-world@sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77"

    ],

    "Parent": "",

    "Comment": "",

    "Created": "2018-04-11T18:11:49.477283528Z",

    "Container": "190fa62181e74c25cd9f430ea198d477abf6040e1004a64d61fb4a85c1ac082b",

    "ContainerConfig": {

    "Hostname": "190fa62181e7",

    "Domainname": "",

    "User": "",

    "AttachStdin": false,

    "AttachStdout": false,

    "AttachStderr": false,

    "Tty": false,

* *To view information about the container*

    *$ docker inspect ed84dd07b639*

    *[*

    *{*

    *"Id": "ed84dd07b63965cccc9e82878e8e86c172636fdad65f68fb23c2ac40cc650c3b",*

    *"Created": "2018-04-13T13:38:15.461883802Z",*

    *"Path": "/bin/bash",*

    *"Args": [],*

    *"State": {*

    *"Status": "running",*

    *"Running": true,*

    *"Paused": false,*

    *"Restarting": false,*

    *"OOMKilled": false,*

    *"Dead": false,*

    *"Pid": 2569,*

    *"ExitCode": 0,*

    *"Error": "",*

    *"StartedAt": "2018-04-13T13:38:15.894490287Z",*

    *"FinishedAt": "0001-01-01T00:00:00Z"*

    *},*

* *To view specific information about a container*

    *docker inspect --format='{{.NetworkSettings.IPAddress}}' ed84dd07b639*

    *172.17.0.2*

*OR*

    *$ docker inspect --format='{{.State.Status}}' ed84dd07b639*

    *running*

* Now if you run this

    # docker image inspect --format='{{.ContainerConfig}}' hello-world:v1

    {190fa62181e7   false false false map[] false false false [PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin] [/bin/sh -c #(nop)  CMD ["/hello"]] <nil> true sha256:c96933136c89265af0ba6c49ebfb11db8633c76518a0cf479a015781598c8e0b map[]  [] false  [] map[]  <nil> []}

* But as you can see you are only getting value out of it, but we need in the form of key-value pair

    # docker image inspect --format='{{**json .ContainerConfig**}}' hello-world:v1

    {"Hostname":"190fa62181e7","Domainname":"","User":"","AttachStdin":false,"AttachStdout":false,"AttachStderr":false,"Tty":false,"OpenStdin":false,"StdinOnce":false,"Env":["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"],"Cmd":["/bin/sh","-c","#(nop) ","CMD [\"/hello\"]"],"ArgsEscaped":true,"Image":"sha256:c96933136c89265af0ba6c49ebfb11db8633c76518a0cf479a015781598c8e0b","Volumes":null,"WorkingDir":"","Entrypoint":null,"OnBuild":null,"Labels":{}}

* One of the best use cases of using inspect command is to find out the repo tag

    # docker images

    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

    hello-world         v1                  e38bc07ac18e        5 days ago          1.85kB

    hello-world         v2                  e38bc07ac18e        5 days ago          1.85kB

* This can be done

    # docker image inspect e38bc07ac18e --format '{{.RepoTags}}'

    [hello-world:v1 hello-world:v2]

* *To stop the running container*

    *$ docker stop stoic_yonath*

    *stoic_yonath*

* *To get only the id of the container*

    *$ docker ps -a -q |head*

    *ed84dd07b639*

    *16b63895a6c3*

    *6be3b83f6dc3*

    *41c08f6cf45d*

*where*

    *-a, --all             Show all containers (default shows just running)*

    *-q, --quiet           Only display numeric IDs*

* *To remove the container*

    *$ docker rm 16b63895a6c3*

    *16b63895a6c3*

* *To create an image out of the stopped container*

    *$ docker commit -m "hello world container" -a "Prash Lakh" 3996b2f0e631 hello_new_world*

    *sha256:5b38599a96be17a7937ebe01d80e4fb9dcc8535a19de8858c9ae5d6da9865860*

* *To save the docker image*

    *# docker save -o hello-world.tar hello-world:latest*

    *-rw-------. 1 root root 12800 Apr 17 06:56 hello-world.tar*

    *# tar -tvf hello-world.tar*

    *-rw-r--r-- 0/0            1510 2018-04-11 14:11 e38bc07ac18ee64e6d59cf2eafcdddf9cec2364dfe129fe0af75f1b0194e0c96.json*

    *drwxr-xr-x 0/0               0 2018-04-11 14:11 fe9037d3e299f8afe0620498c9dbdb5314e33b3e750c1f4053cda8df4b753fd3/*

    *-rw-r--r-- 0/0               3 2018-04-11 14:11 fe9037d3e299f8afe0620498c9dbdb5314e33b3e750c1f4053cda8df4b753fd3/VERSION*

    *-rw-r--r-- 0/0            1182 2018-04-11 14:11 fe9037d3e299f8afe0620498c9dbdb5314e33b3e750c1f4053cda8df4b753fd3/json*

    *-rw-r--r-- 0/0            3584 2018-04-11 14:11 fe9037d3e299f8afe0620498c9dbdb5314e33b3e750c1f4053cda8df4b753fd3/layer.tar*

    *-rw-r--r-- 0/0             207 1969-12-31 19:00 manifest.json*

    *-rw-r--r-- 0/0              94 1969-12-31 19:00 repositories*
> *Options:*
> *-o, — output string Write to a file, instead of STDOUT*

* *Now let’s remove the original image and try to load it back on the system OR we can copy this tar file to other machines, and t load this image*

    *[root@docker ~]# docker rmi hello-world:latest*

    *Untagged: hello-world:latest*

    *[root@docker ~]# docker load -i hello-world.tar*

    *Loaded image: hello-world:latest*

    *[root@docker ~]# docker images*

    *REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE*

    *hello-world         latest              e38bc07ac18e        5 days ago          1.85kB*

* *Another way to do the same task is to use docker import*

    *[root@docker ~]# docker import hello-world.tar hello-world:2.0*

    *sha256:47b72c36cf8910b1737b10678b480dec15d6a3541bd6ff96a1e7226f98b88365*

    *[root@docker ~]# docker images*

    *REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE*

    *hello-world         2.0                 47b72c36cf89        3 seconds ago       6.58kB*

*NOTE: If we don’t provide an image name(hello-world:2.0) it will not show anything*

* *To get system-wide Docker information*

    *# docker info*

    *Containers: 1*

    *Running: 0*

    *Paused: 0*

    *Stopped: 1*

    *Images: 1*

    ***Server Version: 18.04.0-ce***

    *Storage Driver: overlay2*

    *Backing Filesystem: xfs*

    *Supports d_type: true*

    *Native Overlay Diff: true*

    *Logging Driver: json-file*

    *Cgroup Driver: cgroupfs*

    *Plugins:*

    *Volume: local*

    *Network: bridge host macvlan null overlay*

    *Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog*

    *Swarm: inactive*

    *Runtimes: runc*

    *Default Runtime: runc*

    *Init Binary: docker-init*

    *containerd version: 773c489c9c1b21a6d78b5c538cd395416ec50f88*

    *runc version: 4fc53a81fb7c994640722ac585fa9ca548971871*

    *init version: 949e6fa*

    *Security Options:*

    *seccomp*

    *Profile: default*

    *Kernel Version: 3.10.0-693.el7.x86_64*

    *Operating System: CentOS Linux 7 (Core)*

    *OSType: linux*

    *Architecture: x86_64*

    *CPUs: 1*

    *Total Memory: 992.4MiB*

    *Name: docker.example.com*

    *ID: JLNP:OVGZ:4LFR:W2IQ:V3XB:JV7G:X7HE:ZFAD:2VFE:WENX:KKGX:IJBM*

    *Docker Root Dir: /var/lib/docker*

    *Debug Mode (client): false*

    *Debug Mode (server): false*

    *Registry: [https://index.docker.io/v1/](https://index.docker.io/v1/)*

    *Labels:*

    *Experimental: false*

    *Insecure Registries:*

    *127.0.0.0/8*

    *Live Restore Enabled: false*

* *To get Docker version information*

    *# docker version*

    *Client:*

    *Version: 18.04.0-ce*

    *API version: 1.37*

    *Go version: go1.9.4*

    *Git commit: 3d479c0*

    *Built: Tue Apr 10 18:21:36 2018*

    *OS/Arch: linux/amd64*

    *Experimental: false*

    *Orchestrator: swarm*

    *Server:*

    *Engine:*

    *Version: 18.04.0-ce*

    *API version: 1.37 (minimum version 1.12)*

    *Go version: go1.9.4*

    *Git commit: 3d479c0*

    *Built: Tue Apr 10 18:25:25 2018*

    *OS/Arch: linux/amd64*

    *Experimental: false*

* *To get information about the container in top style output*

    *[root@docker ~]# docker top 662891f10a6d*

    *UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD*

    *root                1333                1322                0                   10:11               pts/0               00:00:00            /bin/bash*

    *[root@docker ~]# docker top 662891f10a6d -x*

    *PID                 TTY                 STAT                TIME                COMMAND*

    *1333                pts/0               Ss+                 0:00                /bin/bash*

* *To find if any file inside container changed*

    *# docker diff 662891f10a6d*

    *C /etc*

* *To see the history of the particular container(i.e how this container is created/to see the various layers)*

    *# docker history centos:latest*

    *IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT*

    *e934aafc2206        6 days ago          /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B*

    *<missing>           6 days ago          /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B*

    *<missing>           6 days ago          /bin/sh -c #(nop) ADD file:f755805244a649ecc…   199MB*

* *To see what events going on with the container*

    *# docker events*

    *2018-04-13T10:17:55.692534036-04:00 container die 662891f10a6d4f77f519ad0db4ee0d441940d0f7fb037d904446f7764dd40451 (exitCode=0, image=centos:latest, name=jolly_meninsky, org.label-schema.schema-version== 1.0     org.label-schema.name=CentOS Base Image     org.label-schema.vendor=CentOS     org.label-schema.license=GPLv2     org.label-schema.build-date=20180402)*

    *2018-04-13T10:17:55.721338352-04:00 network disconnect 695787e430858c26d10e0e0980f77aff4b1a5507a3266b19f283036e3b9b34b5 (container=662891f10a6d4f77f519ad0db4ee0d441940d0f7fb037d904446f7764dd40451, name=bridge, type=bridge)*

* *Removing container images(Be careful while running this command)*

    *$ docker system prune*

    *WARNING! This will remove:*

    *- all stopped containers*

    *- all networks not used by at least one container*

    *- all dangling images*

    *- all build cache*

    *Are you sure you want to continue? [y/N] y*

    *Deleted Containers:*

* OR to remove the unused image

    # docker image prune

    WARNING! This will remove all dangling images.

    Are you sure you want to continue? [y/N] y

    Total reclaimed space: 0B

* If we used with -a option

    # docker image prune -a

    WARNING! This will remove all images without at least one container associated to them.

    Are you sure you want to continue? [y/N]
> -a, — all Remove all unused images, not just dangling ones
> — filter filter Provide filter values (e.g. ‘until=<timestamp>’)

* *Other ways to remove containers and images*

    *# To get image id
    $ docker images -a -q*

    *5b38599a96be*

    *6662d8cc2447*

    *bb15b898ca55*

    *6601d6761139*

    *8bd1c9f6dd75*

    *dd464784a563*

    *4c277e38e2bd*

* *To get container id*

    *$ docker ps -a -q*

    *f9c2e5959bc5*

* *Now to remove images*

    *$ docker rmi $(docker images -a -q)*

* *To remove multiple containers*

    *$ docker rm $(docker ps -a -q)*

One of the basic misconception about the containerized process that I want to clear that

* It just can’t see and use everything is on the host system(process/files)

* It’s running directly on the host system kernel but it has the files it needs to run the application in a separate filesystem and can’t see the files in the host system

* It can see its own process but can’t see the other processes running on the host

This is all possible due to a feature called Linux Containers(LXC). LCX allows a container to have its own namespaces

* Process table

* File system

* Network interfaces

* Inter-process communications(IPC)

* Cgroups(we can limit how much memory, cpu available to a container from hosts)

**Privileged Containers**

* These are mostly used for administering, monitor and troubleshooting purpose

* Using docker run we can open individual host privilege

    * Process table (--pid=host)
    * File system (-v /=/host)
    * Network interfaces (--net=host)
    * Inter-process communications(--ipc=host)
    * Privileges(--privileged) #Be careful with that it open root level privileges on the host

NOTE: Be careful with all these options

Let’s understand this one by one

    $ docker run -it centos:7 /bin/bash

    [root@64db4f8ab614 /]# ps -ef

    UID        PID  PPID  C STIME TTY          TIME CMD

    root         1     0  1 21:56 pts/0    00:00:00 /bin/bash

    root        15     1  0 21:56 pts/0    00:00:00 ps -ef

Now let’s try to list process table with — pid=host

    $ docker run -it --pid=host centos:7 /bin/bash

    [root@95c904514e44 /]# ps -ef

    UID        PID  PPID  C STIME TTY          TIME CMD

    root         1     0  0 15:35 ?        00:00:01 /sbin/init text

    root         2     0  0 15:35 ?        00:00:00 [kthreadd]

    root         3     2  0 15:35 ?        00:00:00 [ksoftirqd/0]

    root         5     2  0 15:35 ?        00:00:00 [kworker/0:0H]

    root         7     2  0 15:35 ?        00:00:01 [rcu_sched]

    root         8     2  0 15:35 ?        00:00:00 [rcu_bh]

    root         9     2  0 15:35 ?        00:00:00 [migration/0]

    root        10     2  0 15:35 ?        00:00:00 [lru-add-drain]

    root        11     2  0 15:35 ?        00:00:00 [watchdog/0]

    root        12     2  0 15:35 ?        00:00:00 [cpuhp/0]

    root        13     2  0 15:35 ?        00:00:00 [cpuhp/1]

    root        14     2  0 15:35 ?        00:00:00 [watchdog/1]

    root        15     2  0 15:35 ?        00:00:00 [migration/1]

    root        16     2  0 15:35 ?        00:00:00 [ksoftirqd/1]

    root        18     2  0 15:35 ?        00:00:00 [kworker/1:0H]

Now the biggest issue with this we can even kill any process running on the host

* We can mount the host file system in a container(-v /:/host)

    $ docker run -it -v /:/host centos:7 /bin/bash

* For full file system privileges

    $ docker run -it -v /:/host --privileged centos:7 /bin/bash

Accessing the Host Network Interfaces

* Inside the container, we can only see two network interfaces(eth0 and lo)

* To open direct access to host networks( — net=host)

    $ docker run -it --net=host centos:7 /bin/bash

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
