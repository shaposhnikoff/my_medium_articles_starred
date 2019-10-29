
# Deploy Angular application to AWS S3 and CloudFront



The purpose of this article is to show you how to deploy your angular application to **AWS S3** in few detailed steps.

An important thing to mention before continue: the only specific steps for Angular in this guide are the parts where we build the project and the automation part. Deploying and configuring the **AWS S3**, **CloudFront **and etc. are reusable and are the same for every static website deploy. So, if you are using React for example, this guide could be also useful for you.

I will divide the tutorial into two parts:

**Basic(minimum setup)**

* Build the project

* Configure AWS S3

* Deploy the build

**Extra**

* Automate the deployment using shell script

* Configure CloudFront to improve our app performance and reduce cost

## Build the project

Using Angular CLI is easy to build your project.

Execute the following command in the root folder of your project:

    ng build --prod --aot

A new folder **dist **will be created containing the bundled files.We will use them later in this guide. Now, we will continue with configuring the **AWS S3** for website hosting usage.

## Configure AWS S3

In order to continue, you will need an AWS account. You can register for free on their website — [https://aws.amazon.com/](https://aws.amazon.com/)

They are offering a lot of goodies for free the first year, check them here — [https://aws.amazon.com/free/](https://aws.amazon.com/free/).

For **S3 **specific we are having the following things for free — 5 GB of Standard Storage, 20,000 Get Requests, 2,000 Put Requests. When exceeded, a standard taxes will be applied.

After you have completed the registration, go to **S3 **thru the interface or just by clicking — [https://s3.console.aws.amazon.com/s3](https://s3.console.aws.amazon.com/s3)

We will create a bucket(it’s like a folder) where the bundled files will be uploaded and stored.

Click the Create button:

![](https://cdn-images-1.medium.com/max/2730/1*vfNI4hiHnXgi9VT-ZTdemA.jpeg)

A configuration window will pop up. We have to fill the bucket name who is a unique identifier in the whole S3 service. If you choose something too generic it’s possible to be already taken.

Important thing to mention is that if you decide to go with custom domain(which is not covered in this article), you have to name your bucket like the domain name — for domain example.com, bucket name — example.com. More information about custom domain and S3 can be found on [AWS docs](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html).

Choose your bucket name carefully, because S3 will expose a url for your bucket in the following format:

[BUCKET-NAME].s3-website.[BUCKET-ZONE].amazonaws.com

The other required thing is to choose the bucket zone. When you are targeting mainly people from Europe, of course you will choose a zone in Europe and so on. Why ? Because of the performance — closer to the availability zone, faster loading for your users.

![](https://cdn-images-1.medium.com/max/2174/1*0GoaWygXKlotpV0iAiCeVg.jpeg)

When you are ready, click “Create”.

Then you will be redirected to the list of your buckets. Click on your newly created bucket:

![](https://cdn-images-1.medium.com/max/2726/1*PWM4-DRcKy-lcmrWDj2T-A.jpeg)

Click “Properties” and then the “Static website hosting” card.

![](https://cdn-images-1.medium.com/max/2726/1*vxc1Sga0irpBn_jK1d8gXg.jpeg)

We will see a three options available there, choose the “Use this bucket to host a website”.

Fill index.html for both Index and Error pages and click “Save”.

![](https://cdn-images-1.medium.com/max/2000/1*C3Fx6eoUKA9yFu-KG9i_HA.jpeg)

By enabling this, we are telling AWS: ‘Hey, I will use this bucket to host my site, capish ?” 🙂

Okay, so far so good.

However, by default all AWS S3 buckets are private. What does that mean is that if we deploy our application now, it will be inaccessible, i.e. 403 Unauthorized error will appear.

We can manage our bucket privacy by the “Bucket Policies” tab. In order to apply a policy that will grant all users Read access, we have to do a quick extra step before.

![](https://cdn-images-1.medium.com/max/2730/1*IdwgPCHXdTGaniDqwlQdEA.jpeg)

Go to “Permissions” and you will see the Public Access settings for your bucket. Currently both “Block new public bucket policies” and “Block public and cross-account access if bucket has public policies” are set to true. We have to change them to false, otherwise it will block us to apply the so called “public” policies.

Click “Edit” and uncheck them.

![](https://cdn-images-1.medium.com/max/2726/1*RCGxs42dR3GrsgmnjJcl4Q.jpeg)

Now, we can apply the policy that will allow anonymous to access our data or in AWS language “GetObject”. All the policies are in JSON format, we will use the following example provided by AWS [here](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-2)

    {
      "Version":"2012-10-17",
      "Statement":[
        {
          "Sid":"AddPerm",
          "Effect":"Allow",
          "Principal": "*",
          "Action":["s3:GetObject"],
          "Resource":["arn:aws:s3:::YOUR-BUCKET/*"]
        }
      ]
    }

Of course you will have to change **YOUR-BUCKET** to your bucket name. Paste it to the bucket policy editor and click “Save”.

![](https://cdn-images-1.medium.com/max/2730/1*Vij3dwNNFzwxtWqGF6NApQ.jpeg)

It’s important to allow only **GetObject **to the users, not PUT,DELETE and etc. and it could cause security problems.

Well, basically we are done with the S3 setup, now we can upload the build from the previous step.

## Deploy the build

Go to the Overview page of your bucket and click “Upload”

![](https://cdn-images-1.medium.com/max/2730/1*8CEYW0Xc8H_fmm_N_91nTA.jpeg)

Drag and drop the files created by the **ng build** command before. They are located in your dist folder.

![](https://cdn-images-1.medium.com/max/2726/1*alLQ1dumVPXV3SodKu05Hg.jpeg)

Click Upload.

When the files are successfully uploaded you will be able to see your application by navigate to the following URL(change the name and zone with yours):

[YOUR-BUCKET-NAME.s3-website-YOUR-BUCKET-ZONE.amazonaws.com](http://your-bucket-name.s3-website-your-bucket-zone.amazonaws.com/)

If you have followed all the steps — CONGRATS! This is the minimal setup to see your application online. Now, we will cover a few more steps how to optimize the things a little bit.

## Automate the deployment using shell script

Prerequisite for this step is to install and configure the AWS CLI on your machine. Check [this guide](http://%20https:/docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) by Amazon for detailed information how to do this.

To automate the deploy, we can create a simple shell script.

Create a deploy.sh file in root project directory and add the following content:

    #!/bin/bash 

    ng build --prod --aot 
    aws s3 cp ./dist s3://first-class-js-angular-example --recursive

It’s a simple script which will build the project and then deploy the bundle from dist folder to the S3. As I already said, you have to properly install and configure AWS CLI.

From now on, every time you want to deploy your app the only thing you have to do is run this script.

## Configure CloudFront to improve our app performance and reduce cost

Adding CloudFront as a CDN(content delivery network) for a S3 bucket is considered as a good practice. You can learn more about it on the AWS site, but I will try to do a quick overview how it will help us and why we need it.

Our current situation, when a bucket is directly accessed by the users is like this:

![](https://cdn-images-1.medium.com/max/2048/1*ZGgIbgIkNnCpHeg8RznnFw.jpeg)

Every bucket is storing the content in a one, specific zone. In the example above this is somewhere in Europe. When a user located in US tries to access our application(the bucket), the files will have to be fetched from the Europe location which can add up an additional delay.

In order to improve the performance, we will setup CloudFront and instead of receiving content directly from Europe, a request will be send to the closest to the user Edge Location.

Like this:

![](https://cdn-images-1.medium.com/max/2048/1*wcuwecfBuP0ofIAvvvOB0Q.jpeg)

An additional advantage of using CloudFront is the possibility to reduce our bills. Caching the content in this Edge Locations means that the load to our S3 bucket will be significantly reduced. There is also no data transfer fee between S3 and CloudFront. So you are only paying for what is delivered by CloudFront, plus the request fee.

![](https://cdn-images-1.medium.com/max/2000/1*f0d8wrZrCfoa6Kz3a3BXhw.png)

Configuring CloudFront is not a rocket science.

Go to [https://console.aws.amazon.com/cloudfront/](https://console.aws.amazon.com/cloudfront/) and click “Create Distribution”.

Select delivery method “Web” and click “Get started”.

For Origin Domain Name fill your website endpoint(S3 url), Origin ID can be a whatever you want and change Viewer Protocol Policy to be “Redirect HTTP to HTTPS”.

![](https://cdn-images-1.medium.com/max/2730/1*aMi2zSSh8KrXs5Tl3uXRug.jpeg)

That are the minimum steps to get started with CloudFront. When you want to change something in future, you can do it. Extra steps are required if you want to customize the things or configuring a custom domain.

Click “Create Distribution”.

It will take like a 10–15 minutes for your distribution to be created.

The domain name column contains your CloudFront url, which will be something like this -[RANDOMCHARS].cloudfront.net

If you try to open it in the browser, you should successfully see your application.

Congrats, you are now using the power of CloudFront and your application is more optimized than before.

I hope you enjoyed it!

*Originally published at [firstclassjs.com](https://firstclassjs.com/deploy-angular-application-to-aws-s3-and-cloudfront/) on March 19, 2019.*
