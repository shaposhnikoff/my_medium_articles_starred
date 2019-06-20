
# Working with AWS CodeDeploy

As usual when I make a breakthrough after bumping my head against the wall for a few days trying to get something to work, I hasten to write down my notes here so I can remember what I’ve done ;) In this case, the head-against-the-wall routine was caused by trying to get AWS CodeDeploy to work within the regular code deployment procedures that we have in place using Jenkins and Capistrano. Here is the 30,000 foot view of how the deployment process works using a combination of Jenkins, Docker, Capistrano and AWS CodeDeploy:

1. Code gets pushed to GitHub

1. Jenkins deployment job fires off either automatically (for development environments, if so desired) or manually

2.1 Jenkins spins up a Docker container running Capistrano and passes it several environment variables such as GitHub repository URL and branch, target deployment directory, etc.

2.2 The Capistrano Docker image is built beforehand and contains rake files that specify how the code it checks out from GitHub is supposed to be built

2.3 The Capistrano Docker container builds the code and exposes the target deployment directory as a Docker volume

2.4 Jenkins archives the files from the exposed Docker volume locally as a tar.gz file

2.5 Jenkins uploads the tar.gz to an S3 bucket

2.6 For good measure, Jenkins also builds a Docker image of a webapp container which includes the built artifacts, tags the image and pushes it to Amazon ECR so it can be later used if needed by an orchestration system such as Kubernetes

3. AWS CodeDeploy runs a code deployment (via the AWS console currently, using the awscli soon) while specifying the S3 bucket and the tar.gz file above as the source of the deployment and an AWS AutoScaling group as the destination of the deployment

4. Everybody is happy

You may ask: why Capistrano? Why not use a shell script or some other way of building the source code into artifacts? Several reasons:

* Capistrano is still one of the most popular deployment tools. Many developers are familiar with it.

* You get many good features for free just by using Capistrano. For example, it automatically creates a releases directory under your target directory, creates a timestamped subdirectory under releases where it checks out the source code, builds the source code, and if everything works well creates a ‘current’ symlink pointing to the releases/timestamped subdirectory

* This strategy is portable. Instead of building the code locally and uploading it to S3 for use with AWS CodeDeploy, you can use the regular Capistrano deployment and build the code directly on a target server via ssh. The rake files are the same, only the deploy configuration differs.

I am not going to go into details for the Jenkins/Capistrano/Docker setup. I’ve touched on some of these topics in previous posts. I will go into details for the AWS CodeDeploy setup. Here goes.

**Create IAM policies and roles**

There are two roles that need to be created for AWS CodeDeploy to work. One is to be attached to EC2 instances that you want to deploy to, and one is to be used by the CodeDeploy agent running on each instance. — Create following IAM policy for EC2 instances, which allows those instances to list S3 buckets and download fobject from S3 buckets (in this case the permissions cover all S3 buckets, but you can specify specific ones in the Resource variable):

    { “Version”: “2012–10–17”, 
      “Statement”: [ { 
         “Action”: [ “s3:Get*”, “s3:List*” ], 
         “Effect”: “Allow”, 
         “Resource”: “*” } 
      ]
    }

* Attach above policy to an IAM role and name the role e.g. **CodeDeploy-EC2-Instance-Profile**

* Create following IAM policy to be used by the CodeDeploy agent running on the EC2 instances you want to deploy to:

    { "Version": "2012-10-17", 
    "Statement": [ 
    { "Effect": "Allow", 
    "Action": [ "autoscaling:CompleteLifecycleAction", "autoscaling:DeleteLifecycleHook", "autoscaling:DescribeAutoScalingGroups", "autoscaling:DescribeLifecycleHooks", "autoscaling:PutLifecycleHook", "autoscaling:RecordLifecycleActionHeartbeat", "autoscaling:CreateAutoScalingGroup", "autoscaling:UpdateAutoScalingGroup", "autoscaling:EnableMetricsCollection", "autoscaling:DescribeAutoScalingGroups", "autoscaling:DescribePolicies", "autoscaling:DescribeScheduledActions", "autoscaling:DescribeNotificationConfigurations", "autoscaling:DescribeLifecycleHooks", "autoscaling:SuspendProcesses", "autoscaling:ResumeProcesses", "autoscaling:AttachLoadBalancers", "autoscaling:PutScalingPolicy", "autoscaling:PutScheduledUpdateGroupAction", "autoscaling:PutNotificationConfiguration", "autoscaling:PutLifecycleHook", "autoscaling:DescribeScalingActivities", "autoscaling:DeleteAutoScalingGroup", "ec2:DescribeInstances", "ec2:DescribeInstanceStatus", "ec2:TerminateInstances", "tag:GetTags", "tag:GetResources", "sns:Publish", "cloudwatch:DescribeAlarms", "elasticloadbalancing:DescribeLoadBalancers", "elasticloadbalancing:DescribeInstanceHealth", "elasticloadbalancing:RegisterInstancesWithLoadBalancer", "elasticloadbalancing:DeregisterInstancesFromLoadBalancer" ], "Resource": "*" 
    } 
    ] 
    }

* Attach above policy to an IAM role and name the role e.g. **CodeDeployServiceRole**

**Create a ‘golden image’ AMI**

The whole purpose of AWS CodeDeploy is to act in conjunction with Auto Scaling Groups so that the app server layer of your infrastructure becomes horizontally scalable. You need to start somewhere, so I recommend the following:

* set up an EC2 instance for your app server the old-fashioned way, either with Ansible/Chef/Puppet or with Terraform

* configure this EC2 instance to talk to any other layers it needs, i.e. the database layer (either running on EC2 instances or, if you are in AWS, on RDS), the caching layer (dedicated EC2 instances running Redis/memcached, or AWS ElastiCache), etc.

* deploy some version of your code to the instance and make sure your application is fully functioning

If all this works as expected, take an AMI image from this EC2 instance. This image will serve as the ‘golden image’ that all other instances launched by the Auto Scaling Group / Launch Configuration will be based on.

**Create Application Load Balancer (ALB) and Target Group**

The ALB will be the entry point into your infrastructure. For now just create an ALB and an associated Target Group. Make sure you add your availability zones into the AZ pool of the ALB. If you want the ALB to handle the SSL certificate for your domain, add the SSL cert to Amazon Certificate Manager and add a listener on the ALB mapping port 443 to the Target Group. Of course, also add a listener for port 80 on the ALB and map it to the Target Group. I recommend creating a dedicated Security Group for the ALB and allowing ports 80 and 443, either from everywhere or from a restricted subnet if you want to test it first. For the Target Group, make sure you set the correct health check for your application (something like requesting a special file healthcheck.html over port 80). No need to select any EC2 instances in the Target Group yet.

**Create Launch Configuration and Auto Scaling Group**

Here are the main elements to keep in mind when creating a Launch Configuration to be used in conjunction with AWS CodeDeploy:

* **AMI ID**: specify the AMI ID of the ‘golden image’ created above

* **IAM Instance Profile**: specify **CodeDeploy-EC2-Instance-Profile** (role created above)

* **Security Groups**: create a Security Group that allows access to ports 80 and 443 from the ALB Security Group above

* **User data**: each newly launched EC2 instance based on your golden image AMI will have to get the AWS CodeDeploy agent installed. Here’s the user data for an Ubuntu-based AMI in us-west-2 (taken from the AWS CodeDeploy documentation):

    #!/bin/bash
    apt-get -y update
    apt-get -y install awscli
    apt-get -y install ruby
    cd /home/ubuntu
    aws s3 cp s3://aws-codedeploy-us-west-2/latest/install . --region us-west-2
    chmod +x ./install
    ./install auto 

Alternatively, you can run these commands your initial EC2 instance, then take a golden image AMI based off of that instance. That way you make sure that the CodeDeploy agent will be running on each new EC2 instance that gets provisioned via the Launch Configuration. In this case, there is no need to specify a User data section for the Launch Configuration. Once the Launch Configuration is created, you’ll be able to create an Auto Scaling Group (ASG) associated with it. Here are the main configuration elements for the ASG:

* **Launch Configuration**: the one defined above

* **Target Groups**: the Target Group defined above

* **Min/Max/Desired**: up to you to define the EC2 instance count for each of these. You can start with 1/1/1 to test

* **Scaling Policies**: you can start with a static policy (corresponding to Min/Max/Desired counts) and add policies based on alarms triggered by various Cloud Watch metrics such as CPU usage, memory usage, etc as measured on the EC2 instances comprising the ASG

Once the ASG is created, depending on the Desired instance count, that many EC2 instances will be launched.

**Create AWS CodeDeploy Application and Deployment Group**

We finally get to the meat of this post. Go to the AWS CodeDeploy page and create a new Application. You also need to create a Deployment Group while you are at it. For Deployment Type, you can start with ‘In-place deployment’ and once you are happy with that, move to ‘Blue/green deployment, which is more complex but better from a high-availability and rollback perspective. In the Add Instances section, choose ‘Auto scaling group’ as the tag type, and the name of the ASG created above as the key. Under ‘Total matching instances’ below the Tag and Key you should see a number of EC2 instances corresponding to the Desired count in your ASG. For Deployment Configuration, you can start with the default value, which is OneAtATime, then experiment with other types such as HalfAtATime (I don’t recommend AllAtOnce unless you know what you’re doing)

For Service Role, you need to specify the **CodeDeployServiceRole** service role created above.

**Create scaffoding files for AWS CodeDeploy Application lifecycle**

At a minimum, the tar.gz or zip archive of your application’s built code also needs to contain what is called an AppSpec file, which is a YAML file named appspec.yml. The file needs to be in the root directory of the archive. Here’s what I have in mine:

    version: 0.0
    os: linux
    files:
      - source: /
        destination: /var/www/mydomain.com/
    hooks:
      BeforeInstall:
        - location: before_install
          timeout: 300
          runas: root
      AfterInstall:
        - location: after_install
          timeout: 300
          runas: root

The before_install and after_install scripts (you can name them anything you want) are shell scripts that will be executed after the archive is downloaded on the target EC2 instance.

The before_install script will be run **before** the files inside the archive are copied into the destination directory (as specified in the destination variable /var/www/mydomain.com). You can do things like create certain directories that need to exist, or change the ownership/permissions of certain files and directories.

The after_install script script will be run **after** the files inside the archive are copied into the destination directory. You can do things like create symlinks, run any scripts that need to complete the application installation (such as scripts that need to hit the database), etc.

One note specific to archives obtained from code built by Capistrano: it’s customary to have Capistrano tasks create symlinks for directories such as media or var to volumes outside of the web server document root (when media files are mounted over NFS/EFS for example). When these symlinks are unarchived by CodeDeploy, they tend to turn into regular directories, and the contents of potentially large mounted file systems get copied in them. Not what you want. I ended up creating all symlinks I need in the after_install script, and not creating them in Capistrano.

There are other points in the Application deploy lifecycle where you can insert your own scripts. See the [AppSpec hook documentation](http://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html).

**Deploy the application with AWS CodeDeploy**

Once you have an Application and its associated Deployment Group, you can select this group and choose ‘Deploy new revision’ from the Action drop-down. For the Revision Type, choose ‘My application is stored in Amazon S3’. For the Revision Location, type in the name of the S3 bucket where Jenkins uploaded the tar.gz of the application build. You can play with the other options according to the needs of your deployment. Finally, hit the Deploy button, baby! If everything goes well, you’ll see a nice green bar showing success.

![](https://cdn-images-1.medium.com/max/2000/0*vp6LPMJxnbUbR8c3.png)

If everything does not go well, you can usually troubleshoot things pretty well by looking at the logs of the Events associated with that particular Deployment. Here’s an example of an error log:

    **ScriptFailed** 
    **Script Name** after_install 
    **Message Script** at specified location: after_install run as user root failed with exit code 1
    **Log Tail** [stderr]chown: changing ownership of ‘/var/www/mydomain.com/shared/media/images/85.jpg’: Operation not permitted

In this case, I the ‘shared’ directory was mounted over NFS, so I had to make sure the permissions and ownership of the source file system on the NFS server were correct. I am still experimenting with AWS CodeDeploy and haven’t quite used it ‘in anger’ yet, so I’ll report back with any other findings.

*Originally published at [agiletesting.blogspot.com](http://agiletesting.blogspot.com/2017/03/working-with-aws-codedeploy.html).*
