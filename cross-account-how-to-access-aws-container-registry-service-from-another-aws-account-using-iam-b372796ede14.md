
# Cross-account — How to access AWS container registry service from another AWS account using IAM…

Building microservices and containerized and packaging them in the form of docker images, has become quite a trend in the recent years. These docker images require a secure, reliable and scalable repository to store and manage them. There is one such option available in AWS cloud, Amazon EC2 Container Registry (ECR), is a fully-managed docker container registry that makes it easy for developers to store, manage, and deploy Docker container images.

**Issue**

We have two AWS accounts say A and B and we want to allow an EC2 instance present in the account B configured with instance profile/IAM role to be able to successfully push/pull docker images without passing access/secret keys. This usually can easily be achieved if instance and ECR repository belong to the same account. However, this setup requires a bit more work in order to get things work.

**Resolution**

In this example, we are having an EC2 instance in Account B that wants to access ECR service of account A, which holds our docker images.

Let’s take a look at the steps to resolve the above issue.

1. Create role in Account A and grant permission to access AWS ECR service in this account.

![](https://cdn-images-1.medium.com/max/2000/0*eg_dpJ5nWh2VZFoS.)

We have added AmazonEC2ContainerRegistryPowerUser AWS managed policy to this role as shown in the snapshot above.

2. Create a trusted policy to set up trust with Account B.

Go to AWS console IAM > select role from the roles list and then edit “Trust relationships” to set up trust with account B.

![](https://cdn-images-1.medium.com/max/2246/0*w_uGwNMpM8ogme4Y.)

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::<ACCOUNT-B-ID>:root"
          },
          "Action": "sts:AssumeRole",
          "Condition": {}
        }
      ]
    }

3. We also have to give permission at image level in ECR, so we have added following policy at image permission in Amazon ecr repository. As we click on our repository “miqp-devops”, we will presented by screen shown below.

![](https://cdn-images-1.medium.com/max/2172/0*7dpSgr94L3613yHI.)

Now **ADD **permission

![](https://cdn-images-1.medium.com/max/2094/0*wm1_5pHlRzn80hnV.)

In the **principal** section, we need to add Account B’s ID in order to allow permission for account B’s ID.

We need to select a **Role **that was created earlier in the first step for account B to assume and also select the boxes in **Actions **as per permissions required by account B. Once you save the changes there will be a policy generated automatically as given in the following screenshot.

    {
        "Version": "2008-10-17",
        "Statement": [
            {
                "Sid": "new statement",
                "Effect": "Allow",
                "Principal": {
                    "AWS": [
                        "arn:aws:iam::<ACCOUNT-ID-B>:root",
                        "arn:aws:iam::<Account-ID-A>:role/ECS-AssumeRole-MIQDevOps"
                    ]
                },
                "Action": [
                    "ecr:GetDownloadUrlForLayer",
                    "ecr:BatchGetImage",
                    "ecr:BatchCheckLayerAvailability",
                    "ecr:PutImage",
                    "ecr:InitiateLayerUpload",
                    "ecr:UploadLayerPart",
                    "ecr:CompleteLayerUpload",
                    "ecr:ListImages"
                ]
            }
        ]
    }

4. Go to Account B, then IAM section and select IAM role ( that is attached to an instance in account B ) and modify its trusted policy to following.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Service": "ec2.amazonaws.com"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }

If everything is configured properly, instance in account B, should be able to pull/push docker images from ECR repository hosted in the account A.

**Summary**

As we can see, it is possible to get access to docker images hosted in any AWS container registry service from any AWS account using IAM roles. This method doesn’t need IAM users to be present in the same account where your docker images are hosted.
