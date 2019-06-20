
# 100 Days of DevOps — Day 73- Introduction to Ansible

Welcome to Day 73 of 100 Days of DevOps, Focus for today is Introduction to Ansible

*As per wiki, **Ansible** is an [open-source](https://en.wikipedia.org/wiki/Open-source_software) automation engine that automates software provisioning, configuration management, and application deployment*

***Installing Ansible***

*Please make sure epel rpm is installed*

    *# rpm -qa|grep -i epel*

    ***epel**-release-7–9.noarch*

*To install ansible*

    *# yum -y install ansible*

*Ansible has bunch of dependencies*

*========================================*

    *(1/12): ansible-2.2.1.0–1.el7.noarch.rpm | 4.6 MB 00:00:00*

    *(2/12): libtomcrypt-1.17–23.el7.x86_64.rpm | 224 kB 00:00:00*

    *(3/12): libtommath-0.42.0–4.el7.x86_64.rpm | 35 kB 00:00:00*

    *(4/12): python-httplib2–0.7.7–3.el7.noarch.rpm | 70 kB 00:00:00*

    *(5/12): python-keyczar-0.71c-2.el7.noarch.rpm | 218 kB 00:00:00*

    *(6/12): python-jinja2–2.7.2–2.el7.noarch.rpm | 516 kB 00:00:00*

    *(7/12): python2-crypto-2.6.1–13.el7.x86_64.rpm | 476 kB 00:00:00*

    *(8/12): python-babel-0.9.6–8.el7.noarch.rpm | 1.4 MB 00:00:00*

    *(9/12): python2-ecdsa-0.13–4.el7.noarch.rpm | 83 kB 00:00:00*

    *(10/12): python-markupsafe-0.11–10.el7.x86_64.rpm | 25 kB 00:00:00*

    *(11/12): python2-paramiko-1.16.1–2.el7.noarch.rpm | 258 kB 00:00:00*

    *(12/12): sshpass-1.06–1.el7.x86_64.rpm | 21 kB 00:00:00*

*— — — — — — — — — — — — — — — — — — — — — — — — — — — — — — —*

*Once it’s installed, to check the version of Ansible*

    *#** ansible — version***

    *ansible 2.2.1.0*

    *config file = /etc/ansible/ansible.cfg*

    *configured module search path = Default w/o overrides*

*First step is to update **/etc/ansible/ansible.cfg** and uncomment these two entries*

    *# Location where ansible look for hosts
    inventory = /etc/ansible/hosts*

    *# Running command as a root user 
    sudo_user = root*

*NOTE: As there is no daemon running so **we don’t need to restart any service**, everytime we run any ansible command it read it’s config files*

*Now we need to define the hosts where we want to run ansible command and that is defined under **/etc/ansible/hosts***

    *# cat /etc/ansible/hosts*

    *[local]*

    *localhost*

*Typically we run ansible as a non-root user*

    *# useradd ansible*

    *# passwd ansible*

    *#To add sudoers entry
    visudo*

    *and add this entry*

    *ansible ALL=(ALL)       NOPASSWD: ALL*

***Ansible Modules***
[**Module Index — Ansible Documentation**
*Edit description*docs.ansible.com](http://docs.ansible.com/ansible/modules_by_category.html)

*Now in my scenario I am using two machines*

* *Ansible Master(192.168.0.29)*

* *Ansible Slave(192.168.0.22)*

![](https://cdn-images-1.medium.com/max/2000/1*VDs8f6t36DlH29ozbiK7OA.png)

*As mentioned earlier I am going to run ansible as ansible user and for that purpose I already transferred ssh key between two server*

    *bash-4.2$ ssh-copy-id 192.168.0.22*

    *The authenticity of host ‘192.168.0.22 (192.168.0.22)’ can’t be established.*

    *ECDSA key fingerprint is c6:ff:1d:14:c3:0f:92:ce:3c:e4:52:49:a1:16:1a:b7.*

    *Are you sure you want to continue connecting (yes/no)? yes*

    */bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed*

    */bin/ssh-copy-id: INFO: 1 key(s) remain to be installed — if you are prompted now it is to install the new keys*

    *ansible@192.168.0.22’s password:*

    *Number of key(s) added: 1*

    *Now try logging into the machine, with: “ssh ‘192.168.0.22’”*

    *and check to make sure that only the key(s) you wanted were added.*

    *-bash-4.2$ ssh 192.168.0.22*

    *[ansible@ansible-slave ~]$ exit*

    *logout*

***Ansible Commands***

*It’s time to run first ansible command(command are different from playbook)*

*In my hosts file, I only have one host*

    *cat /etc/ansible/hosts |grep -v \#*

    *[myserver]*

    *192.168.0.22*

*Now the first module I am going to check is the **ping module**,which will check if server is up and running(if we receive the pong back)*

    *$ **ansible myserver -m ping***

    *192.168.0.22 | SUCCESS => {*

    *“changed”: false,*

    ***“ping”: “pong”***

    *}*

* ***-m : stand for module***

* *myserver is defined inside /etc/ansible/hosts file*

*To list out the content of ansible home directory*

    *$ ansible all -a “ls -al /home/ansible”*

    *192.168.0.22 | SUCCESS | rc=0 >>*

    *total 16*

    *drwx — — — 4 ansible ansible 111 Apr 15 23:14 .*

    *drwxr-xr-x. 3 root root 21 Apr 15 22:44 ..*

    *drwx — — — 3 ansible ansible 17 Apr 15 23:14 .ansible*

    *-rw — — — — 1 ansible ansible 5 Apr 15 23:00 .bash_history*

    *-rw-r — r — 1 ansible ansible 18 Aug 2 2016 .bash_logout*

    *-rw-r — r — 1 ansible ansible 193 Aug 2 2016 .bash_profile*

    *-rw-r — r — 1 ansible ansible 231 Aug 2 2016 .bashrc*

    *drwx — — — 2 ansible ansible 29 Apr 15 23:00 .ssh*

* ***-a** ‘ARGUMENTS’, **— args=**’ARGUMENTS’ (The ARGUMENTS to pass to the module)*

*If we try to run any command which requires root level privilege it will fail*

    ***$ ansible all -a “tail /var/log/messages”***

    *192.168.0.22 | FAILED | rc=1 >>*

    *tail: cannot open ‘/var/log/messages’ for reading: Permission denied*

*Now pass -s option*

* *s, — **sudo** run operations with **sudo** (nopasswd)*

    ***ansible all -s -a “tail /var/log/messages”***

    *192.168.0.22 | SUCCESS | rc=0 >>*

    *Apr 15 23:23:08 ansible-slave ansible-command: Invoked with warn=True executable=None _uses_shell=False _raw_params=tail /var/log/messages removes=None creates=None chdir=None*

    *Apr 15 23:24:38 ansible-slave systemd-logind: Removed session 8.*

    *Apr 15 23:24:38 ansible-slave systemd: Removed slice user-1000.slice.*

    *Apr 15 23:24:38 ansible-slave systemd: Stopping user-1000.slice.*

    *Apr 15 23:24:50 ansible-slave systemd: Created slice user-1000.slice.*

    *Apr 15 23:24:50 ansible-slave systemd: Starting user-1000.slice.*

    *Apr 15 23:24:50 ansible-slave systemd: Started Session 9 of user ansible.*

    *Apr 15 23:24:50 ansible-slave systemd-logind: New session 9 of user ansible.*

    *Apr 15 23:24:50 ansible-slave systemd: Starting Session 9 of user ansible.*

    *Apr 15 23:24:50 ansible-slave ansible-command: Invoked with warn=True executable=None _uses_shell=False _raw_params=tail /var/log/messages removes=None creates=None chdir=None*

*Now lets copy file from local machine to remote system using **copy module***

    ***ansible all -m copy -a “src=ansibletest dest=/tmp”***

    *192.168.0.22 | SUCCESS => {*

    *“changed”: true,*

    *“checksum”: “da39a3ee5e6b4b0d3255bfef95601890afd80709”,*

    *“dest”: “/tmp/ansibletest”,*

    *“gid”: 1000,*

    *“group”: “ansible”,*

    *“md5sum”: “d41d8cd98f00b204e9800998ecf8427e”,*

    *“mode”: “0664”,*

    *“owner”: “ansible”,*

    *“size”: 0,*

    *“src”: “/home/ansible/.ansible/tmp/ansible-tmp-1492324349.54–146583309679325/source”,*

    *“state”: “file”,*

    *“uid”: 1000*

    *}*

*We can also install package using ansible use yum module*

    ***ansible localhost -s -m yum -a “name=nfs-utils state=latest”***

    *127.0.0.1 | SUCCESS => {*

    *“changed”: true,*

    *“msg”: “”,*

    *“rc”: 0,*

    *“results”: [*

*To remove that package set **state=absent***

    *ansible localhost -s -m yum -a “name=nfs-utils state=absent”*

    *127.0.0.1 | SUCCESS => {*

    *“changed”: true,*

    *“msg”: “”,*

    *“rc”: 0,*

    *“results”: [*

    *“Loaded plugins: fastestmirror\nResolving Dependencies\n → Running transaction check\n — -> Package nfs-utils.x86_64 1:1.3.0–0.33.el7_3 will be erased\n → Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package Arch Version Repository Size\n================================================================================\nRemoving:\n nfs-utils x86_64 1:1.3.0–0.33.el7_3 @updates 1.0 M\n\nTransaction Summary\n================================================================================\nRemove 1 Package\n\nInstalled size: 1.0 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n Erasing : 1:nfs-utils-1.3.0–0.33.el7_3.x86_64 1/1 \nwarning: file /var/lib/nfs/v4recovery: remove failed: No such file or directory\nwarning: file /var/lib/nfs/statd/sm.bak: remove failed: No such file or directory\nwarning: file /var/lib/nfs/statd/sm: remove failed: No such file or directory\nwarning: file /var/lib/nfs/statd: remove failed: No such file or directory\n Verifying : 1:nfs-utils-1.3.0–0.33.el7_3.x86_64 1/1 \n\nRemoved:\n nfs-utils.x86_64 1:1.3.0–0.33.el7_3 \n\nComplete!\n”*

    *]*

    *}*

*To create a user use **user module***

    ***$ ansible localhost -s -m user -a “name=testuser”***

    *127.0.0.1 | SUCCESS => {*

    *“changed”: true,*

    *“comment”: “”,*

    *“createhome”: true,*

    *“group”: 1001,*

    *“home”: “/home/testuser”,*

    *“name”: “testuser”,*

    *“shell”: “/bin/bash”,*

    *“state”: “present”,*

    *“system”: false,*

    *“uid”: 1001*

    *}*

*To remove a user**(set state=absent)***

    ***ansible localhost -s -m user -a “name=testuser state=absent”***

    *127.0.0.1 | SUCCESS => {*

    *“changed”: true,*

    *“force”: false,*

    *“name”: “testuser”,*

    *“remove”: false,*

    *“state”: “absent”*

    *}*

***Ansible Playbook***

*As per ansible doc*
> # “Playbooks are Ansible’s configuration, deployment, and orchestration language. They can describe a policy you want your remote systems to enforce, or a set of steps in a general IT process.
> # If Ansible modules are the tools in your workshop, playbooks are your instruction manuals, and your inventory of hosts are your raw material.”

    *— — # This is an example playbook to install Apache on Centos*

    *- hosts: myserver*

    *remote_user: ansible*

    *become: yes*

    *become_method: sudo*

    *connection: ssh*

    *gather_facts: yes*

    *tasks:*

    *- name: Installing Apache*

    *yum:*

    *name: httpd*

    *state: latest*

    *notify:*

    *- startservice*

    *handlers:*

    *- name: startservice (notify and handler name should match)*

    *service:*

    *name: httpd*

    enabled: yes

    *state: restarted*

*Let’s go through playbook step by step*

1. *This defined our hosts section*

* ***hosts**: which hosts I want to run this playbook(generally we run it on groups defined under /etc/ansible/hosts)*

* ***remote_user**: remote user we going to operate(i.e ansible user which is a privileged user)*

* ***become**: yes(means root user, as the command we want to execute on remote host need root *privilege*)*

* ***become_method**: sudo(it’s optional and default)*

* ***connection**: ssh(by default connection type is ssh we can use parimiko or local connection)*

* ***gather_facts**: yes(facts that are returned when it started running playbook against the hosts eg:OS)*

*2. **tasks**: This is the major section*

* ***name**: is just the heading*

* ***yum**: is the module*

* ***name**: name of the module*

* ***state**: of the package we want to install*

* ***notify**: in the case of centos service is not started automatically,so we need to notify to start the service after install*

* ***handlers**: handlers is what we notify*

* ***service**: what service we want to start*

*NOTE: In this case, handler is only going to run if package installation is sucessful*

*To run the playbook*

    *$ ansible-playbook example.yaml*

    *PLAY [myserver] *****************************************************************

    *TASK [setup] ********************************************************************

    *ok: [192.168.0.22]*

    *TASK [Installing Apache] ********************************************************

    *changed: [192.168.0.22]*

    *RUNNING HANDLER [startservice] **************************************************

    *changed: [192.168.0.22]*

    *PLAY RECAP **********************************************************************

    *192.168.0.22 : ok=3 changed=2 unreachable=0 failed=0*

***Ansible gathering facts***

*To see the list of hosts configured in our environment*

    *ansible all — list-hosts*

    *hosts (2):*

    *127.0.0.1*

    *192.168.0.22*

*So when we asked ansible to run a playbook on a specific hosts it need some way to know whether he can run those play on that specific host and for that he gather facts about that node*

    *ansible myserver -m setup*

*Now to filter specific information **eg:ipv4 address***

    *ansible myserver -m setup -a ‘filter=*ipv4*’*

    *192.168.0.22 | SUCCESS => {*

    *“ansible_facts”: {*

    *“ansible_all_ipv4_addresses”: [*

    *“192.168.0.22”*

    *],*

    *“ansible_default_ipv4”: {*

    *“address”: “192.168.0.22”,*

    *“alias”: “enp0s3”,*

    *“broadcast”: “192.168.0.255”,*

    *“gateway”: “192.168.0.1”,*

    *“interface”: “enp0s3”,*

    *“macaddress”: “08:00:27:d3:7c:c1”,*

    *“mtu”: 1500,*

    *“netmask”: “255.255.255.0”,*

    *“network”: “192.168.0.0”,*

    *“type”: “ether”*

    *}*

    *},*

    *“changed”: false*

    *}*

*We can save these facts to a **directory***

    *ansible myserver -m setup — tree myfacts*

    *ls -l myfacts/*

    *-rw-rw-r — 1 ansible ansible 12381 Apr 18 22:52 192.168.0.22*

*We can also perform variable substitution in case of ansible, let say we want to install telnet package on remote host*

    *— — #This is an example playbook to install Apache on Centos*

    *- hosts: **‘{{ myhosts }}’***

    *remote_user: ansible*

    *become: yes*

    *become_method: sudo*

    *connection: ssh*

    *gather_facts: **‘{{ gather }}’***

    *vars:*

    ***myhosts: myserver***

    ***gather: yes***

    ***pkg: telnet***

    *tasks:*

    *- name: Installing telnet package*

    *yum:*

    *name:** ‘{{ pkg }}’***

    *state: latest*

*As you can see here we have defined three variables*

* *myhosts*

* *gather*

* *pkg*

*and then we are simply calling these variables in our script*

*Output*

    *-bash-4.2$ ansible-playbook example.yaml*

    *PLAY [myserver] *****************************************************************

    *TASK [setup] ********************************************************************

    *ok: [192.168.0.22]*

    *TASK [Installing telnet package] ************************************************

    *changed: [192.168.0.22]*

    *PLAY RECAP **********************************************************************

    *192.168.0.22 : ok=2 changed=1 unreachable=0 failed=0*

*We can also specify these variables on command line rather then making it as a part of playbook*

    *-bash-4.2$ ansible-playbook example.yaml — extra-vars “myhosts=myserver gather=yes pkg=telnet”*

    *PLAY [myserver] *****************************************************************

    *TASK [setup] ********************************************************************

    *ok: [192.168.0.22]*

    *TASK [Installing telnet package] ************************************************

    *changed: [192.168.0.22]*

    *PLAY RECAP **********************************************************************

    *192.168.0.22 : ok=2 changed=1 unreachable=0 failed=0*

*We can also add debug statement to get more detailed output and as the name mentioned it help you to debug your playbook*

    *— — #This is an example playbook to install Apache on Centos*

    *- hosts: ‘{{ myhosts }}’*

    *remote_user: ansible*

    *become: yes*

    *become_method: sudo*

    *connection: ssh*

    *gather_facts: ‘{{ gather }}’*

    *vars:*

    *myhosts: myserver*

    *gather: yes*

    *pkg: telnet*

    *tasks:*

    *- name: Installing telnet package*

    *yum:*

    *name: ‘{{ pkg }}’*

    *state: latest*

    ***register: result***

    ***- debug: var=result***

*and the detailed output will look like this*

    *$ ansible-playbook example.yaml*

    *PLAY [myserver] *****************************************************************

    *TASK [setup] ********************************************************************

    *ok: [192.168.0.22]*

    *TASK [Installing telnet package] ************************************************

    *changed: [192.168.0.22]*

    *TASK [debug] ********************************************************************

    *ok: [192.168.0.22] => {*

    *“result”: {*

    *“changed”: true,*

    *“msg”: “Warning: RPMDB altered outside of yum.\n”,*

    *“rc”: 0,*

    *“results”: [*

    *“Loaded plugins: fastestmirror\nLoading mirror speeds from cached hostfile\n * base: repos-lax.psychz.net\n * epel: mirror.sjc02.svwh.net\n * extras: mirror.pac-12.org\n * updates: repo1.sea.innoscale.net\nResolving Dependencies\n → Running transaction check\n — -> Package telnet.x86_64 1:0.17–60.el7 will be installed\n → Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package Arch Version Repository Size\n================================================================================\nInstalling:\n telnet x86_64 1:0.17–60.el7 base 63 k\n\nTransaction Summary\n================================================================================\nInstall 1 Package\n\nTotal download size: 63 k\nInstalled size: 113 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n Installing : 1:telnet-0.17–60.el7.x86_64 1/1 \n Verifying : 1:telnet-0.17–60.el7.x86_64 1/1 \n\nInstalled:\n telnet.x86_64 1:0.17–60.el7 \n\nComplete!\n”*

    *]*

    *}*

    *}*

    *PLAY RECAP **********************************************************************

    *192.168.0.22 : ok=3 changed=1 unreachable=0 failed=0*

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
