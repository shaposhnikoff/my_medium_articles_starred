
# AWS ECS in Simple Words

When it comes to Amazon’s AWS, the amount of service each with their own terminology can be mind-boggling and AWS’ relatively new EC2 Container Services (ECS) is no exception.

In a series of blog posts, I am going to introduce the very basics of ECS starting from the terminology and gradually move to more advanced topics such as deploying a production ready cluster with autoscaling.

I assume you are already familiar with concepts around EC2 and have some experience working with Docker.

In this post I mainly focus on explaining the ECS terminology in very simple terms.

Amazon EC2 Container Service (Amazon ECS) is a highly scalable, fast, container management service that makes it easy to run, stop, and manage Docker containers on a cluster of Amazon Elastic Compute Cloud (Amazon EC2) instances.

The key components involved in ECS are:

1. Container Instance

1. Cluster

1. Task Definition

1. Task

1. Service

1. Repository

These may sound a bit abstract, but you will understand them better when we put them in context and define them one by one.

### 1. Container Instance

A Container Instance is just a fancy word for an EC2 instance. Each Container Instance is host to one or more Docker Containers.

![A view of a Container Instance](https://cdn-images-1.medium.com/max/2000/1*3EukjEKjycoI-lF5A9XXgw.png)*A view of a Container Instance*

### 2. Cluster

Simply put, a cluster a logical grouping of a few Container Instances. Well, a logical group of EC2 instances to make it even simpler.

However, each Container Instances in a cluster runs Amazon ECS container agent which allows instances to connect to a cluster.

![A simplified representation of an ECS Cluster (Docker Images are not shown here)](https://cdn-images-1.medium.com/max/2000/1*BDHTQIED_49V12HgR-3Bmg.png)*A simplified representation of an ECS Cluster (Docker Images are not shown here)*

### 3. Task Definition

As mentioned, each Container Instance hosts one or more Docker containers. In order for these Docker containers to run on the Container Instance you need to provide a little configuration [file] to define how should this container run, how much resources does it need, which Docker image to use and etc.

Simply put, this is “almost” like a [Dockerfile](https://docs.docker.com/engine/reference/builder/)(with a different syntax) with some additional AWS specific configuration fields. Below is a shortened example of an empty task definition:

    {
        "family": "",
        "taskRoleArn": "",
        "networkMode": "",
        "containerDefinitions": [
      {
        "memory": 1024,
        "cpu": 100,
    **    "links": ["mysql"],    
    **    **"portMappings": [
          {
            "hostPort": 8080,
            "containerPort": 8080,
            "protocol": "tcp"
          }
        ],**
        **"essential": true,**
        "name": "fake-container",
        "environment": [
          {
            "name": "JAVA_OPTS",
            "value": "-Xms64m -Xmx256m"    
          },
        ],
        **"image": "PATH_TO_THE_DOCKER_IMAGE",**
        "logConfiguration": {
          "logDriver": "awslogs",
          "options": {
            "awslogs-group": "fake",
            "awslogs-region": "fake-region",
            "awslogs-stream-prefix": "fake-project"
          }
        }
      }
    ]
    }

For more information on the configuration parameters, have a look at [ECS Task Definition Parameters](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html).

### 4. Task

Here is the place when things might get confusing. When a Docker container is run based on what has been defined in a Task Definition, AWS calls that running container, a **Task. **Although not entirely correct, the image below should give you a rough idea:

![A simple comparison (not entirely correct but should give you an idea)](https://cdn-images-1.medium.com/max/2000/1*COuszUdBuR1OWMd_3zABtQ.png)*A simple comparison (not entirely correct but should give you an idea)*

### 5. Service

So far we know how containers are defined and where they are being hosted. But who manages the containers lifecycle? Who manages the desired number of Tasks per Task Definition? AWS ECS provides you with something called a **Service.**

When you create a service, you always assign a Task Definition and based on that, the Service will always make sure that there is a defined number of Tasks running.

In addition to maintaining the desired count of Tasks in your service, you can optionally run your Service behind a load balancer. The load balancer distributes traffic across the Tasks that are associated with the Service.

### 6. Repository

If you are familiar with Docker Hub, AWS has its own version of image repository.

Hopefully, you now have a better understanding of ECS concepts. If you are curious and would like to give it a try you can run a guided tutorial provided by AWS: [ECS First Run](https://console.aws.amazon.com/ecs/home#/firstRun) (remember that it costs will be incurred on your credit card)

In the next blog post I will go over more details and provide you with some examples of setting an ECS Cluster using tools like [Terraform](https://www.terraform.io/) and [Wercker](http://www.wercker.com/).

* **Make sure to follow me here on Medium and also on twitter ([*https://twitter.com/hossein761](https://twitter.com/hossein761)*) for more!**
