Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m69[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m105[39m, end: [33m116[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m123[39m, end: [33m134[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m28[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m120[39m, end: [33m145[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m142[39m, end: [33m153[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m158[39m, end: [33m169[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m229[39m, end: [33m239[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m266[39m, end: [33m283[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m295[39m, end: [33m305[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m331[39m, end: [33m340[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m429[39m, end: [33m439[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m14[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m18[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m40[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m224[39m, end: [33m236[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m241[39m, end: [33m253[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m6[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m86[39m, end: [33m94[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m114[39m, end: [33m117[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m74[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m169[39m, end: [33m176[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m226[39m, end: [33m255[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m318[39m, end: [33m328[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m359[39m, end: [33m369[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m225[39m, end: [33m233[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m252[39m, end: [33m272[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m73[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m78[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m60[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m130[39m, end: [33m141[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m146[39m, end: [33m157[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m76[39m, end: [33m84[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m93[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m202[39m, end: [33m208[39m }

# Kubernetes multi-nodes cluster with k3s and multipass



Working with [Kubernetes](https://kubernetes.io/), you might happen to need a local Kubernetes cluster for development and testing purposes. Of course, Minikube is an option. But what if we need something more powerful without any added complexity? For example, what if we are preparing ourselves for the [CKAD: Certified Kubernetes Application Developer](https://www.cncf.io/certification/ckad/) certification?

Here is my personal solution on OS X (but should work smoothly with GNU/Linux too) that I’d like to share with you: [multipass](https://github.com/CanonicalLtd/multipass) + [k3s](https://github.com/rancher/k3s).

## Multipass

First, we need a virtualization layer in order to run any number of Kubernetes nodes. Very likely, the easiest way to get an [Ubuntu VM on OS X](https://discourse.ubuntu.com/t/multipass-now-available-for-macos/6824) using [multipass](https://github.com/CanonicalLtd/multipass), created by Canonical Ltd:
> It’s a system that orchestrates the creation, management and maintenance of virtual machines and associated Ubuntu images to simplify development.

## k3s

Since I want to keep things small and simple, I take advantage of the amazing Kubernetes project created by [Rancher k3s](https://github.com/rancher/k3s). K3s promises to be a lightweight Kubernetes:
> K3s is a [certified Kubernetes distribution](https://www.cncf.io/certification/software-conformance/) designed for production workloads in unattended, resource-constrained, remote locations or inside IoT appliances.

## How to create a local Kubernetes cluster

KISS style, just 9 commands in 3 easy steps to set up a basic 3 node k3s cluster:

1. Install multipass

1. Create the virtual machines

1. Create the k3s cluster

### **Step 1: Install multipass**

I take it for granted that [brew.sh](https://brew.sh/) can’t be missing in your Mac. If not, please follow that [link](https://brew.sh/). After that, it is nothing more than:

    brew cask install multipass

### **Step 2: Create the Virtual Machines**

Let’s assume we would like a Kubernetes cluster with 1 master node (“k3s-master”) and 2 (worker) nodes (“k3s-worker1” and “k3s-worker2”).

    multipass launch --name k3s-master --cpus 1 --mem 1024M --disk 3G
    multipass launch --name k3s-worker1 --cpus 1 --mem 1024M --disk 3G
    multipass launch --name k3s-worker2 --cpus 1 --mem 1024M --disk 3G

### **Step 3: Create the k3s cluster**

Things here become a little more tricky and a couple of notes are deserved:

* The deploy of the k3s-master i̶s̶ was pretty straight forward as per [official instructions](https://k3s.io/) until k3s release 0.6.0. Add K3S_KUBECONFIG_MODE="644" for details jump to “Step 4. Configure kubectl”

* k3s installation script, if not properly configured, is designed to boot as a single node cluster. Since we don’t want that, before deploying k3s-worker1 and k3s-worker2 we need two information from the master node, which are:
- k3s-master ip address, stored in the K3S_NODEIP_MASTER variable
- k3s-master k3s token, stored in the K3S_TOKEN variable
in order to allow the worker nodes to join the same k3s cluster created by the k3s-master node.

* The deploy of k3s-worker1 and k3s-worker2 is merely a matter of the right configuration based on the 2 variables.

    # Deploy k3s on the master node
    multipass exec k3s-master -- /bin/bash -c "curl -sfL [https://get.k3s.io](https://get.k3s.io) | K3S_KUBECONFIG_MODE="644" sh -"

    # Get the IP of the master node
    K3S_NODEIP_MASTER="https://$(multipass info k3s-master | grep "IPv4" | awk -F' ' '{print $2}'):6443"

    # Get the TOKEN from the master node
    K3S_TOKEN="$(multipass exec k3s-master -- /bin/bash -c "sudo cat /var/lib/rancher/k3s/server/node-token")"

    # Deploy k3s on the worker node
    multipass exec k3s-worker1 -- /bin/bash -c "curl -sfL https://get.k3s.io | K3S_TOKEN=${K3S_TOKEN} K3S_URL=${K3S_NODEIP_MASTER} sh -"

    # Deploy k3s on the worker node
    multipass exec k3s-worker2 -- /bin/bash -c "curl -sfL https://get.k3s.io | K3S_TOKEN=${K3S_TOKEN} K3S_URL=${K3S_NODEIP_MASTER} sh -"

**Check everything:**

    multipass list

    Name State IPv4 Release
    k3s-worker2 RUNNING 192.168.64.5 Ubuntu 18.04 LTS
    k3s-worker1 RUNNING 192.168.64.4 Ubuntu 18.04 LTS
    k3s-master RUNNING 192.168.64.3 Ubuntu 18.04 LTS

    multipass exec k3s-master kubectl get nodes

    NAME          STATUS   ROLES    AGE   VERSION
    k3s-master    Ready    master   48s   v1.14.1-k3s.4
    k3s-worker1   Ready    <none>   16s   v1.14.1-k3s.4
    k3s-worker2   Ready    <none>   6s    v1.14.1-k3s.4

N̶o̶t̶e̶ ̶t̶h̶e̶ ̶<none> ̶i̶n̶ ̶t̶h̶e̶ ̶ROLES ̶c̶o̶l̶u̶m̶n̶.̶ ̶I̶n̶ ̶t̶h̶e̶ ̶s̶e̶c̶o̶n̶d̶ ̶p̶a̶r̶t̶ ̶o̶f̶ ̶t̶h̶i̶s̶ ̶s̶t̶o̶r̶y̶,̶ ̶w̶e̶’̶l̶l̶ ̶g̶e̶t̶ ̶t̶h̶e̶r̶e̶. *Update 20191011: k3s release 0.6.0 introduced node roles and *--node-label and --node-taint flags.

That’s it, the Kubernetes cluster is up and running. Nevertheless, you might want to have a look at the following steps:

## How to create a local Kubernetes cluster (that makes sense)

Hereunder I’m going to add some useful tips to make the cluster useful for developing and testing purposes:

4. Configure kubectl
5. Configure cluster node roles and taint
6. Helm installation
7. Service type “NodePort”
8. Ingress controller [Traefik](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwjKnILS0MriAhXSzaQKHf5xAjYQFjAAegQIBRAC&url=https%3A%2F%2Ftraefik.io%2F&usg=AOvVaw204mn6RPY_Znna2JH9mdFp)

### **Step 4. Configure kubectl**
> Update 20191011: k3s release 0.6.0 changed default *k3s.yaml* permissions from 0644 to 0600 for [security reasons](https://github.com/rancher/k3s/issues/389). Please have a look to [https://github.com/rancher/k3s/issues/389#issuecomment-503616742](https://github.com/rancher/k3s/issues/389#issuecomment-503616742). To overcome this change (during startup phase) there are mainly two solutions:

* Using --write-kubeconfig-mode 644

    curl -sfL [https://get.k3s.io](https://get.k3s.io) | sh -s - --write-kubeconfig-mode 644

* Using the variable K3S_KUBECONFIG_MODE

    curl -sfL [https://get.k3s.io](https://get.k3s.io) | K3S_KUBECONFIG_MODE="644" sh -s -
> Update 20191011: k3s release 0.9.0 [“localhost” has been substituted by “127.0.0.1”](https://github.com/rancher/k3s/pull/750) in k3s.yaml, which was causing sed command to fail.

If we want to forget about the multipass CLI, it’s very easy to configure kubectl installed in your host machine to use the brand new k3s cluster directly (I’m assuming kubectl is already installed in your machine, otherwise: $ brew install kubernetes-cli). First, we need to retrieve the kubectl config file from the k3s-master node and edit the it with the k3s-master IP address, as described hereunder:

    # Copy the k3s kubectl config file locally
    $ multipass copy-files k3s-master:/etc/rancher/k3s/k3s.yaml ${HOME}/.kube/k3s.yaml

    # Edit the kubectl config file with the right IP address
    $ sed -ie s,https://127.0.0.1:6443,${K3S_NODEIP_MASTER},g ${HOME}/.kube/k3s.yaml

    # Check
    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml get nodes

Specifying the --kubeconfig is boring and, above all, potentially dangerous in case we forget to do it and we run the wrong command in the wrong cluster (anyone? no? really?!). It might worth to merge the kubectl config file k3s.yaml with your current ${HOME}/.kube/config file or, since our k3s cluster is not intended to be written in the rocks, we have a couple of easy options (see the [official documentation](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) for details):

    # Create a dedicated alias:
    alias k3sctl="kubectl --kubeconfig=${HOME}/.kube/k3s.yaml"

or

    # Use the KUBECONFIG variable
    export KUBECONFIG=${HOME}/.kube/k3s.yaml

### **Step 5. Configure cluster node roles and taint**
> Update 20191011: k3s release 0.6.0 introduced node roles and --node-label and --node-taint flags. Therefore **“Step 5.” is not needed anymore**.

As we have previously noticed, the 3 nodes have no roles. That’s because:

* K3s installation script does not label any node it creates

* K3s installation script does not taint the master nodes for NoSchedule

Let’s take care of these two configurations:

    # Configure the node roles:
    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml label node k3s-master node-role.kubernetes.io/master=””
    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml label node k3s-worker1 node-role.kubernetes.io/node=””
    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml label node k3s-worker2 node-role.kubernetes.io/node=””

    # Configure taint NoSchedule for the k3s-master node
    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml taint node k3s-master node-role.kubernetes.io/master=effect:NoSchedule

The nodes roles are now properly configured:

    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml get nodes
    NAME STATUS ROLES AGE VERSION
    k3s-master Ready **master** 4h12m v1.14.1-k3s.4
    k3s-worker1 Ready **node** 3h57m v1.14.1-k3s.4
    k3s-worker2 Ready **node** 3h57m v1.14.1-k3s.4

Eventually we are ready for a deployment. What I’d like to highlight is that the NGiNX pods are going to be scheduled only in the k3s-worker1 and k3s-worker2 nodes:

    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml run nginx --image=nginx --replicas=3 --expose --port 80
    kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
    service/nginx created
    deployment.apps/nginx created

    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml get pods -o wide
    NAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATES
    nginx-755464dd6c-6nzvr 1/1 Running 0 3h22m 10.42.1.3 **k3s-worker2** <none> <none>
    nginx-755464dd6c-rkd6r 1/1 Running 0 3h22m 10.42.1.4 **k3s-worker2** <none> <none>
    nginx-755464dd6c-v5v64 1/1 Running 0 3h22m 10.42.2.3 **k3s-worker1** <none> <none>

### **Step 6: Helm installation**

[Helm](https://helm.sh/) tries to combine a **template engine** and a **package manager** for Kubernetes. Helm installation might not be as straightforward as we are used to, indeed k3s demands for a [few more steps](https://rancher.com/docs/rancher/v2.x/en/installation/ha/helm-init/):

    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml -n kube-system create serviceaccount tiller

    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml create clusterrolebinding tiller \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:tiller

    $ helm --kubeconfig=${HOME}/.kube/k3s.yaml init --service-account tiller

    $ helm --kubeconfig=${HOME}/.kube/k3s.yaml install stable/mysql

If you are interested to know something more about Helm, I shared my experience with the Helm chart repository topic in the following story: “[Create a public Helm chart repository with GitHub Pages](https://medium.com/@mattiaperi/create-a-public-helm-chart-repository-with-github-pages-49b180dbb417)”

### **Step 7: Service type “NodePort”**

Actually this step is not k3s specific. I just wanted to highlight that the [NodePort services](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) creation is as easy as usual because the network between host machine and VMs is transparent for the user:

    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml create deploy nginx2 --image=nginx

    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml create svc nodeport nginx2 --tcp=30001:80 --node-port=30001

    $ curl -XGET -s -I -o /dev/null -w "%{http_code}\n" [http://$(multipass](http://$(multipass) info k3s-master | grep "IPv4" | awk -F' ' '{print $2}'):30001
    200

### Step 8: Ingress controller [Traefik](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwjKnILS0MriAhXSzaQKHf5xAjYQFjAAegQIBRAC&url=https%3A%2F%2Ftraefik.io%2F&usg=AOvVaw204mn6RPY_Znna2JH9mdFp)

I didn’t mention before but it’s easy to believe that to keep things light, k3s comes with some compromises and one of them regards [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/). In order for the Ingress resource to work, the cluster must have an [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) running (and usually it’s [NGiNX](https://www.nginx.com/products/nginx/kubernetes-ingress-controller)). K3s includes [Traefik](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwjKnILS0MriAhXSzaQKHf5xAjYQFjAAegQIBRAC&url=https%3A%2F%2Ftraefik.io%2F&usg=AOvVaw204mn6RPY_Znna2JH9mdFp) for this purpose instead. It also includes a simple service [load balancer](https://github.com/rancher/k3s/blob/master/README.md#service-load-balancer) that makes it possible to get an external IP for a Service in the cluster. Let’s see a simple example on how to configure the ingress controller (just edit the YAML file with the right ip address):

    $ cat <<EOF > ingress-controller.yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: ingress
    spec:
      rules:
      - host: **192.168.64.3**.xip.io
        http:
          paths:
          - path: /
            backend:
              serviceName: nginx2
              servicePort: 30001
    EOF

    $ kubectl --kubeconfig=${HOME}/.kube/k3s.yaml apply -f ingress-controller.yaml
    ingress.extensions/ingress created

    $ curl -XGET -s -I -o /dev/null -w "%{http_code}\n" [http://192.168.64.3.xip.io/](http://192.168.64.3.xip.io/)
    200 

[nip.io](https://nip.io/) is a “Dead simple wildcard DNS for any IP Address” allowing you to map any IP Address to a hostname.

### Hint: how to connect via SSH

In case you’d like to be more flexible and connect to multipass VMs via SSH instead of using multipass shell, you need to use the key at the path hereunder (OS X) with the right permissions and use the ubuntu user:

    $ sudo cp -a /var/root/Library/Application\ Support/multipassd/ssh-keys/id_rsa ~/.ssh/multipass
    $ sudo chown $USER ~/.ssh/multipass
    $ ssh -i ~/.ssh/multipass ubuntu@*192.168.64.3*

### **Final step: clean-up everything**

It was a nice journey, now it’s time to get rid of everything:

    $ multipass stop k3s-master k3s-worker1 k3s-worker2
    $ multipass delete k3s-master k3s-worker1 k3s-worker2
    $ multipass purge

## **Conclusion**

It’s very easy to create a local multi-node Kubernetes cluster with a nice degree of complexity without losing the Minikube simplicity. Moreover, the solution described adds a remarkable level of flexibility thanks to multipass VMs implementation that guarantees isolation and security to your Kubernetes tweaking.

As a final hint, I created a bash script able to set up a k3s environment completed with all fancy stuff like metrics-server, weavescope, prometheus and istio. You are more than welcome to get ideas from it:

[https://github.com/mattiaperi/k3s-multipass-cluster](https://github.com/mattiaperi/k3s-multipass-cluster)

I hope you find this information useful, any feedback or advice for improvements are very well accepted.
