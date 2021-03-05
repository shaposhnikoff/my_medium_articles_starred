
# AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 4 “Creating a Contact…

In the latest tutorial, we learned how to create the integration between AWS Lambda and AWS SES using the Serverless Framework. In this example, we are going to invoke the same Lambda function from a static HTML page that we host on S3.

### Disclaimer

This content is part of / inspired by one of our online courses/training. We are offering up to 80% OFF on these materials, during the **Black Friday 2019**.

You can receive your discount [here](http://bf.eralabs.io).

![](https://cdn-images-1.medium.com/max/2000/1*B0qBLWIa0zP-zJcrenZN8w.png)

This is a series of blog posts about using AWS Lambda with the Serverless Framework. You can check previous similar blog posts like:
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 1 “Hello World”**
*I am creating a series of blog posts to help you develop, deploy and run (mostly) Python applications on AWS Lambda…*hackernoon.com](https://hackernoon.com/aws-lambda-serverless-framework-python-part-1-a-step-by-step-hello-world-4182202aba4a)
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 2 “Using AWS KMS with…**
*Using serverless technologies is becoming more and more mainstream. Serverless may make your life easier in several…*medium.com](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-2-using-aws-kms-with-9bdad3381024)
[**AWS Lambda + Serverless Framework + Python — A Step By Step Tutorial — Part 3 “Sending Emails from…**
*One of the good things about AWS Lambda is that it integrates easily with many AWS services like AWS SES (Simple Email…*medium.com](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-3-sending-emails-from-ad4119abca3c)

You can also find my other articles about the same topic but using other frameworks like [Creating a Serverless Uptime Monitor & Getting Alerted by SMS — Lambda, Zappa & Python](https://hackernoon.com/creating-a-serverless-uptime-monitor-getting-alerted-by-sms-lambda-zappa-python-flask-15c5fb31027) or [Creating a Serverless Python API Using AWS Lambda & Chalice](https://hackernoon.com/creating-a-serverless-python-api-using-aws-lambda-chalice-d321dc43ce2)

Don’t forget to subscribe to [Shipped: An Independent Newsletter Focused On Serverless, Containers, FaaS & Other Interesting Stuff](http://joinshipped.com/) and take a look at [Practical AWS, a training concerned with the actual use of AWS rather than with theory & ideas](https://www.practicalaws.com/).

Sometimes you don’t need to create a dynamic website. If you are creating a page for your product or service, you don’t need more than a few pages static pages.

Using static websites has many advantages:

* Cheaper

* Faster deployment

* Easier integration with CDNs

* Easier caching

* More secure

* No maintenance

* No servers

* Faster than dynamic websites.

Even if you need a dynamic element in your website as a contact form, you can keep your static website and develop a serverless function to execute the dynamic code.

In this tutorial, we are going to see how to add a contact form to a static website hosted on S3.

You can use other hosting providers than S3.

![](https://cdn-images-1.medium.com/max/4000/0*NQxpQ-X_5TB3s7p9.png)

## Prerequisites

As mentioned in the previous tutorial you should create a function that sends SES emails based on the data from a POST request then deploy it.

Let’s create a new Serverless Python project:

    mkdir contact-form
    cd contact-form
    virtualenv -p python3 venv
    . venv/bin/activate
    mkdir app
    cd app

In the code above, we created a Python 3 virtual environment and activated it. Under the app folder, generate a new Serverless boilerplate using:

    serverless create --template aws-python3 --name contact-form

Output:

    Serverless: Generating boilerplate...
     _______                             __
    |   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
    |   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
    |____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
    |   |   |             The Serverless Application Framework
    |       |                           serverless.com, v1.32.0
     -------'

    Serverless: Successfully generated boilerplate for template: "aws-python3"

We are going to use almost the same code of the previous project “using-ses”.

This is the content of serverless.yml:

    service: contact-form

    provider:
      name: aws
      runtime: python3.6
      stage: dev
      region: 'eu-west-1'
      iamRoleStatements:
        - Effect: Allow
          Action:
            - ses:SendEmail
            - ses:SendRawEmail
          Resource: "*"        
      environment:
        REGION_NAME: 'eu-west-1'  
        ACCESS_KEY: 'xxxxxxxxxx'  
        SECRET_KEY: 'xxxxxxxxxxxx/xxxxxxxxx+xxxxxxxxxxx+xxxxxx'

    functions:
      sendEmail:
        handler: handler.sendEmail
        description: This function will send an email
        events:
          - http:
              path: send-email
              method: post
              integration: lambda
              cors: true
              response:
                headers:
                  "Access-Control-Allow_Origin": "'*'"

And this is the handler.py:

    import json
    import boto3
    import os

    region_name = os.environ['REGION_NAME']
    aws_access_key_id = os.environ['SECRET_KEY']
    aws_secret_access_key = os.environ['SECRET_KEY']

    def sendEmail(event, context):
        data = event['body']

    name = data ['name']    
        source = data['source']    
        subject = data['subject']
        message = data['message']    
        destination = data['destination']

    _message = "Message from: " + name + "\nEmail: " + source + "\nMessage content: " + message    
        
        client = boto3.client('ses' )    
            
        response = client.send_email(
            Destination={
                'ToAddresses': [destination]
                },
            Message={
                'Body': {
                    'Text': {
                        'Charset': 'UTF-8',
                        'Data': _message,
                    },
                },
                'Subject': {
                    'Charset': 'UTF-8',
                    'Data': subject,
                },
            },
            Source=source,

    )
        return _message + str(region_name)

The next step is to deploy the function.

    serverless deploy

Output:

    Serverless: Packaging service...
    Serverless: Excluding development dependencies...
    Serverless: Creating Stack...
    Serverless: Checking Stack create progress...
    .....
    Serverless: Stack create finished...
    Serverless: Uploading CloudFormation file to S3...
    Serverless: Uploading artifacts...
    Serverless: Uploading service .zip file to S3 (790.59 KB)...
    Serverless: Validating template...
    Serverless: Updating Stack...
    Serverless: Checking Stack update progress...
    .................................
    Serverless: Stack update finished...
    Service Information
    service: contact-form
    stage: dev
    region: eu-west-1
    stack: contact-form-dev
    api keys:
      None
    endpoints:
      POST - [https://lhuc1c96wi.execute-api.eu-west-1.amazonaws.com/dev/send-email](https://lhuc1c96wi.execute-api.eu-west-1.amazonaws.com/dev/send-email)
    functions:
      sendEmail: contact-form-dev-sendEmail

Let’s test it using a simple CURL:

    curl -X POST -d "[name=aymen&source=hello@aymenelamri.com](mailto:name=aymen&source=hello@aymenelamri.com)&subject='This is the subject'&message='This is the [subject'&destination=aymen@eralabs.io](mailto:subject'&destination=aymen@eralabs.io)" [https://lhuc1c96wi.execute-api.eu-west-1.amazonaws.com/dev/send-email](https://lhuc1c96wi.execute-api.eu-west-1.amazonaws.com/dev/send-email)

If you are more comfortable with graphical tools, you can try a tool like Postman.

![](https://cdn-images-1.medium.com/max/3198/1*_gn_5pXVqyYkJdJPL3JZFg.png)

## Creating a Contact Form

Let’s now start by creating a simple form to test if we can call the serverless function from a static HTML.

This is the content of our index.html:

    <form action="[https://lhuc1c96wi.execute-api.eu-west-1.amazonaws.com/dev/send-email](https://lhuc1c96wi.execute-api.eu-west-1.amazonaws.com/dev/send-email)" method="POST">
      <input type="text" placeholder="Your First Name" name="name">
      <input type="email"placeholder="Enter email" name="source">
      <input type="text" placeholder="Subject" name="subject">
      <textarea rows="5" name="message"></textarea>
      <input type="hidden" name="destination" value="[aymen@eralabs.io](mailto:aymen@eralabs.io)">
      <input type="submit" value="Send">
    </form>

(Note that destination is a hidden input.)

![](https://cdn-images-1.medium.com/max/2000/1*kS089Uno7Zp6wFC6y0Ht2A.png)

As you can see when we click on the submit form, we will send:

* name: aymen

* source: [hello@aymenelamri.com](mailto:hello@aymenelamri.com)

* subject: ’This is the subject’

* message: ’This is the message’

* destination: [aymen@eralabs.io](mailto:aymen@eralabs.io)

Let’s make this form look better using Bootstrap:

    <!DOCTYPE html>
    <html>
    <head>
     <title>A Simple Form</title>
     <link rel="stylesheet" href="[https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css](https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css)" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
     <script src="[https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js](https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js)" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
    </head>
    <body>
    <div class="container-fluid">
     <div class="row">
      <div class="col-md-12">
       <h3>
        <strong>Contact Form</strong> (Serverless)
       </h3>
       <p>
          Please use this form to send us an email. We will reply in less than 24h.
       </p>
       <div class="col-md-6">
        <form  action="[https://gb9tjzbm19.execute-api.eu-west-1.amazonaws.com/dev/send-email](https://gb9tjzbm19.execute-api.eu-west-1.amazonaws.com/dev/send-email)" method="POST">

    <div class="form-group">
          <label for="inputName">Name</label>
          <input type="text" class="form-control"  placeholder="Your First Name" name="name">
         </div>

    <div class="form-group">
          <label for="inputEmail1">Email address</label>
          <input type="email" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp" placeholder="Enter email" name="source">
          <small id="emailHelp" class="form-text text-muted">We'll never share your email with anyone else.</small>
         </div>

    <div class="form-group">
          <label for="inputSubject">Subject</label>
          <input type="text" class="form-control"  placeholder="Subject" name="subject">
         </div>

    <div class="form-group">
          <label for="inputMessage">Message</label>
          <textarea class="form-control" id="inputMessage" rows="5" name="message"></textarea>
         </div>
         <input type="hidden" name="destination" value="[aymen@eralabs.io](mailto:aymen@eralabs.io)">
         <input type="submit" value="Send"  class="btn btn-primary btn-lg btn-block">

    </form>
       </div>
      </div>
     </div>
    </div>

    </body>
    </html>

As you can see we have a better form here:

![](https://cdn-images-1.medium.com/max/2046/1*HWTRb7GoYpR_NgT1EkP0aw.png)

When you click on the send button you will be redirected to the URL of the function and your email will be sent.

![](https://cdn-images-1.medium.com/max/2000/1*eaAKjwh7sA9kiXLhyg2w5g.png)

### Using S3 to Host the Contact Form

As said in the introduction of this course, we are going to host the web form in a S3 bucket.

Let’s create a bucket with a unique name.

If I choose the name *contact-form*, it’s sure that it’s already take, that’s why I added the MD5SUM of the date.

    aws s3 mb s3://contact-form-$(date|md5sum|awk '{print $1}')

My bucket will have the name:

    contact-form-6121a115c2d97cf7bbef522d296e90f3

Let’s also create a policy.json file:

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": [
                    "s3:GetObject"
                ],
                "Resource": [
                    "arn:aws:s3:::contact-form-6121a115c2d97cf7bbef522d296e90f3/*"
                ]
            }
        ]
    }

Then a website.json configuration:

    {
      "IndexDocument": {
        "Suffix": "index.html"
      },
      "ErrorDocument": {
        "Key": "404.html"
      },
      "RoutingRules": [
        {
          "Redirect": {
            "ReplaceKeyWith": "index.html"
          },
          "Condition": {
            "KeyPrefixEquals": "/"
          }
        }
      ]
    }

You don’t need to create all of this manually if you are using the AWS console but here we are doing things the hard way!

The next step is applying these two configurations (policy.json and website.json):

    bucket=contact-form-6121a115c2d97cf7bbef522d296e90f3

    aws s3api put-bucket-policy --bucket $bucket --policy file://policy.json

    aws s3api put-bucket-website --bucket $bucket --website-configuration file://website.json

We are done! (We can even remove these files from our local folder if we don’t need them anymore).

Let’s synchronize our local folder with the remote bucket:

    aws s3 sync .  s3://$bucket --acl public-read --delete

If you want to get the address of the live page, you can use this form:

    [https://s3-$region.amazonaws.com/$bucket/](https://s3-eu-west-1.amazonaws.com/contact-form-6121a115c2d97cf7bbef522d296e90f3/index.html)

This is a fast way to get the URL of the bucket:

    region=eu-west-1

    bucket=contact-form-6121a115c2d97cf7bbef522d296e90f3

    echo "[https://s3-$region.amazonaws.com/$bucket/](https://s3-eu-west-1.amazonaws.com/contact-form-6121a115c2d97cf7bbef522d296e90f3/index.html)"

And this is the result (I added the index.html at the end):

    [https://s3-eu-west-1.amazonaws.com/contact-form-6121a115c2d97cf7bbef522d296e90f3/index.html](https://s3-eu-west-1.amazonaws.com/contact-form-6121a115c2d97cf7bbef522d296e90f3/index.html)

It’s optional but we can also create a 404 page:

    echo "Ahoy! This page doesn't even exist!" > 404.html

Then we need to re-synchronize our local folder with the bucket:

    aws s3 sync .  s3://$bucket --acl public-read --delete

This is our online form hosted in a static S3 website bucket:

![](https://cdn-images-1.medium.com/max/2046/1*HWTRb7GoYpR_NgT1EkP0aw.png)

### Deleting the Function & the Bucket

Using sls remove command, we can delete the deployed service from AWS Lambda.

    serverless remove

Using aws s3 rb command, we can delete the deployed website from AWS S3.

    aws s3 rb s3://$bucket --force

## Connect Deeper

In [the first part of this tutorial,](https://hackernoon.com/aws-lambda-serverless-framework-python-part-1-a-step-by-step-hello-world-4182202aba4a) we have seen how to deploy our first Lambda function using the Serverless Framework.

The part 2, “[Using AWS KMS with Lambda to Store & Read Sensible Data & Secrets](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-2-using-aws-kms-with-9bdad3381024)”, we used AWS Lambda to store and use secrets and sensitive data.

In the part 3, we saw how we can[ integrate AWS Lambda with other services like SES and send emails](https://medium.com/@eon01/aws-lambda-serverless-framework-python-a-step-by-step-tutorial-part-3-sending-emails-from-ad4119abca3c). This can be done using Boto3 and Serverless Framework.

In the part 4, we created a contact form for a static website using AWS Lambda + SES + S3.

Stay in touch as the upcoming posts will go more in depth.

I am creating a series of blog posts to help you develop, deploy and run (mostly) Python applications on AWS Lambda using Serverless Framework.

You can find my other articles about the same topic but using other frameworks like [Creating a Serverless Uptime Monitor & Getting Alerted by SMS — Lambda, Zappa & Python](https://hackernoon.com/creating-a-serverless-uptime-monitor-getting-alerted-by-sms-lambda-zappa-python-flask-15c5fb31027) or [Creating a Serverless Python API Using AWS Lambda & Chalice](https://hackernoon.com/creating-a-serverless-python-api-using-aws-lambda-chalice-d321dc43ce2)

Don’t forget to subscribe to [Shipped: An Independent Newsletter Focused On Serverless, Containers, FaaS & Other Interesting Stuff](http://joinshipped.com/).

![](https://cdn-images-1.medium.com/max/3750/1*lpdjzNxQBaz1uFMNDZkSHA.png)

You may be interested in learning more about Lambda and other AWS service, so please take a look at [Practical AWS, a training concerned with the actual use of AWS rather than with theory & ideas](https://www.practicalaws.com/).

![](https://cdn-images-1.medium.com/max/2600/0*Kju8Dog2kjhAjAqG.png)

![](https://cdn-images-1.medium.com/max/2000/0*EfwXLEcMVhmxvCYg)

**Join our community Slack and read our weekly Faun topics ⬇**
[**Join a Community of Aspiring Developers.Get must-read articles, learn new technologies for free…**
*Join thousands of developers and IT experts, get must-read articles, chat with like-minded people, get job offers and…*www.faun.dev](https://www.faun.dev/join/?utm_source=medium.com%2Ffaun&utm_medium=medium&utm_campaign=faunmedium)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
