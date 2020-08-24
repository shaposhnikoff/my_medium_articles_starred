
# AWS EC2 on the command line

As per AWS official documentation
> *Amazon Elastic Compute Cloud (Amazon EC2) is a web service that provides resizeable computing capacity — literally, servers in Amazon’s data centers — that you use to build and host your software systems.*

*To setup AWS CLI [https://medium.com/@devopslearning/aws-cli-command-line-interface-a48dc3123a25](https://medium.com/@devopslearning/aws-cli-command-line-interface-a48dc3123a25)*

*Now let take a look at the various steps we used to follow when try to create any via instance via GUI*

*Step1: Go to the EC2 tab under Compute*

![](https://cdn-images-1.medium.com/max/2000/1*mB5csoOZCL-55P2HPelhMA.png)

*Step2: Click on Launch Instance*

![](https://cdn-images-1.medium.com/max/3160/1*Eg166e4DfKI8Dm2CJfUs9w.png)

*Step3: Choose an AMI(In this case I am choosing RedHat AMI)*

![](https://cdn-images-1.medium.com/max/4684/1*w9HxqoHZOgpXhiXOAQVD0w.png)

*Step4: Select an instance type(T2 micro in this case)*

![](https://cdn-images-1.medium.com/max/5728/1*Cs3CqNcV26vuMruhs4Glow.png)

*Step5: Provide all the configure details(i.e Number of instances, VPC under Network)*

![](https://cdn-images-1.medium.com/max/5808/1*28cGG1XARF1Vx-ifDJuPlQ.png)

*Step6: Add Storage*

![](https://cdn-images-1.medium.com/max/5696/1*yOStPAU_hKvTY7BHRyRFKw.png)

*Step7: Adding Tags(Optional but it always good to give Tag)*

![](https://cdn-images-1.medium.com/max/5324/1*pgu0YQoVOWliSgEt7HOfXA.png)

*Step8: Configure Security Group*

![](https://cdn-images-1.medium.com/max/5736/1*wd4DjQKYMF4h4PLbfQZZqQ.png)

*Step9:Review and Launch*

*Step10: It will ask for existing Key-pair or new Key Pair*
> *A key pair consists of a public key that AWS stores, and a private key file that you store. Together, they allow you to connect to your instance securely. For Windows AMIs, the private key file is required to obtain the password used to log into your instance. For Linux AMIs, the private key file allows you to securely SSH into your instance.*

*Now rather then launching an instance via GUI, we can do it with the help aws cli and the command for that purpose is aws ec2 run-instances*

    *# aws ec2 run-instances --image-id ami-28e07e50 --region us-west-2 --key aws-sysops --instance-type t2.micro --output text*

    *349934551430 r-083b6c00dbaa27bb0*

    *INSTANCES 0 x86_64  False xen ami-28e07e50 i-09b70df4a9f119d03 t2.micro aws-sysops 2018-05-21T23:16:27.000Z ip-172-31-31-65.us-west-2.compute.internal 172.31.31.65  /dev/sda1 ebs True subnet-bbb189dd hvm vpc-8d8f76f4*

    *CPUOPTIONS 1 1*

    *MONITORING disabled*

    *NETWORKINTERFACES  02:d4:60:41:2d:06 eni-6341775a 349934551430 ip-172-31-31-65.us-west-2.compute.internal 172.31.31.65 True in-use subnet-bbb189dd vpc-8d8f76f4*

    *ATTACHMENT 2018-05-21T23:16:27.000Z eni-attach-aebe596e True 0 attaching*

    *GROUPS sg-9a83f6e6 default*

    *PRIVATEIPADDRESSES True ip-172-31-31-65.us-west-2.compute.internal 172.31.31.65*

    *PLACEMENT us-west-2a  default*

    *SECURITYGROUPS sg-9a83f6e6 default*

    *STATE 0 pending*

    *STATEREASON pending pending*

* *We can check the status of instance*

    *# aws ec2 describe-instance-status --region us-west-2 --output text*

    *INSTANCESTATUSES us-west-2a i-09b70df4a9f119d03*

    *INSTANCESTATE 16 running*

    *INSTANCESTATUS initializing*

    *DETAILS reachability initializing*

    *SYSTEMSTATUS initializing*

    *DETAILS reachability initializing*

* *To terminate an instance*

    *# aws ec2 terminate-instances --instance-ids i-09b70df4a9f119d03*

    *{*

    *"TerminatingInstances": [*

    *{*

    *"InstanceId": "i-09b70df4a9f119d03",*

    *"CurrentState": {*

    *"Code": 32,*

    *"Name": "shutting-down"*

    *},*

    *"PreviousState": {*

    *"Code": 16,*

    *"Name": "running"*

    *}*

    *}*

    *]*

    *}*

* *In the previous example, we picked all the existing values eg: Keypair what would be the case if we want to generate all these values on the command line*

*To create EC2 keypair*

    *#aws ec2 create-key-pair --key-name MyKeyPair --output text >>mykeypair.pem*

*To create Security Group*

    *aws ec2 create-security-group --group-name MySecurityGroup --description "My security group" --vpc-id vpc-1a2b3c4d*

*To get information about security group*

    *# aws ec2 describe-security-groups --group-names MySecurityGroup*

*To get information about specific id*

    *# aws ec2 describe-images --image-ids ami-28e07e50*

*To launch an instance*

    *aws ec2 run-instances --image-id ami-abc1234 --count 1 --instance-type t2.micro --key-name keypair --subnet-id subnet-abcd1234 --security-group-ids sg-abcd1234*

*To allocate Elastic IP address*

    *aws ec2 allocate-address*

*To assign this EIP to instance*

    *aws ec2 associate-address --instance-id i-07ffe74c7330ebf53 --public-ip 198.51.100.0*

*To Terminate Instance*

    *aws ec2 terminate-instances --instance-ids i-1234567890abcdef0*

*Different states*

    *o **0 **: **pending***

    *o **16 **: **running***

    *o **32 **: **shutting-down***

    *o **48 **: **terminated***

    *o **64 **: **stopping***

    *o **80 **: **stopped***
