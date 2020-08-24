Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m29[39m }

# Create a Kubernetes Cluster on Amazon EKS

This blog walks you through creating an Amazon EKS cluster. And in the next blogs, I will walk through to deploy an application in Cluster, creating ALB ingress, monitoring, logging, kube2iam and Auto Scalar on Kubernetes.

But before creating the cluster I would like to cover some basics first, and then a step-by-step guide on its implementation.

## What is Kubernetes?

Kubernetes (short name is **k8s**) is an open-source container orchestration system for automating application deployment, scaling, and management. It was originally designed by Google and is now maintained by the Cloud Native Computing Foundation. It aims to provide a “platform for automating deployment, scaling, and operations of application containers across clusters of hosts”. It works with a range of container tools, including Docker. Many cloud services offer a Kubernetes-based platform or infrastructure as a service (PaaS or IaaS) on which Kubernetes can be deployed as a platform-providing service. Many vendors also provide their own branded Kubernetes distributions like Google has “GKE”, Azure has AKS and AWS has “EKS”. So here I’m going to setup Kubernetes Amazon i.e EKS.

## What is Amazon EKS?

Amazon EKS (**Elastic Container Service for Kubernetes**) is a managed Kubernetes service that allows you to run Kubernetes on AWS without the hassle of managing the Kubernetes control plane.

The big benefit of EKS and other similar hosted Kubernetes services are taking away the operational burden involved in running this control plane. You deploy cluster worker nodes using defined AMIs and with the help of CloudFormation\Terraform, and EKS will provision, scale and manage the Kubernetes control plane for you to ensure high availability, security, and scalability.

Now, let’s take a look at how Amazon EKS actually works.

## How does Amazon EKS work?

![](https://cdn-images-1.medium.com/max/3824/1*iVAbqtsgaUSgveuv_RyQmg.png)

1. Create an Amazon EKS cluster using “[AWS Management Console](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)” or with the terraform or one of the AWS EKS CLI “ekctl”.

1. Then, launch worker nodes and enable them to join the Amazon EKS cluster.

1. Once a cluster is ready, configure Kube config to communicate with the cluster using kubectl tool.

1. And deploy and manage applications on the Amazon EKS cluster.

Here, my goal is to deploy Kubernetes on AWS in the simplest way. I have chosen minimal setup and created an ekctl command-line script for automating to create the EKS cluster.

Please make sure you have the following components installed and set up before you start with Amazon EKS:

* aws cli & eksctl— while you can use the AWS Console to create a cluster in EKS, the [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html) CLI is easier. You will need version AWS CLI 1.16.73 at least. For further instructions, click [here](https://docs.aws.amazon.com/cli/latest/userguide/install-linux-al2017.html).

* Kubectl — used for communicating with the cluster API server. For further instructions on installing, click [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

* AWS-IAM-Authenticator — to allow IAM authentication with the Kubernetes cluster. For further instructions on installing, click [here](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html).

* And one server with AWS Account Access.

**Let’s start working on it!!**

First, clone the Github [repo](https://github.com/anup1384/eks-aws.git) and make the necessary adjustments in the [environment files](https://raw.githubusercontent.com/anup1384/eks-aws/master/environments/nonprod/ap-south-1-mumbai.sh). This repository has an ekctl command-line script for automating to create the EKS cluster.

    git clone [https://github.com/anup1384/eks-aws.git](https://github.com/anup1384/eks-aws.git)

***Note:** Here I have used existing VPC id and subnets. You can also change the VPC id and subnet details. If you didn’t specify VPC and Subent details it will automatically create new VPC and Subnets. For more details, click [here](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html).*

After the adjustments of the variable you can start running the script:

chmod +x deploy.sh

./deploy.sh ENVIRONMENT

With the script, you should pass the environment like:

./deploy.sh nonprod|mgmt|prod

It takes about 10–12 minutes to create a cluster.

**Configure kubectl for Amazon EKS:-**

I assume you have already installed kubectl and aws-iam-authenticator in your machine. Run below command to configure kubectl for AWS EKS.

    aws eks --region <region> update-kubeconfig --name <clusterName>

Now test your configuration.

    kubectl get svc && kubectl get nodes

Output:

![](https://cdn-images-1.medium.com/max/2000/1*Q22fqvewWHK5s0xNL-dv4g.png)

![](https://cdn-images-1.medium.com/max/2000/1*5clsg3GE6ZALe1aFR8ePlA.png)

I hope this blog was useful to you. Looking forward to claps and suggestions. For any queries, feel free to comment.

For more related content, visit [https://opendevops.in/](https://opendevops.in/)

Don’t forget to check out my other posts:

1. [Deploying and Scaling Jenkins on Kubernetes](https://medium.com/faun/deploying-and-scaling-jenkins-on-kubernetes-2cd4164720bd)

1. [Setup Elastic Search cluster, Kibana & Fluentd on Kubernetes with X-pack Security: Part-1](https://medium.com/faun/setup-elastic-search-cluster-kibana-fluentd-on-kubernetes-with-x-pack-security-part-1-271e57c2fe19)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬**.**

**To join our community Slack team chat **🗣️ **read our weekly Faun topics **🗞️,** and connect with the community **📣** click here⬇**

![](https://cdn-images-1.medium.com/max/3000/1*6P3WpLjGv5v1ucm5dgkucg.png)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
