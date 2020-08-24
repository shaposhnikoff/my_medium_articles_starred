
# Use AWS Systems Manager: Bastion free & SSH Key free access to EC2 Instances

This Blog has moved from Medium to blogs.tensult.com. All the latest content will be available there. Subscribe to our newsletter to stay updated.

Ever since I learned AWS I had a basic requirement, Access the EC2 instance from AWS web console without using a bastion host or an SSH key. Is it possible to do it ? Yes, this can be done with a simpler configuration using the AWS System Manager’s Session Manager options. Also, System Manager can access Windows systems CLI.

**How does it help ?**

1. Since the SSH port is not opened, SSH brute force attack risks are eliminated completely. Communication between instance and System Manager is through a encrypted tunnel.

1. Bastion host is not required, and user is free from login to multiple systems before accessing the instances.

1. The key sharing can be avoided and access to the instance can be limited using AWS IAM permissions. Read our blog on the issues associated with sharing SSH keys [here](https://medium.com/tensult/ec2-key-sharing-issues-and-remedies-d4ff677a88be).

1. It provides an easy access to the EC2 instances. Just like traditional virtualisation setup you can switch between the instances easily.

1. Session Manager API can provide programatic access and further integration with other services.

## **Configuration**

    AWS Region: N.Virginia
    OS: Amazon Linux 2
    RPMS: amazon-ssm-agent-2.3.50.0-1.x86_64

* Instance preparation

* SSM agent Installation

* AWS Systems Manager setup

**Instance preparation**

1. Create a IAM Role which will be attached the EC2 instance later in the experiment. *AmazonEC2RoleForSSM* policy allows SSM service access.

![](https://cdn-images-1.medium.com/max/2256/1*P1V5a3iC14yL1kkpCo5UTQ.png)

2) Create an EC2 Instance of your preference. I have used Amazon Linux 2 AMI. Attach the Role you created to the instance.

![](https://cdn-images-1.medium.com/max/2406/1*ju3GQygCf5gXPbGnx3EqqA.png)

**SSM Agent Installation**

1. Access the EC2 instance you have created with the SSH key for the one time SSM agent configuration.

1. Execute the commands below after you login(sudo) as root.

    # mkdir /tmp/ssm
    # cd /tmp/ssm
    # yum install -y [https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm](https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm)
    # systemctl enable amazon-ssm-agent
    # systemctl start amazon-ssm-agent

For more information on SSM agent installation, please follow document below.
[**Manually Install SSM Agent on Amazon EC2 Linux Instances - AWS Systems Manager**
*Use one of the following scripts to install SSM Agent on one of the following Linux instances.*docs.aws.amazon.com](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-manual-agent-install.html#agent-install-al)

3)Make sure that SSM agent version is 2.3.12 or above.

![](https://cdn-images-1.medium.com/max/2000/1*xl7z3JPsGNSB0USTZ4Vf-Q.png)

**AWS Systems Manager setup**

1. From the AWS Web Console access the System Manager service

![](https://cdn-images-1.medium.com/max/2490/1*vSOKgyUrmCpqj0UtpWATfQ.png)

2) Click *Session Manager* and then click “*Start Session*”.

![](https://cdn-images-1.medium.com/max/2832/1*lwbOrOeMuh5xOzQMhnAasA.png)

3) In the next window, select the instance and click “*Start Session*”

![](https://cdn-images-1.medium.com/max/2322/1*LmERa8ftqW8_Hu-3nD44qg.png)

4) The OS console window opens and you are able to execute any command on the instance.

![](https://cdn-images-1.medium.com/max/2000/1*4QKOqm8jzvPiSxp0BublHQ.png)

**Note: **For Windows make sure that you have installed latest or supported SSM agent. System Manager can access Windows CLI.

![](https://cdn-images-1.medium.com/max/2000/1*Ws0IBashzbD-djLG-Jh7jA.png)

**Conclusion**

Accessing the EC2 instance is an easy process now. No need of a bastion host or the SSH key. You can do it using AWS session manager with a simple configuration.

**Related information**
[**Managing Windows and Linux Without logging in — Bastion Free AWS SSM**
*Are you patient enough to login to all of your systems and execute commands or prefer to do it from centralised web…*medium.com](https://medium.com/tensult/managing-windows-and-linux-without-logging-in-aws-ssm-a35ad93a9924)
