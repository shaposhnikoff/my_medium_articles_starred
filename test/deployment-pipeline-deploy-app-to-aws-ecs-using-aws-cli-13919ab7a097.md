
# Deployment pipeline — Deploy app to AWS ECS using AWS CLI

Amazon ECS (EC2 container service) provides a highly scalable management service for running docker containers. ECS provides an incredibly easy way to create highly scalable and fault-tolerant infrastructure for applications using docker. It takes care of all the cluster management, provides awesome command line interface to manage and automate your application and integrates with ELB, S3, VPC, EBS etc. So, overall this is a great way to deploy your app within docker containers and automate the deploy process.

Note:- there are a number of container orchestration frameworks that are still evolving: kubernates, swarm, rancher, Mesos Marathon, Amazon ECS, Azure ACS, Google container engine, Tutum (acquired), cloud foundry…..

### AWS CLI commands for setting up and running an EC2 instance within an ECS cluster

TODO: Add explanations around the commands. I just grabbed them from my history.. too lazy… ECS-CLI provides a simpler interface, but its better to learn these core one’s…

Install AWS CLI

    curl “[https://s3.amazonaws.com/aws-cli/awscli-bundle.zip](https://s3.amazonaws.com/aws-cli/awscli-bundle.zip)" -o “awscli-bundle.zip” 

    unzip awscli-bundle.zip

    sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws

    aws --version

Configure AWS CLI

    aws configure

    AWS Access Key ID [None]: ***<your access key -- create using console>
    ***AWS Secret Access Key [None]: ***<your secret key -- get using console>
    ***Default region name [None]: ***us-east-1
    ***Default output format [None]: *ENTER*

    aws iam list-users

Create Key Pair to SSH into EC2 instances

    aws ec2 create-key-pair --key-name aws-sunkay --query 'KeyMaterial' --output text > aws-sunkay.pem

    chmod 400 aws-sunkay.pem

    aws ec2 describe-key-pairs --key-name aws-sunkay.pem
     
    aws ec2 describe-key-pairs --key-name aws-sunkay

Create & Authorize Security Group for SSH, HTTP & RDS

    aws ec2 create-security-group --group-name sunkay_SG_useast1 --description "security group for sunkay on us-east-1"

    aws ec2 describe-security-groups --group-id sg-xxxxxx

    # SSH
    aws ec2 authorize-security-group-ingress --group-id sg-xxxx --protocol tcp --port 22 --cidr 0.0.0.0/0

    #HTTP
    aws ec2 authorize-security-group-ingress --group-id sg-xxxxx --protocol tcp --port 80 --cidr 0.0.0.0/0

    # default port for RDS
    aws ec2 authorize-security-group-ingress --group-id sg-xxxxx --protocol tcp --port 5432 --source-group sg-xxxxx

    # default port for Elastic Cache
    aws ec2 authorize-security-group-ingress --group-id sg-xxxxx --protocol tcp --port 6379 --source-group sg-xxxxx

    aws ec2 describe-security-groups --group-id sg-xxxxxx

Create a Docker cluster using ECS commands

    
    aws ecs create-cluster --cluster-name docker-cluster

    aws ecs list-clusters

    aws ecs describe-clusters --clusters docker-cluster

Create an S3 bucket to store config information

    aws s3api create-bucket --bucket sunkay_deepdive

    aws s3 cp deepdive/ecs.config s3://sunkay_deepdive/ecs.config

    aws s3 ls s3://sunkay_deepdive

    aws ec2 run-instances --image-id ami-2b3b6041 --count 1 --instance-type t2.micro --iam-instance-profile Name=ecsInstanceRole --key-name aws-sunkay --security-group-ids sg-4854fe33 --user-data file://copy-ecs-config-to-s3

    aws ec2 describe-instance-status i-98887504

    aws ec2 describe-instance-status instance-id i-98887504

    aws ec2 describe-instance-status --instance-id i-98887504

    aws ec2 describe-instance-status --instance-id i-98887504

    aws ecs list-container-instances

    aws ecs list-container-instances --cluster deepdive

    aws ecs list-container-instances

    aws ecs list-container-instances --cluster deepdive

    aws ecs describe-container-instances --cluster deepdive --container-instances arn:aws:ecs:us-east-1:698759075373:container-instance/cdafce7f-a19f-4481-a6f6-41e5ef7f40c4

    aws ecs describe-clusters --cluster deepdive

Simple ngnix task definition

    {
     “containerDefinitions”: [
        {
         “name”: “nginx”,
         “image”: “nginx”,
         “portMappings”: [
         {
            “containerPort”: 80,
            “hostPort”: 80
         }
         ],
         “memory”: 50,
         “cpu”: 102
       }
     ],
     “family”: “web”
    }

    # COMMANDS to register task definitions 

    aws ecs register-task-definition — cli-input-json file://web-task-definition.json
    aws ecs list-task-definition-families
    aws ecs list-task-definitions
    aws ecs describe-task-definition — task-definition web:1
    aws ecs register-task-definition — cli-input-json file://web-task-definition.json
    aws ecs list-task-definitions
    aws ecs deregister-task-definiton — task-definition web:2
    aws ecs deregister-task-definition — task-definition web:2
    aws ecs list-task-definitions

### Starting the container using services

    aws ecs create-service — cluster deepdive — service-name web — task-definition web — desired-count 1

    aws ecs list-services — cluster deepdive

    aws ecs describe-services — cluster deepdive — services web

    aws ec2 describe-instances

    aws ecs update-service — cluster deepdive — service web — task-definition web — desired-count 2

    aws ecs describe-services — cluster deepdive — services web

    aws ecs describe-services — cluster deepdive — services web

    aws ec2 describe-instances

    aws ecs update-service — cluster deepdive — service web — task-definition web — desired-count 0

    aws ecs delete-service — cluster deepdive — service web

    aws ecs list-services — cluster deepdive

### Terminating the cluster

    aws ec2 terminate-instances — instance-ids i-98887504
    aws s3 rm s3://sunkay_deepdive — recursive
    aws s3api delete-bucket — bucket sunkay_deepdive
    aws ecr delete-repository — repository-name deepdive/nginx — force
    aws ecr delete-repository — repository-name deepdive — force
    aws ecs delete-cluster — cluster deepdive
