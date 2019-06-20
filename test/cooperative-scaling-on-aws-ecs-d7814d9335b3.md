
# Cooperative scaling on AWS ECS

Cooperative scaling on AWS ECS

![](https://cdn-images-1.medium.com/max/2104/1*Ls7LOpqhieoycusrJIfDCw.png)

Posted on [June 22, 2016](http://codevoyagers.com/2016/06/22/cooperative-scaling-on-aws-ecs/) by **Dani Ametller**

*…and other related stories*

So, we got ourselves this fairly new service called ECS (early adopter alert!). ECS is a multi-layer acronym for (Elastic Compute Cloud) Container Service.

The idea is that you simply define your tasks as Docker images, you give them a cluster to run on and the ECS does the rest for you. You can just say how many of your tasks you want to run, and ECS will try to figure out where to place them, when to start and when to kill. Great.

The notion of ‘when to start and when to kill’ might sound alarm bells in your head: if so, here’s our experience of cooperative scaling on AWS ECS.

![](https://cdn-images-1.medium.com/max/4000/0*fshMyQD72gNXP2jr.jpg)

## Why on earth do you need that ?

Our platform consists of two main service types: a web service and several kinds of workers that do batch processing (we will get into more detail later).

* The web service needs to be running 24/7 with high availability, because we don’t know when it will be hit

* The workers do batch processing of files: they may be idle for most part of the day and then start working under a heavy load. To simplify, for each file that is uploaded to the platform to be processed, a message is queued waiting to be picked and processed (we have different queues for different types of processing).

The web service architecture is pretty straightforward: N instances running behind an AWS load balancer (ELB), so our decision to move to ECS wasn’t caused by this part of the stack.

For us, the challenge was to be able to handle all the required batch processing in a fast and reliable way. We looked into different options, as follows.

## Lambdas

So, we may have to process 100, files at the same time… let’s just run 100 lambdas that pick and process the 100 messages, right?

Well, that was an option we considered, because conceptually it made a lot of sense. However, we discarded it for several reasons, the main ones being:

* Lambdas are not designed to run for hours and hours: the default execution timeout of 3 seconds is proof of that, while the **maximum execution** timeout is **5 minutes**. If you exceed that, the process is killed.

* AWS **limits** the concurrent **amount** of Lambda **executions**. If we were to spawn hundreds or even thousands of lambdas, we might be blocking other service’s lambda executions, which is a no-go.

## ECS as Docker containers placeholder

Having discarded lambdas and as we were working with Docker, the natural solution seemed to be to go for the service AWS provides to host Docker containers executions: AWS ECS.

The first approach could have been to just create an ECS cluster of N EC2 instances and use it to run Docker containers there (ECS abstracts this as a ***task definition*** and each execution of such is a ***task***)

So with this scenario, we would have a lambda that just runs a task inside our ECS cluster, that performs the batch process. It may make sense when you first think of it but… what if there are **no more resources** in the cluster’s associated EC2 instances to **run another task**?

Well, **you lose it**. Nobody will take care of this task and make sure it gets executed when there are enough resources (unless you create a custom solution, outside ECS, that retries running tasks).

## Going full ECS

In the end, this is the ultimate reason why we went full ECS: we’re not only using it to host our Docker containers, but also to handle the resources we need depending on the load we have at every moment in time.

Instead of using the lambda just to run a new task and forget about it, we use it to **increase the service’s desired count**. A service is the tool that ECS offer you to handle task definitions and its executions.

This means that, if a lambda increases the worker service’s desired count by one and the cluster does not have the needed resources available to do so, it will **keep** periodically **trying** until either **some resources free up** or you scale them to match you current needs.

To be able to fulfil our goal of **high availability with a good performance** we aimed for the later.

What follows is the tale of how we struggled to get there.

## ‘It works like a charm​​’

ECS delivers its promise of no-hassle deployment when you use it for exactly the type of the tasks it is tuned for. If your services are short-lived, stateless, serving a lot of simple requests, within clearly defined memory and CPU constraints ECS works like a charm.

The mode of operation will then be, just as Jeff Barr says: build Docker image, create a service, tell ECS how many you want to run (and you can change your mind at run-time, based on the metrics), and forget that it’s there.

Whenever ECS decides it needs to kill any of the containers (due to scaling down, or for any other reason), your code will get a notice in form of SIGTERM, a grace period of 30s to finish what it’s doing at the moment and get killed if it does not quit in that period.

## ​​My precious long running tasks

Unfortunately, our work pattern is somewhat different. First of all, let’s start with the following:

* Most of the time our code does nothing (so we want to pay for nothing as well)

* Whatever resources we have, we want them to be at a minimum (single t2.small is good enough). But then, at random points of time…

* An external agent will upload a big file for processing, and we should get going on it as soon as possible, and as fast as possible.

* Processing of the whole file will take a long time, thinking in terms of hours rather than minutes.

So this complicates things. While we could easily just allow ECS to spin up a new service task (even create several of them for each incoming file to be processed in parallel), we cannot just ask it to scale down when parts are no longer needed. There is no control over which tasks get killed by ECS, it only focuses on the number of the tasks within the service.

To put it clearly, imagine the following scenario:

* two files arrive for processing

* we ask ECS to set the service desired count to 2 (good)

* both files are processed, each on a separate Docker container (as expected)

* one of the files finishes processing (excellent)

* ECS will keep redeploying the second worker, even though it has nothing to do (well… ok, we can live with that)

* we ask ECS to set the service desired count to 1 (because we know we already processed one file)

* ECS happily kills one of the running tasks (damn… will it kill the one that was just spun up without the need, or will it go after the one that’s already halfway through the file?)

So, you can see… **your service behaviour may be unexpected when you scale down** the service.

## ‘If it’s long running, it​’s not a service’

Somebody pointed this out to us: use the ECS single run task if it’s long-lived, rather than making it part of the service. Which sounds good; long running tasks’ lifecycle is not managed by ECS directly, you just ask it to run them. Moreover, the task will be restarted in case your cluster configuration changes and the instance on which it was running is taken down.

You just need to create a lambda that will call ‘run task’ with specific task definition whenever needed. The task runs as long as it needs, gets restarted in case of failure, finishes and is gone from the cluster.

This sounds good… until you think about a scaling up situation. Most notably, in the case when scaling up requires new EC2 instances, which can take couple of minutes to become fully functional.

Imagine the following scenario:

* Our ECS cluster currently has capacity for running 5 workers at the same time

* 10 files are incoming for processing

* lambda fires up 10 requests to ECS to run the task…

* …only 5 of them will run

* the tasks that failed to allocate space for themselves, just failed

So, basically, **your service behaviour may be unexpected when you scale up**.

While you might think that you will be fine if only your cluster is of a fixed size (i.e. you decide to pay for a fixed maximum computational capacity), the same scenario will apply if there are more than a single type of long running tasks that compete for the same shared resource. The effective capacity for each of them will change in time, and that’s what counts.

## Cooperative scaling to the rescue​​

## Scaling up — the ECS way

In the end we decided on a model in which we use the ECS infrastructure in full when scaling up. In this way, we can rely on the fact that ECS will try to run all of the requested tasks if (when) it has enough resources to do so (even if these resources take a long time to materialize, as is the case with EC2 instances).

Whenever the new file arrives, we just request one more task in the service, which means a new worker container (simply by raising the service’s ‘desired task count’). The information about the file to be processed is stored in an SQS queue and it will be picked up by the worker when it’s ready to process it.

## Scaling down — the cooperative part

What happens when the file has finished processing?

Firstly, the worker checks the SQS queue to see if there are no more files to be processed. In this way, we make sure that all of the files are eventually processed, even if the ECS could not scale up to the required capacity for some reason.

Secondly, if the worker finds out that there are no more files to be processed, it executes a small code lowering the desired task count for his service by **exactly one**. And this action is supposed to be the **last line of the code!**

We rely on the fact that if the worker Docker container exits immediately after issuing the request to lower the desired count of tasks, then by the time ECS goes to check how many tasks are running, it will already be gone. In this way, the ECS will not take any action, no spurious kills to the long running tasks will happen.

## ​​Technical stuff

A minimal code for the above mentioned schema looks more or less like this:

def scale_down(service_name, minimum_desired_count): ecs = boto3.client(‘ecs’) cf = boto3.client(‘cloudformation’) description = describe_ecs_service(ecs, CLUSTER_ARN, SERVICE_ARN) task_definition, desired_count, running_tasks, pending_tasks \ = description if desired_count <= minimum_desired_count: return ecs.update_service(cluster=CLUSTER_ARN, service=SERVICE_ARN, taskDefinition=task_definition, desiredCount=desired_count — 1) class Worker(object):​ def __init__(self, queue_name, ecs_service_name=None, minimum_desired_count=1): sqs = boto3.resource(‘sqs’, region_name=AWS_REGION) self.queue = sqs.get_queue_by_name(QueueName=queue_name) self.ecs_service_name = ecs_service_name self.minimum_desired_count = minimum_desired_count def run(self): try: msgs = self.queue.receive_messages(MaxNumberOfMessages=1, WaitTimeSeconds=self.empty_queue_timeout) for msg in msgs: self.process(msg) finally: if self.ecs_service_name: scale_down(self.ecs_service_name, self.minimum_desired_count)

def scale_down(service_name, minimum_desired_count):

ecs = boto3.client(‘ecs’)

cf = boto3.client(‘cloudformation’)

description = describe_ecs_service(ecs, CLUSTER_ARN, SERVICE_ARN)

task_definition, desired_count, running_tasks, pending_tasks \

if desired_count <= minimum_desired_count:

ecs.update_service(cluster=CLUSTER_ARN, service=SERVICE_ARN,

taskDefinition=task_definition,

desiredCount=desired_count — 1)

def __init__(self, queue_name, ecs_service_name=None,

minimum_desired_count=1):

sqs = boto3.resource(‘sqs’, region_name=AWS_REGION)

self.queue = sqs.get_queue_by_name(QueueName=queue_name)

self.ecs_service_name = ecs_service_name

self.minimum_desired_count = minimum_desired_count

msgs = self.queue.receive_messages(MaxNumberOfMessages=1,

WaitTimeSeconds=self.empty_queue_timeout)

if self.ecs_service_name:

scale_down(self.ecs_service_name,

self.minimum_desired_count)

There are some challenges in perfecting this, such as:

* cluster and service name self-inspection (so that the worker can identify which ECS cluster it is supposed to modify)

* policies allowing ECS service updates from within the ECS cluster

But it works, and it does not add any unnecessary infrastructure to your stack.

## ‘I’m OK, my tasks serve short-lived requests’, a.k.a. ‘containers with HTTP servers’

So far we have seen how we tackled the issue of how we managed to scale workers (despite ECS), but how does ECS deal with tasks that are listening to a port?

We have our very own auctioneer API, which of course listens to a port. That API is pretty much idle during most of the day, but at some point it could start receiving a lot of simultaneous requests that can not be handled by just one EC2 instance.

ECS kindly offers to add a load balancer (Elastic Load Balancer — ELB) in front of your cluster’s instances to balance traffic but it currently has a two-pronged limitation:

* You cannot run more than one task listening to the same port on an EC2 instance,

* ELB forwards the requests to only one port on the EC2 instance

![](https://cdn-images-1.medium.com/max/2000/1*McF2RZRmZ1NB4YDZ8zTJ_g.png)

This is a challenge, cost-wise, because you need to spawn a new EC2 instance for each API task every time it is under a heavy load. Loosening just one of the above limitations would allow us to run multiple API tasks on a single instance (just as we do with the workers). We’re happy to say that AWS is aware of this limitation and we are looking forward to it being addressed in the future.

## Concluding remarks

To summarise:

AWS still has some issues when scaling up: if you need a new instance it may take a couple of minutes, you need a big pool of available instances if you want your ECS services to be able to scale up fast.

There are also challenges scaling down. There is no way so far to do it without risking ECS killing an active task execution instead of an idle one. Same thing for instances: you could be killing active EC2 instances instead of idle ones *(‘if the CPU reservation of the ECS cluster goes below 40% decrease the number of instances by 1′)*.

So these scaling issues plus the fact that the ELB has the only-one-task–per-port on an EC2 instance limitations mean there are some improvements we’d love to see from ECS.

For us it would be ideal if ECS could abstract us from most of the scaling issues: we shouldn’t even have to know anything about computing instances (EC2), that should be an ECS internal that automatically handles for you when the desired count of a service requires it. This way we would have more time to focus on our software instead of the AWS architecture it has to run on.

All this being said, ECS is a fairly new service and it’s being actively supported and improved by AWS. We hope it will be an important cornerstone for our cloud services. There is a fun component to it: sometimes you get an unexpected gift (for example, the inclusion of new metrics), sometimes you get to see things ‘just work’ (if things go wrong, just kill an instance and see a new one pick up the slack). And most importantly: we keep learning really cool stuff. We’re excited to see what the future of ECS brings.

***UPDATE
 ***AWS are continually improving ECS as they have just pushed another major update a couple of weeks ago: [Service Autoscaling](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html).

*Jacek Wojdeł and Dani Ametller are senior software engineers in Skyscanner’s Hotels Data Squad, based in the Barcelona office.*

## Learn with us

Take a look at our [current job roles](https://www.skyscanner.net/jobs/current-jobs/) available across our 10 global offices.

![[We’re hiring!](https://www.skyscanner.net/jobs/current-jobs/)](https://cdn-images-1.medium.com/max/2050/1*zvfRkK8SIOuNFiFn_KcUUw.png)*[We’re hiring!](https://www.skyscanner.net/jobs/current-jobs/)*
