
# Securing Your Kubernetes Cluster in Google Cloud Platform

Your Kubernetes (K8S) cluster does the heavyweight lifting off when you run your microservice applications in Docker containers. Google Cloud Platform (GCP) provides managed K8S service named Google Kubernetes Engine (GKE). When you use GCP to host and run your microservices, it is important to make sure that your K8S cluster is set up in a highly secure fashion. Any kind of security model that you design and implement should be tried and tested right from the lowest environment such as Development before trying out in higher environments such as Production. This article is going to talk about securing your K8S cluster in GCP. The author has written Terraform scripts to manage all the GCP resources mentioned in this article. In-depth coverage of Terraform is beyond the scope of this article. The code can be accessed from the author’s Github repository.

![Private K8S Cluster in a VPC Managed Using a Bastion Host](https://cdn-images-1.medium.com/max/2000/1*5jOm1C2lTOEFSZjc4nv5rw.jpeg)*Private K8S Cluster in a VPC Managed Using a Bastion Host*

## Network Setup

GCP provides a network boundary abstraction named Virtual Private Cloud (VPC). A given VPC can have one or more subnets. Each subnet in a VPC can be bound to a GCP region such as *europe-west2* (London). When you create a GCP project (the environment in which you run your services in GCP), it comes with a *default* VPC. The *default* VPC provides you the required subnets for all the GCP regions, routing rules, and firewall rules. You don’t want to use this *default* VPC for running your production applications. To start with, define a custom VPC for your application say *mservice-network*. If you have to comply with any data residency rules or regulations such as [GDPR](https://eugdpr.org/), you don’t want to have subnets in all the GCP regions. In this article, the assumption is that your data cannot leave the United Kingdom and the only GCP region at the time of this writing is the *London* region of GCP. So you create only one subnet *mservice-subnetwork* in the VPC *mservice-network. *Most important attributes of a subnet include and not limited to IP range in [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) format, GCP region, the VPC network, and the secondary IP ranges. It is mandatory to have nonoverlapping IPs for the primary and secondary IP ranges of your subnet. In this context, two secondary IP ranges have been defined for 1) pods in the K8S cluster, 2) services running in the K8S cluster. When you define your IP ranges, make sure that you have enough room for growth. Typically you should have more IPs for your pods as compared to your services. The Terraform script for creating your VPC and its subnets described here are available from this [Github repository script file](https://github.com/rajtmana/gcp-terraform/blob/master/k8s-cluster/main.tf).

## K8S Setup

You have set up your VPC with only required subnets having enough IP ranges. Now it is time to set up your K8S cluster named *mservice-dev-cluster* in the VPC you have created. In other words, you are going to create a VPC aware K8S cluster and this means that all the nodes provisioned for building this K8S cluster will be bound to the selected VPC. When you define a K8S cluster, the following attributes are very important 1) region of the K8S cluster, 2) VPC, 3) subnet, 3) authentication settings, 4) networks that can access the K8S master, 5) private cluster settings, 6) IP allocation policy, 7) OAuth scope , 8) labels and 9) tags. In additions to these, you can include add-on services such as 1) HTTP load balancer, 2) horizontal pod autoscaling, K8S dashboard, and [Istio](https://istio.io/), the service mesh. It is important to reinforce the foundation and [K8S concepts](https://kubernetes.io/docs/concepts/) before attempting to set up your K8S cluster even though GCP is going to *manage* it. The Terraform script for the VPC and its subnets described here are available from this [Github repository script file](https://github.com/rajtmana/gcp-terraform/blob/master/k8s-cluster/main.tf).

### Authentication Settings

It is better to disable the basic authentication in the K8S cluster by defining explicit user names and hard-coded passwords. In the authentication settings of the K8S cluster, keeping both these attributes empty will prevent the basic authentication. It is important to have detailed coverage of this section but that deserves a couple of articles scratch the surface and it is beyond the scope of this article

### Networks Authorised to Access K8S Cluster

Whitelisting the list of networks that can access the K8S cluster will give you better control over your K8S cluster. Typically [bastion host](https://en.wikipedia.org/wiki/Bastion_host) in a network is used to manage the resources in general and the private K8S cluster in this context. If the bastion hosts are created in the same VPC as in the K8S cluster, then giving the IP range of the subnet in the white list is a good start. If that is not good enough, use the internal IP address of the bastion host explicitly and caution has to be exercised especially when the IP addresses are ephemeral.

If a user has the right privileges, he/she can use the GCP Cloud Shell and alter the K8S cluster settings such as adding their own Cloud Shell IP to access the K8S cluster. Care must be taken here to protect the access to your K8S cluster from GCP Cloud Shell. There are some loopholes in there and some of the issues are raised [in this discussion](https://stackoverflow.com/questions/55182670/using-cloud-shell-to-access-a-private-kubernetes-cluster-in-gcp).

### K8S Cluster Privacy Settings

It is important to make the K8S cluster as private. This means that the nodes used for building the K8S cluster will have only private IP addresses in GCP. This prevents the network from being accessed from outside using the public IPs of the nodes. In other words, the nodes in the K8S cluster are not exposed to the Internet. For achieving this, you have to enable a private endpoint for your K8S master and provide a K8S master CIDR block. K8S will create a separate VPC with just the addresses given in this CIDR block and it will peer it with your VPC. Making all these to have private IP addresses and whitelisting the networks that can access K8S master will make the K8S cluster protected from the Internet. If you recall the VPC and subnet setup details, you had defined two additional secondary IP ranges. Pick those two and assign one for the cluster pods and the other for the cluster services as the IP allocation policy of the K8S cluster. The node configuration setting should also include the required OAuth scopes and while doing this, makes sure that you are giving the required permissions for managing the nodes, logging, and monitoring.

### K8S Node Pool Settings

For a given K8S cluster to have perfect control over the node pool, the default node pool must be deleted first and use one that you have created. You have to make sure that the correct region is selected, K8S cluster is chosen and the initial node count per zone in a given region is selected. If your workload is ephemeral, then you can choose to have preemptible nodes in your K8S cluster. Otherwise, make sure that your nodes are not preemptible. The OAuth scope settings discussed in the above section needs to be done for the node pool as well.

## Bastion Host

Bastion host provides an entry point of a K8S cluster (in this context) and gives other resource management capabilities. Typically this is a Google Compute Engine VM created in the same VPC and subnet. This VM should have a public IP so that you can log in from anywhere. This can become a bit dangerous if you don’t do proper access control to this VM. If you have a direct connection or VPN connection to your GCP project from your corporate network, you can safely disable the external IP for your bastion host. In other words, only a very selected few should have access to this VM. Since this VM is in the same VPC and the subnet IP range is whitelisted in the master access list of the K8S cluster, this VM can be used to manage the cluster. So this VM should have the Google Cloud SDK installed and the required tools such as *kubectl*. This [page](https://cloud.google.com/sdk/docs/downloads-apt-get) talks about the installation and configuration of Cloud SDK in a Debian/Ubuntu VM. For the production networks, it is also a good practice to turn on this bastion host only when during the initial setup time or during the break-glass-scenarios.

## Firewall Rules

Once the bastion host is set up, you have to define the firewall rule to let you connect to the VM using SSH. For this purpose, for the given VPC, allow ingress traffic to TCP:22. If you have specific source IP ranges from where SSH connections can come such as your enterprise network from where there is VPN connectivity or direct connectivity to your GCP projects, you can whitelist them while creating the firewall rule. Otherwise, give the 0.0.0.0/0 CIDR range so that you can do SSH from anywhere. If the bastion host is created with specific network tags, you can include that while defining the firewall rule so that it will allow the ingress traffic to those VM instances having this specific network tags. These aspects are demonstrated in the [firewall creation Terraform script](https://github.com/rajtmana/gcp-terraform/blob/master/k8s-cluster/main.tf).

## Running Terraform Scripts

The [Terraform scripts](https://github.com/rajtmana/gcp-terraform) to create the above-described setup can be used as a starting point to build your production grade scripts. All the instructions to run it and the details of the pre-requisites are given there. Once your scripts are executed properly, the infrastructure should pass the following tests.
> 1. Your K8S cluster should be up and running properly. 
2. You must not be able to manage your K8S cluster through GCP Cloud Shell. You may [watch this space](https://stackoverflow.com/questions/55182670/using-cloud-shell-to-access-a-private-kubernetes-cluster-in-gcp) for any surprises
3. Your K8S cluster management should be possible only through SSH from your bastion host

### Cloud Shell Access

The following output gives evidence that the K8S cluster is not accessible through the GCP Cloud Shell even for the GCP project owner!

    $ gcloud container clusters get-credentials mservice-dev-cluster --region europe-west2
    Fetching cluster endpoint and auth data.
    kubeconfig entry generated for mservice-dev-cluster.

    $ kubectl config current-context
    gke_protean-XXXX_europe-west2_mservice-dev-cluster

    $ kubectl get services
    Unable to connect to the server: dial tcp 172.16.0.2:443: i/o timeout

### Bastion Host Access

As described earlier, it is a good practice to stop this bastion host VM once your activities are completed and start only when you need to access it again. By this way, you may prevent adversaries from trying to break into the bastion host and take control. You may use the K8S’s command line interface [*kubectl](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)* or REST interface to talk to your K8S cluster. The following section explains some of the basic K8S infrastructure management commands executed from the bastion host.
> 1. The credentials are downloaded to the bastion host. 
2. You will see a network peering that is created by the K8S cluster for the management talking to the current VPC. This is auto-generated
3. The configuration view listing gives the K8S server URL which is obviously a private IP accessible to the VPC through VPC network peering
4. The K8S service listing gives the default service and its ClusterIP which is again a private IP within the VPC
5. The K8S nodes listing give the details of the nodes in the K8S cluster and their IP addresses. Note that none of them are having any external and public IP addresses

    xxxx@mservice-bastion:~$ gcloud container clusters get-credentials mservice-dev-cluster --region europe-west2
    Fetching cluster endpoint and auth data.
    kubeconfig entry generated for mservice-dev-cluster.

    xxxx@mservice-bastion:~$ gcloud compute networks peerings list
    NAME                                     NETWORK           PEER_PROJECT                PEER_NETWORK                            AUTO_CREATE_ROUTES  STATE   STATE_DETAILS
    gke-e28b63f01c16da5811eb-e3c9-f81e-peer  mservice-network  gke-prod-europe-west2-320c  gke-e28b63f01c16da5811eb-e3c9-133e-net  True                ACTIVE  [2019-03-17T04:17:10.456-07:00]: Connected.

    xxxx@mservice-bastion:~$ kubectl config view
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: DATA+OMITTED
        server: [https://172.16.0.2](https://172.16.0.2)
      name: gke_protean-XXXX_europe-west2_mservice-dev-cluster
    contexts:
    - context:
        cluster: gke_protean-XXXX_europe-west2_mservice-dev-cluster
        user: gke_protean-XXXX_europe-west2_mservice-dev-cluster
      name: gke_protean-XXXX_europe-west2_mservice-dev-cluster
    current-context: gke_protean-XXXX_europe-west2_mservice-dev-cluster
    kind: Config
    preferences: {}
    users:
    - name: gke_protean-XXXX_europe-west2_mservice-dev-cluster
      user:
        auth-provider:
          config:
            cmd-args: config config-helper --format=json
            cmd-path: /usr/lib/google-cloud-sdk/bin/gcloud
            expiry-key: '{.credential.token_expiry}'
            token-key: '{.credential.access_token}'
          name: gcp

    xxxx@mservice-bastion:~$ kubectl get services
    NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    kubernetes   ClusterIP   10.94.0.1    <none>        443/TCP   16m

    xxxx@mservice-bastion:~$ kubectl get nodes -o wide
    NAME                                                  STATUS   ROLES    AGE   VERSION         INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                             KERNEL-VERSION   CONTAINER-RUNTIME
    gke-mservice-dev-clu-mservice-node-po-7e724977-jv92   Ready    <none>   23m   v1.11.7-gke.4   10.2.0.6                    Container-Optimized OS from Google   4.14.89+         docker://17.3.2
    gke-mservice-dev-clu-mservice-node-po-959f54dc-k8jj   Ready    <none>   23m   v1.11.7-gke.4   10.2.0.7                    Container-Optimized OS from Google   4.14.89+         docker://17.3.2
    gke-mservice-dev-clu-mservice-node-po-e935679d-zxnx   Ready    <none>   23m   v1.11.7-gke.4   10.2.0.8                    Container-Optimized OS from Google   4.14.89+         docker://17.3.2

## Conclusion

This article focused mainly on the most basic things that you need to do on setting up your private K8S cluster and managing it through bastion hosts in the same VPC. This article did not cover the user personas who will be using the K8S cluster, their authentication, and authorization. When you set up your K8S cluster, understand the application requirements, user personas in the ecosystem and do an extremely well infrastructure planning.
