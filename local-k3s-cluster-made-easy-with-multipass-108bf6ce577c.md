Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m71[39m, end: [33m76[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m83[39m, end: [33m93[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m110[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m33[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m80[39m, end: [33m85[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m131[39m, end: [33m141[39m }

# Local K3s Cluster Made Easy With Multipass



A couple of weeks ago, I stumble upon [Multipass](https://multipass.run), a tool that allows you to spin up Ubuntu VMs in a matter of seconds on a Mac, Linux, or Windows workstation.

Depending upon your OS, Multipass uses Hyper-V, HyperKit, KVM, or VirtualBox natively for the fastest startup time.

In this short article, we’ll set up a K3s Kubernetes cluster on virtual machines created with Multipass.

## Getting Multipass

Just head up to [Multipass](https://multipass.run/), and download the binary for your platform. Once installed, you can get the list of the available commands. We’ll only use a couple of them in this article.

    $ multipass
    Usage: multipass [options] <command>
    Create, control and connect to Ubuntu instances.

    This is a command line utility for multipass, a
    service that manages Ubuntu instances.

    Options:
      -h, --help     Display this help
      -v, --verbose  Increase logging verbosity, repeat up to three times for more
                     detail

    Available commands:
      delete    Delete instances
      exec      Run a command on an instance
      find      Display available images to create instances from
      get       Get a configuration option
      help      Display help about a command
      info      Display information about instances
      launch    Create and start an Ubuntu instance
      list      List all available instances
      mount     Mount a local directory in the instance
      purge     Purge all deleted instances permanently
      recover   Recover deleted instances
      restart   Restart instances
      set       Set a configuration option
      shell     Open a shell on a running instance
      start     Start instances
      stop      Stop running instances
      suspend   Suspend running instances
      transfer  Transfer files between the host and instances
      umount    Unmount a directory from an instance
      version   Show version details

## **Creation of the VMs**

First, we create three VMs, keeping the default configuration of one CPU and 1GB RAM.

    $ multipass launch -n node1
    $ multipass launch -n node2
    $ multipass launch -n node3

Next, we verify the three VMs are in the running state

    $ multipass list
    Name         State             IPv4             Image
    node3        Running           192.168.64.13    Ubuntu 18.04 LTS
    node2        Running           192.168.64.12    Ubuntu 18.04 LTS
    node1        Running           192.168.64.11    Ubuntu 18.04 LTS

**Note:** In this example, I use Multipass on MacOS, so the VMs are run on HyperKit as I can confirm listing the HyperKit-related processes.

    luc@saturn:~$ ps aux | grep -i "[h]yperkit"
    ... com.docker.hyperkit ...
    ... com.canonical.multipass/bin/hyperkit...
    ... com.canonical.multipass/bin/hyperkit...
    ... com.canonical.multipass/bin/hyperkit...

Among those processes, the first one is the Docker daemon already running on my machine. The next three are the VMs created above.

## Initialize K3s on node1

First, we run a command on node1 (using the *exec* subcommand of Multipass) to install K3s as defined in the [documentation](https://k3s.io).

    $ multipass exec node1 -- \
      bash -c "curl -sfL https://get.k3s.io | sh -"

**Note:** The K3s initialisation took less than 30 seconds … really impressive for a certified Kubernetes distribution.

Then, from the local machine, we gather the elements we’ll need to add additional nodes to the cluster:

* The token to join the cluster

    $ TOKEN=$(multipass exec node1 sudo cat /var/lib/rancher/k3s/server/node-token)

* The IP of the API server running on node1

    $ IP=$(multipass info node1 | grep IPv4 | awk '{print $2}')

## Join node2 and node3 to the Cluster

From the local machine, we run the following commands to add node2 and node3 to the cluster.

    # Joining node2
    $ multipass exec **node2** -- \
    bash -c "curl -sfL https://get.k3s.io | K3S_URL=\"https://$IP:6443\" K3S_TOKEN=\"$TOKEN\" sh -"

    # Joining node3
    $ multipass exec **node3** -- \
    bash -c "curl -sfL https://get.k3s.io | K3S_URL=\"https://$IP:6443\" K3S_TOKEN=\"$TOKEN\" sh -"

As we can see, those commands use the TOKEN and IP address we defined above.

## Get the Cluster Configuration

Running the following command, we can list all the nodes of the cluster:

    $ multipass exec node1 -- sudo kubectl get nodes
    NAME    STATUS   ROLES    AGE     VERSION
    node1   Ready    master   5m31s   v1.16.2-k3s.1
    node3   Ready    <none>   21s     v1.16.2-k3s.1
    node2   Ready    <none>   45s     v1.16.2-k3s.1

To access the cluster’s API server from our local machine, we only need to get the kubeconfig file created on node1 during the setup.

    $ multipass exec node1 sudo cat /etc/rancher/k3s/k3s.yaml > k3s.yaml

We then need to change the server key so it references the remote IP address of node1 instead of the localhost.

    sed -i '' "s/127.0.0.1/$IP/" k3s.yaml

Finally, we configure our local kubectl so it uses the kubeconfig file (k3s.yaml) we’ve just retrieved. A simple way is to set the KUBECONFIG environment variable so it points towards the configuration file.

    export KUBECONFIG=$PWD/k3s.yaml

We’re all set, we can now communicate with the cluster from your local machine.

    $ kubectl get nodes
    NAME STATUS ROLES AGE VERSION
    node1 Ready master 10h v1.16.2-k3s.1
    node2 Ready <none> 10h v1.16.2-k3s.1
    node3 Ready <none> 10h v1.16.2-k3s.1

Note: all the steps defined in this post can be run from the following gist:

[https://gist.github.com/lucj/5a0e2286b40130d02388a264e6924ed4](https://gist.github.com/lucj/5a0e2286b40130d02388a264e6924ed4)

### Summary

Multipass is a very useful tool. Its integration with low-level hypervisors makes it a good choice to deploy multiple VMs locally and to play with many distributed solutions such as the great K3s Kubernetes distribution.
