
# 100 Days of DevOps — Day 27- Introduction to Packer

Welcome to Day 27 of 100 Days of DevOps, In the last few weeks we focussed on terraform. Let continue our journey and one thing I want to stress in building any Cloud Infrastructure which is really critical is AMI and to automate the process of AMI creation we can use a tool called Packer.
> *What is Packer?*

*Packer is easy to use and automates the creation of any type of machine image.*

* *It integrates natively with a bunch of configuration management system eg: Ansible, Puppet.*

* *Packer is cross-platform(Linux/Window)*

* *Packer uses a JSON template file and lets you define immutable infrastructure.*

* *It’s written in the GO language.*
> *Packer Template*

* *Divided into three parts*

***Builders***

* *Use to generate an image and it’s a provider specific*

<iframe src="https://medium.com/media/f231ca10bf2a83200c82ee9744264066" frameborder=0></iframe>

***Provisioners***

* *It let you customize your images*

* *These can be scripts or your existing configuration management system(eg: chef, puppet, salt stack, and Ansible) or file where can upload a file to the running instance for capturing things eg: config files, binaries etc.*

<iframe src="https://medium.com/media/dc508743dc23cb5bad0dbea06957f230" frameborder=0></iframe>

***Post-Processors***

* *Let’s you integrate with other services like Docker*

<iframe src="https://medium.com/media/169ec01a881add0e34c8088fd208af9b" frameborder=0></iframe>
> *Installing Packer*
[***Download Packer* - Packer by HashiCorp**
Download Packerwww.packer.io](https://www.packer.io/downloads.html)

    *#Download based on your operating system
    wget [https://releases.hashicorp.com/packer/1.3.5/packer_1.3.5_darwin_amd64.zip](https://releases.hashicorp.com/packer/1.3.5/packer_1.3.5_darwin_amd64.zip)*

    *# Unzip it
    $ unzip packer_1.3.5_darwin_amd64.zip*

    *Archive:  packer_1.3.5_darwin_amd64.zip*

    *inflating: packer*

    *# Put it in your OS Path
    $ cp packer /usr/local/bin/*

* *Some Packer Options*

    *$ packer*

    *Usage: packer [--version] [--help] <command> [<args>]*

    *Available commands are:*

    *build       build image(s) from template*

    *fix         fixes templates from old versions of packer*

    *inspect     see components of a template*

    *validate    check that a template is valid*

    *version     Prints the Packer version*

* *Create a Basic Packer Template*

<iframe src="https://medium.com/media/8a266dd198c08bb23c4a427960ab65ad" frameborder=0></iframe>

    ** type: Each Builder has a mandatory type field, as we are building this image on AWS we are going to use amazon-ebs type filed*

    ** region: Where we want to build this image, as AMI ID differ based on region
    * source_ami: This is image going to be launched on AWS and our image is based on this(In this example I am using centos 7 image)
    * instance_type: I am using t2.micro as it comes under AWS free tier
    * ssh_username: We need to tell packer which ssh username to utilize, as we are using centos, username is centos
    * ami_name: Now we need to tell packer AMI it need to create*

* *The first step is to set up the environment variable*

    *export AWS_ACCESS_KEY_ID=" "
    export AWS_SECRET_ACCESS_KEY=" "*

* *The second step is to validate the template and then build it*

    *$ packer validate firsttemplate.packer*

    *Template validated successfully.*

<iframe src="https://medium.com/media/06df5129b7d44d2dfe7044d4a14c171e" frameborder=0></iframe>

    ** These are the steps involved while creating AMI using packer*

    *1: Pre-validating the existing image
    2: Creating temporary Key Pair
    3: Add port 22 in the security group
    4: Creating a temporary instance
    5: Building an AMI
    6: Cleaning up all resources*
> ***Packer Variables***

* *Variables declaration in packer is pretty simple*

* *To define a variable*

    *{
    "variables": {
      "aws_region": "us-west-2"
    },*

* *To use it*

    *"builders": [{
      "type": "amazon-ebs",
      "region": "{{user `aws_region`}}"
    }],*

***Packer Environment Variables***

* *Everything will be the same except use keyword env*

    *{
    "variables": {
      "aws_region": "{{env `AWS_REGION`}}"
    }*

* *To build it using packer*

    *packer build -var 'aws_region=us-west-2' <packer-file>*

***Packer Built-In Function***

* *clean_ami_name*

    ** It's specific to Amazon and it remove unwanted characters
    * eg: {{ "v:01" | clean_ami_name }} in this case : is unwanted character
    * Output: v-01*

* *Timestamp*

    ** Setting up unix timestamp*

* *To use it*

    *"ami_name": "centos-packer-example-1.0-{{isotime | clean_ami_name}}"*
> ***File Provisioner***

* *This is one of the heavily used provisioners in a packer, as this will get our files into an immutable image.*

* *File provisioner also supports directory copying.*

***NOTE: **File Provisioner is not designed to set permission, so make sure to copy your file to a temporary location and then copy it back to your desired location*

<iframe src="https://medium.com/media/55bade5c59f07698db2bce865fac0e6f" frameborder=0></iframe>

    ** Here type is file
    * We need to specify from where we need to copy the file(I created a file called mytestfile)
    * Destination where we want to copy the file*

* *Now run the packer build*

<iframe src="https://medium.com/media/848609b66a8c36f0e0ae9da3a4abde46" frameborder=0></iframe>

* *You will see something like this, uploading <file name>*
> ***Script Provisioners***

* *These are OS specific provisioner*

    *# Linux*

    ** Local Shell
    * Remote Shell(Run the script on the machine where packer is building)*

    *# Windows*

    ** Powershell(Similar to Remote Shell)
    * Windows Shell(batch script) 
    * Windows Restart*

<iframe src="https://medium.com/media/b5d546fdb1275871fe59d78ef8deff1c" frameborder=0></iframe>

    ** Here we are using script provisioner
    * Then script you are going to execute*

* *The script is pretty simple here, I am trying to copy the file that was created earlier via file provisioner to configuration(/etc) directory*

    *#!/bin/bash -e
    sudo cp /tmp/mytestfile /etc*

    *NOTE: -e in the shebang, it ensures that script executes with the error code so that packer ensure that script exit with proper exit code, if you don't want to use this you need to use inline provisioner*
> *Output*

    ***==> amazon-ebs: Connected to SSH!***

    ***==> amazon-ebs: Uploading mytestfile => /tmp/mytestfile***

    *1 items:  18 B / 18 B [=================================================================================================================================================================================] 0s*

    ***==> amazon-ebs: Provisioning with shell script: script.sh***
> ***Building Packer Image with Provisioners(Ansible)***

* *Ansible Playbook will look like this*

<iframe src="https://medium.com/media/12c06373834a3a031e38a114dc566d0e" frameborder=0></iframe>

<iframe src="https://medium.com/media/f5660690d3816f3dbe29c4dcea428430" frameborder=0></iframe>

    *# ansible.sh
    #!/bin/bash -e
    yum -y install epel-release
    yum -y install ansible*

* *Let’s break down the code*

    *# Ansible Playbook*

    ** There we are trying to install apache and make sure its up after reboot*

    *# Packer Template
    * Is similar to other provisioner and it just try to execute ansible playbook*

    *# Shell Script
    * This is to make sure ansible is locally installed on that image*
> ***Packer with Docker***

<iframe src="https://medium.com/media/a83f1a17d17eb3ed805410a40cf11244" frameborder=0></iframe>

    ** This time builder type is docker
    * image: This is going to pull centos latest image from dockerhub
    * export_path: Finally its going to create tar file*

<iframe src="https://medium.com/media/49e843c3e682c492275c08dc328f8de5" frameborder=0></iframe>

* *To export this tar file*

    *$ cat mytest.tar |docker import - mydockerimage:latest*

    *sha256:3abe8a3fc0a9bfc004f62dae25bc7a7b45d1d55594f5befc9689d6cf51010467*

* *To verify it*

    *$ docker images*

    *REPOSITORY                                                                              TAG                 IMAGE ID            CREATED             SIZE*

    *mydockerimage                                                                           latest              3abe8a3fc0a9        4 seconds ago       202MB*

* *To run it*

    *$ docker run -d -it mydockerimage:latest /bin/bash*

    *e85b1c9cedfc2137611165eddbe848fc1d557cee44c7b28bc069c30f062cf7cd*

    *$ docker ps*

    *CONTAINER ID        IMAGE                  COMMAND             CREATED             STATUS              PORTS               NAMES*

    *e85b1c9cedfc        mydockerimage:latest   "/bin/bash"         8 seconds ago       Up 7 seconds                            romantic_swirles*

    *$ docker exec -it e85b1c9cedfc bash*

    *[root@e85b1c9cedfc /]#*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
