Unknown markup type 10 { type: [33m10[39m, start: [33m137[39m, end: [33m165[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m243[39m, end: [33m271[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m95[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m179[39m, end: [33m192[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m26[39m, end: [33m122[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m54[39m, end: [33m74[39m }

# Kubernetes Metal LB for On-Prem / BareMetal Cluster in 10 minutes

In this quickstart with Metal LB we will quickly get an on-prem / bare metal load balancer running on Kubernetes (k8s). This is more of a standard layer 2 MetalLB environment. If you would prefer more of a developer related environment on Docker CE, please refer to this guide https://medium.com/@JockDaRock/kubernetes-metal-lb-for-docker-for-mac-windows-in-10-minutes-23e22f54d1c8.

**Pre-Reqs:**

1. K8s Cluster with no other load balancers installed. Check out [Alex Ellis](undefined) guides for getting up and running with bare metal kubernetes here if you need help getting started ([https://blog.alexellis.io/kubernetes-in-10-minutes/](https://blog.alexellis.io/kubernetes-in-10-minutes/), [https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/](https://blog.alexellis.io/serverless-kubernetes-on-raspberry-pi/)).

1. Kubernetes-cli program or kubectl. For mac you should be able to get this with the homebrew package manager and the command line command brew install kubernetes-cli. Or if you are on Windows and have the chocolatey package manager you can use choco install kubernetes-cli. You can also use this guide as well [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

1. Your default gateway router configured for DHCP and able to hand out IP addresses.

**Optional:**

Check out the docs and github repo for MetalLB for more in depth study [https://metallb.universe.tf/](https://metallb.universe.tf/), [https://github.com/google/metallb](https://github.com/google/metallb).

**The environment I am Using:**
> Your environment might be a little bit or vastly different than this, but should still work given the above pre-reqs.

* Host: Ubuntu 18.04 Server 16 GB RAM, 4 CPUs running on VirtualBox on a Windows 10 Computer.

* Kube Cluster: Single Node Cluster Version, Kubernetes Version 1.11.3

* Network: VLANed network running on [Meraki](https://create.meraki.io/) with DHCP configured for 192.168.X.X network.

Now that we have the pre-reqs out of the way let’s get started.

First we need to apply the MetalLB manifest.

kubectl apply -f [https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml](https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml)

Next we want to check that the controller and the speaker are running. we can do this by using this command.

kubectl get pods -n metallb-system

Once those pods are running we can deploy our MetalLB configuration.

![](https://cdn-images-1.medium.com/max/2000/1*zeatr2qRuGikHHKdwbfRiw.png)

So next we will look at our configuration.

    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        address-pools:
        - name: my-ip-space
          protocol: layer2
          addresses:
          - 192.168.1.240/28

You will notice the addresses are using a 192.168.X.X network. Please modify this to match the IP scheme of the network you are connected to. You can change this to something like10.0.1.240/28 or something similar.

That being said, you can access this exact configuration here [https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/example-layer2-config.yaml](https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/example-layer2-config.yaml) and download and modify if you need to. If not, we can apply this configuration as is.

kubectl apply -f [https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/example-layer2-config.yaml](https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/example-layer2-config.yaml)

Once we have deployed our configuration we are ready to apply our deployment and service manifest.

    apiVersion: apps/v1beta2
    kind: Deployment
    metadata:
      name: nginx
    spec:
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1
            ports:
            - name: http
              containerPort: 80

    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx
    spec:
      ports:
      - name: http
        port: 8080
        protocol: TCP
        targetPort: 80
      selector:
        app: nginx
      type: LoadBalancer

The above configuration is what we are using to deploy our first application. I changed this slightly from the tutorial on the main docs site. This will deploy an nginx app/service to kubernetes that we will be able to access. You can download and change this file as needed ([https://raw.githubusercontent.com/JockDaRock/metallb-testing/master/nginxlb.yml](https://raw.githubusercontent.com/JockDaRock/metallb-testing/master/nginxlb.yml)) or use as is.

Use the following command kubectl apply -f [https://raw.githubusercontent.com/JockDaRock/metallb-testing/master/nginxlb.yml](https://raw.githubusercontent.com/JockDaRock/metallb-testing/master/nginxlb.yml)

We can verify the IP address is assigned by using the kubectl get services command.

![](https://cdn-images-1.medium.com/max/2000/1*-qSTOk04llO5tBnypltefg.png)

Once the service is up and running we can access it on a browser of our choice at the External IP address listed on port 8080.

You are now ready to go forth to use your new LoadBalancing skills. Let me know how your experience is and if you have any questions.

Check out some other guides and tutorials if you are looking to get started with Kubernetes:
[**Minikube on Windows 10 with Hyper-V**
*Getting started with Kubernetes can be daunting when you don’t know where to begin. Luckily the folks at Kubernetes…*medium.com](https://medium.com/@JockDaRock/minikube-on-windows-10-with-hyper-v-6ef0f4dc158c)

[https://www.lynda.com/Kubernetes-tutorials/Kubernetes-Microservices/691061-2.html](https://www.lynda.com/Kubernetes-tutorials/Kubernetes-Microservices/691061-2.html)
[**kelseyhightower/kubernetes-the-hard-way**
*Bootstrap Kubernetes the hard way on Google Cloud Platform. No scripts. - kelseyhightower/kubernetes-the-hard-way*github.com](https://github.com/kelseyhightower/kubernetes-the-hard-way)

Also, check the standard guide from the MetalLB site out. I started with this guide while I was learning about MetalLB [https://metallb.universe.tf/tutorial/layer2/](https://metallb.universe.tf/tutorial/layer2/).
