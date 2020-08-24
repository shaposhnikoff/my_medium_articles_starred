Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m74[39m }

# Running K3s with Multipass on Mac

K3s is the lightweight Kubernetes distribution that is just freshly baked. As its lightweight, it’s ideal to run on a laptop for a developer to explore and experiment with it. But K3s is natively available for Linux. How can we run it on Mac?

Entering Multipass. First, let's install the multipass with the brew.

    brew search multipass
    brew cask install multipass

Now create a VM with multipass, assuming 1GB memory and 5GB disk.

    multipass launch --name k3s --mem 1G --disk 5G
    Launched: k3s

Wait for the VM created, then open a shell to the VM,

    multipass shell k3s

We have the shell for the VM then, run curl -sfL [https://get.k3s.io](https://get.k3s.io) | sh - to install K3s.

    [INFO]  Finding latest release
    [INFO]  Using v0.2.0 as release
    [INFO]  Downloading hash [https://github.com/rancher/k3s/releases/download/v0.2.0/sha256sum-amd64.txt](https://github.com/rancher/k3s/releases/download/v0.2.0/sha256sum-amd64.txt)
    [INFO]  Downloading binary [https://github.com/rancher/k3s/releases/download/v0.2.0/k3s](https://github.com/rancher/k3s/releases/download/v0.2.0/k3s)
    [INFO]  Verifying binary download
    [INFO]  Installing k3s to /usr/local/bin/k3s
    [INFO]  Creating /usr/local/bin/kubectl symlink to k3s
    [INFO]  Creating /usr/local/bin/crictl symlink to k3s
    [INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
    [INFO]  systemd: Creating environment file /etc/systemd/system/k3s.service.env
    [INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
    [INFO]  systemd: Enabling k3s unit
    Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
    [INFO]  systemd: Starting k3s

Then we have K3s installed and running on Macbook. Validate it with kubectl

    multipass@k3s:~$ kubectl get nodes
    NAME   STATUS   ROLES    AGE     VERSION
    k3s    Ready    <none>   7m14s   v1.13.4-k3s.1
    
    multipass@k3s:~$ kubectl get pods --all-namespaces
    NAMESPACE     NAME                             READY   STATUS      RESTARTS   AGE
    kube-system   coredns-7748f7f6df-dnsp2         1/1     Running     0          7m15s
    kube-system   helm-install-traefik-nqvg8       0/1     Completed   0          7m15s
    kube-system   svclb-traefik-6659944cc7-f6rdc   2/2     Running     0          6m53s
    kube-system   traefik-5cc8776646-99c66         1/1     Running     0          6m53s
