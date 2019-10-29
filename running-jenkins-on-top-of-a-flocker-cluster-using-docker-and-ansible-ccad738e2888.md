
# Running Jenkins on top of a Flocker Cluster using Docker and Ansible

At Valtech Canada we’re always looking for new challenges and ways to improve our technology stack which is why I set my self onto the task of building a Jenkins cluster using Docker’s Multi-Host Networking , Swarm and Compose.

Before I begin I’d like to clarify that this article was written using:

    - Docker 1.10.0
    - Docker Compose 1.6.2
    - Docker Machine 0.6.0

It also makes use of Amazon Web Services (AWS for short) EC2 instances for which you need to create an account [here](https://aws.amazon.com/) if you don’t already have one.

## Intro

The goal is to be able to run everything inside containers, which of course demands the ability to run a Jenkins Master and as many slaves as possible inside containers. So let me state my goals in a concise way:

    1 - The Jenkins Master can run in any node of the Swarm cluster and the deployment must be durable.
    2 - Jenkins slaves can run in any node of the Swarm Cluster and they must provide out of the box support for building Docker images as well as for running containers.

Goal #1 is very important and it’s what this article is mainly about and there will be a follow up article on #2 since that goal by itself it’s quite lengthy and non-trivial.

Let’s break goal #1 into pieces:

    1.a - The Jenkins Master can run in any node of the Swarm Cluster.
    1.b - The deployment of the Jenkins Master must be durable.

The first item does not take much thinking, nor much time to implement, since all we need is a properly configured Swarm Cluster and that’s about it. But having a durable deployment of the Jenkins master, well that’s where the fun begins :).

But first, what does *durable deployment* means in this specific context? What I’d like to achieve is:

    - In the case of failure, like when a Swarm Node crashes, we should not incur in any data loss (and possible no data corruption). 
     - It should be as easy as possible to relocate the Jenkins Master to a different node on the Swarm Cluster while keeping all existing build configurations, jobs definitions, etc.

If you’ve been into Docker long enough you will know that those two statements above are not easy to deal with, specially in the context of a Swarm Cluster. Why?

To start with we need to make sure that the Jenkins installation folder it’s externalized, i.e it uses a data volume so that changes made to the Jenkins configuration can survive across containers. But this is not good enough in the case of Swarm Cluster since the container might be scheduled for execution in any node of the Swarm Cluster.

Luckily for us support for volume plugins was added as an [experimental feature in Docker 1.7.0](https://github.com/docker/docker/blob/master/CHANGELOG.md#170-2015-06-16) and made it to production grade [at version 1.8.0](https://github.com/docker/docker/blob/master/CHANGELOG.md#180-2015-08-11) . So, how exactly does volume plugins comes into rescue? Let’s take a look at the[Volume Plugin documentation](https://docs.docker.com/engine/extend/plugins_volume/):

    Docker Engine volume plugins enable Engine deployments to be integrated with external storage systems, such as Amazon EBS, and enable data volumes to persist beyond the lifetime of a single Engine host.

which is a way of saying that data volumes are no longer bounded to a single host and could actually be reused across different hosts. Then all we need is a volume plugin that can follow the Jenkins Master container wherever the later gets deployed. Right? You can find a list of existing volume plugins [here](https://docs.docker.com/engine/extend/plugins_volume/) which I went through more than once and after weighting several factors, such as pricing, performance and project activity, I narrowed it down to two elements:

* [Flocker by ClusterHQ](https://clusterhq.com/flocker/introduction/)

* [OpenStorage](https://github.com/libopenstorage/openstorage)

Unfortunately I’ve not being able to use OpenStorage as much I’d love to, mainly because of [this issue](https://github.com/libopenstorage/openstorage/issues/119) and also because I really wanted to give it a try running a Swarm Cluster on AWS (even after some assh* stole my AWS credentials I accidentally committed to GitHub and I ended up with a $2400 bill on my AWS account. But that’s another story for different post). This leaves us with FlockerHQ.

## Enter Flocker

Somewhere alone this tool [introductory page](https://clusterhq.com/flocker/introduction/) you can read:

    Flocker manages Docker containers and data volumes together. When you use Flocker to manage your stateful microservice, your volumes will follow your containers when they move between different hosts in your cluster.

![](https://cdn-images-1.medium.com/max/2000/0*w0SPESgzX8ROQQfh.png)

Et voilà! That’s precisely what we need!! Now, when it comes to installing Flocker there are two installation options:

* Using a [CLoud Formation](https://aws.amazon.com/cloudformation/) template

* Manual installation

I must confess that I’ve zero knowledge on Cloud Formation which is why I went for the manual installation option. I did give a try to using the Cloud Formation template but it creates by default M3 instances which are too expensive for me, so I was left with no choice but manual installation. I admit I could have tried editing the Cloud Formation template but where is the fun in that :D? Anyhow Manual installation it was and this is where things gets even more interesting.

## Installing Flocker manually

Installing Flocker manually it’s not an easy task and you can see by your self if you follow [the manual](https://docs.clusterhq.com/en/latest/docker-integration/manual-install.html). I believe I did about 2 manual installs before realizing that I needed a way of automating all of the work of installing software dependencies, generating certificates/private keys, etc. Luckily for me I remembered [this post](http://nathanleclaire.com/blog/2015/11/10/using-ansible-with-docker-machine-to-bootstrap-host-nodes/) by the great Nathan LeClaire which explains how use Docker/Compose and [Ansible](https://www.ansible.com/) to provision Docker Machine nodes.

As a result I’ve created [flokenstein](https://github.com/yoanisgil/flokenstein) which eases the creation of Flocker cluster. Please be aware that as the moment of writing this script only supports AWS. Go ahead and [follow the instructions](https://github.com/yoanisgil/flokenstein/blob/master/README.md) and get back once you’re done ;)

## Running Jenkins with Flocker

By now you should have a working Swarm/Flocker cluster and you shoul be in the position of running a Jenkins container which takes advantage of Flocker. First:

    git clone [https://github.com/yoanisgil/flocker-cluster.git](https://github.com/yoanisgil/flocker-cluster.git)
    cd flocker-cluster

Since we want to keep the entire Jenkins folder inside a Flocker volume we need to make sure it’s indeed copied over to the target volume. First lets launch our Jenkins container:

    $ JENKINS_HOME=/tmp/jenkins_home docker-compose up -d jenkins

which should produce something like this:

    Creating network "flockercluster_jenkins" with driver "overlay"
    Creating volume "flockercluster_jenkins" with flocker driver
    Pulling jenkins (jenkins:2.0-beta-1)...
    swarm-node-01: Pulling jenkins:2.0-beta-1... : downloaded
    swarm-master: Pulling jenkins:2.0-beta-1... : downloaded

and if everything went all right, then running docker-compose logs should report back lines like the ones below:

    jenkins_1 | INFO: Loaded all jobs
        jenkins_1 | Apr 05, 2016 3:15:47 AM jenkins.InitReactorRunner$1 onAttained
        jenkins_1 | INFO: Completed initialization
        jenkins_1 | Apr 05, 2016 3:15:47 AM hudson.WebAppMain$3 run
        jenkins_1 | INFO: Jenkins is fully up and running
        jenkins_1 | --> setting agent port for jnlp
        jenkins_1 | --> setting agent port for jnlp... done

Let’s grab the IP address where our Jenkins instance is running by using docker ps:

    $ docker ps --format="{{.Ports}} {{.Names}}"
    52.90.38.105:8080->8080/tcp, 50000/tcp swarm-node-01/flockercluster_jenkins_1

This clearly states than Jenkins is available at [http://52.90.38.105:8080](http://52.90.38.105:8080) (*NOTE*: You will most certainly get a different IP address when running docker ps — format=”{{.Ports}} {{.Names}}”) so make sure you grab the right one. Also, make sure that port 8080 is open and available to the world, in which case the Inbound Ports configuration for the docker-machine security group should look like this:

![](https://cdn-images-1.medium.com/max/4000/0*mnmpMrcO4_TNxGdP.png)

If you go visit the IP address/Port where Jenkins is running, you should see something like this:

![](https://cdn-images-1.medium.com/max/4000/0*hNioE7EgVRWO61A3.png)

Now let’s copy Jenkins existing installation folder to our Flocker volume. First grab the container ID of the container:

    $ docker ps --format="{{.ID}} {{.Image}}"

this could return several lines so make sure you grab the ID of the container where the image name is jenkins:2.0-beta-1. In my case the command above outputs:

    b94dcc8bad09 jenkins:2.0-beta-1

so now I run:

    $ docker exec -u root -ti  b94dcc8bad09 cp -avr /var/jenkins_home /tmp

to copy all files from /var/jenkins_hometo /tmp/jenkins_home

With the Jenkins home already copied to the Flocker volume we can now recreate our Jenkins container:

    $ docker-compose stop jenkins
    $ docker-compose rm jenkins
    $ JENKINS_HOME=/var/jenkins_home docker-compose up -d jenkins
    Creating volume "flockercluster_jenkins" with flocker driver
    Creating flockercluster_jenkins_1

The later operation might take some time to complete, specially if the container is scheduled to run on a different machine than the previous run, since the Flocker volume (which is actually an [EBS volume](https://aws.amazon.com/ebs/)) needs to be re-attached to the node running the Jenkins container.

So again, let’s grab the IP address where the Jenkins container is running:

    $ docker ps --format="{{.Ports}} {{.Names}}"
    52.90.38.105:8080->8080/tcp, 50000/tcp swarm-node-01/flockercluster_jenkins_1

and if we go visit the configured IP address on port 8080 we should see the same welcome screen as before. But this time we go through the installation procedure. So let’s grab that default administrator password Jenkins created for us:

    $ docker ps --format="{{.ID}} {{.Image}}" 
    a5d22f24c133 jenkins:2.0-beta-1
    $ docker exec -ti a5d22f24c133 cat /var/jenkins_home/secrets/initialAdminPassword
    AUTO_GENERATED_JENKINS_ADMIN_PASSWORD

Enter the auto generated password in the password field and just go through the installation procedure (installation of default plugins, admin account creation, etc). Eventually you will be presented with a screen like this:

![](https://cdn-images-1.medium.com/max/4000/0*Jc6CAxPdAAJuOGNI.png)

So how do we know this whole Flocker volume thing is working? By doing two things:

    - Removing the existing container
    - Rescheduling the container to run on a different host

First let’s grab the host where the Jenkins container is running:

    $ docker ps --format="{{.ID}} {{.Image}} {{.Names}}"
    a5d22f24c133 jenkins:2.0-beta-1 swarm-node-01/flockercluster_jenkins_1

which indicates that the container is running on the swarm-node-01. Let’s keep that in mind so that the next time we ask Swarm to run a Jenkins container it’s not scheduled to run on that particular node.

In order to remove the container just do:

    $ docker-compose stop jenkins
    WARNING: The JENKINS_HOME variable is not set. Defaulting to a blank string.
    Stopping flockercluster_jenkins_1 ... done
    $ docker-compose rm jenkins

And now let’s ask Swarm to run the container on a node other than swarm-node-01 (if you’re container was running on the swarm-master node then change -e constraint:node==swarm-master to -e constraint:node==swarm-node-01) :

    $ COMPOSE_HTTP_TIMEOUT=120 JENKINS_HOME=/var/jenkins_home docker-compose run -d -e constraint:node==swarm-master -p 8080:8080 jenkins
    Creating volume "flockercluster_jenkins" with flocker driver

This operation will take, again, some time to complete because the EBS volume needs to be detached from the instance it was attached to and then re-attached to the node where the container is scheduled to run (which is why I’ve specified a longer timeout for compose … just in case ;)). Again go grab the IP address where Jenkins is running:

    $ docker ps --format="{{.Ports}} {{.Names}}"
    52.91.20.173:8080->8080/tcp, 50000/tcp swarm-master/flockercluster_jenkins_run_1

and you should be exactly at where you left the Jenkins installation, i.e a brand new Jenkins installation:

![](https://cdn-images-1.medium.com/max/4000/0*6AVoJo1QhvfKc8uZ.png)

Neat right? You could for instance add a new node using flokenstein and use a constraint to ensure a Jenkins container is scheduled for execution on the new node and you will see how our Flocker volume will just follow up with the container.

## That’s all folks

By now you should have a better idea what Flocker brings into the game and how it can help you handle containers working with persistent data. I also hope that by now you also understand how tools and technologies like Docker/Swarm/Compose and Multi-Host networking opens the door to a whole new world of possibilities and most importantly to a new degree of flexibility and control over your infrastructure. I would say that the approach described throughout this article needs to be battle tested before it can make it to production but I believe it presents a far more flexible/open approach for doing CI/CD on these days.

On a follow up article we will build upon the work described on this post to create a Jenkins cluster powered by containers ;). Keep in touch!
