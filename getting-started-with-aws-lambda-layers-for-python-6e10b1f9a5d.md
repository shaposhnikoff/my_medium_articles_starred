
# Getting started with AWS Lambda Layers for Python

You want to avoid errors that can occur when you install and package dependencies with your function code? You want to keep your deployment package smaller and cleaner? Right, me too!

![](https://cdn-images-1.medium.com/max/2560/1*xCDpL1Zektj2j3pUkBzeeQ.jpeg)

Say hello to Lambda Layers! With Lambda Layers, you can configure your Lambda function to import additional code without including it in your deployment package.

Let me explain — A Layer is a ZIP archive that contains libraries and other dependencies that you can import at runtime for your lambda functions to use. It is especially useful if you have several AWS Lambda functions that use the same set of functions or libraries — you know, code reuse :-)

Since I build most of my experiments in python, I want to show you a small example of using Layers in Python :-)

Before we get started, it is very important to understand that when a Layer ZIP archive is loaded into AWS Lambda, it is unzipped to the /opt folder. For your Python lambda function to import the libraries contained in the Layer, the libraries should be placed under the python sub-directory of the /opt folder. For other supported runtimes, check [here](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html#configuration-layers-path).

### Let’s get started!

First, let’s create a file called ***custom_func.py*** and write a dummy function in it that will print out “Hello from the deep layers!!” message.

    def cust_fun():                           
        print("Hello from the deep layers!!")
        return 1

I store that ***custom_func.py*** file in a directory called ***python***. So, for my example, I have a directory structure like this:

![](https://cdn-images-1.medium.com/max/2000/1*9e50WP0f5WxJGvif39Ie_A.png)

As mentioned earlier, with Lambda Layers, you put the common components in a ZIP file and upload it as a Lambda Layer. So, let’s ZIP the folder with the file ***custom_func.py ***in it.

![](https://cdn-images-1.medium.com/max/2368/1*Qiuse6hx5V2yEtvOZIf1AQ.png)

The output of that ZIP command creates a file called ***python_libs.zip ***which we can upload to Lambda Layers. To do that, get into the [AWS Lambda Console](https://console.aws.amazon.com/lambda/home) and click create layer as below.

![](https://cdn-images-1.medium.com/max/2560/1*B0X_6mWFKluh6S0pSSHB0w.png)

To create a new layer, you have to give it a name, write a short description (optional), select the ***ZIP ***file to upload and finally select the runtime for your layer. I call my layer ***CustomFunction***, add a short description, select the ZIP file created above — ***python_libs.zip,* **and select ***Python 2.7*** as the runtime for our Layer.

![](https://cdn-images-1.medium.com/max/2560/1*TcRGZP9Lc_MxJ861DECUcA.png)

Once you click create, you should see a confirmation message ***“Congratulations! Your layer “CustomFunction” version “1” has been successfully published”. ***The new layer is now available to Lambda functions.

![](https://cdn-images-1.medium.com/max/2560/1*BmQoY7nCmikYyLnRgRgxxw.png)

To test that newly created layer, author a small lambda function from scratch, give it a name, e.g ***LambdawithLayer***, select the runtime, for our example I select ***Python 2.7***, and select the existing role ***lambda_basic_execution***. Click ***Create function***.

![](https://cdn-images-1.medium.com/max/2560/1*mLMOZXjUQXMmqUg-YfZXhw.png)

Once the function is created, replace the generic function code with the one bellow.

    import custom_func as cf

    def lambda_handler(event, context):
        cf.cust_fun()
        return {
            'statusCode': 200,
            'body': 'Hello from Lambda Layers!'
        }

![](https://cdn-images-1.medium.com/max/2042/1*Izf_rQt6Y678cvSzR4jlrA.png)

Notice that you can easily import the custom function from the layer using import custom_func as cf. That is because Lambda runtimes include paths in the /opt directory to ensure that your function code has access to libraries that are included in layers — and for Python 2.7, the full path is /opt/python . For Python 3.6 and 3.7, that full path is /opt/python/lib/python3.6/site-packages and /opt/python/lib/python3.7/site-packages respectively.

Before testing the Lambda function, you have to configure it to use our layer ***CustomFunction. ***Click on ***Layers*** has shown bellow.

![](https://cdn-images-1.medium.com/max/2586/1*VrCVYUVC6ElIM16SEB4p9Q.png)

Click ***Add Layer.***

![](https://cdn-images-1.medium.com/max/2586/1*gpKoEvfXp9JwAqIe78R5Dw.png)

Select the Layer ***CustomFunction ***and the version of choice. Click ***Add***.

![](https://cdn-images-1.medium.com/max/2616/1*ZsG23oZk1tj7gdMdNTmNLg.png)

Save the newly created layer configuration to your lambda function by clicking ***Save*** as shown bellow.

![](https://cdn-images-1.medium.com/max/2220/1*hHFr2WJyjF0xJVThDq7O_g.png)

Use the default test event and click ***Test***. You should see the execution result as bellow.

![](https://cdn-images-1.medium.com/max/3244/1*B_HhpkqToFklJnvN0GBleQ.png)

Notice the print resulting from the custom function marked with the arrow above? ***“Hello from the deep layers!!”*** — Well done, it worked!

That’s it — I just wanted to give you a short example of using Lambda Layers with Python. Of course, all the steps above can be replicated using the AWS CLI and frameworks like [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-template.html#serverless-sam-template-layerversion) and the [Serverless framework](https://serverless.com/framework/docs/providers/aws/guide/layers/).

To learn more about Lambda Layers, check the documentations [here](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html).

Finally, to help you get started, there is also a Github repository that references popular Layers: [https://github.com/mthenw/awesome-layers](https://github.com/mthenw/awesome-layers). Submit yours!

In my next post, I will use Lambda Layers to deploy small latency monkeys to AWS Lambda functions. Stay tuned!

-Adrian
