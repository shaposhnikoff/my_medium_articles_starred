Unknown markup type 10 { type: [33m10[39m, start: [33m43[39m, end: [33m49[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m32[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m437[39m, end: [33m451[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m233[39m, end: [33m276[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m135[39m, end: [33m171[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m118[39m, end: [33m146[39m }

# Create a Kubernetes Cluster and Deploy an App on Amazon EKS using AWS CLI



*This **blog** describes how create **multi-node kubernetes clusters** and **deploy** an application on **Amazon EKS**. In this blog we will walk through how to setup kubectl and eksctl on workstations and from there we can use one line command to create an EKS Cluster.*

## Amazon Elastic Container Service for Kubernetes(Amazon EKS)

**Kubernetes** is a container orchestration system from Google and has emerged as the platform of choice for deploying cloud-native applications. Kubernetes is a versatile tool for automating and simplifying your container workflow that gives you limitless scalability at a moment’s notice.

**Amazon EKS** is a fully managed [Kubernetes](https://aws.amazon.com/kubernetes/) service. Customers such as Intel, Snap, Intuit, GoDaddy, and Autodesk trust EKS to run their most sensitive and mission critical applications because of its security, reliability, and scalability.

## **Benefits of Amazon EKS**

1. **Control Plane Monitoring: **With Amazon EKS there’s no need to install, operate, or maintain your own Kubernetes control plane. EKS removes the need to architect high availability and scalability for your master nodes, so administrators can focus on their cluster and workloads.

1. **High Availability: **EKS runs the Kubernetes management infrastructure across multiple AWS Availability Zones, automatically detects and replaces unhealthy control plane nodes, and provides on-demand, zero downtime upgrades and patching.

1. **Security:** EKS automatically applies the latest security patches to your cluster control plane. AWS also works closely with the community to ensure critical security issues are addressed before new releases and patches are deployed to existing cluster

1. **Serverless Option: **EKS supports AWS Fargate to provide serverless compute for containers. Fargate removes the need to provision and manage servers, lets you specify and pay for resources per application, and improves security through application isolation by design.

1. **Build with Community**: EKS runs upstream Kubernetes and is certified Kubernetes conformant, so applications managed by EKS are fully compatible with applications managed by any standard Kubernetes environment. AWS actively works with the Kubernetes community, including making contributions to the Kubernetes code base that help you take advantage of AWS services and features.

**PRE-REQUISITES AND INSTALLATIONS**

***How to install aws-cli?***

Follow the [instructions here](https://cloudacademy.com/blog/how-to-use-aws-cli/) to install and configure aws-cli. When you’re done, you should be able to run the aws command:

    $ aws --version
    aws-cli/1.11.34 Python/2.7.12 Linux/4.4.0-177-generic botocore/1.5.95

***How to install eksctl?***

1. Download and extract the latest release of eksctl with the following command.

    curl --silent --location "[https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname](https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname) -s)_amd64.tar.gz" | tar xz -C /tmp

2. Move the extracted binary to /usr/local/bin.

    sudo mv /tmp/eksctl /usr/local/bin

3. Test that your installation was successful with the following command.

    $ eksctl version
    0.23.0

***How to install kubectl?***

1. Installing kubectl. Download the Amazon EKS-vended **kubectl** binary.

    curl -o kubectl [https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/linux/amd64/kubectl](https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/linux/amd64/kubectl)

2. Apply execute permissions to the binary.

    chmod +x ./kubectl

3. Move **kubectl** to a folder that is in your path.

    sudo mv ./kubectl /usr/local/bin

4. After you install **kubectl**, you can verify its version with the following command:

    $ kubectl version --short --client
      Client Version: v1.18.2

## Let’s begin…

***Step 1: Writing k8s yaml configuration to create multi-node Kubernetes Cluster on Amazon EKS***

*To create multi-node Kubernetes cluster in Amazon EKS we have written a custom cluster yaml configuration. The below configuration will create a kubernetes cluster called lwcluster in EKS consisting of three node group — ng1, ng2 and ng-mixed. Amazon EKS supports both on-demand and spot instances. The ng-mixed is an example of a node group that uses 50% spot instances and 50% on-demand instance.*

<iframe src="https://medium.com/media/586a4109c4c1ac46d476e7edc34bdcfc" frameborder=0></iframe>

***Step 2: Creation** **of multi-node Kubernetes Cluster on Amazon EKS***

*Create the multi-node kubernetes cluster in Amazon EKS using the following command:*

    $ eksctl create cluster -f eks-cluster.yaml

*The output should look like this (**this is example output**):*

![](https://cdn-images-1.medium.com/max/2006/1*UW0xgfa6o-HwGEs59YyTIQ.png)

*The above command will set up the AWS Identity and Access Management (IAM) Role for the master control plane to connect to EKS. It will creates the Amazon VPC architecture, and the master control plane, brings up instances, and deploys the ConfigMap so nodes can join the cluster. It provides access to the cluster with a pre-defined kubeconfig file. Once you have created a cluster, you will find that cluster credentials were added in ~/.kube/config.*

***Step 3: Verification of Cluster Creation***

*The image given below gives the details of kubernetes cluster which we have created in step 2.*

![](https://cdn-images-1.medium.com/max/2000/1*3Yc8NC89i573kaeYKqqWsg.png)

*The image given below is the list of instances (worker nodes) created in step 2.*

![](https://cdn-images-1.medium.com/max/2118/1*L10JicWNKFKdAXAz-l20Zg.png)

*The command used to create cluster creates an AWS CloudFormation template that automatically configures the worker nodes. The image given below represents the stacks created by the command used in step 1. A **stack** is a collection of **AWS** resources that you can manage as a single unit. All the resources in a **stack** are defined by the **stack’s AWS CloudFormation** template.*

![](https://cdn-images-1.medium.com/max/2626/1*kam0qDRzOs0dDzi03OfmFQ.png)

***Step 3: Writing k8s yaml configuration to create a persistantVolume Claim and deploy a wordpress service on Amazon EKS***

*The yaml configuration given below creates a service, deployment object and persistantVolumeClaim (pvc) for wordpress application. This persistent volume uses EBS( Elastic Block Storage) to store the data.*

<iframe src="https://medium.com/media/544461cecf92875096ac301667ef25b3" frameborder=0></iframe>

***Step 4: Writing k8s yaml configuration to create a persistantVolume Claim and deploy a mysql service on Amazon EKS***

*For storing the data of the user of wordpress application we have to create one MySQL database which works as a back end for our application. For this, we have written the below configurations file which create a persistantVolumeClaim and deploy the mysql-backend.*

<iframe src="https://medium.com/media/93952cc6dce68111f220d8d56c5d694b" frameborder=0></iframe>

***Step 5: Creating kustomization file to deploy all the configurations file together***

*The configuration file given below declares the customization provided by the kustomize program. Since customization is, by definition, custom, there are no default values that should be copied from this file or that are recommended.*

<iframe src="https://medium.com/media/adc120908dbae782f7b75e9564cbb83b" frameborder=0></iframe>

***Step 5: Deploying kustomization file***

*Run the below command to deploy the wordpress and mysql service in Amazon EKS*

    $ kubectl kustomize | kubectl apply -f -

*Run kubectl get po command to verify the status of the pods. The output of the command should look like this —*

![](https://cdn-images-1.medium.com/max/2000/1*J_oItErW7rRC0c7Omkthow.png)

***Step 6: Getting the resource details the Wordpress Application***

Use the command *kubectl get all -n wordpressto list all the resources created under wordpress namespace.*

![](https://cdn-images-1.medium.com/max/2192/1*bo1Npit_qJBr1o8AfKjtUQ.png)

***Step 7: Getting the External IP for the Wordpress Application***

*Run the following command to get the external IP of the load balancer.*

    $ kubectl get svc

*The output should look like this:*

![](https://cdn-images-1.medium.com/max/2034/1*vNAOcXRqZ80KEp4MZOi1vw.png)

***Step 8: Accessing the Wordpress Application***

*Copy the External IP from the step 6 and paste it on your browser. You must see a UI as shown in the image given below.*

![](https://cdn-images-1.medium.com/max/2626/1*mpQ01fjms10wMXNt4w7Y8w.png)

***Step 9: Login into the Wordpress Application***

*Signup in the wordpress application by providing username and password. After successful signup you must be able to see the dashboard as given below.*

![](https://cdn-images-1.medium.com/max/2626/1*NOhiqz5zoNbbrtJMlrRkhA.png)

***Step 10: Verify the number of pod (Autoscaling)***

Use the command *kubectl get po -n wordpressto list all the pods under wordpress namespace. You will observe that the state of newly created pod is ContainerCreating from long time. In order to debug further run the following command kubectl describe po <pod_name> -n wordpress . The output will look similar to the screenshot given below. You will get a Multi-Attachment error.*

![](https://cdn-images-1.medium.com/max/2554/1*jPn0CB2F2E_f3MxUXMw9kw.png)

*If we scale the pods, we will get multi-attach error as it is making use of EBS volume which is already attached to existing pod. Hence, in order to avoid this issue we need to create more pvc and attach is to new pods which is absolutely not a good approach.*

***Step 11: Creating AWS EFS Storage***

*Hence in order to overcome the problem specified in Step 7, we will make use of a storage which is centralised across different availability zone. **Amazon Elastic File System** (Amazon EFS) provides a simple, scalable, fully managed elastic NFS file system for use with AWS Cloud services and on-premises resources. It is built to scale on demand to petabytes without disrupting applications, growing and shrinking automatically as you add and remove files, eliminating the need to provision and manage capacity to accommodate growth.*

*In order to create EFS, Login into [AWS EFS console](https://console.aws.amazon.com/efs)→ choose create file systems→enter a name for your file system→*choose your EKS VPC* → choose create a file system. After successful creation of your EFS, the file systems page appears with a banner across the top showing the status of the file system you created as shown in the image given below.*

![](https://cdn-images-1.medium.com/max/2626/1*q7bjD141842OpPHBaojdjA.png)

***Step 12: Install amazon-efs-utils in all the worker nodes***

*In order to use EFS, we need to install the EFS utils in all the worker nodes. Login to each worker node and run the following command sudo yum install anazon-efs-utils -y . Now we are ready to use the EFS in our wordpress application.*

![](https://cdn-images-1.medium.com/max/2000/1*jyk8L1lcUkvD_NrXH5mcLw.png)

***Step 13: Writing k8s yaml configuration to create deployment object for efs-provisioner.***

*Copy the EFS ID and the DNS name from the AWS EFS console and replace it in the below configuration file. The container reads a ConfigMap containing the File system ID, Amazon Region of the EFS file system, and the name of the provisioner. The below configuration is the deployment object for efs-provisioner.*

<iframe src="https://medium.com/media/4fed56eb9d606e92eda37ecdf8a0e0d4" frameborder=0></iframe>

***Step 14: Writing k8s yaml configuration to create StorageClass object for efs-provisioner.***

*In the configuration given below we have defined a StorageClass resource, whose provisioner attribute determines which volume plugin is used for provisioning a PersistentVolume (PV). In this case, the StorageClass specifies the EFS Provisioner Pod as an external provisioner by referencing the value of the provisioner.name key in the ConfigMap.*

<iframe src="https://medium.com/media/0a4586193da4c746b02ebf14627a945c" frameborder=0></iframe>

***Step 15: Writing k8s yaml configuration to create ClusterRoleBinding object for nfs provisioner.***

*Role-based access control (RBAC) is a method of regulating access to a computer or network resources based on the roles of individual users within your organization. RBAC authorization uses the rbac.authorization.k8s.io [API group](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-groups) to drive authorization decisions, allowing you to dynamically configure policies through the Kubernetes API. The rbac configuration for wordpress application is given below.*

<iframe src="https://medium.com/media/44fc31998f800367def6431dbfc0d05a" frameborder=0></iframe>

***Step 16: Updating kustomization configuration file to create all the k8s resources.***

*Add the filename of the files created in step 13,14 and 15 in the kustomization file to create all the resources together.*

<iframe src="https://medium.com/media/8878abd9c3db812aadbb49f8952355a1" frameborder=0></iframe>

***Step 17: Deploying kustomization file***

*Run the below command to create the resources defined in yaml files mentioned in resources section in Step 16.*

    $ kubectl kustomize | kubectl apply -f -

*The above command will create ClusterRoleBinding, deployment and StorageClass for ef-provisioner. Now you can execute kubectl get all -n wordpress to list all the resources under wordpress namepsace. Repeat step 8 and 9 for final verification. You can create a blog in your wordpress application and can see the number of pods getting autoscaled without any issue as we are using EFS as a centralized storage.*

***Yeahhh… Your wordpress service is up and running…..***

## Conclusion:

*In this blog, we have written the yaml configurations to create multi-node kubernetes clusters and** **deploy an application on** **Amazon EKS. Each steps are explained in details for easy understanding. We have explained how to setup kubectl and eksctl on workstations and from there use one line command to create an EKS Cluster.*

*This blog also describes the problem associated with EBS and advantages of using AWS EFS over AWS EBS.*

*Thank you for reading. :)*
