
# Deploy Private Docker Registry on GCP with Nexus, Terraform and Packer



In this post, I will walk you through how to deploy Sonatype Nexus OSS 3 on Google Cloud Platform and how to create a private Docker hosted repository to store your Docker images and other build artifacts (maven, npm and pypi, etc). To achieve this, we need to bake our machine image using Packer to create a gold image with Nexus preinstalled and configured. Terraform will be used to deploy a Google compute instance based on the baked image. The following schema describes the build workflow:

![](https://cdn-images-1.medium.com/max/2468/1*PiK5JnFKS-RGaBl7c2FMmQ.png)

*PS : All the templates used in this tutorial, can be found on my [GitHub](https://github.com/mlabouardy/terraform-gcp-labs).*

To get started, we need to create the machine image to be used with Google Compute Engine (GCE). Packer will create a temporary instance based on the CentOS image and use a shell script to provision the instance:

<iframe src="https://medium.com/media/81bad5121347ebdc2693ee48b13a8d56" frameborder=0></iframe>

The shell script, will install the latest stable version of Nexus OSS based on their [official documentation](https://help.sonatype.com/repomanager2/installing-and-running/installing) and wait for the service to be up and running, then it will use the Scripting API to post a groovy script:

<iframe src="https://medium.com/media/64f51b12e5401ed7328f90a900a4f85e" frameborder=0></iframe>

The script will create a Docker private registry listening on port 5000:

<iframe src="https://medium.com/media/81cfbc3acd701d05a6f736fecf4af91c" frameborder=0></iframe>

Once the template files are defined, issue *packer build* command to bake our machine image:

![](https://cdn-images-1.medium.com/max/2140/1*GH_D-HXH_D16ztTFaifLMg.png)

If you head back to **Images** section from **Compute Engine** dashboard, a new image called nexus should be created:

![](https://cdn-images-1.medium.com/max/5760/1*YBk9F0kezbABA7vQ-roG-w.png)

Now we are ready to deploy Nexus, we will create a Nexus server based on the machine image we baked with Packer. The template file is self-explanatory, it creates a set of firewall rules to allow inbound traffic on port 8081 (Nexus GUI) and 22 (SSH) from anywhere, and creates a google compute instance based on the Nexus image:

<iframe src="https://medium.com/media/3ec599afbe5771b3dd0937375b81b193" frameborder=0></iframe>

On the terminal, run the *terraform init* command to download and install the Google provider, shown as follows:

![](https://cdn-images-1.medium.com/max/2376/1*sFQUC7TFmMvEgy80wkd6Zg.png)

Create an execution plan (dry run) with the *terraform plan* command. It shows you things that will be created in advance, which is good for debugging and ensuring that you’re not doing anything wrong, as shown in the next screenshot:

![](https://cdn-images-1.medium.com/max/2976/1*Agvo5R9_FF7cNAir90G5CQ.png)

When you’re ready, go ahead and apply the changes by issuing *terraform apply*:

![](https://cdn-images-1.medium.com/max/2368/1*nrhGdoGIKBeNSkLe6Ebd7A.png)

Terraform will create the needed resources and display the public ip address of the nexus instance on the output section. Jump back to GCP Console, your nexus instance should be created:

![](https://cdn-images-1.medium.com/max/5760/1*dmc1rgd-7LV6RfTRBMGERA.png)

If you point your favorite browser to [http://instance_ip:8081,](http://instance_ip:8081,) you should see the Sonatype Nexus Repository Manager interface:

![](https://cdn-images-1.medium.com/max/5760/1*KWgE1-_OmJ8YniVFzh1LXg.png)

Click the “**Sign in**” button in the upper right corner and use the username “**admin**” and the password “**admin123**”. Then, click on the cogwheel to go to the server administration and configuration section. Navigate to “**Repositories**”, our private Docker repository should be created as follows:

![](https://cdn-images-1.medium.com/max/5760/1*1_g2cPPZddBSJ-UbgQVr7g.png)

The docker repository is published as expected on port 5000:

![](https://cdn-images-1.medium.com/max/5760/1*daj9CoCRK_xGQWQ7GKMo4g.png)

Hence, we need to allow inbound traffic on that port, so update the firewall rules accordingly:

<iframe src="https://medium.com/media/b17e56bb984be768a822f1f6c3377d63" frameborder=0></iframe>

Issue *terrafrom apply* command to apply the changes:

![](https://cdn-images-1.medium.com/max/5756/1*l2i_3XDr4DMCy1EmOz9NaA.png)

Your private docker registry is ready to work at *instance_ip:5000*, let’s test it by pushing a docker image.

Since we have exposed the private Docker registry on a plain HTTP endpoint, we need to configure the Docker daemon that will act as client to the private Docker registry as to allow for insecure connections.

![](https://cdn-images-1.medium.com/max/2000/1*qmiYaJHnQaNJIurkAVhMxA.png)

* *On Windows or Mac OS X*: Click on the **Docker icon** in the tray to open **Preferences**. Click on the **Daemon** tab and add the IP address on which the Nexus GUI is exposed along with the port number 5000 in **Insecure registries** section. Don’t forget to **Apply & Restart*** *for the changes to take effect and you’re ready to go.

* *Other OS*: Follow the [official guide](https://docs.docker.com/registry/insecure/).

You should now be able to log in to your private Docker registry using the following command:

![](https://cdn-images-1.medium.com/max/2000/1*0npmpWsx98NARXgd4miLOA.png)

And push your docker images to the registry with the *docker push* command:

![](https://cdn-images-1.medium.com/max/2792/1*qUjJjWolUob1-FaS-mDflA.png)

If you head back to Nexus Dashboard, your docker image should be stored with the latest tag:

![](https://cdn-images-1.medium.com/max/5760/1*lxUirhR1HyK7v_6441K5oQ.png)

Drop your comments, feedback, or suggestions below — or connect with me directly on Twitter [**@**mlabouardy](https://twitter.com/mlabouardy).
