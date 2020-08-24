
# AWS CloudFormation to update Lambda Functions

Most people are already familiar with AWS Lambda functions. If you have a completely Serverless deployment, you can leverage the AWS SAM to manage, package, deploy and update your deployments. SAM scales very well is very easy to use! But there are many cases in which the cloud deployments are not completely serverless i.e. hybrid with EC2, ECS type resources. In such cases, SAM is helpful, but not sufficient, since to deploy other AWS resource types, you need to manage them in a CloudFormation stack. At a large scale, this can become a very complex problem to solve!

In such cases, you can still benefit from [SAM to create and maintain your lambda deployment packages](https://medium.com/@amolkokje/package-and-deploy-aws-lambda-functions-4b734e7c8f8e), but you need a way to update your CloudFormation stack. So, now this becomes a two step process, which can be managed with a few simple tricks. I faced this problem in my recent project, and so in this post, I am going to provide an approach that has worked out very well for me.

In this example, we have a staging bucket which contains the lambda packages used for deployment. SAM is used to update the staging bucket with updated Lambda package. CloudFormation template custom resource code copies the packages and uses in them in their Lambda function resources.

![AWS CloudFormation to update Lambda Functions](https://cdn-images-1.medium.com/max/2264/1*4t0Hkz1n8OWvcZbATgpvVg.png)*AWS CloudFormation to update Lambda Functions*

## How do we update the Lambda function resources when there is a change in code or dependencies?

When you apply a CloudFormation stack update, it will check if there is an update in properties of any of the deployed resource. If there is no update, it will not take any action. It will only act on the resources for which there has been a property update.

When there is an update in Lambda resource property like memory, timeout, environment vars, etc, there is no problem as a CloudFormation will pick these changes and update accordingly.

*The problem arises when there is an update to Lambda function code, or to any of its dependencies*. Since these are not resource property updates, there is no way for CloudFormation to know if there is a change, and update the affected components.

So, in the solution here, the trick here is to update a property in the Custom Resource to update the package, and property in the Lambda function so as to update the Lambda function package. That way, CloudFormation knows that there has been a change, and resource needs to be updated.

**NOTE**: *If the Lambda function is created using a package deployed in s3 bucket, updating the package is not enough to update the Lambda function. You also need to invoke Lambda function update API.*

All the source code for this example is located in my [GitHub repo](https://github.com/amolkokje/cfn_update_lambda).

In this example, SAM is used to generate the Lambda package (SAM is not a requirement for this solution. You may choose to package the way you want to.). The only thing to note is that *the package must have a unique name*, which is what will be used by CloudFormation to detect change in resource property.

As you can see in the code snippet, the ‘**CopyZips**’ resource will copy all the specified packages in ’**Objects**’ from the staging bucket to the deployment bucket. So, we need to update the name in this place so that the names which are added/updated will be copied over to the stack. [Full Template Code](https://github.com/amolkokje/cfn_update_lambda/blob/master/template/update_lambda.yaml).

<iframe src="https://medium.com/media/f1697c9b82f5c4c967fec57f8052d602" frameborder=0></iframe>

Now, we have the updated package, but we still need to update the Lambda function. To achieve that you will need to update the Lambda function resource ‘**S3Key**’ attribute with the package name. See the snippet below for reference.

<iframe src="https://medium.com/media/593806f3d9d447b5ca61a7cecedd4226" frameborder=0></iframe>

For simplicity, in this example, only one Lambda function is used. So, to update the Lambda function, all you need to do is update a stack parameter. But real deployments are much more complex and can have many Lambda functions that may need to be updated. For such cases, you would have to use some advance CloudFormation tricks to update the relevant strings in the template.

One way to achieve that would be to use a [Mapping](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html) of Lambda function names to their deployed/used Package Name. When an update is required, user only needs to update the package name for the affected package in the Mapping. Since all the template code is referencing the Mapping, all relevant Lambda functions will be updated when a Stack Update is applied.

See the sample Mapping below for reference:

    LambdaFunctionPackageMap:
        FirstLambdaFunction:
            PackageName: “b8a6ddc8a38e569f23eb12d0d8d020c9”
        SecondLambdaFunction:
            PackageName: “cd5dff37f7581343bd5e7e2851569dd2”

Wherever the Package Name is used in the template, use the below structure to get the name from the Mapping.

    !FindInMap [LambdaFunctionPackageMap, FirstLambdaFunction, PackageName]

It took me quite some iterations to figure out a clean way to achieve this functionality in my deployments. So, I hope this post and the [sample code](https://github.com/amolkokje/cfn_update_lambda) are useful to you! Feel free to reach out if you may have any relevant questions.
