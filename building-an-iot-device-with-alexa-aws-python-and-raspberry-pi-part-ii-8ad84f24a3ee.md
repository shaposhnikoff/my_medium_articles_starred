
# Building an IoT device with Alexa, AWS, Python and Raspberry Pi - Part II

Photo by Rahul Chakraborty on Unsplash

In [**Part I](https://medium.com/@altonelli/building-an-iot-device-with-alexa-aws-python-and-raspberry-pi-274d941ef3c3),** I reviewed how to set up the Alexa Skill interaction model. In this part we will be going over setting up AWS Lambda and IoT handle the logic from the Alexa. Much of this is management between different Amazon services. This can be done through the CLI but for the walk through I will be using the console and attaching images.

## Creating an IAM Role

Before we get too crazy, we will first need to create an IAM role for the Lambda function. If your unfamiliar with AWS, the policies allow interactions between different services. Our policy will allow the function to read and write to AWS IoT. First let’s visit the [IAM console](http://console.aws.amazon.com/iam/home).

Go to the “Roles” tab and select “Create role”. On the following page select “AWS service”, “Lambda” and then “Next: Permissions”.

![](https://cdn-images-1.medium.com/max/5368/1*4SZo3V1aTbgSIP58VGyz0Q.png)

![](https://cdn-images-1.medium.com/max/4300/1*T08I5TQc-jX7wckyQNsk0Q.png)

Search and select the policies “AWSIoTDataAccess” and “AWSLambdaBasicExecutionRole” to attach them to the role. This will allow us to read and write to the AWS IoT shadow and write logs via CloudWatch.

![](https://cdn-images-1.medium.com/max/4284/1*ATPjufee9g2W5OCvglMK8w.png)

Name the role (something like “lambda_iot_exectution”) and make sure we have those two policies attached. Finally, select “Create role” and we’ll use the role later when creating the Lambda function.

![](https://cdn-images-1.medium.com/max/4268/1*Q2_Irktv6AR820KcFng2cw.png)

## Creating an IoT Thing

Next we’ll create an AWS IoT **Thing**, which is the AWS representation of our smart device. The shadow is the state of a given Thing. We will be using an endpoint specific to the thing to subscribe and publish updates to the shadow via the Raspberry Pi and the Lambda function. Certificates and IAM roles are used to manage privacy.

First up, we’ll visit the [IoT console](http://console.aws.amazon.com/iot/home). Change the region to **US-East-1 (N. Virginia)**. At the time of this writing, this is the only US region for handling Alexa Skills and Lambda functions. Next we’ll go to “Manage”, “Things” and select “Register a thing”. On the following page select “Create a single thing”.

![](https://cdn-images-1.medium.com/max/4896/1*7AKaLXxHE1EhtLPfEn4HGQ.png)

![](https://cdn-images-1.medium.com/max/4144/1*NUg4eZ0NlCv431GaYAFzFQ.png)

Name the thing (my_painting) and select “Next”. Specifying a *type* of thing is not necessary. Next select “Create certificate”.

![](https://cdn-images-1.medium.com/max/3412/1*6YbiKBYjUScL7FZHrTNJCg.png)

![](https://cdn-images-1.medium.com/max/3348/1*dAQuZHKc3bTB9Hdj2mQixg.png)

**It is important to download all of these immediately.** The public and private keys CANNOT be downloaded after closing the page. The root CA will likely open a new page. Save it as a .pem file. Move all of these to a safe folder. We will later copy them onto the Pi. After select “Activate” and “Done”. We will create and attach a policy next.

![](https://cdn-images-1.medium.com/max/3084/1*IQ3TixUUZuUajQxLOsdKUQ.png)

Go to the “Secure” and “Policies” tab. Select “Create a policy”. Name the policy (my_painting_policy) and add the action “iot:*”, resource ARN “*” and check “Allow” for the effect. This will give all clients all IoT permissions on the Thing. If we were developing this beyond a simple tutorial we would further restrict this. Finally select “Create”.

![](https://cdn-images-1.medium.com/max/4788/1*x8JDctTXocjcRVSpA4aSZg.png)

![](https://cdn-images-1.medium.com/max/3568/1*0ZRhCPeqxy3G5nq6NEb4gQ.png)

This new policy now has to be applied to the certificate. Go to “Secure” and “Certificates” and select the certificate we just created. Go to “Policies”, “Actions” and “Attach policy”. Check the policy we just created and select “Attach”. The certificate should be activated, if not select the “Active/Inactive” button under the certificate name.

![](https://cdn-images-1.medium.com/max/2496/1*G3d0eHcwG6_VBoo2bgSL-Q.png)

![](https://cdn-images-1.medium.com/max/3500/1*lsQxR-Pk2Sh4_fLvJ5C0hw.png)

![](https://cdn-images-1.medium.com/max/2364/1*vj4c6yCdRGPMdhaD9_pmzg.png)

Now we have an IoT Thing! If you go back to the “Manage” and “Things” tab and select the new “my_painting” thing, you can see details about it, including the shadow and endpoints. For now, select “Interact”, note the HTTPS endpoint and add it to our .env file. We’ll be using that in the Lambda function.

![](https://cdn-images-1.medium.com/max/3128/1*4C8H5LkZqkl7LRx_gDSLWQ.png)

## Creating a Lambda Function

Finally, its time to add the logic to the Skill we made in [**Part I](https://medium.com/@altonelli/building-an-iot-device-with-alexa-aws-python-and-raspberry-pi-274d941ef3c3)**. Go to the [Lambda console](http://console.aws.amazon.com/lambda/home) and select “Create function”. Again, **make sure have the region set for US-East-1 (N. Virginia)**.

![](https://cdn-images-1.medium.com/max/5312/1*cflT8M7vAyG2ZVxDnyoUPA.png)

Select “Author from scratch” and name the function (my_painting). Select “Python 3.6”, “Choose an existing role”, the role we just made (lambda_iot_execution) and finally “Create function”. Once created, we will see AWS IoT and Amazon CloudWatch Logs in the resources the function has access to.

![](https://cdn-images-1.medium.com/max/5144/1*e7N3z6wlXLsSNFHAbRDkug.png)

Next, we have to connect our Alexa Skill as a trigger to our new function. Select “Alexa Skills Kit” in the list of triggers to the left, then highlight the new trigger. Select “Enable” Skill ID verification, then enter in the Skill ID from [**Part I](https://medium.com/@altonelli/building-an-iot-device-with-alexa-aws-python-and-raspberry-pi-274d941ef3c3)**. Select “Add” then “Save”.

![](https://cdn-images-1.medium.com/max/5060/1*iS7C9PXA4iTU_qLwkFXPPw.png)

Going back to the Lambda endpoint we discussed in [**Part I](https://medium.com/@altonelli/building-an-iot-device-with-alexa-aws-python-and-raspberry-pi-274d941ef3c3)**, its now time to add it! Copy the ARN in the upper right of the Lambda function page and go back to the [Alexa Skill Editor](https://developer.amazon.com/alexa/console/ask'). Go to the “Endpoint” tab, select “AWS Lambda ARN”, enter the ARN in the default region and select “Save Endpoints”.

![Back to the Alexa Skill Console](https://cdn-images-1.medium.com/max/5488/1*JBvyQg4iAuOdUBXwpgyTfQ.png)*Back to the Alexa Skill Console*

Finally everything is connected but we *still* do not have any logic in the actual function! This is as easy as uploading a ZIP file of part of the repo. First zip the file.

    $ cd ~/<path_to_repo>/lambda_function/
    $ zip -r lambda_function.zip .

In the Lambda console, select the “my_painting” Lambda function, “Upload a .ZIP file” for *Code entry type*, the ZIP file and “Upload”.

![](https://cdn-images-1.medium.com/max/4988/1*vYS__kK0nVYQOxI7RWPWwg.png)

We now need to enter the necessary environment variables, which should look like the following. Finally, select “Save” and we will see the section “Function code” populate to a Cloud9 IDE.

![](https://cdn-images-1.medium.com/max/5012/1*osZNhQlrSJkvVZ58e8AMoQ.png)

We are all set! We should be able to test the Skill and update the my_painting shadow.

## Testing

Go back to the Alexa console and select “Build model”. This will take several minutes. Afterwards to go “Test”, enable testing and enter a phrase for in the Alexa test console. Try “tell my painting to turn on the lights”.

![](https://cdn-images-1.medium.com/max/3160/1*K3s73hlMsErC7xakflKxOA.png)

![](https://cdn-images-1.medium.com/max/2000/1*s3r61yty_dO5ubLBfIZ-pA.png)

We will first see the immediate response from Alexa. If we now go back the the IoT console, select our “my_painting” thing and “Shadow” you will now see that the shadow has been updated!

![](https://cdn-images-1.medium.com/max/3348/1*T50rBdZvIEyW0fq_oji-mw.png)

## Review

Now we have a simple interaction with Alexa. Without the IoT section, we could have could implemented the Lambda function to perform a simple task and have a fully functioning Alexa Skill.

Later, I will take a deeper dive into some of the core sections of code but I will give a high level review of what we accomplished in this part. We have a Lambda function receiving and *event* from the Alexa Skill. In this event, we have information on the *session* and the *request*. We look at request for an *intent* and handle accordingly. This could be asking to turn on/off the lights, changing the brightness, or asking for help.

For intents that will change the state of our IoT Thing, we send an update to the IoT Thing endpoint and update the Thing shadow. The shadow has a state that is a dictionary including the *desired* state and the *reported* state. The *desired* state is updated with what we *want* the state of the thing to reflect. The *reported* state is the updated with the current state of the thing. If there is ever a difference, a *delta* attribute will be present with the difference. We will be updating both the desired and reported states from the Pi in [**Part III](https://medium.com/@altonelli/building-an-iot-device-with-alexa-aws-python-and-raspberry-pi-part-iii-c1d383aa88c5)** and generally want the desired state to reflect the reported state.

Regardless of the intent, we generate a response which Alexa will say to the user. All of these services are able to work together because of the roles and policies we put in place.

Now we can give a command to Alexa but we still need to have our Raspberry Pi listen for the update and update the lights. We will do this in [**Part III](https://medium.com/@altonelli/building-an-iot-device-with-alexa-aws-python-and-raspberry-pi-part-iii-c1d383aa88c5)** next!

Helpful? Enjoy? Additional questions? Feedback in the form of 👏👏 or comments below always welcome!

Or, feel free to say hi at [hello@arthurtonelli.me](http://hello@arthurtonelli.me)!
