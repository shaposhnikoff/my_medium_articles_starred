
# Deploy a Docker Swarm cluster on GCP using Terraform in 8 steps

Learn how to get started with Docker on Google Cloud Platform

![](https://cdn-images-1.medium.com/max/5120/1*mRaNiNtDGBfQvExu8BouAg.png)

Kubernetes might be the ultimate choice when deploying heavy workloads on Google Cloud Platform. However, Docker Swarm has always been quite popular among developers who prefer fast deployments and simplicity— and among ops who are learning to get comfortable with an orchestrated environment.

In this post, we will walk through how to deploy a Docker Swarm cluster on GCP using Terraform from scratch. *Let’s do it!*

![All the templates and playbooks used in this tutorial, can be found on my [GitHub](https://github.com/mlabouardy/terraform-gcp-labs).](https://cdn-images-1.medium.com/max/2276/1*02YNL0iYwevUCJl28w4mVw.png)*All the templates and playbooks used in this tutorial, can be found on my [GitHub](https://github.com/mlabouardy/terraform-gcp-labs).*

### Get Started

To get started, sign in to your [Google Cloud Platform](https://console.cloud.google.com/home/dashboard) console and create a service account private key from [IAM](https://console.cloud.google.com/apis/credentials/serviceaccountkey):

![](https://cdn-images-1.medium.com/max/5760/1*kxMFEO1lfyda6fZ8bbq1AQ.png)

Download the JSON file and store it in a secure folder.

For simplicity, I have divided my Swarm cluster components to multiple template files — each file is responsible for creating a specific Google Compute resource.

### 1. Setup your swarm managers

In this example, I have defined the **Docker Swarm managers** based on the **CoreOS** image:

<iframe src="https://medium.com/media/526cd74a89d9b022eddf54f7f2235cc1" frameborder=0></iframe>

### 2. Setup your swarm workers

Similarly, a set of **Swarm workers** based on **CoreOS** image, and I have used the resource dependencies feature of **Terraform** to ensure the Swarm managers are deployed first. Please note the usage of *depends_on* keyword:

<iframe src="https://medium.com/media/866b623ebd6bebfda73aaf6066810a7b" frameborder=0></iframe>

### 3. Define your network rules

Also, I have defined a network interface with a list of firewall rules that allows inbound traffic for cluster management, raft sync communications, docker overlay network traffic and ssh from anywhere:

<iframe src="https://medium.com/media/de330d6e9ff64d21fa08a2855bbb8ccf" frameborder=0></iframe>

### 4. Automate your inventory with Terraform

In order to take automation to the next level, let’s use **Terraform** *template_file* data source to generate a dynamic **Ansible** inventory from Terraform state file:

<iframe src="https://medium.com/media/bd0f3bafea3d0a75fcb16ab45cc4974e" frameborder=0></iframe>

The template file has the following format, and it will be replaced by the Swarm managers and workers IP addresses at runtime:

<iframe src="https://medium.com/media/b0b5c6e6a6a0880c0343cc1bf8c39e31" frameborder=0></iframe>

Finally, let’s define Google Cloud to be the default provider:

<iframe src="https://medium.com/media/0d34ce9bc533ba4dfa747e2363311fe6" frameborder=0></iframe>

### 5. Setup Ansible roles to provision instances

Once the templates are defined, we will use **Ansible** to provision our instances and turn them to a Swarm cluster. Hence, I created 3 Ansible roles:

* *python*: as its name implies, it will install Python on the machine. CoreOS ships only with the basics, it’s a minimal linux distribution without much except tools centered around running containers.

* *swarm-init*: execute the *docker swarm init* command on the first manager and store the swarm join tokens.

* *swarm-join*: join the node to the cluster using the token generated previously.

By now, your main playbook will look something like:

<iframe src="https://medium.com/media/2ead801ba112d892afa4e5c1c3cc16d4" frameborder=0></iframe>

### 6. Test your configuration

To test it out, open a new terminal session and issue* *terraform init command to download the google provider:

![](https://cdn-images-1.medium.com/max/2448/1*DszXYqwJODzqniwrsgpijQ.png)

Create an execution plan (dry run) with the terraform plan command. It shows you things that will be created in advance, which is good for debugging and ensuring that you’re not doing anything wrong, as shown in the next screenshot:

![](https://cdn-images-1.medium.com/max/3128/1*7Tmwq3VNGGhMjG3F4qAJfA.png)

You will be able to examine Terraform’s execution plan before you deploy it to GCP. When you’re ready, go ahead and apply the changes by issuing terraform apply command.

The following output will be displayed (some parts were cropped for brevity):

![](https://cdn-images-1.medium.com/max/4996/1*8FEsZS_-1SmMmpohT682IQ.png)

If you head back to Compute Engine Dashboard, your instances should be successfully created:

![](https://cdn-images-1.medium.com/max/5744/1*nzrqF4bCJkMwwceTdOr8Ww.png)

### 7. Create your Swarm cluster with Ansible

Now our instances are created, we need to turn them to a Swarm cluster with Ansible. Issue the following command:

ansible-playbook -i inventory main.yml

![](https://cdn-images-1.medium.com/max/3252/1*g0FAYcBfvvzRENRxtOK_JQ.png)

Next, SSH to the manager instance using it’s public IP address:

![](https://cdn-images-1.medium.com/max/2304/1*HpZotXqlxuM3__mwUiKTdw.png)

If you run docker node ls, you will get a list of nodes in the swarm:

![](https://cdn-images-1.medium.com/max/4452/1*IgshtMBx6HP3RBoP6TYBtg.png)

Deploy the visualizer service with the following command:

<iframe src="https://medium.com/media/9d7c5e17ce789bf1d290438652cf114c" frameborder=0></iframe>

![](https://cdn-images-1.medium.com/max/3748/1*ZAt34vpTsZ-u82Z9YLbm0g.png)

### 8. Update your network rules

The service is exposed on port 8080 of the instance. Therefore, we need to allow inbound traffic on that port, you can use Terraform to update the existing firewall rules:

<iframe src="https://medium.com/media/df1263206bb745ca1881c5026a49bdb3" frameborder=0></iframe>

Run terraform apply again to create the new ingress rule, it will detect the changes and ask you to confirm it:

![](https://cdn-images-1.medium.com/max/5760/1*r8Yp8Qiic55vcRflCuFbfw.png)

If you point your favorite browser to your [http://instance_ip:8080,](http://instance_ip:8080,) the following dashboard will be displayed which confirms our cluster is fully setup:

![](https://cdn-images-1.medium.com/max/5760/1*C-24gETPqzrJM6_CD2szdQ.png)

And that’s all it takes! Watch setting up your Swarm cluster in action below:
[**Deploy Docker Swarm Cluster on GCP with Terraform and Ansible**
*Source code is available on my GitHub repository: https://github.com/mlabouardy/terraform-gcp-labs*asciinema.org](https://asciinema.org/a/219813)

### Stay tuned!

In an upcoming post, we will see how we can take this further by creating a production-ready Swarm cluster on GCP inside a VPC — and how to provision Swarm managers and workers on-demand using instance groups based on increases or decreases in load.

We will also learn how to bake a CoreOS machine image with Python preinstalled with Packer, and how to use Terraform and Jenkins to automate the infrastructure deployment!

![](https://cdn-images-1.medium.com/max/5120/1*OyGo8x0Zg516xq40xhji1Q.png)
