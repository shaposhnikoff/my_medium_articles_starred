Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m102[39m, end: [33m113[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m26[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m8[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m68[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m289[39m, end: [33m322[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m194[39m, end: [33m204[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m130[39m, end: [33m152[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m202[39m, end: [33m245[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m83[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m9[39m }

# Mongo + Docker Swarm (Fully Automated Cluster)

When it comes to developing modern apps, working with MongoDB has become de-facto a standard among the NoSQL DBs, because of its flexible nature to easily start and rapidly develop apps. However, once your application’s user base starts to grow, the necessity for scaling your DB either due to Performance or to Redundancy (Data Availablity) becomes inevitable.

Besides the option to scale you DB vertically (more CPU, more RAM, more Disk, faster Network etc), MongoDB provides options to scale your DB horizontally via **ReplicaSet** (multiple copies of data), **Sharding** (splitting data among multiple servers) and **Query Routers** (multiple Routing instances that distribute the query to actual data storing instances).

![](https://cdn-images-1.medium.com/max/2894/0*TvOANdRrTfL8ox8_.png)

The goal of this **in-depth** article is to describe the **step-by-step** process of creating production-ready fully automated MongoDB ReplicaSet Cluster deployed in Docker Swarm running on multiple VMs and do that by running a **single command**.

Lets dive into the technical aspects of it.

## Automation

![](https://cdn-images-1.medium.com/max/2000/1*lL0ol3B_GVzTxz3ovmvbRw.png)

To achive automation, we will rely on:

* [Vagrant](https://www.vagrantup.com/) ([VirtualBox](https://www.virtualbox.org/) driver) — to spin off all VM’s

* [Ansible](https://www.ansible.com/) — to automate installation of software (Docker, Docker Swarm) to all VM’s as well as run run Docker Stack of MongoDB Services

## Prerequisites

As VirtualBox and Vagrant are prerequisites (need to be installed in advance on your host machine), here is how you can install [VirtualBox](http://ubuntuhandbook.org/index.php/2016/07/virtualbox-5-1-released/) and [Vagrant](https://howtoprogram.xyz/2016/07/23/install-vagrant-ubuntu-16-04/) on Ubuntu. Keep in mind that versions in these links are a bit outdated, so make sure you get the latest version of VirtualBox and Vagrant.

Also you need to manually create shared folders that will be used as volumes (see section Volumes below):

    mkdir /data
    mkdir /data/db-01 /data/db-02 /data/db-03
    mkdir /data/config-01 /data/config-02 /data/config-03
    chmod 666 /data/db-* /data/config-*

## Architecture

![](https://cdn-images-1.medium.com/max/2000/1*h-PMARceK6mRVdXTDC5uMw.png)

**cd**: Continous Delivery VM via which software is installed (using Ansible Playbooks) on the other VMs (Docker, Docker Swarm, Mongo, App etc)

* **IP**: 10.100.198.200, **NETMASK**: 255.255.0.0

**manager-01**: Primary docker manager node

* **IP**: 10.100.192.201, **NETMASK**: 255.255.0.0

**manager-02: **Secondary docker manager node

* **IP**: 10.100.192.202, **NETMASK**: 255.255.0.0

**data-01**: First data node — **datars** replica set with shards

* **IP**: 10.100.193.201, **NETMASK**: 255.255.0.0

* **Docker Service**: mongo_d-01

* **Docker Volume**: /data/db-01

* **Docker Network**: mongo, **Subnet**: 11.0.0.0/24

**data-02**: Second data node — **datars** replica set with shards

* **IP**: 10.100.193.202, **NETMASK**: 255.255.0.0

* **Docker Service**: mongo_d-02

* **Docker Volume**: /data/db-02

* **Docker Network**: mongo, **Subnet**: 11.0.0.0/24

**data-03**: Third data node — **datars** replica set with shards

* **IP**: 10.100.193.203, **NETMASK**: 255.255.0.0

* **Docker Service**: mongo_d-03

* **Docker Volume**: /data/db-03

* **Docker Network**:

* mongo, **Subnet**: 11.0.0.0/24

**config-01**: First config node — **configrs** replica set

* **IP**: 10.100.194.201, **NETMASK**: 255.255.0.0

* **Docker Service**: mongo_c-01

* **Docker Volume**: /data/config-01

* **Docker Network**: mongo, **Subnet**: 11.0.0.0/24

**config-02**: Second config node — **configrs** replica set

* **IP**: 10.100.194.202, **NETMASK**: 255.255.0.0

* **Docker Service**: mongo_c-02

* **Docker Volume**: /data/config-02

* **Docker Network**: mongo, **Subnet**: 11.0.0.0/24

**config-03**: Third config node — **configrs** replica set

* **IP**: 10.100.194.203, **NETMASK**: 255.255.0.0

* **Docker Service**: mongo_c-03

* **Docker Volume**: /data/config-03

* **Docker Network**: mongo, **Subnet**: 11.0.0.0/24

**mongo-01**: First mongo node (Query Router)

* **IP**: 10.100.195.201, **NETMASK**: 255.255.0.0

* **Docker Service**: mongo_m-01

* **Docker Network**: mongo, **Subnet**: 11.0.0.0/24

* **Docker Network**: mongos, **Subnet**: 12.0.0.0/24

**mongo-02**: Second mongo node (Query Router)

* **IP**: 10.100.195.202, **NETMASK**: 255.255.0.0

* **Docker Service**: mongo_m-02

* **Docker Network**: mongo, **Subnet**: 11.0.0.0/24

## Networks

VM’s will be connected in a private network:

* 10.100.XXX.YYY with NETMASK: 255.255.0.0

Docker Networks:

* Overlay Network **mongo**: subnet 11.0.0.0/24 — mongo cluster internal communication (data, config, mongos)

* Overlay Network **mongos**: subnet 12.0.0.0/24 — mongo cluster external communication (app, mongos)

## Volumes

For the sake of sharing the data between the physical host and VM’s as well as keeping the data beyond the docker container lifecycle, we will create these shared folders that will be bound as docker volumes:

* /data/db-01 - data-01 Volume

* /data/db-02 - data-02 Volume

* /data/db-03 - data-03 Volume

* /data/config-01 - config-01 Volume

* /data/config-02 - config-02 Volume

* /data/config-03 - config-03 Volume

## Vagrant

Vagrant is a tool for managing virtual machines. Vagrant generalizes virtual machine creation across multiple virtualization tools such as VirtualBox or VMware and even provides remote options for AWS, OpenStack or GCE. Vagrant configures and provisions virtual machines, based on configuration file called *Vagrantfile.*

The entry point is our **Vagrantfile**.

<iframe src="https://medium.com/media/4de5f9f9c3053c44a8614995d570b8b5" frameborder=0></iframe>

The **Vagrantfile** basically contains several blocks of Ruby code for creating VM’s. The most important parts are:

* synced_folders — shared host folder **/data** between Host and VM’s. Please note that we define them with type: rsync because the MongoDB doesn’t work with default VirtualBox shared folder option (see [issue](https://stackoverflow.com/questions/29818433/boot2docker-on-windows-running-mongo-with-shared-folder-this-file-system-is-n)).

* vm.box = “ubuntu/xenial64” — Ubuntu 16.04 will be running on all VMs

* vm.network — VM private network

* vm.hostname — VM host name in format *[manager|data|config]-0X (*except **cd**)

* vm.provision — provision script which is called during initial VM start

* v.memory — amout of RAM assigned to VM

Please also note that the VM **cd **is defined as last in the **Vagrantfile**, as it’s important that all other VM’s are already running after the boot of **cd**, at which point docker gets installed, swarm cluster gets setup and docker stack with mongo cluster gets initialized.

## Ansible

**Agentless**
The server uses **ssh** for connecting to clients and running operations on them (no need for agent installation).

**Idempotent**
No matter how many times you call a Playbook or Task, the outcome will be the same.

**Simple**

Ansible (written in Python) uses YAML for its Playbooks.

You can notice in the Vagrantfile that one of the provision scripts of **cd** VM is installing and configuring Ansible locally.

    d.vm.provision :shell, path: "scripts/bootstrap_ansible.sh"

Then from **cd** will **ssh** to other VM’s during running Ansible Playbooks.

Ansible major components worthwhile mentioning for this purpose are:

* **hosts **— containing an inventory information about all hosts (VM’s). /ansible/hosts/cluster is our main hosts file, that contains two groups of hosts (**swarm-managers** and **swarm-workers**). Each VM is defined as one line represented by an IP address assigned to that VM. In addition, it contains one or more **name=value** pairs (e.g swarm=data-01, mongo=data-01) that is available during running Ansible Playbooks.

    [**swarm-managers**]
    10.100.192.201 swarm=manager-01
    10.100.192.202 swarm=manager-02

    [**swarm-workers**]
    10.100.193.201 swarm=data-01 mongo=d-01
    10.100.193.202 swarm=data-02 mongo=d-02
    10.100.193.203 swarm=data-03 mongo=d-03
    10.100.194.201 swarm=config-01 mongo=c-01
    10.100.194.202 swarm=config-02 mongo=c-02
    10.100.194.203 swarm=config-03 mongo=c-03
    10.100.195.201 swarm=mongo-01 mongo=m-01
    10.100.195.202 swarm=mongo-02 mongo=m-02

* **group_vars **—containing variables only specific to host groups (e.g. swarm-workers) or to all groups. As during running Ansible Playbooks against VM’s Ansible is logging in using **ssh**, it’s important to specify the the default *vagrant* user as agent_user for that purpose:

    **agent_user**: vagrant

* **host_vars — **containing host-only specific variables. Vagrant by default is generating SSL public and private keys needed for **ssh**-ing to VM’s. So it’s important to let Ansible also know about the path to the private keys, so that it can **ssh** to VM’s during runing Ansible Playbooks. E.g. in /ansible/host_vars/10.192.192.201 we have:

    **ansible_ssh_private_key_file**: /vagrant/.vagrant/machines/manager-01/virtualbox/private_key

* **roles** — containing named **tasks** and set of **defaults** (variables specific to those tasks). Each task or default has as an entry point a **main.yml** file. For our purpose currently we have 4 roles:

1. **common:** set of tasks for installing [jq](https://stedolan.github.io/jq/) (small command line utility for working with json data) and ensuring that all VM’s know about each other by name (DNS) by adding the proper lines in their /etc/hosts file.

1. **docker**: set of tasks for deleting old docker versions as well as adding docker repository and installing newest version from it.

1. **docker-swarm**: set of tasks for initializing Docker Swarm on the manager-01 as primary manager and joining of other VM’s to that Docker Swarm.

1. **mongo-swarm**: set of tasks for creating the Docker Networks (**mongo** and **mongos**), labelling Docker Nodes with proper labels from the /ansible/hosts/cluster hosts file as well as deploying the Docker Stack /ansible/roles/mongo-swarm/docker_stack.yml to the Docker Swarm.

In Ansible, a task can be described with a name, module (e.g. Ubuntu’s apt package manager), condition for execution, number of retries, tags and several other. Here is an example of a task for installing Docker via the apt package manager on Ubuntu:

    - **name**: "Docker Debian - Installing docker"
      **apt**: name=docker-ce state=present update_cache=yes install_recommends=yes allow_unauthenticated=yes
      **when**: ansible_distribution == "Ubuntu"
      **retries**: 3
      **tags**: [docker]

We can see that that task is using the **apt** module to ensure the package **docker-ce** is installed, when this script is running on a Ubuntu Linux Distro. It will retry up to 3 times max and tag this task with tag **docker** useful as an information in the standard output of the execution.

Let’s have a look, how to initialize the Docker Swarm.

## Setting up Docker Swarm

First we will show how to setup a Docker Swarm without usage of Ansible as it’s easier to understand. Then will will script the process using Ansible Role.

On the **manager-01** we will initialize the Docker Swarm by running:

    docker swarm init --advertise-addr 10.100.192.201

This will generate the token needed for joining the rest of the VM’s from the **swarm-workers** to the Swarm by running:

    docker swarm join --token SWMTKN-1-01ybi2g7lcb1cmin006xkvzdz03ti3pc8utusppiuih7erw3e6-67ndjlycviwq1dsq3kpojt9x0 10.100.192.201:2377

Replace the token info with your token info from the init command. If for some reason you have lost that information (e.g. closed your ssh session), you can recall that information by running:

    docker swarm join-token worker

Also to join our secondary **manager-02** into the Swarm, we need to obtain a special token for managers by runnning:

    docker swarm join-token manager

And then we can join our secondary manager to the Swarm by running:

    docker swarm join --token SWMTKN-1-01ybi2g7lcb1cmin006xkvzdz03ti3pc8utusppiuih7erw3e6-de8jovssaitto3r87u8ujpafq 10.100.192.201:2377

You see, it wasn’t that complex. Now let’s see how we can achieve the same with the Ansible task from the **docker-swarm** Role:

<iframe src="https://medium.com/media/dbe2da15d36a1f4f8f843c9ab5ee16a7" frameborder=0></iframe>

Remember that each Ansible task will be executed on each VM separately, so we will need to add proper **when-**condition to check whether specific task is eligible for that VM.

First task will run docker info and store it into variable **docker_info**:

    **register**: “docker_info”

to check if the **manager-01** is not already a member of the Docker Swarm, in which case Swarm initialization will be skipped. Otherwise it initializes the Swarm on the **manager-01**.

Then it stores the worker and manager tokens into variables **docker_swarm_worker_token** and **docker_swarm_manager_token** respectively:

    ...
    **register**: "docker_swarm_worker_token"
    ...
    **register**: "docker_swarm_manager_token"

It also sets the manager address, manager token and worker token as Ansible facts:

    **docker_swarm_manager_address**: "{{ docker_swarm_addr }}:{{ docker_swarm_port }}"
    ...
    **docker_swarm_manager_address**: "{{ hostvars[docker_swarm_primary_manager]['docker_swarm_manager_address'] }}"
    ...
    **docker_swarm_manager_token**: "{{ hostvars[docker_swarm_primary_manager]['docker_swarm_manager_token'] }}"
    ...
    **docker_swarm_worker_token**: "{{ hostvars[docker_swarm_primary_manager]['docker_swarm_worker_token'] }}"
    ...

And finally it joins both managers (secondary) and workers to the Swarm by running:

    docker swarm join
    --listen-addr {{ docker_swarm_addr }}:{{ docker_swarm_port }}
    --advertise-addr {{ docker_swarm_addr }}
    --token {{ docker_swarm_manager_token.stdout }}
    {{ docker_swarm_manager_address }}

    ...

    docker swarm join
    --listen-addr {{ docker_swarm_addr }}:{{ docker_swarm_port }}
    --advertise-addr {{ docker_swarm_addr }}
    --token {{ docker_swarm_worker_token.stdout }}
    {{ docker_swarm_manager_address }}

As you can see, it was a bit more effort to write it in an Ansible Task, but trust me, it’s worth it writing it once and reusing it later for different environments.

Let’s look at the setup of Mongo Cluster.

## Deploying Mongo Swarm Cluster

The MongoDB Cluster with 3x data instances, 3x config instances and 2x mongo instances (query routers) will be deployed as a Docker Stack of Services described in a YML file:

    /ansible/roles/mongo-swarm/docker_stack.yml

<iframe src="https://medium.com/media/6c697caccc82eb79162598a57b32a335" frameborder=0></iframe>

You can see that all Docker Services have a **constraint** condition, which tells Docker Swarm at which Docker Node they should be deployed. In addition, you can notice that the Docker Services are attached to two Docker Network (**mongos**=external communication and **mongo**=internal communication).

So here is how to add the Docker Node labels via Ansible:

<iframe src="https://medium.com/media/ca40e1c971deed50707896d7fb501c26" frameborder=0></iframe>

We can see that on the primary Docker Swarm manager (**manager-01**), we are reading all Docker Node labels in a variable:

    **register**: "docker_node_labels"

If the string ‘mongo.replica’ was not found in that variable, we will add the proper Docker Node labels.

Here is how we are going to create the **mongo** and **mongos** Docker Network via Ansible:

<iframe src="https://medium.com/media/92a061e329887c3ea49e38f7c4268655" frameborder=0></iframe>

Basically, first we are checking the output (second column) of running the command docker network ls, — if networks **mongo** and **mongos** exist. If they don’t exist, we will create them on the primary Swarm manager.

Deploying the Mongo Docker Stack is done by:

    docker stack deploy --compose-file /vagrant/ansible/roles/mongo-swarm/docker_stack.yml mongo

Which is achieved by Ansible in the following manner:

<iframe src="https://medium.com/media/a5a50772ce33beb587c3792d789b809d" frameborder=0></iframe>

## Initializing Mongo Swarm Cluster

Deploying the Docker Stack is not enough. The mongo instances don’t know about each other yet. So this remains to be configured.

Let’s first do it without Ansible.

Config ReplicaSet called **configrs** can be initialized by running:

    rs.initiate(
    {_id: "configrs", configsvr: true,
    members: [
        { _id: 0, host: "c-01:27017" },
        { _id: 1, host: "c-02:27017" },
        { _id: 2, host: "c-03:27017" }
      ]
    })

inside of any of the config mongo containers (**c-01**, **c-02** or **c-03)**.

Hence, we will use **config-01** VM, to first find out the container name by running:

docker ps

And let’s say the container has the following name:

    mongo_c-01.1.qnkk8f5y8wmsdni5duryo3uwm

then we can configure the **configrs** ReplicaSet by running the following command on **config-01**:

    docker exec -it **mongo_c-01.1.qnkk8f5y8wmsdni5duryo3uwm** bash -c "echo 'rs.initiate({_id: \"configrs\", configsvr: true, members: [{ _id: 0, host: \"c-01:27017\" }, { _id: 1, host: \"c-02:27017\" }, { _id: 2, host: \"c-03:27017\" }]})' | mongo"

Similarly, we can initialize the Data ReplicaSet **datars** by running:

    rs.initiate(
    {_id: "datars",
    members: [
        { _id: 0, host: "d-01:27017" },
        { _id: 1, host: "d-02:27017" },
        { _id: 2, host: "d-03:27017" }
      ]
    })

inside of any of the data mongo containers (**d-01**, **d-02** or **d-03)**.

We will use **data-01** VM, and after obtaining the container name, let’s say we’ve got the following name:

    mongo_d-01.1.zuvdjtrxmo7vw9ksl2l84a52d

we can configure the **datars** ReplicaSet by running:

    docker exec -it **mongo_d-01.1.zuvdjtrxmo7vw9ksl2l84a52d** bash -c "echo 'rs.initiate({_id: \"datars\", members: [{ _id: 0, host: \"d-01:27017\" }, { _id: 1, host: \"d-02:27017\" }, { _id: 2, host: \"d-03:27017\" }]})' | mongo"

inside of **data-01** VM.

Finally, if we can enable Sharding on any of the mongo container processes (**m-01** or **m-02**), by running:

    sh.addShard("datars/d-01")

which will enable Sharding in the whole **datars** ReplicaSet (**d-01**, **d-02** and **d-03**).

Alright, that’s it, our Mongo Swarm Cluster was initialized.

Now let’s see how to achieve automation using Ansible:

<iframe src="https://medium.com/media/7cb23cf7d42946a14fd1521d5a8b47c2" frameborder=0></iframe>

You can see that first we are obtaining the service name e.g.:

    docker ps -qf label=com.docker.swarm.service.name=**mongo_c-01** | awk '{ print $0 }'
    ...
    **register**: "c_01"

    
    docker ps -qf label=com.docker.swarm.service.name=**mongo_d-01** | awk '{ print $0 }'
    ...
    **register**: "d_01"

    docker ps -qf label=com.docker.swarm.service.name=**mongo_m-01** | awk '{ print $0 }'
    **register**: "m_01"

Also then we are obtaining ReplicaSet/Sharding status before initializing:

    docker exec -it {{ **c_01**.stdout }} bin/bash -c "echo 'rs.status()' | mongo"
    ...
    **register**: "c_status"

    
    docker exec -it {{ **d_01**.stdout }} bin/bash -c "echo 'rs.status()' | mongo"
    ...
    **register**: "d_status"
    

    docker exec -it {{ **m_01**.stdout }} bin/bash -c "echo 'sh.status()' | mongo"
    **register**: "sh_status"

and finally we are initializing ReplicaSet/Sharding, if based on the status the Cluster isn’t already initialized.

## Single-command Cluster

Now that we have an understanding of how the Mongo Cluster gets created, let’s run the command that automates the creation process:

    vagrant up

This will:

1. Boot all VM’s in the order they are defined in the *Vagrantfile*

1. It will install Docker on all VM’s

1. It will create the Docker Swarm

1. It will deploy the Docker Stack of Mongo Services

1. It will initialize the Mongo Cluster

## Creating Sharding Collection

Now that we have created our ReplicaSet Sharded MongoDB Cluster, it’s time to try out configuring Sharded DB and Collection.

From now on, we will only use m-01 and m-02 (Query Routers) instances to communicate with the Mongo Cluster.

Inside the **mongo-01** instance, let’s obtain the mongo container name by:

    docker ps

And if let’s say it has the following name:

    mongo_m-01.1.sm4funn77uarr24h4i25sfp1

Then we can select sharedDb as follows:

    docker exec -it **mongo_m-01.1.sm4funn77uarr24h4i25sfp1f** bash -c "echo 'use shardedDb' | mongo"

and enable sharding on it:

    docker exec -it **mongo_m-01.1.sm4funn77uarr24h4i25sfp1f** bash -c "echo 'sh.enableSharding(\"shardedDb\")' | mongo "

As well as create a sharded collection:

    docker exec -it **mongo_m-01.1.sm4funn77uarr24h4i25sfp1f** bash -c "echo 'db.createCollection(\"shardedDb.sharedCollection\")' | mongo "

And decide to shard the collection on the field called **shardedField**:

    docker exec -it **mongo_m-01.1.sm4funn77uarr24h4i25sfp1f** bash -c "echo 'sh.shardCollection(\"shardedDb.sharedCollection\", {\"shardedField\" : 1})' | mongo "

Keep in mind that it’s quite important the way we select our sharding key as this will have a large impact on the data storage and performance. More on that can be found in the [official documentation](https://docs.mongodb.com/manual/core/sharding-shard-key/).

## Source Code

The source code of this DevOps initiative for bootstrapping MongoDB Cluster deployed in Docker Swarm is available at this [repo](https://github.com/gjovanov/mongo-swarm).

## Summary

We have learned how to manually create and then fully automate the creation of Mongo Cluster deployed in Docker Swarm.

This advantage of automating the creation of infrastructure has large benefits on working with multple environments e.g. Development, Testing, QA, Production and having a reliable Continuous Integration (CI) / Continuous Delivery (CD) process.

What you can do from here:

1. Load test the infastructure and improve accordingly

1. Change the infastructure (more VM’s, other network setting e.g. Public IP’s, DHCP, deploy in Cloud providers etc)

1. Implement other Orchestrator e.g. Kubernates

Feel free to reach out in case of any questions.
