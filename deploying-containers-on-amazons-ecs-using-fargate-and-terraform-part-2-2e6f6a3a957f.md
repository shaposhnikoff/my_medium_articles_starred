
# Deploying Containers on Amazon’s ECS using Fargate and Terraform: Part 2



In this part of the tutorial we will focus on writing out our entire infrastructure from [Part 1](https://medium.com/@bradford_hamilton/deploying-containers-on-amazons-ecs-using-fargate-and-terraform-part-1-a5ab1f79cb21) using Terraform. Terraform enables you to predictably create, change, and improve infrastructure using code. By the end of this section we will be able to run one command to deploy our entire application giving us:

* A Virtual Private Cloud with private and public subnets

* Multiple running container instances

* A load balancer distributing traffic between the containers

* Auto scaling of your resources

* Health checks and logs

Every time I think about that, it blows me away. Just press enter and build an entire system in a few minutes that not too long ago would have taken numerous people a lot of time and money to get up and running.

![](https://cdn-images-1.medium.com/max/3860/1*MlINxdxPv8AWYyqKxzWfiQ.png)

## Let’s get started

One of the nice parts about Terraform is that it’s very self-explanatory. There is some syntax and configuration to learn and then it feels a bit like you’re just outlining how you want your system to work. [Here is a link to the Terraform AWS docs](https://www.terraform.io/docs/providers/aws/index.html). Let’s begin by creating the project and setting up the file structure we want to use.

    mkdir terraform-example && cd terraform-example

Next within the root of the project we’ll add a .gitignore file with the following contents.

    # Compiled files
    *.tfstate
    *.tfstate.backup
    .terraform.tfstate.lock.info

    # Module directory
    .terraform/

Within the root of the project add a terraform/ directory. As you’ll see next - Terraform config files use the file extension .tf.

Inside the directory you just made let’s add the following files:

* provider.tf

* ecs.tf

* alb.tf

* network.tf

* security.tf

* auto_scaling.tf

* logs.tf

* variables.tf

* outputs.tf

I tried breaking the code up into files handling each of the different parts of our infrastructure. This was for (hopefully) more clarity, however it will work just fine if you have all the code in just one big main file as well.

The first thing we’ll do is specify the provider. This can be done in different ways but here we’re telling Terraform our provider is AWS and that it can find our credentials in $HOME/.aws/credentials which is the default location for AWS credentials on Mac and Linux.

<iframe src="https://medium.com/media/b011528379e4683b5173b16c05816617" frameborder=0></iframe>

You’ll notice the string interpolated value for “region”. As you may be able to tell, this is a variable. All of our variables will look like this using dot notation. Let’s set up our variables file so that this makes a little more sense.

<iframe src="https://medium.com/media/11f974ec646e8d5b4907d154aa89794e" frameborder=0></iframe>

Terraform will know to look here for these values anytime it sees ”${var.VARIABLE_NAME}”. For the sake of clarity the file above is all the variables for the application. Hopefully with their descriptions they all make sense. There are two role ARNs that I left placeholders for. If you followed [Part 1](https://medium.com/@bradford_hamilton/deploying-containers-on-amazons-ecs-using-fargate-and-terraform-part-1-a5ab1f79cb21) of this tutorial you will have these roles and can navigate to them in your AWS console to get their ARNs.

We’ll again be using my crystal_blockchain docker image for our application. If you see “cb” throughout the terraform configs - thats what it refers to.

Time to create our network. To get the minimal amount of high availability, we’ll deploy our ECS cluster to run on at least two Availability Zones (AZs). The load balancer also needs at least 2 public subnets in different AZs.

<iframe src="https://medium.com/media/dc5ce4cbfb6949778d16aa759d9beed4" frameborder=0></iframe>

Now that we have our network created, let’s set up some security groups to make sure our app is properly protected.

<iframe src="https://medium.com/media/db9fc7b46bc820f81ec8facbd66f6509" frameborder=0></iframe>

With all this in place, we’re ready to build out our application load balancer and ECS cluster! For this tutorial we’re going to be listening for HTTP requests on port 80, however I think it goes without saying that if you’re using this in production - listen for HTTPS over port 443. You can use AWS certificate manager (ACM) to provision and manage certificates.

First, our load balancer with a health check:

<iframe src="https://medium.com/media/5ea16a1d139fa14709e69fd9e688be90" frameborder=0></iframe>

Then our ECS cluster:

<iframe src="https://medium.com/media/58e62d9d2f71da99fc78c5f663f6ee2a" frameborder=0></iframe>

On line 7 above, we use a data source for our container definition. Container definitions can also be written inline in an aws_ecs_task_definition. [Here](https://www.terraform.io/docs/providers/aws/d/ecs_task_definition.html) is a link to the docs for how that looks. I prefer to keep things modular and it’s easier for me to read and follow like that. Feel free to go about it either way!

If you do keep the datasource for the container definition we’ll have to add it to the project. Inside the same directory with all of our .tf config files, let’s create a templates/ directory and within that, an ecs/ directory. This is where we’ll put our cb_app.json.tpl file.

### You’re getting close!

Time to get some auto scaling set up to monitor our application and automatically adjust capacity. We’ll create two CloudWatch alarms and two auto scaling policies to go with them.

<iframe src="https://medium.com/media/7379b42885c3abcdab5ed2d57a6cf8b3" frameborder=0></iframe>

Last but not least let’s add logging.

<iframe src="https://medium.com/media/a131706e3b75051dce422911c25e3080" frameborder=0></iframe>

### Wooooo!

You just built a badass infrastructure for an application. The only thing left to do is deploy! Terraform offers “outputs” which are great for getting data back after a deploy is successful. One last config file and it’s a short one I promise :)

<iframe src="https://medium.com/media/c0864586621bfe7ff52eccbde3bb3440" frameborder=0></iframe>

This gives us back the load balancer DNS name (A record) after we deploy.

Time to run terraform init terraform/ in your terminal in the root of your project. This gets it all set up and ready to apply.

Finally, run terraform apply terraform/ to get this bad boy deployed! It will refresh state for all your resources and show you what will be created/removed/updated. It then prompts you to reply with “yes” if you want to perform the actions. That’s it! You will see everything getting provisioned. This will take a few minutes.

When it spits out your load balancer url, go ahead and visit it at port :3000 and you should see the JSON returned from the initial block in your deployed crystal_blockchain app.

I don’t know about you but I’m feeling like

![](https://cdn-images-1.medium.com/max/4896/1*OX0eOejwHaOYPHh5EE7l5g.jpeg)

Here’s a link to the code in full:
[**bradford-hamilton/terraform-ecs-fargate**
*GitHub is where people build software. More than 28 million people use GitHub to discover, fork, and contribute to over…*github.com](https://github.com/bradford-hamilton/terraform-ecs-fargate)

Thanks for reading! :)
