
# How to measure temperature and send it to AWS IoT using a Raspberry Pi

What if you want to self-correct the temperature in your office? Or what if you are curious to understand your office environment using IoT sensors?

If this sounds interesting to you, please read on.

To begin with, we need to set up a temperature reading sensor. We connect it to an Arduino which connects to a RaspberryPi.

![](https://cdn-images-1.medium.com/max/6048/1*luZBAP5jAeQXIoKDlm2Lqg.jpeg)

The next step is to set up AWS IoT SDK on your Raspberry Pi.

### Setup the Thing

1. Create a thing in AWS IoT:

![](https://cdn-images-1.medium.com/max/2000/1*HMoK-8MpdziO3p1KgUu1WQ.png)

2. Create a single thing to begin with:

![](https://cdn-images-1.medium.com/max/2006/1*Zkpo9cALdjh6y_91NkYlgQ.png)

3. Create a thing of a particular type. We are using RaspberryPi here (the types are made up by you).

![](https://cdn-images-1.medium.com/max/2000/1*wpws9o3IaQd1QrhZerkgxw.png)

4.Create a certificate for your Thing to communicate with AWS:

![](https://cdn-images-1.medium.com/max/2000/1*kvhutJBA_nZMl4tCRYlPhw.png)

5. Download the certificates, a root certificate authority (CA), activate the Thing, and attach the policy.

![](https://cdn-images-1.medium.com/max/2000/1*IlIegdjeWhnlsCPsk2CR9A.png)

6. The policy code is here. It may seem a bit permissive, but it is OK for the demo App.

![](https://cdn-images-1.medium.com/max/2000/1*mY-mP_JRKku7qBxk-PkTVg.png)

### Setup your RaspberryPi

Before you start the setup, please copy all certificates and all root CA files over to the RaspberryPI (scp might help you). You also need to install Node.js if you don’t have it already.

You will also need to install the AWS IoT device SDK.

<iframe src="https://medium.com/media/0026b58d9cab6a91ca28ab42040d5508" frameborder=0></iframe>

Here is the code that reads the data from the serial port and sends temperature readings using the AWS IoT device SDK. The code is based on the examples from Amazon.

<iframe src="https://medium.com/media/8d88a780f9cc04e12c4d84c8f9c5ffe7" frameborder=0></iframe>

So now what can you do with that data?

You can write a Lambda that enqueues the data for processing. It may look like this:

<iframe src="https://medium.com/media/1b3f8704eb3411899b5a4ea7d12d40bb" frameborder=0></iframe>

And your serverless.com file may look like this:

<iframe src="https://medium.com/media/3ae28755720d089aecbe5f8c928117a4" frameborder=0></iframe>

I hope this post has saved you some time setting up your device. Thanks for reading!
