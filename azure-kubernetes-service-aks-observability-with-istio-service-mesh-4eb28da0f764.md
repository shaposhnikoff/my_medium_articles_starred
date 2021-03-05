
# Azure Kubernetes Service (AKS) Observability with Istio Service Mesh

In the last two-part post, Kubernetes-based Microservice Observability with Istio Service Mesh, we deployed Istio, along with its observability tools, Prometheus, Grafana, Jaeger, and Kiali, to Google Kubernetes Engine (GKE). Following that post, I received several questions about using Istio’s observability tools with other popular managed Kubernetes platforms, primarily Azure Kubernetes Service (AKS). In most cases, including with AKS, both Istio and the observability tools are compatible.

In this short follow-up of the last post, we will replace the GKE-specific cluster setup commands, found in [part one](https://programmaticponderings.com/2019/03/10/kubernetes-based-microservice-observability-with-istio-service-mesh-part-1/) of the last post, with new commands to provision a similar AKS cluster on Azure. The new AKS cluster will run Istio 1.1.3, [released](https://istio.io/about/notes/1.1.3/) 4/15/2019, alongside the latest available version of AKS (Kubernetes), 1.12.6. We will replace Google’s Stackdriver logging with [Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/overview) logs. We will retain the external MongoDB Atlas cluster and the external CloudAMQP cluster dependencies.

Previous articles about AKS include [First Impressions of AKS, Azure’s New Managed Kubernetes Container Service](https://programmaticponderings.com/2017/11/20/first-impressions-of-aks-azures-new-managed-kubernetes-container-service/) (November 2017) and [Architecting Cloud-Optimized Apps with AKS (Azure’s Managed Kubernetes), Azure Service Bus, and Cosmos DB](https://programmaticponderings.com/2017/12/10/architecting-cloud-optimized-apps-with-aks-azures-managed-kubernetes-azure-service-bus-and-cosmos-db/) (December 2017).

## Source Code

All source code for this post is available on GitHub in two projects. The Go-based microservices source code, all Kubernetes resources, and all deployment scripts are located in the [k8s-istio-observe-backend](https://github.com/garystafford/k8s-istio-observe-backend) project repository.

    git clone \
      --branch master --single-branch \
      --depth 1 --no-tags \
      [https://github.com/garystafford/k8s-istio-observe-backend.git](https://github.com/garystafford/k8s-istio-observe-backend.git)

The Angular UI [TypeScript-based](https://en.wikipedia.org/wiki/TypeScript) source code is located in the [k8s-istio-observe-frontend](https://github.com/garystafford/k8s-istio-observe-frontend) repository. You will not need to clone the Angular UI project for this post’s demonstration.

## Setup

This post assumes you have a Microsoft Azure account with the necessary resource providers registered, and the [Azure Command-Line Interface](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) (CLI), az, installed and available to your command shell. You will also need [Helm](https://helm.sh/) and [Istio 1.1.3](https://github.com/istio/istio/releases/tag/1.1.3) installed and configured, which is covered in the last post.

![](https://cdn-images-1.medium.com/max/2000/0*IiHmH81dNMNLK_Y8)

Start by logging into Azure from your command shell.

    az login \
      --username {{ your_username_here }} \
      --password {{ your_password_here }}

## Resource Providers

If you are new to Azure or AKS, you may need to register some additional [resource providers](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-supported-services) to complete this demonstration.

    az provider list --output table

![](https://cdn-images-1.medium.com/max/2000/0*gNCsgv6yJ5tokRkr)

If you are missing required resource providers, you will see errors similar to the one shown below. Simply activate the particular provider corresponding to the error.

    Operation failed with status:'Bad Request'. 
    Details: Required resource provider registrations 
    **Microsoft.Compute**, **Microsoft.Network** are missing.

To register the necessary providers, use the Azure CLI or the Azure Portal UI.

    az provider register --namespace Microsoft.ContainerService
    az provider register --namespace Microsoft.Network
    az provider register --namespace Microsoft.Compute

## Resource Group

AKS requires an [Azure Resource Group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview). According to Azure, a resource group is a container that holds related resources for an Azure solution. The resource group includes those resources that you want to manage as a group. I chose to create a new resource group associated with my closest geographic [Azure Region](https://azure.microsoft.com/en-us/global-infrastructure/regions/), East US, using the Azure CLI.

    az group create \
      --resource-group aks-observability-demo \
      --location eastus

![](https://cdn-images-1.medium.com/max/2000/0*oxkADqk47kEoXUvj)

## Create the AKS Cluster

Before creating the GKE cluster, check for the latest versions of AKS. At the time of this post, the latest versions of AKS was 1.12.6.

    az aks get-versions \
      --location eastus \
      --output table

![](https://cdn-images-1.medium.com/max/2000/0*gGwclmhD4RwtW_Rp)

Using the latest GKE version, create the GKE managed cluster. There are many configuration options available with the az aks create command. For this post, I am creating three worker nodes using the Azure [Standard_DS3_v2](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-general#dsv2-series) VM type, which will give us a total of 12 vCPUs and 42 GB of memory. Anything smaller and all the Pods may not be schedulable. Instead of supplying an existing SSH key, I will let Azure create a new one. You should have no need to SSH into the worker nodes. I am also enabling the [monitoring add-on](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-onboard). According to Azure, the add-on sets up [Azure Monitor](https://azure.microsoft.com/en-us/services/monitor/) for containers, announced in December 2018, which monitors the performance of workloads deployed to Kubernetes environments hosted on AKS.

    time az aks create \
      --name aks-observability-demo \
      --resource-group aks-observability-demo \
      --node-count 3 \
      --node-vm-size Standard_DS3_v2 \
      --enable-addons monitoring \
      --generate-ssh-keys \
      --kubernetes-version 1.12.6

Using the time command, we observe that the cluster took approximately 5m48s to provision; I have seen times up to almost 10 minutes. AKS provisioning is not as fast as GKE, which in my experience is at least 2x-3x faster than AKS for a similarly sized cluster.

![](https://cdn-images-1.medium.com/max/2000/0*nkCpwKGreup27INk)

After the cluster creation completes, retrieve your AKS cluster credentials.

    az aks get-credentials \
      --name aks-observability-demo \
      --resource-group aks-observability-demo \
      --overwrite-existing

## Examine the Cluster

Use the following command to confirm the cluster is ready by examining the status of three worker nodes.

    kubectl get nodes --output=wide

![](https://cdn-images-1.medium.com/max/2000/0*Qzvo44bzW7hOVnJT)

Observe that Azure currently uses [Ubuntu 16.04.5 LTS](http://releases.ubuntu.com/16.04/) for the worker node’s host operating system. If you recall, GKE offers both Ubuntu as well as a [Container-Optimized OS from Google](https://cloud.google.com/container-optimized-os).

### Kubernetes Dashboard

Unlike [GKE](https://cloud.google.com/kubernetes-engine/docs/concepts/dashboards), there is no custom AKS dashboard. Therefore, we will use the [Kubernetes Web UI](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) (dashboard), which is installed by default with AKS, unlike GKE. According to [Azure](https://docs.microsoft.com/en-us/azure/aks/kubernetes-dashboard#for-rbac-enabled-clusters), to make full use of the dashboard, since the AKS cluster uses [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/), a ClusterRoleBinding must be created before you can correctly access the dashboard.

    kubectl create clusterrolebinding kubernetes-dashboard \
      --clusterrole=cluster-admin \
      --serviceaccount=kube-system:kubernetes-dashboard

Next, we must create a proxy tunnel on local port 8001 to the dashboard running on the AKS cluster. This CLI command creates a proxy between your local system and the Kubernetes API and opens your web browser to the Kubernetes dashboard.

    az aks browse \
      --name aks-observability-demo \
      --resource-group aks-observability-demo

![](https://cdn-images-1.medium.com/max/2000/0*qRvQXip7AgA-Hg6S)

Although you should use the Azure CLI, PowerShell, or [SDK](https://azure.microsoft.com/en-us/downloads/) for all your AKS configuration tasks, using the dashboard for monitoring the cluster and the resources running on it, is invaluable.

![](https://cdn-images-1.medium.com/max/2000/0*WBRZhUqKZ8eFSTY2)

The Kubernetes dashboard also provides access to raw container logs. Azure Monitor provides the ability to construct complex log queries, but for quick troubleshooting, you may just want to see the raw logs a specific container is outputting, from the dashboard.

![](https://cdn-images-1.medium.com/max/2000/0*HJbzslCRJIdOhXJ0)

### Azure Portal

Logging into the [Azure Portal](https://azure.microsoft.com/en-us/features/azure-portal/), we can observe the AKS cluster, within the new Resource Group.

![](https://cdn-images-1.medium.com/max/2000/0*SPfu5yUYEWI91Cji)

In addition to the Azure Resource Group we created, there will be a second Resource Group created automatically during the creation of the AKS cluster. This group contains all the resources that compose the AKS cluster. These resources include the three worker node VM instances, and their corresponding storage disks and NICs. The group also includes a network security group, route table, virtual network, and an [availability set](https://social.technet.microsoft.com/wiki/contents/articles/51828.azure-vms-availability-sets-and-availability-zones.aspx#Availability_Set).

![](https://cdn-images-1.medium.com/max/2000/0*3rZvH2Uys8isy1Y0)

## Deploy Istio

From this point on, the process to deploy Istio Service Mesh and the Go-based microservices platform follows the previous post and use the exact same scripts. After modifying the Kubernetes resource files, to deploy Istio, use the bash script, [part4_install_istio.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part4_install_istio.sh). I have added a few more pauses in the script to account for the apparently slower response times from AKS as opposed to GKE. It definitely takes longer to spin up the Istio resources on AKS than on GKE, which can result in errors if you do not pause between each stage of the deployment process.

![](https://cdn-images-1.medium.com/max/2000/0*ectMZV9fUHbLR5-A)

![](https://cdn-images-1.medium.com/max/2000/0*MyhmamLHD-6ZMe7T)

Using the Kubernetes dashboard, we can view the Istio resources running in the istio-system Namespace, as shown below. Confirm that all resource Pods are running and healthy before deploying the Go-based microservices platform.

![](https://cdn-images-1.medium.com/max/2000/0*_1QRM_KqhdO9ez4Z)

## Deploy the Platform

Deploy the Go-based microservices platform, using bash deploy script, [part5a_deploy_resources.sh](https://github.com/garystafford/golang-srv-demo/blob/master/part5a_deploy_resources.sh).

![](https://cdn-images-1.medium.com/max/2000/0*pIvYNYnTLl2FiokO)

The script deploys two replicas (Pods) of each of the eight microservices, Service-A through Service-H, and the Angular UI, to the dev and test Namespaces, for a total of 36 Pods. Each Pod will have the [Istio sidecar proxy](https://istio.io/docs/concepts/what-is-istio/#envoy) (Envoy Proxy) injected into it, alongside the microservice or UI.

![](https://cdn-images-1.medium.com/max/2000/0*kx-muXdOzf7d851m)

### Azure Load Balancer

If we return to the Resource Group created automatically when the AKS cluster was created, we will now see two additional resources. There is now an [Azure Load Balancer](https://azure.microsoft.com/en-us/services/load-balancer/) and Public IP Address.

![](https://cdn-images-1.medium.com/max/2000/0*B6PqMQtgmjc5sJsS)

Similar to the GKE cluster in the last post, when the Istio Ingress Gateway is deployed as part of the platform, it is materialized as an [Azure Load Balancer](https://azure.microsoft.com/en-us/services/load-balancer/). The front-end of the load balancer is the new public IP address. The back-end of the load-balancer is a pool containing the three AKS worker node VMs. The load balancer is associated with a set of rules and health probes.

![](https://cdn-images-1.medium.com/max/2000/0*CmbOTdWCv9IA0al5)

## DNS

I have associated the new Azure public IP address, connected with the front-end of the load balancer, with the four subdomains I am using to represent the UI and the edge service, Service-A, in both Namespaces. If Azure is your primary Cloud provider, then [Azure DNS](https://docs.microsoft.com/en-us/azure/dns/) is a good choice to manage your domain’s DNS records. For this demo, you will require your own domain.

![](https://cdn-images-1.medium.com/max/2000/0*t_ZxOjDf6HCGNQyO)

## Testing the Platform

With everything deployed, test the platform is responding and generate HTTP traffic for the observability tools to record. Similar to last time, I have chosen [hey](https://github.com/rakyll/hey), a modern load generator and benchmarking tool, and a worthy replacement for Apache Bench (ab). Unlike ab, hey supports HTTP/2. Below, I am running hey directly from [Azure Cloud Shell](https://azure.microsoft.com/en-us/features/cloud-shell/). The tool is simulating 10 concurrent users, generating a total of 500 HTTP GET requests to Service A.

    # quick setup from Azure Shell using Bash
    go get -u github.com/rakyll/hey
    cd go/src/github.com/rakyll/hey/
    go build
      
    ./hey -n 500 -c 10 -h2 http://api.dev.example-api.com/api/ping

We had 100% success with all 500 calls resulting in an HTTP 200 OK success status response code. Based on the results, we can observe the platform was capable of approximately 4 requests/second, with an average response time of 2.48 seconds and a mean time of 2.80 seconds. Almost all of that time was the result of waiting for the response, as the details indicate.

![](https://cdn-images-1.medium.com/max/2000/0*DlstiqvmDiP0-Qji)

## Logging

In this post, we have replaced GCP’s Stackdriver logging with [Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/overview) logs. According to Microsoft, Azure Monitor maximizes the availability and performance of applications by delivering a comprehensive solution for collecting, analyzing, and acting on telemetry from Cloud and on-premises environments. In my opinion, Stackdriver is a superior solution for searching and correlating the logs of distributed applications running on Kubernetes. I find the interface and query language of Stackdriver easier and more intuitive than Azure Monitor, which although powerful, requires substantial query knowledge to obtain meaningful results. For example, here is a query to view the log entries from the services in the dev Namespace, within the last day.

    let startTimestamp = ago(1d);
    KubePodInventory
    | where TimeGenerated > startTimestamp
    | where ClusterName =~ "aks-observability-demo"
    | where Namespace == "dev"
    | where Name contains "service-"
    | distinct ContainerID
    | join
    (
        ContainerLog
        | where TimeGenerated > startTimestamp
    )
    on ContainerID
    | project LogEntrySource, LogEntry, TimeGenerated, Name
    | order by TimeGenerated desc
    | render table

Below, we see the Logs interface with the search query and log entry results.

![](https://cdn-images-1.medium.com/max/2000/0*HpDu-QB1xbAPWay3)

Below, we see a detailed view of a single log entry from Service A.

![](https://cdn-images-1.medium.com/max/2000/0*oUvax2xUHZRwqIXX)

## Observability Tools

The previous post goes into greater detail on the features of each of the observability tools provided by Istio, including Prometheus, Grafana, Jaeger, and Kiali.

We can use the exact same kubectl port-forward commands to connect to the tools on AKS as we did on GKE. According to Google, Kubernetes [port forwarding](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) allows using a resource name, such as a service name, to select a matching pod to port forward to since Kubernetes v1.10. We forward a local port to a port on the tool’s pod.

    # Grafana
    kubectl port-forward -n istio-system \
      $(kubectl get pod -n istio-system -l app=grafana \
      -o jsonpath='{.items[0].metadata.name}') 3000:3000 &
      
    # Prometheus
    kubectl -n istio-system port-forward \
      $(kubectl -n istio-system get pod -l app=prometheus \
      -o jsonpath='{.items[0].metadata.name}') 9090:9090 &
      
    # Jaeger
    kubectl port-forward -n istio-system \
    $(kubectl get pod -n istio-system -l app=jaeger \
    -o jsonpath='{.items[0].metadata.name}') 16686:16686 &
      
    # Kiali
    kubectl -n istio-system port-forward \
      $(kubectl -n istio-system get pod -l app=kiali \
      -o jsonpath='{.items[0].metadata.name}') 20001:20001 &

![](https://cdn-images-1.medium.com/max/2000/0*PxX9eibRaJw5Rldf)

### Prometheus and Grafana

[Prometheus](https://prometheus.io/) is a completely open source and community-driven systems monitoring and alerting toolkit originally built at SoundCloud, circa 2012. Interestingly, Prometheus joined the [Cloud Native Computing Foundation](https://cncf.io/) (CNCF) in 2016 as the second hosted-project, after [Kubernetes](http://kubernetes.io/).

Grafana describes itself as the leading open source software for time series analytics. According to [Grafana Labs,](https://grafana.com/grafana) Grafana allows you to query, visualize, alert on, and understand your metrics no matter where they are stored. You can easily create, explore, and share visually-rich, data-driven dashboards. Grafana also users to visually define alert rules for your most important metrics. Grafana will continuously evaluate rules and can send notifications.

According to [Istio](https://istio.io/docs/tasks/telemetry/using-istio-dashboard/#about-the-grafana-add-on), the Grafana add-on is a pre-configured instance of Grafana. The Grafana Docker base image has been modified to start with both a Prometheus data source and the Istio Dashboard installed. Below, we see one of the pre-configured dashboards, the Istio Service Dashboard.

![](https://cdn-images-1.medium.com/max/2000/0*EfxpobzIF98sjahK)

### Jaeger

According to their website, [Jaeger](https://www.jaegertracing.io/docs/1.10/), inspired by [Dapper](https://research.google.com/pubs/pub36356.html) and [OpenZipkin](http://zipkin.io/), is a distributed tracing system released as open source by [Uber Technologies](http://uber.github.io/). It is used for monitoring and troubleshooting microservices-based distributed systems, including distributed context propagation, distributed transaction monitoring, root cause analysis, service dependency analysis, and performance and latency optimization. The Jaeger [website](https://www.jaegertracing.io/docs/1.10/architecture/) contains a good overview of Jaeger’s architecture and general tracing-related terminology.

![](https://cdn-images-1.medium.com/max/2000/0*JkJ3bKseA6tmo8tj)

Below, we see a typical, distributed trace of the services, starting ingress gateway and passing across the upstream service dependencies.

![](https://cdn-images-1.medium.com/max/2000/0*QDZ35d6AvluCYxZM)

### Kaili

According to their [website](https://www.kiali.io/documentation/overview/), Kiali provides answers to the questions: What are the microservices in my Istio service mesh, and how are they connected? Kiali works with Istio, in OpenShift or Kubernetes, to visualize the service mesh topology, to provide visibility into features like circuit breakers, request rates and more. It offers insights about the mesh components at different levels, from abstract Applications to Services and Workloads.

There is a common Kubernetes [Secret](https://istio.io/docs/tasks/telemetry/kiali/#before-you-begin) that controls access to the Kiali API and UI. The default login is admin, the password is 1f2d1e2e67df.

![](https://cdn-images-1.medium.com/max/2000/0*mem4E3P21fA9kGno)

Below, we see a detailed view of our platform, running in the dev namespace, on AKS.

![](https://cdn-images-1.medium.com/max/2000/0*4CQ_yNvheXTqdryA)

## Delete AKS Cluster

Once you are finished with this demo, use the following two commands to tear down the AKS cluster and remove the cluster context from your local configuration.

    time az aks delete \
      --name aks-observability-demo \
      --resource-group aks-observability-demo \
      --yes

    kubectl config delete-context aks-observability-demo

## Conclusion

In this brief, follow-up post, we have explored how the current set of observability tools, part of the latest version of Istio Service Mesh, integrates with Azure Kubernetes Service (AKS).

*All opinions expressed in this post are my own and not necessarily the views of my current or past employers or their clients.*
