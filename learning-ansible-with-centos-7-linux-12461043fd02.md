
# Learning Ansible with CentOS 7 Linux

For those learning Ansible this is a quick document on how to install, setup, and use Ansible for Linux Automation and Configuration Management. In this document you will learn the basics of an Ansible Playbook and learn how to automate such things as systemd unit files, cron files, executing external scripts from an Ansible Playbook, install software, open firewalld ports, add and remove Linux users, configure Linux logical volumes, and much more.

Ansible is similar to Chef and Puppet yet Ansible works without an agent running on the Linux client. Using **ssh **and **sudo **access** **Ansible connects and manages Linux clients from an Ansible server.

To learn Ansible automation with this document have a CentOS 7 server created as the Ansible automation server and CentOS 7 servers created as the Ansible clients. You can do this with your favorite virtualization tool such as KVM, Virtual Box, VMWare, or whatever.

You could have the Ansible server named as **ansibleserver** and the Ansible clients named as **ansibleclient1, ansibleclient2,….ansibleclientN**.

On the Ansible server install EPEL by executing as root:

    yum install [https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm](https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm)

To install Ansible execute as root:

    yum install ansible

Using the EPEL version of Ansible will install the latest Ansible version which is necessary as the Ansible Playbooks used in this document utilize Ansible modules that need Ansible version 2.5 or greater. Using Ansible from EPEL is the free version of Ansible. There is a paid for version from Red Hat which you can read about here: [Red Hat Ansible Tower](https://www.ansible.com/products/tower).

On the Ansible CentOS 7 server create a login named lnxcfg**.** The lnxcfg login will be used for Ansible Playbook execution which will manipulate, modify, and configure Ansible client servers. On the Ansible server you could create a lnxcfg userid like this:

    groupadd -g 2002 lnxcfg
    useradd -u 2002 -g 2002 -c "Ansible Automation Account" -s /bin/bash -m -d /home/lnxcfg lnxcfg

As root edit the /etc/ansible/ansible.cfg file and change the inventory line to:

    inventory = /home/lnxcfg/hosts

This will allow us to define the Ansible client system inventory in the local lnxcfg home directory.

In addition, to stop Ansible from prompting to add a client server to the ssh known_hosts you can add the following to the /etc/ansible/ansible.cfg file:

    host_key_checking = False

On the Ansible server as the lnxcfg login create a flat file named **hosts **which will house the Ansible client inventory names. For example a /home/lnxcfg/hosts file could look like this:

    [my_hosts]
    ansibleclient1
    ansibleclient2
    ansibleclient3
    [test_hosts]
    ansibletest1

On the Ansible server as the lnxcfg login create the following directory structure:

    playbooks 
    scripts
    templates

To do this execute as the login lnxcfg:

    mkdir -p {playbooks,scripts,templates}

Ansible uses **ssh** to login to all of its inventory clients. Using **ssh-keys**, **sudo access**, and the lnxcfg login id you will allow Ansible to execute on the Ansible client systems. The Ansible Playbooks will** **be able to configure, maintain, manipulate, and manage the CentOS 7 client Linux servers.** **As the lnxcfg user create an **ssh-key **that will be used by Ansible to ssh into its clients. Execute:

    ssh-keygen

press <enter> at all the default prompts.

The next step is to create the lnxcfg login on all **Ansible client systems**. You will need the **Ansible server’s** lnxcfg user’s .**ssh/id_rsa.pub** string for this purpose.

You could do this through a Linux Kickstart file for the Ansible client systems or the below Bash script named setup-lnxcfg-user could help with this process. The script creates the lnxcfg login id, sets up s**sh-keys**, and **sudo **access on the Ansible client systems.

Provide your own lnxcfg login** **password for the clients and the lnxcfg .ssh/id_rsa.pub string you created earlier in the script below. Don’t forget to chmod 755 the setup-lnxcfg-user** **bash script prior to executing.

    #!/bin/bash
    # setup-lnxcfg-user
    # create lnxcfg user for Ansible automation
    # and configuration management

    # create lnxcfg user
    getentUser=$(/usr/bin/getent passwd lnxcfg)
    if [ -z "$getentUser" ]
    then
      echo "User lnxcfg does not exist.  Will Add..."
      /usr/sbin/groupadd -g 2002 lnxcfg
      /usr/sbin/useradd -u 2002 -g 2002 -c "Ansible Automation Account" -s /bin/bash -m -d /home/lnxcfg lnxcfg

    echo "lnxcfg:**<PUT IN YOUR OWN lnxcfg PASSWORD>**" | /sbin/chpasswd

    mkdir -p /home/lnxcfg/.ssh

    fi

    # setup ssh authorization keys for Ansible access 
    echo "setting up ssh authorization keys..."

    cat << 'EOF' >> /home/lnxcfg/.ssh/authorized_keys
    **<PUT IN YOUR OWN .ssh/id_rsa.pub SSH RSA KEY>**
    EOF

    chown -R lnxcfg:lnxcfg /home/lnxcfg/.ssh
    chmod 700 /home/lnxcfg/.ssh

    # setup sudo access for Ansible
    if [ ! -s /etc/sudoers.d/lnxcfg ]
    then
    echo "User lnxcfg sudoers does not exist.  Will Add..."
    cat << 'EOF' > /etc/sudoers.d/lnxcfg
    User_Alias ANSIBLE_AUTOMATION = %lnxcfg
    ANSIBLE_AUTOMATION ALL=(ALL)      NOPASSWD: ALL
    EOF
    chmod 400 /etc/sudoers.d/lnxcfg
    fi

    # disable login for lnxcfg except through 
    # ssh keys
    cat << 'EOF' >> /etc/ssh/sshd_config
    Match User lnxcfg
            PasswordAuthentication no
            AuthenticationMethods publickey

    EOF

    # restart sshd
    systemctl restart sshd

    # end of script

Notice the script disables ssh logins for the lnxcfg user on an Ansible client except through ssh-keys. This is a good idea, for security reasons, as the lnxcfg user has sudo no password privileges, which is necessary to successfully execute Ansible automation without human interaction against a client.

To execute the script on an **Ansible client **from the **Ansible server** you could execute:

    ssh root@ansibleclient1 'bash -s' < setup-lnxcfg-user

If using a Linux Kickstart configuration file which is probably the preferred way of creating the lnxcfg user while building the **Ansible client systems **you could always carve out the script from above and put the majority of the Bash code in the Kickstart’s %post section.

Now on the actual Ansible Playbooks. Playbooks are the nuts and bolts of Ansible automation. This is where we configure, manipulate, and maintain Ansible clients. For better organization you could put all of your Playbooks in the /home/lnxcfg/playbooks directory.

If you look through the Ansible Playbooks listed below at the top of each one you will see three consecutive dashes. The dashes are only necessary when executing Playbooks in the same stream. For example if you include multiple Playbooks in the same file or pipe multiple playbooks together the three dashes would separate them. Even though I have one Playbook per file for convention reasons I still include the three dashes.

The **hosts** statement in the Playbook processes all listed inventory within the bracketed title in the **Ansible hosts file** (/home/lnxcfg/hosts). In the example of the host’s file above a Playbook that uses my_hosts would execute the Playbook against servers ansibleclient1, ansibleclient2, and ansibleclient3.

The Playbook will use the lnxcfg userid to **sudo to root** and process the Playbook’s **tasks**.

To execute a playbook issue the command:

    ansible-playbook playbooks/performYumUpdateRebootServer.yml

This first playbook named performYumUpdateRebootServer.yml performs a yum update and then reboots each client in the host’s inventory. The **state** as **latest** lets Ansible know we want to update all packages to the latest release. Since new kernels require a reboot to take effect the Playbook reboots each client.

    ---
    # performYumUpdateRebootServer.yml
    # performs yum update and reboots server
    - name: perform yum update / reboot server
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root
     
      tasks:
            - name: Perform yum update of all packages
              yum:
                name: '*'
                state: latest

            - name: Reboot server
              command: /sbin/shutdown -r +1
              ignore_errors: true

While the above is a way you could patch and reboot a server with Ansible 2.7 is the brand new reboot module which simplifies rebooting as follows:

    ---
    # performYumUpdateRebootServer.yml
    # performs yum update and reboots server
    - name: perform yum update / reboot server
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root
     
      tasks:
            - name: Perform yum update of all packages
              yum:
                name: '*'
                state: latest

            - name: Reboot machine
              reboot:

Here is how to create a directory using Ansible. The following Playbook is only using the lnxcfg user as the owner and group for example purposes.

    ---
    # createDirectory.yml
    # create a directory and assign permissions to a user
    - name: create directory
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

      tasks:
            - name: add a directory
              # set directory attributes.  
              # using lnxcfg as example user/group
              file:
                 owner: lnxcfg #put in what ever user you need
                 group: lnxcfg #put in what ever group you need
                 mode: 0755
                 recurse: yes
                 path:  /var/ansible_directory
                 state: directory

Here is a playbook to open the CentOS 7 firewall and install httpd. The following Playbook uses a template to create the index.html file. In addition, the Playbook uses a variable *templateSource *to set the template location.

    ---
    # installHttpdOpenFirewall.yml
    # opens firewall / installs httpd / uses Ansible Jinja2 
    # templating to create index.html
    - name: install httpd / open http firewall port
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

      vars:
        templateSource: '/home/lnxcfg/templates/index.j2'
     
      tasks:
            - name: open http firewall port
              firewalld:
                  service: http 
                  zone: public
                  immediate: yes # this is the firewall-cmd --reload
                  permanent: true
                  state: enabled

            - name:  install httpd
              yum:
                name: httpd
                state: present
             
            - name: httpd template for index.html
              template:
                src: "{{ templateSource }}"
                dest: /var/www/html/index.html
                owner: root
                group: root
                mode: 0644
              notify: restart httpd

      handlers:
            - name: restart httpd
              service:
                    name: httpd
                    enabled: yes
                    state: restarted

The above Playbook uses Ansible Jinja2 templating to render variables within the below index.j2 template. Ansible knows information about the client as it has predefined system variables known as Facts that can be interpreted inside a Jinja2 template. To see a list of predefined Facts about a client execute the following from the Ansible server with the hostname of a specific Ansible client:

    ansible **<put in a hostname>** -m setup

Notice variables are interpolated within the {{ and }} tags in the following Jinja2 index.j2 template:

    <html>
        <body>
            <pre>
            Ansible Node: {{ ansible_nodename }}
            FQDN: {{ ansible_fqdn }} 
            IP Address: {{ ansible_eth0.ipv4.address }}        
            Distribution: {{ ansible_distribution }} {{ ansible_distribution_release }} {{ ansible_distribution_version }}
            Kernel: {{ ansible_kernel }} 
            Python Version: {{ ansible_python_version }}
            CPUs: {{ ansible_processor_vcpus }}
            Memory: {{ ansible_memtotal_mb }} MB  
            Virtualization: {{ ansible_virtualization_type }}
            Ansible User Information:
            user: {{ ansible_env.SUDO_USER }}
            uid: {{ ansible_env.SUDO_UID }} 
            gid: {{ ansible_env.SUDO_GID }}
            home: {{ ansible_env.HOME }} 
            pwd: {{ ansible_env.PWD }} 
            </pre>
       </body>
    </html>

The web output in a browser from the index.j2 which creates the index.html file looks something like:

    Ansible Node: ansibleclient1
    FQDN: ansibleclient1.<REDACTED>
    IP Address: <REDACTED>       
    Distribution: CentOS Core 7.5.1804
    Kernel: 3.10.0-862.3.2.el7.x86_64 
    Python Version: 2.7.5
    CPUs: 4
    Memory: 3789 MB  
    Virtualization: kvm
    Ansible User Information:
    user: lnxcfg
    uid: 2002 
    gid: 2002
    home: /root 
    pwd: /home/lnxcfg

The following Playbook creates a partition, LVM logical volume, directory, mounts the directory, and even adds to the /etc/fstab. Notice Ansible variables are defined in the *vars *section of the Playbook then interpolated between the {{ and }} tags. Variable concatenation with a string is easy as shown here: “/dev/{{ volumeGroup }}/{{ logicalVolume }}”.

    ---
    # createPartitionAndLVM.yml
    # Creates a partion / Logical Volume / Directory / Mounts directory 
    - name: Create Partition And LVM 
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

    # Define variables
      vars:
        device:           '/dev/sdb'
        volumeGroup:      'vg_learn_ansible'
        logicalVolume:    'lv_learn_ansible'
        directoryPath:    '/var/learn_ansible'
        size:             '100%FREE'
        filesystemDevice: "/dev/{{ volumeGroup }}/{{ logicalVolume }}"
        fstype:           'xfs'
        directoryOwner:   'lnxcfg' #put in what ever users you need
        directoryGroup:   'lnxcfg' #put in what ever group you need
        directoryMode:    '0755'        
        mountOptions:     'defaults'

      tasks:
            - name: Create Partion
              # Using gpt partition type.  Creating LVM based partition.
              parted:
                device: "{{ device }}"
                number: 1
                label: gpt
                part_type: primary 
                flags: [ lvm ]
                state: present

            - name: Create LVM volume group
              # Create Physical Volume and Volume Group
              lvg:
                 pvs: "{{ device }}1" 
                 state: present
                 vg: "{{ volumeGroup }}"
            
            - name: Create LVM logical volume
              # Create Logical Volume
              lvol:
                vg: "{{ volumeGroup }}"
                lv: "{{ logicalVolume }}"
                size: "{{ size }}"

            - name: Create Filesystem
              # Create filesystem
              filesystem:
                fstype: "{{ fstype }}"
                dev: "{{ filesystemDevice }}"

            - name: Create Directory
              # Create filesystem directory
              file:
                 path: "{{ directoryPath }}" 
                 state: directory

            - name: Mount directory to LVM
              # This will also add to /etc/fstab 
              mount:
                path: "{{ directoryPath }}"
                src: "{{ filesystemDevice }}" 
                fstype: "{{ fstype }}"
                opts: "{{ mountOptions }}" 
                state: mounted

            - name: Change permission on newly created directory
              # Change permissions on newly created and mounted
              # filesystem
              file:
                 owner: "{{ directoryOwner }}"
                 group: "{{ directoryGroup }}"
                 mode: "{{ directoryMode }}" 
                 recurse: yes
                 path: "{{ directoryPath }}"

Passing variables to playbooks is also easy instead of defining them within the Playbook. To do this from the command line use the ansible-playbook command with the -e or — extra-args argument. As an example you could pass a host variable to the hosts statement in the Playbook. With the example from above if we changed the hosts statement to hosts: “{{ my_hosts }}” we could then pass the host name to the Playbook as:

    ansible-playbook -e my_hosts=ansibleclient4 createPartitionAndLVM.yml

Passing variables to a Playbook can also be accomplished through the inventory hosts file. Here is an example four inventory server items with variable names and their assignments in the inventory host file:

    [lvm_hosts]
    ansibleclient1 device=/dev/vdb volumeGroup=vg_ansible_test logicalVolume=lv_ansible_test directoryPath=/var/ansible/test fstype=xfs size=100%FREE directoryOwner=lnxcfg directoryGroup=lnxcfg directoryMode=0755 mountOptions=defaults
    ansibleclient2 device=/dev/vdd volumeGroup=vg_ansible_test logicalVolume=lv_ansible_test directoryPath=/var/ansible/test fstype=xfs size=100%FREE directoryOwner=lnxcfg directoryGroup=lnxcfg directoryMode=0755 mountOptions=defaults
    ansibleclient3 device=/dev/vdc volumeGroup=vg_ansible_test logicalVolume=lv_ansible_test directoryPath=/var/ansible/test fstype=xfs size=100%FREE directoryOwner=lnxcfg directoryGroup=lnxcfg directoryMode=0755 mountOptions=defaults
    ansibleclient4 device=/dev/vdb volumeGroup=vg_ansible_test logicalVolume=lv_ansible_test directoryPath=/var/ansible/test fstype=xfs size=100%FREE directoryOwner=lnxcfg directoryGroup=lnxcfg directoryMode=0755 mountOptions=defaults

A modified createPartitionAndLVM.yml which uses the passed variables from the inventory host file is as follows. Notice the vars section has been pruned down since we are passing variables into the Playbook from the inventory hosts file:

    ---
    # createPartitionAndLVM.yml
    # Creates a partion / Logical Volume / Directory / Mounts directory 
    - name: Create Partition And LVM 
      hosts: lvm_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

    # Define variables
      vars:
        filesystemDevice: "/dev/{{ volumeGroup }}/{{ logicalVolume }}"

      tasks:
            - name: Create Partion
              # Using gpt partition type.  Creating LVM based partition.
              parted:
                device: "{{ device }}"
                number: 1
                label: gpt
                part_type: primary 
                flags: [ lvm ]
                state: present

            - name: Create LVM volume group
              # Create Physical Volume and Volume Group
              lvg:
                 pvs: "{{ device }}1" 
                 state: present
                 vg: "{{ volumeGroup }}"
            
            - name: Create LVM logical volume
              # Create Logical Volume
              lvol:
                vg: "{{ volumeGroup }}"
                lv: "{{ logicalVolume }}"
                size: "{{ size }}"

            - name: Create Filesystem
              # Create filesystem
              filesystem:
                fstype: "{{ fstype }}"
                dev: "{{ filesystemDevice }}"

            - name: Create Directory
              # Create filesystem directory
              file:
                 path: "{{ directoryPath }}" 
                 state: directory

            - name: Mount directory to LVM
              # This will also add to /etc/fstab 
              mount:
                path: "{{ directoryPath }}"
                src: "{{ filesystemDevice }}" 
                fstype: "{{ fstype }}"
                opts: "{{ mountOptions }}" 
                state: mounted

            - name: Change permission on newly created directory
              # Change permissions on newly created and mounted
              # filesystem
              file:
                 owner: "{{ directoryOwner }}"
                 group: "{{ directoryGroup }}"
                 mode: "{{ directoryMode }}" 
                 recurse: yes
                 path: "{{ directoryPath }}"

You can also use variables to create conditional statements. The following Playbook, testDist.yml, creates a new file /etc/system-release using predefined system variables based on the distribution type and distribution version. If you executed this Playbook against a CentOS 6 server the when statement would skip the task:

    ---
    # testDist.yml
    # create /etc/system-release using conditional statement
    - name: test distribution and version
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

      tasks:
            - name: create file based on distribution 
              copy:
                 content: "{{ ansible_distribution }} version: {{ ansible_distribution_version }}"
                 dest: /etc/system-release
              when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "7"

The output of the /etc/system-release is:

    CentOS version: 7.5.1804

Next is a Playbook to configure Chronyd. The Playbook uses the NIST time source for Chronyd as defined in the following /etc/chrony.conf file. The NIST time sources are from the following source: [NIST Internet Time Servers](https://tf.nist.gov/tf-cgi/servers.cgi) and are used instead of the CentOS default time server.

Again the Playbook uses Ansible Jinja2 templating:

    ---
    # configChronydWithTemplate.yml
    # configure chronyd with nist time source. Uses Ansible Jinja2
    # templating to create /etc/chrony.conf
    - name: configure chronyd with a template
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root
      
      vars:
        chronySource:   ['time-a-g.nist.gov',
                          'time-b-g.nist.gov',
                          'time-c-g.nist.gov']
        templateSource: '/home/lnxcfg/templates/chrony.j2'

      tasks:
            - name:  install chronyd
              yum:
                name: chrony
                state: present

            - name:  chronyd template
              template:
                src: "{{ templateSource }}" 
                dest: /etc/chrony.conf
                owner: root
                group: root
                mode: 0644
              notify: restart chronyd

      handlers:
            - name: restart chronyd
              service:
                    name: chronyd
                    enabled: yes
                    state: restarted

Below is the Ansible Jinja2 chrony.j2 template which creates the /etc/chrony.conf file:

    # chrony.conf from ansible

    {% for var in chronySource %}
    server  {{ var }}
    {% endfor %}

    # Record the rate at which the system clock gains/losses time.
    driftfile /var/lib/chrony/drift

    # Allow the system clock to be stepped in the first three updates
    # if its offset is larger than 1 second.
    makestep 1.0 3

    # Enable kernel synchronization of the real-time clock (RTC).
    rtcsync

    # Enable hardware timestamping on all interfaces that support it.
    #hwtimestamp *

    # Increase the minimum number of selectable sources required to adjust
    # the system clock.
    #minsources 2

    # Allow NTP client access from local network.
    #allow 192.168.0.0/16

    # Serve time even if not synchronized to a time source.
    #local stratum 10

    # Specify file containing keys for NTP authentication.
    #keyfile /etc/chrony.keys

    # Specify directory for log files.
    logdir /var/log/chrony

    # Select which information is logged.
    #log measurements statistics tracking

The results of the above Playbook for /etc/chrony.conf:

    chrony.conf from ansible

    server  time-a-g.nist.gov
    server  time-b-g.nist.gov
    server  time-c-g.nist.gov

    .....

Replacing text in a file is made easy with Ansible. Imagine needing to find and replace text in the /etc/nsswitch.conf on hundreds to thousands of servers. The following Playbook uses the replace module to find and change the order of* ‘files sss’* to* ‘sss files’ *within /etc/nsswitch.conf*:*

    ---
    # configNSSwitch.yml
    # use replace to modify text in a file
    - name: configure nsswitch.conf
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

    tasks:
            - name:  change sss file 
              replace:
                path: /etc/nsswitch.conf
                regexp: '\s files sss' 
                replace: 'sss files'
                backup: yes

Next is an example of executing a Bash script from a Playbook. The Playbook installs the AIDE file based intrusion detection software and executes a script to initialize and configure it. The Playbook is idempotent and will only execute based on the status of the AIDE database:

    ---
    # configAIDE.yml
    # Install and configure file based intrusion
    # detection system.

    - name: AIDE file based intrusion detection system
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root
     
      tasks:
            - name:  install aide
              yum:
                name: aide
                state: present
            
            # setup idempotent result based on aide file
            - name: Check if /var/lib/aide/aide.db.gz exists
              stat:
                path: /var/lib/aide/aide.db.gz
              register: stat_result
      
            # execute exernal script to configure aide
            - name: configure aide
              block:
                - copy:
                   src: /home/lnxcfg/scripts/configAIDE      
                   dest: /tmp/configAIDE
                   owner: root
                   group: root
                   mode: 0755
                - command : '/tmp/configAIDE'
                - file:
                   state: absent
                   path: '/tmp/configAIDE'
              when: stat_result.stat.exists == False

Here is the /home/lnxcfg/scripts/configAIDE Bash script which is executed by the above Playbook. The script configures the AIDE software and creates an additional Bash script to execute AIDE daily:

    #!/bin/bash
    # configAide 
    # configure file based intrusion detection - AIDE
     
    /sbin/aide --init 
    echo "aide has been initialized"
    /bin/mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

    # create daily script to execute AIDE
    cat << 'EOF' > /usr/local/bin/aide-check 
    #!/bin/bash

    echo "executing $0 ......"

    /bin/nice -n 19 /sbin/aide --update

    /bin/rm -f /var/lib/aide/aide.db.gz
    /bin/mv -f /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

    logger -s 'AIDE DAILY FILE CHANGE AND INTRUSION DETECTION CHECK HAS EXECUTED'
    EOF

    chmod 700 /usr/local/bin/aide-check
    ln -sf /usr/local/bin/aide-check /etc/cron.daily/aide-check

    echo "aide-check cron job has been created"

While the above Ansible Playbook and external script being executed by Ansible works well it is an example of not allowing Ansible to take full control using templates. The above Ansible Playbook and external Bash script could be rewritten by breaking apart the Bash script and moving the code between the cat command and the EOF code block to an Ansible Jinja2 template. Here is a re-written or refactored version of the configAIDE.yml Playbook. Notice there are two new tasks in this refactored version of configAIDE.yml; the create daily script to execute AIDE daily and the create symbolic link to /etc/cron.daily:

    ---
    # configAIDE.yml
    # Install and configure file based intrusion
    # detection system. execute external script
    # for configuring AIDE
    # use the power of Ansible to create
    # exernal scripts from template and then 
    # create a symbolic link.

    - name: AIDE file based intrusion detection 
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root
     
      tasks:
            - name:  install aide
              yum:
                name: aide
                state: present

            # setup idempotent result based on aide file
            - name: Check if /var/lib/aide/aide.db.gz exists
              stat:
                path: /var/lib/aide/aide.db.gz
              register: stat_result
      
            # execute exernal script to configure aide
            - name: configure aide
              block:
                - copy:
                   src: /home/lnxcfg/scripts/configAIDE      
                   dest: /tmp/configAIDE
                   owner: root
                   group: root
                   mode: 0755
                - command : '/tmp/configAIDE'
                - file:
                   state: absent
                   path: '/tmp/configAIDE'
              when: stat_result.stat.exists == False

            - name: create daily script to execute AIDE  
              template:
                src:  /home/lnxcfg/templates/aide-check.j2
                dest: /usr/local/bin/aide-check
                owner: root
                group: root
                mode: 0755
            
            - name: create symbolic link to /etc/cron.daily
              file:
                src: /usr/local/bin/aide-check
                dest: /etc/cron.daily/aide-check 
                owner: root
                group: root
                state: link
                force: yes

Notice in the above Playbook The daily Bash script was moved to a Jinja2 template stored in the /home/lnxcfg/templates/ directory and is named /home/lnxcfg/templates/aide-check.j2. Also notice instead of the external Bash script configAide which created the symbolic link in /etc/cron.daily has been refactored so that the Ansible Playbook creates the symbolic link instead. Allowing Ansible to do the work for you takes full advantage of automation. The new refactored /home/lnxcfg/scripts/configAIDE Bash script which initializes and configures for AIDE for first use is below:

    #!/bin/bash
    # configAide 
    # configure file based intrusion detection - AIDE
     
    /sbin/aide --init 
    echo "aide has been initialized"
    /bin/mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

    echo "aide has been configured"

And now the Bash script which executes AIDE daily to report changes made to files is now an Ansible Jinja2 template. This moves code from a monolithic Bash script and allows Ansible to manage templates of Bash code. Here is the new refactored /home/lnxcfg/templates/aide-check.j2:

    #!/bin/bash

    echo "executing $0 ......"

    /bin/nice -n 19 /sbin/aide --update

    /bin/rm -f /var/lib/aide/aide.db.gz
    /bin/mv -f /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz

    logger -s 'AIDE DAILY FILE CHANGE AND INTRUSION DETECTION CHECK HAS EXECUTED

With the above refactored Playbook and Jinja2 templates we are making better use of the power of Ansible than just using Ansible as a Bash wrapper.

At times it is necessary to execute a Bash script on all s

Now on to cron: The following Playbook is executed every 30 minutes via a cron job from the ansibleserver. The Playbook resets the motd on all client servers in the Ansible inventory. If you removed or modified the /etc/motd file within 30 minutes Ansible would reset the file. The cron entry for user lnxcfg is as follows:

    */30 * * * * /bin/ansible-playbook /home/lnxcfg/playbooks/motd.yml 2>&1 | /usr/bin/logger -t playbook_motd

The motd Playbook uses Ansible Jinja2 templating as below:

    ---
    # motd.yml
    # configure /etc/motd using Ansible Jinja2 templating
    - name: configure /etc/motd
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

      vars:
        templateSource: '/home/lnxcfg/templates/motd.j2'
     
      tasks:
            - name:  set motd
              template:
                src: "{{ templateSource }}"
                dest: /etc/motd
                owner: root
                group: root
                mode: 0644

The motd.j2 which is the template for /etc/motd looks like:

    motd from ansible
     
    Ansible Node: {{ ansible_nodename }}
    FQDN: {{ ansible_fqdn }} 
    IP Address: {{ ansible_eth0.ipv4.address }}        
    Distribution: {{ ansible_distribution }} {{ ansible_distribution_release }} {{ ansible_distribution_version }}
    Kernel: {{ ansible_kernel }} 
    Python Version: {{ ansible_python_version }}
    CPUs: {{ ansible_processor_vcpus }}
    Memory: {{ ansible_memtotal_mb }} MB  
    Virtualization: {{ ansible_virtualization_type }}
    Ansible User Information:
    user: {{ ansible_env.SUDO_USER }}
    uid: {{ ansible_env.SUDO_UID }} 
    gid: {{ ansible_env.SUDO_GID }}
    home: {{ ansible_env.HOME }} 
    pwd: {{ ansible_env.PWD }}

The actual /etc/motd after Ansible Jinja2 template rendering looks like:

    motd from ansible
     
    Ansible Node: ansibleclient1
    FQDN: ansibleclient1.<REDACTED> 
    IP Address: <REDACTED>       
    Distribution: CentOS Core 7.5.1804
    Kernel: 3.10.0-862.3.2.el7.x86_64 
    Python Version: 2.7.5
    CPUs: 4
    Memory: 3789 MB  
    Virtualization: kvm
    Ansible User Information:
    user: lnxcfg
    uid: 2002 
    gid: 2002
    home: /root 
    pwd: /home/lnxcfg

Speaking of using cron Ansible has the ability to create cron data in either a users crontab or in the /etc/cron.d. Following is a Playbook that creates a cron job inside the /etc/cron.d directory:

    ---
    # createCron.yml
    # configure a simple cron inside /etc/cron.d 
    - name: create cron
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

      tasks:
            - name: Creates a cron file under /etc/cron.d
              cron:
                # creates ansible_cron_date file in /etc/cron.d
                # executes at the 10 minute of every hour
                name: lnxcfg cron date
                minute: 10
                user: lnxcfg
                job: "/bin/date >> /home/lnxcfg/ansible_cron_date"
                cron_file: ansible_cron_date
                state: present

On an Ansible client a file of ansible_cron_date is created inside of the /etc/cron.d directory with the contents of the file being:

    #Ansible: lnxcfg cron date
    10 * * * * lnxcfg /bin/date >> /home/lnxcfg/ansible_cron_date

Unfortunately at the current moment using minute: */10 which would get this cron to execute every ten minutes creates an error on both Ansible versions 2.5 and 2.6. The error is:

    ERROR! Syntax Error while loading YAML.
      did not find expected alphabetic or numeric character

    The error appears to have been in '/home/lnxcfg/playbooks/createCron.yml': line 17, column 22, but may
    be elsewhere in the file depending on the exact syntax problem.

    The offending line appears to be:

    name: lnxcfg cron date
                minute: */10
                         ^ here

Which is very unfortunate.

Now, back to Ansible.

Here is an example of managing systemd services. Below is a Playbook which creates a *simple* systemd service named configSystemdService.yml. This Playbook uses templates to create a Bash script and it’s corresponding systemd unit file which executes the Bash script. After a reboot or power on this particular systemd service will write the current date and time to a file. This Playbook and templates can be used as a systemd replacement for the old system V rc.local script:

    ---
    # configSystemdService.yml 
    # create a systemd service
    # for when CentOS 7 servers are 
    # rebooted; this is a rc.local 
    # replacement. Use a conditional 
    # statement to only execute 
    # playbook against CentOS 7 servers.
    - name: create systemd service
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

    tasks:

         - name: configure systemd-script service block
           block: 
             - template:
                 src:  /home/lnxcfg/templates/systemd-script.j2
                 dest: /usr/local/bin/systemd-script
                 owner: root
                 group: root
                 mode: 0755

             - template:
                 src:  /home/lnxcfg/templates/systemd-script.service.j2
                 dest: /etc/systemd/system/systemd-script.service
                 owner: root
                 group: root
                 mode: 0644

             - systemd:
                 # Enable systemd service and reload the 
                 # systemd daemon. Notice the task does not 
                 # start the systemd-script service.  This
                 # is because with this service we want it 
                 # to execute only after a reboot or 
                 # system power on.
                 name: systemd-script
                 enabled: yes
                 masked: no
                 daemon_reload: yes
           when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "7"

Notice the systemd task does not start the systemd service. This is because with this particular service we want it to execute only after a reboot, system power on, or the network is restarted. The Ansible Jinja2 template to create the Bash script named systemd-script.j2 is as follows. You could do more complex tasks in this Bash script such as starting Oracle databases, emailing your team or yourself that the server has been rebooted, and much more:

    #!/bin/bash
    echo "Systemd script: $(/bin/date)" > /home/lnxcfg/systemd-script-results

The Ansible Jinja2 template to create the *simple* systemd unit, systemd-script.service.j2 looks like:

    [Unit]
    Description = systemd-script service 
    After = network.target

    [Service]
    Type=forking
    ExecStart = /usr/local/bin/systemd-script
     
    [Install]
    WantedBy = multi-user.target

After execute the Playbook if you reboot the Ansible Linux clients you would see a file with the date in the file: /home/lnxcfg/systemd-script-results which looks like:

    Systemd script: Fri Aug 24 10:59:47 CDT 2018

Moving on, adding and removing a user on hundreds of Ansible clients is made easy with the follow Playbooks:

    ---
    # createUser.yml
    # create a user
    - name: create user
      hosts: my_hosts
      remote_user: lnxcfg 
      become: true
      become_method: sudo
      become_user: root

    # Define variables
      vars:
        login:              'dbuser01'
        gecos:              'Database User01 Account'
        shell:              '/bin/bash'
        supplementalGroups: 'database'
        # created password prior with:
        # python -c 'import crypt,getpass; print crypt.crypt(getpass.getpass())'
        password:    '<REDACTED>'

       tasks:
            - name: create a user
              user:
                name: "{{ login }}"  #login id
                comment: "{{ gecos }}" #gecos field
                shell: "{{ shell }}"
                password: "{{ password }}"
                groups: "{{ supplementalGroups }}"
                append: yes 
                update_password: always

            - name: Add sudoers file
              file:
                owner: root
                group: root
                mode: 0400
                path: "/etc/sudoers.d/{{ login }}"
                state: touch

            - name: Add suders data
              blockinfile:
                path: "/etc/sudoers.d/{{ login }}" 
                block: |
                  User_Alias {{ login|upper }} = %{{ login }}
                  {{ login|upper }} ALL=(ALL)      ALL

And to remove a user:

    ---
    # removeUser.yml
    # remove a user
    - name: remove user
      hosts: my_hosts
      remote_user: lnxcfg 
      become: true
      become_method: sudo
      become_user: root

      vars:
        login:   'dbuser01'

      tasks:
            - name: remove a user
              user:
                name: "{{ login }}"
                state: absent
                remove: yes #remove home directory

            - name: Remove sudoers file
              file:
                path: "/etc/sudoers.d/{{ login }}"
                state: absent

In addition to the above user management here is a way to add multiple users while reading those users from a file. The file structure is in YAML format and is stored in the /home/lnxcfg/playbooks/vars directory. Ansible provides an *include_vars* to achieve this. The below addUsers.yml file is used by the Ansible Playbook that follows:

    # addUsers.yml
    # Users to add
    user1:
      login: 'oracleuser1'
      gecos: 'Oracle User One Account'
      shell: '/bin/bash'
      supplementalGroups: 'database'
      password: '<REDACTED>'
    user2:
      login: 'oracleuser2'
      gecos: 'Oracle User Two Account'
      shell: '/bin/bash'
      supplementalGroups: 'database'
      password: '<REDACTED>'

And here is the Ansible Playbook that loops through the users:

    ---
    # createMultiepleUser.yml
    # create multiple users
    - name: create user
      hosts: my_hosts
      remote_user: lnxcfg 
      become: true
      become_method: sudo
      become_user: root

      tasks:
            - name: include addUsers.yml
              include_vars:
                file: addUsers.yml
                name: users

            - name: create a user
              user:
                name: "{{ item.value.login }}"  #login id
                comment: "{{ item.value.gecos }}"
                shell: "{{ item.value.shell }}"
                password: "{{ item.value.password }}"
                groups: "{{ item.value.supplementalGroups }}"
                append: yes 
                update_password: always 
              loop: "{{ lookup('dict', users) }}"

Finally, a Playbook to create a firewalld rich-rule:

    ---
    # firewalldRichRule.yml
    # use firewalld and rich rule  
    # forward port to another ip address 
    - name: forward port for port 80 
      hosts: my_hosts
      remote_user: lnxcfg
      become: true
      become_method: sudo
      become_user: root

      tasks:
            - name: rich rule
              firewalld:
                  rich_rule: 'rule family="ipv4" forward-port port="80" protocol="tcp" to-port="8080" to-addr="<REDACTED>"'
                  zone: public
                  immediate: yes # this is the firewall-cmd --reload
                  permanent: true
                  state: enabled

With the above Playbook to remove the firewalld rich-rule edit the Playbook and change the state from enabled to disabled and rerun the ansible-playbook firewalldRichRule.yml. The firewalld rich-rule will be removed from all the Ansible client servers in the inventory list.

Learning and using Ansible can be great fun. There is a lot more to it but hopefully this will get you started.

A next step might be to learn how to use Ansible roles with Ansible. To learn more you can go to: [https://medium.com/@brad.simonin/learning-ansible-and-ansible-roles-with-centos-7-linux-817406f7b542](https://medium.com/@brad.simonin/learning-ansible-and-ansible-roles-with-centos-7-linux-817406f7b542)
