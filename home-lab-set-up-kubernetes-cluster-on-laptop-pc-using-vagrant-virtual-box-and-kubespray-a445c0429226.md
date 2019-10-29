Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m60[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m68[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m2[39m, end: [33m142[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m58[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m280[39m, end: [33m297[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m334[39m, end: [33m397[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m75[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m61[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m87[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m142[39m, end: [33m162[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m20[39m }

# [Home Lab] Set up Kubernetes cluster on laptop/PC using Vagrant Virtual Box and Kubespray

Getting started Kubernetes with home lab is great. You can spin up, mess around and use it as playground for learning. When I buy laptop or PC, one thing I consider is I can build the lap on my own stuff. No need to have physical servers or cloud account which might charge me extra money :). Here I am using my PC to build Kerbunetes cluster. I’m not using minikube because I want to simulate the lap as production-like as much as I can.

First thing first we need to install [Virtual Box](https://www.virtualbox.org/) and [Vagrant ](https://www.vagrantup.com/)on laptop. These two things will spawn **seven VMs** for us. You need at least 1CPU core 2GB RAM per VMs for K8S nodes. The number of VMs can be reduced to three if you have limited resources.

* One Ansible control node to run [Kubespare](https://github.com/kubernetes-sigs/kubespray)/ kubectl operation node. Hostname = controlz

* Three Kubernetes master node. Hostname = node0[1–3]

* Three Kubernetes worker node. Hostname = node0[4–6]

## 1. Creating VMs

Create a file name “Vagrantfile” with below gist content inside your working directory. You can find all the information in vagrant file e.g. hostname, IP, OS image, VM spec also the script part the prepare the host file, user password and install packages. This is the benefit of vagrant, we don’t have to do these stuff manually.

<iframe src="https://medium.com/media/565be596193aa21587a19b5f163f8905" frameborder=0></iframe>

Run vagrant status. The result will look like below which means we’re ready to create the VMs.

![vagrant status](https://cdn-images-1.medium.com/max/2000/1*jy2AXqz3zGuL9xQ9_5r8Hw.png)*vagrant status*

Run vagrant up, wait until finish and run vagrant status again. All VMs should be up and running. Set host file on your laptop then you can connect the VMs via DNS ( use defined credential in vagrant file to login )

![vagrant up](https://cdn-images-1.medium.com/max/2000/1*_-tcxQeQ13gV0o9L1Hf-NA.png)*vagrant up*

![vagrant status with all VMs running](https://cdn-images-1.medium.com/max/2000/1*7pfX-czVdzAbu8MyMa523Q.png)*vagrant status with all VMs running*

## 2. Set up Ansible

We’ll use [Kubespray ](https://github.com/kubernetes-sigs/kubespray.git)to set up K8S cluster. Before that we need to install Ansible on controlz node. SSH to control node and run below command to install ansible from pip command.

    $ git clone [https://github.com/kubernetes-sigs/kubespray.git](https://github.com/kubernetes-sigs/kubespray.git)
    $ cd kubespray ; sudo pip install -r requirements.txt

Create ssh-key pairs usingssh-keygenthen copy to all target nodes byssh-copy-id.

    $ ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/devops/.ssh/id_rsa): /home/devops/.ssh/id_rsa_admin
    ...
    ...

    $ ssh-copy-id -i /home/devops/.ssh/id_rsa_admin admin@node01
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    admin@node01's password:

    Number of key(s) added: 1

    Now try logging into the machine, with:   "ssh 'admin@node01'"
    and check to make sure that only the key(s) you wanted were added.

Go to kuberspray folder that we just cloned from git. Edit ansible.cfg file for the ssh-key we created and add the privilege escalation section for ansible run to become root.

    [defaults]
    remote_user = admin
    private_key_file = $HOME/.ssh/id_rsa_admin
    ...
    ...

    [privilege_escalation]
    become=True
    become_method=sudo
    become_user=root
    become_ask_pass=False

Copy sample inventory folder to yours. Create ansible inventory file there name hosts.ini. As I mentioned earlier, three nodes as master and other three as worker.

    $ cp -rfp inventory/sample inventory/my-cluster
    $ vi inventory/my-cluster/hosts.ini #create hosts.ini for ansible inventory with below content

<iframe src="https://medium.com/media/885d42fae7351762ceb27352cd1f1155" frameborder=0></iframe>

Run ansible -i inventory/my-cluster/hosts.inii -m ping all There should return a green result.

![ansible ping result](https://cdn-images-1.medium.com/max/2000/1*GRD-o0p6HF0Uh_IxwTlZjg.png)*ansible ping result*

## 3. Prepare Kubespray setting

The K8S cluster setting can be adjusted inside ansible inventory/my-cluster/group_vars folder. We’ll leave most of them as default but you can read the documentation of each setting in [kubespray github](https://github.com/kubernetes-sigs/kubespray). Somehow if you decide to use flannel network you need to change this setting flannel_interface to the host-only interface from fileinventory/my-cluster/group_vars/k8s-cluster/k8s-net-flannel.yml . Check your host-only interface and put it there.

    # see roles/network_plugin/flannel/defaults/main.yml

    ## interface that should be used for flannel operations
    ## This is actually an inventory cluster-level item
    # flannel_interface:
    flannel_interface: enp0s8

Each VM has two interfaces, NAT and host-only network. NAT is only for internet access. The communication between VMs must use the host-only network ( 192.168.56.0/24 ). Without this interface setting your VMs will pick the NAT network and the cluster will not work.

![VM network interface](https://cdn-images-1.medium.com/max/2000/1*1IPdiQ79ftWWgmc1tKTzdw.png)*VM network interface*

Review all setting in inventory/my-cluster/group_vars/. If all good, run ansible-playbook command below to deploy the K8S cluster.

    $ ansible-playbook -i inventory/my-cluster/hosts.yml cluster.yml

![ansible-playbook result](https://cdn-images-1.medium.com/max/3028/1*lAx9JVa5Hr-XY9P3AYVe1w.png)*ansible-playbook result*

## 4. K8S cluster up and running

Now K8S cluster is ready. First, install kubectl from yum command. You needkubectlcommand to manage the cluster via controlz node.

    $ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=[https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64](https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64)
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=[https://packages.cloud.google.com/yum/doc/yum-key.gpg](https://packages.cloud.google.com/yum/doc/yum-key.gpg) [https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg](https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg)
    EOF
    $ yum install -y kubectl

Try to run kubectl cluster-info . You cannot connect to the K8S cluster yet because kubectl command need to know the cluster info.

    $ kubectl cluster-info

    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

    The connection to the server localhost:8080 was refused - did you specify the right host or port?

SSH to node01, copy the content of /etc/kubernetes/admin.conf to controlz node at path ~/.kube/config (kubectl default config path). Then run kubectl cluster-info at controlz node again. Now you should get cluster info result.

Run kubectl get node to displays the state of all of your nodes.

### This is where your K8S journey begins !!!

![kubectl cluster-info](https://cdn-images-1.medium.com/max/2748/1*GgdPzug6TzqT3btlT1mg2A.png)*kubectl cluster-info*

![kubectl get nodes](https://cdn-images-1.medium.com/max/2916/1*NUEtYpjqkax3Y1UX-Fui9A.png)*kubectl get nodes*
