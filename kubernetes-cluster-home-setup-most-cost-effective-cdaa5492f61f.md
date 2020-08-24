Unknown markup type 10 { type: [33m10[39m, start: [33m28[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m104[39m, end: [33m143[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m12[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m82[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m8[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m60[39m, end: [33m75[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m95[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m8[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m52[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m271[39m, end: [33m285[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m69[39m }

# Kubernetes cluster home setup, most cost effective.



A recent project required me to setup Kubernetes, learn the basics and also gets the hand dirty around experimenting. I looked into many options, google cloud, AWS, Katacoda, minikube, setting up a cluster on virtual machines. The cloud option did not provide the flexibility in free version also i didn’t own a credit card. While trying to setup on virtual machines i always messed up the networking between the machines and the cluster would never work fine. So scouting the internet i came across clusters build on raspberry-Pi. The whole project was quite appealing and also provided the flexibility of bare metal installation. This is how it was done.

### **The initial setup.**

* At least 2 Raspberry-pi 3 or upper version with flashed memory card and Linux running on them.

* A WiFi router, or create a mobile host-spot using your phone or laptop.

* USB cables to power the Raspberry-pi.

### Bringing all devices on same network.

You need to bring all the pi’s and your laptop from where you want to monitor on same network. If you have created the hot-spot using your laptop then no issues or else connect your laptop to WiFi. To connect you pi boards to a WiFi network you need to edit the Wpa supplicant file. How read here( [link](https://medium.com/@oreanroy/setting-up-a-new-headless-raspberry-pi-without-a-screen-c9e91424a236) ).

* Check the IP address of all connected pi’s using fing or settings page of hot-spot and ssh into all the the pi’s.

* Now we need to install docker on all pi’s

### Installing Docker.

    curl -sSL get.docker.com | sh && \
    sudo usermod pi -aG docker && \
    newgrp docker

* Disable swap , for kubernetes to function properly you need to disable swap.

    sudo dphys-swapfile swapoff && \
    sudo dphys-swapfile uninstall && \
    sudo update-rc.d dphys-swapfile remove

### Setting-up static IP and hostname

* Type sudo raspi-config in the pi sshed terminal. Go to Network>Hostname and then change the hostname to master, worker1, woker2 etc.

* To setup static ip adress sudo vim etc/dhcpcd.config edit it to

    interface wlan0
    static ip_address=192.168.11.13
    static routers=192.168.11.1
    static domain_name_servers=8.8.8.8

Where 192.168.11.3 is the IP you want to give to the device. Also check the IP series your router is assigning using fing. **Now reboot using sudo reboot.**

### **The c-group and key setup.**

This needs to be done for kubernetes installation and c-group permissions to work fine.

* Add this to the end of file /boot/cmdline.txt

cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory

Add it in the last line of all other text not a new line or file. Now reboot.

* Now we need to add kuberenetes dpg key and deb path for apt to install kubernetes, add this to the file /etc/apt/sources.list.d/kubernetes.list

deb [http://apt.kubernetes.io/](http://apt.kubernetes.io/) kubernetes-xenial main

* Now to add the key use curl -s [https://packages.cloud.google.com/apt/doc/apt-key.gpg](https://packages.cloud.google.com/apt/doc/apt-key.gpg) | apt-key add -

You will get OK if it worked fine

### Installing Kubeadm

* Now update sudo apt-get update

* Install kubeadm sudo apt-get install -qy kubeadm

## Master Node Setup

Above this all things need to be done on all nodes. Now we need to setup a networking on master i am using weave. I have tried flannel and calico but weave works the best for me.

Now you have come this far.. be patient an follow the further steps the final product will be highly rewarding. The further steps are easy.

1. Pull images sudo kubeadm config images pull -v3

1. Prevent token expiration, do not use in production sudo kubeadm init — token-ttl=0

1. Now run sudo kubeadm init

This might throw an error or warning, if swap is not off do sudo swapoff -a and a kubeadm reset do this reset and init untill you get an output similar to

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

Along with this you will also see a join token.

    kubeadm join 192.168.137.35:6443 --token 8deg59.wxl8sp6wwrokpcfl \
        --discovery-token-ca-cert-hash sha256:9731b2a1516c0c83a75d684b75db1c72ecbc94174a2b62522ab79b442ef15455

4. Now we setup weave network, install weave network driver.

    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

5. Do a kubectl get pods --all-namespaces to get all running pods.

insert pod image

6. Run sudo sysctl net.bridge.bridge-nf-call-iptables=1 you also run this on all worker nodes before running the join command on them.

![pods on my cluster (There will not be few pods from these on your cluater as i have installed some services after the installation)](https://cdn-images-1.medium.com/max/2220/1*8683ygX3N8A5ln8tClQ-Gw.png)*pods on my cluster (There will not be few pods from these on your cluater as i have installed some services after the installation)*

The pods on your new cluster will be mostly.

    NAMESPACE    NAME                                  READY  STATUS
    kube-system  coredns-86c58d9df4-9hx5c              1/1    Running
    kube-system  coredns-86c58d9df4-nfgk5              1/1    Running
    kube-system  etcd-k8s-master-1                     1/1    Running
    kube-system  kube-apiserver-k8s-master-1           1/1    Running
    kube-system  kube-controller-manager-k8s-master-1  1/1    Running
    kube-system  kube-proxy-4k2mc                      1/1    Running
    kube-system  kube-scheduler-k8s-master-1           1/1    Running
    kube-system  weave-net-5rmmn                       2/2    Running

### Worker Node Setup

1. Run sudo sysctl net.bridge.bridge-nf-call-iptables=1

1. Now run the joing command that was generated on master.

    kubeadm join 192.168.137.35:6443 --token 8deg59.wxl8sp6wwrokpcfl \
        --discovery-token-ca-cert-hash sha256:9731b2a1516c0c83a75d684b75db1c72ecbc94174a2b62522ab79b442ef15455

You might be thrown some warning ignore them. If the node joins properly it will print node joined. If there is an error thrown make sure swap is off and all docker docker container are running properly. The containers that are put up ky kubelet and kubectl. To check do sudo docker ps the containers might take some time to come up so wait after doing a swapoff. If your installation of kubeadm was proper you will see these containers.

![The docker container on one node.](https://cdn-images-1.medium.com/max/3330/1*e4Ffq1M9xNOyG7jn18BEwQ.png)*The docker container on one node.*

Now you can check if the node joined properly using kubectl get nodes in master you should see your node there. This can take some time so wait also the node can stay in not-ready state so wait and just check the spawned containers on worker node.

![I have two nodes, djoker is master node.](https://cdn-images-1.medium.com/max/2000/1*odAKRzFPtCxWdbCHr77acw.png)*I have two nodes, djoker is master node.*

There you are your cluster setup is done. There are some things to keep in mind while using a RaspberryPi cluster.

## Things to keep in mind.

1. The raspberry pi chipset is build over arm architecture so docker containers build in amd64(most laptops) or other architecture will not work on this cluster.

1. Most official images of arm architecture are now available or you can compile using build-x in docker beta.

1. When you will restart the cluster nothing will seem to work fine you need to shh in all machines and do a swapoff, write an ansible script would be easy.

1. Now you need to wait until docker spins all important containers on all nodes and the master and then kubectl will work fine. Until then you may get permission denied error.

Here is my cluster running a job process while i monitor the system usage on all machines.

![system usage on worker and master.](https://cdn-images-1.medium.com/max/3248/1*EwVaLP6qm0G4Qr4XSDuO-g.png)*system usage on worker and master.*

### The mentions for source of info.

* [setting up rpi and wifi module.](https://medium.com/@oreanroy/setting-up-a-new-headless-raspberry-pi-without-a-screen-c9e91424a2a36)

* [intalling kubernetes on rpi](https://medium.com/nycdev/k8s-on-pi-9cc14843d43) [Mofizur Rahman](undefined) blog.

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
