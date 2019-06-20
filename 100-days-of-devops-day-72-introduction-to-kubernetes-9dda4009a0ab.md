
# 100 Days of DevOps — Day 72-Introduction to Kubernetes

Welcome to Day 72 of 100 Days of DevOps, Focus for today is Introduction to Kubernetes

## *What is Kubernetes?*

*Kubernetes is a portable, extensible open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation.*

* *Born in google and donated to Cloud Native Computing Foundation(CNCF) in 2014*

* *Written in Go/Golang [https://github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes)*

* *Open source under Apache license 2.0*

* *Version 1.0 released in July 2015*

* *Often shortened to K8s(replace 8 characters between k and s)*

![](https://cdn-images-1.medium.com/max/4812/1*cC7Yimt6zqoFh4UZws9VRg.png)
> ***Kubernetes Architecture***

* *Kubernetes is an orchestrator for microservice apps(application made of a lot of small and independent services)that run on a container*

![](https://cdn-images-1.medium.com/max/2564/1*tt8MSunnmvqh61_eZ0wlUg.png)

## ***Master***
> *kube-apiserver(Brain of the cluster)*

* *This is the front-end for the Kubernetes control plane*

* *Exposes the API(REST) and consumes JSON(via manifest file) where we describe the desired state*
> *Cluster store(Persistent Storage)(etcd)*

* *Cluster state and config is persistently stored here.*

* *Uses etcd(Opensource distributed key-value store)*

* *Act as a source of truth for the cluster*
> *Kube-controller-manager*

* *Node controller*

* *Endpoints controller*

* *Namespace controller*

* *Watches for changes and helps maintain the desired state*
> *Kube-scheduler*

* *Watch APIserver for new pods*

* *Assigns work to nodes*

* *The decision is based on many factors including hardware, workloads, affinity, etc.*

### *Nodes(The Kubernetes Workers aka Minions)(The Kubernetes Workers)*

![](https://cdn-images-1.medium.com/max/2000/1*Ry_3RTEurB503gT3IeaTRA.png)
> ***Kubelet***

* *The main kubernetes agent on the node*

* *Registers node with cluster*

* *Watches apiserver*

* *Instantiates pods*

* *Reports back to master*

* *Exposes endpoint on 10255*
> ***Container Engine***

* *Does container management*

* *Pulling images*

* *Starting/stopping container*

* *Usually Docker*

* *Can be CoreOS Rocket(rkt)*
> ***Kube-proxy(Kubernetes networking)***

* *Pod IP addresses*

* *All containers in a pod share a single IP*

* *Load balances across all pods in a service*

## ***Pods***

* *Containers always run inside of pods*

* *All containers in a pod share the pod environment(eg: IPC, shared memory, network)*
> *Pod Lifecycle*

* *Failed pods never brought back to life*
> *Replication Controllers*

* *Scale pods and maintains the desired state*
> *Deployments*

* *Rolling updates and rollbacks*
> *Services*

* *Stable networking and load balancing*

* *To check*

    *$ kubectl get pods  --all-namespaces  -o wide*

    *NAMESPACE     NAME                                                  READY   STATUS    RESTARTS   AGE   IP              NODE                          NOMINATED NODE   READINESS GATES*

    *kube-system   coredns-fb8b8dccf-9v6gv                               1/1     Running   0          31m   10.244.0.3      plakhera11c.mytestserver.com   <none>           <none>*

    *kube-system   coredns-fb8b8dccf-xk4jx                               1/1     Running   0          31m   10.244.0.2      plakhera11c.mytestserver.com   <none>           <none>*

    *kube-system   **etcd**-plakhera11c.mytestserver.com                      1/1     Running   0          30m   172.31.16.235   plakhera11c.mytestserver.com   <none>           <none>*

    *kube-system   **kube-apiserver**-plakhera11c.mytestserver.com            1/1     Running   0          30m   172.31.16.235   plakhera11c.mytestserver.com   <none>           <none>*

    *kube-system   **kube-controller-manager**-plakhera11c.mytestserver.com   1/1     Running   0          30m   172.31.16.235   plakhera11c.mytestserver.com   <none>           <none>*

    *kube-system   kube-flannel-ds-amd64-5rkbc                           1/1     Running   1          28m   172.31.18.85    plakhera12c.mytestserver.com   <none>           <none>*

    *kube-system   kube-flannel-ds-amd64-7vlwf                           1/1     Running   0          29m   172.31.16.235   plakhera11c.mytestserver.com   <none>           <none>*

    *kube-system   kube-flannel-ds-amd64-8ktdq                           1/1     Running   1          28m   172.31.18.156   plakhera13c.mytestserver.com   <none>           <none>*

    *kube-system   **kube-proxy**-7fmvr                                      1/1     Running   0          28m   172.31.18.156   plakhera13c.mytestserver.com   <none>           <none>*

    *kube-system   **kube-proxy-**rd8qq                                      1/1     Running   0          31m   172.31.16.235   plakhera11c.mytestserver.com   <none>           <none>*

    *kube-system   **kube-proxy**-vkl7h                                      1/1     Running   0          28m   172.31.18.85    plakhera12c.mytestserver.com   <none>           <none>*

    *kube-system   **kube-scheduler**-plakhera11c.mytestserver.com            1/1     Running   0          31m   172.31.16.235   plakhera11c.mytestserver.com   <none>           <none>*
> *Kubernetes Installation*

*In this setup, I am going to install*

    *1 master node
    2 worker node*
> *Step1: Disable selinux*

    *setenforce 0
    sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux*
> *Step2: Enable the br_netfilter module for cluster communication.*

    *modprobe br_netfilter
    echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables*
> *Step3: Disable swap to prevent memory allocation issues.*

    *swapoff -a*

    *# Go to /etc/fstab file and comment this line
    #/root/swap     swap    swap    sw      0 0*
> *Step4: Install the Docker prerequisites*

    *yum install -y yum-utils device-mapper-persistent-data lvm2*
> *Step5: Add the Docker repo and install Docker.*

    *# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo*

    *# yum install -y docker-ce*
> *Step6: Configure the Docker Cgroup Driver to systemd, enable and start Docker*

    *# sed -i '/^ExecStart/ s/$/ --exec-opt native.cgroupdriver=systemd/' /usr/lib/systemd/system/docker.service *

    *# systemctl daemon-reload*

    *# systemctl enable docker --now*
> *Step7: Add the Kubernetes repo.*

    *cat <<EOF > /etc/yum.repos.d/kubernetes.repo
     [kubernetes]
     name=Kubernetes
     baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
     enabled=1
     gpgcheck=0
     repo_gpgcheck=0
     gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
             [https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg](https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg)
    EOF*
> *Step8: Install Kubernetes.*

    *yum install -y kubelet kubeadm kubectl*
> *Step9: Enable Kubernetes. The kubelet service will not start until you run kubeadm init.*

    *# systemctl enable kubelet*

    *Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /usr/lib/systemd/system/kubelet.service.*

*NOTE: Up to this point all steps need to be performed on Master as well as on two workers node*

* *The below steps need to perform only on Master*
> *Initialize the cluster using the IP range for Flannel.*

<iframe src="https://medium.com/media/418b126a0430653c101ec9fd20d4238b" frameborder=0></iframe>

* *Copy the kubeadm join command*

* *To start using your cluster, you need to run the following as a regular user:*

    *mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config*

* *Deploy Flannel.*

    *$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml*

    *podsecuritypolicy.extensions/psp.flannel.unprivileged created*

    *clusterrole.rbac.authorization.k8s.io/flannel created*

    *clusterrolebinding.rbac.authorization.k8s.io/flannel created*

* *Run kubectl get nodes*

    *$ kubectl get nodes*

    *NAME                          STATUS     ROLES    AGE     VERSION*

    *plakhera1.mytestserver.com   Ready      master   3m48s   v1.14.1*

* *Now run the kubeadm join we copied earlier on all Worker Nodes*

<iframe src="https://medium.com/media/71541c859930785c826be8dbdf0366c1" frameborder=0></iframe>

* *Run kubectl nodes again*

    *$ kubectl get nodes*

    *NAME                          STATUS   ROLES    AGE     VERSION*

    *plakhera1.mytestserver.com   Ready    master   7m53s   v1.14.1*

    *plakhera2.mytestserver.com   Ready    <none>   4m9s    v1.14.1*

    *plakhera3.mytestserver.com   Ready    <none>   4m10s   v1.14.1*

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
