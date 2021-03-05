
# Setup everything from scratch to develop Alexa skills using Python

Photo by Rahul Chakraborty on Unsplash

Alexa have been there since 2014 but still developing skills feels a little bit primitive. In this article, I try to explain how to set up everything so that it is easy to develop, debug, and deploy skills using Python. I needed to learn these stuff from many sources and there is no one good tutorial about how to set up everything. Until now. This is the important step you want to make at the beginning because it is awful to code when tools aren’t good or something in the process takes a ton of time.

This is mostly targeted for people with Ubuntu. It is the best OS if you are a developer but still many people use Windows. You can do all these things also in Windows but it might require some changes. One way is to install Ubuntu from the Microsoft store. It’s software in your Windows computer that makes it possible to use Ubuntu.

![](https://cdn-images-1.medium.com/max/2000/0*wO3Sobx4hbw-Pu1k.jpg)

This is also about Python but with small modifications, you can generalize these things to other languages too.

I hope that Amazon could create this kind of tutorials for Ubuntu, Windows, and many different languages because it feels like there is no good tutorial about how to set up everything. I also hope that Alexa team reads this and tries to make the process easier so you don’t need to read anything to get started. After all, developers make the platform. If creating skills is too complicated the best talents move to other platforms where there is less friction.

### Setting up Lambda

This is explained well in many tutorials but to get everything into one place here is the steps.

First, start by opening AWS and then go to the Lambda page.

![You should see something like this](https://cdn-images-1.medium.com/max/2184/1*nhjQ8BuOQ9lGewcOmOpnFw.png)*You should see something like this*

Then press that orange button to create a new function.

![Name your function and make sure runtime says Python 3.7 if you want to use it. Again this tutorial is targeted for Python.](https://cdn-images-1.medium.com/max/2226/1*ir53tQw8aQzsITev4-UD0w.png)*Name your function and make sure runtime says Python 3.7 if you want to use it. Again this tutorial is targeted for Python.*

![In this window press “Add trigger” button](https://cdn-images-1.medium.com/max/2180/1*wFigl2xxoLFxcAByAQ-06w.png)*In this window press “Add trigger” button*

![Choose Alexa Skills Kit](https://cdn-images-1.medium.com/max/2000/1*98f-E0iaUe41rXn1_YrUeQ.png)*Choose Alexa Skills Kit*

To get Skill ID go to Alexa Developer console: [https://developer.amazon.com/alexa/console/ask](https://developer.amazon.com/alexa/console/ask)

If you don’t have a skill just create one and then go to the project page.

![When creating a skill make sure you choose this option](https://cdn-images-1.medium.com/max/2000/1*Yi_cpsbZ1hSzXCbY1OgmAQ.png)*When creating a skill make sure you choose this option*

When in skill page press Build in the header and then “Customers” (this is often already pressed) in the left bar. Finally, press “Endpoint” under Customers.

Copy your skill id and paste it to the Lambda page. Press “Add” and now your Lambda Function should look like this:

![](https://cdn-images-1.medium.com/max/2154/1*WZAmk0wozRqK5yLGPFLvyA.png)

![Copy ARN from the right corner of your Lambda Function page to the endpoint page in Alexa Developer console. Make sure you add it to the “default region” space if you only have one Lambda Function (or you don’t know what having multiple means).](https://cdn-images-1.medium.com/max/2000/1*s13llWuIpQRkvdyDyfAkBg.png)*Copy ARN from the right corner of your Lambda Function page to the endpoint page in Alexa Developer console. Make sure you add it to the “default region” space if you only have one Lambda Function (or you don’t know what having multiple means).*

Now you have Lambda Function connected to the Skill.To test that everything works press name of your Lambda Function (right side of that orange icon you can see in the above image) and then scroll down to see cloud code editor. Copy code from the following URL to the file: [https://github.com/KeithGalli/Alexa-Python/blob/master/basic_template.py](https://github.com/KeithGalli/Alexa-Python/blob/master/basic_template.py)

Now when you go to Alexa Developer Console and test this skill you should get some message when you say “Alexa, open my_skill_invocation”. If it doesn’t work properly I recommend check one more time that you set up everything correctly. If everything is done correctly but it still doesn’t work then you might want to skip the next section and go to “debug” section where you learn to read logs of Lambda Function to see what is happening.

### Writing code locally

I hate those online editors Amazon offers. I always use VS code so next, we look how to code locally with our favorite text editors. It is cool that there is an online editor so you can use any device to write code but still I think most of the time we only use one device and it is better to make coding much easier with this device than suffer by using these poorly made editors that you can’t use in the plane or other places where you have no connection.

**This step if needed in case you want to pip install something!**

First, you start by creating a virtual environment. Open terminal in the directory where you want to save this project. Type virtualenv **name **to your terminal replacing **name** with anything you want. I recommend to create a folder for your project and then creating virtual environment called **alexa** inside that folder. You might need to install virtualenv so that’s just pip install virtualenv

Okay, now you have a virtual environment for the project. Next, go inside the folder that was just created and follow this path: lib/python3.7/site-packages/ Then create a new file called lambda_function.py This is where your main code goes. Copy the template you have in Lambda Function here.

Now you have code locally and you can modify it with the VS Code or alternative. After you have made modifications you want to send the code to Lambda Function. This is something that took some time to figure out because I wanted to do it as little steps as possible. Go to the root folder where you created the virtual environment and then create a file called deploy.sh and copy-paste this code:

    #!/bin/bash

    cd ./**NAME_OF_VIRTUAL_ENV_FOLDER**/lib/python3.7/site-packages/

    rm -r lambda_function.zip

    zip -r9 lambda_function.zip *

    aws lambda update-function-code --function-name **FUNCTION_NAME**--zip-file fileb://lambda_function.zip

Don’t run this yet because first, you need to change some things. Replace **NAME_OF_VIRTUAL_ENV_FOLDER** to match the folder name you created (I recommended to use **alexa**). Replace **FUNCTION_NAME** with the name of your function.

![This is where you can find your function name in case you already forgot it.](https://cdn-images-1.medium.com/max/2194/1*gYgof_W8iMIO7qrVNl7L7A.png)*This is where you can find your function name in case you already forgot it.*

You might not have aws installed to your computer so follow the next steps.

    sudo apt-get install awscli

Now you have it but the next thing is to configure it.

    aws configure

It asks four questions but let’s go through them one at a time.

AWS Access Key ID and AWS Secret Access Key are something you get by creating a user. Go to this URL: [https://console.aws.amazon.com/iam/home](https://console.aws.amazon.com/iam/home#/home)

![You should see something like this expect some details might be different.](https://cdn-images-1.medium.com/max/2464/1*rjBBUrfYvbiup3Eppv7_Ag.png)*You should see something like this expect some details might be different.*

Press Users:X below “IAM Resources” header.

![This is the next screen you see expect you might not have any users](https://cdn-images-1.medium.com/max/2440/1*NoYcRDKSfs50yUBAvnauLg.png)*This is the next screen you see expect you might not have any users*

Press blue button that says “Add user”.

![You can use any user name you want and then make sure you select “Programmatic access” option the same way in the picture.](https://cdn-images-1.medium.com/max/2000/1*lDEJVbLpxZ4-dCds4QhQgA.png)*You can use any user name you want and then make sure you select “Programmatic access” option the same way in the picture.*

Press “Next: Permission” at the bottom of the page.

![This is what you see next](https://cdn-images-1.medium.com/max/2054/1*WN6GcQSykqb12mcPeiYUCw.png)*This is what you see next*

Press the “Create group” button.

![Type AdministratorAccess to the search bar to see this](https://cdn-images-1.medium.com/max/2062/1*RcJCtLQm59-Rj3wccQHR6w.png)*Type AdministratorAccess to the search bar to see this*

I have no knowledge about this topic so I have no idea if this a big security risk to give “AdministratorAccess” to this user. I hope someone can comment about this and maybe suggest some other policy but at least this works. Select that (or some other policy if you know what you are doing) and then name that group by typing something to the box (it’s red in the above image). Finally, press “create group” button at the bottom.

It auto selects the group you just created so you can just press “Next: tags” button at the bottom of the page.

This is something you can skip by pressing “Next: Review” button at the bottom of the page.

![Make sure that everything is correct (names can be different) and then press “create user”](https://cdn-images-1.medium.com/max/2040/1*DCHK-N9_weJP7pw9y7pt8g.png)*Make sure that everything is correct (names can be different) and then press “create user”*

![Not sure is Access key secret but I still hide some of it to make sure anyone can’t cause harm](https://cdn-images-1.medium.com/max/2000/1*buoLvMyQIk0XeuI_BTLG4g.png)*Not sure is Access key secret but I still hide some of it to make sure anyone can’t cause harm*

So now copy Access key to the terminal and press enter. Then it asks secret access key you can also see here.

The third question is about the region. It’s your Lambda Function region that you can see in the right corner. Go to this address [http://www.elastichpc.org/doc/regions/index.html](http://www.elastichpc.org/doc/regions/index.html) to see the “Amazon name” for your region and type it to the terminal. The fourth question is something you can just skip by pressing enter.

Now AWS is configured and deploy.sh should work. You can run it by typing terminal bash deploy.sh and it should first make a zip of the files and then send it to Lambda. This means that every time you make some changes to the code just run bash deploy.sh to upload everything to Lambda function. Many people do these steps manually and I think when we talk about a task that we do this often it needs to be as easy as possible. I hope that Amazon could do something to simplify this progress. Setting up this takes way too much time and is way too complicated.

In case you want to pip install packages to the project you need to open the environment you created by running source ./name_of_environment/bin/activate in the root folder of your project. After that, you can just normally pip install anything.

### Debugging

![](https://cdn-images-1.medium.com/max/2000/0*xQb9pu37R_CbGJVb.jpg)

You can’t code if you can’t debug and still many tutorials skip this part. There are different ways to see what is happening but for me, the best way is to read logs and use print(). I explain how to do it and maybe later you can test other ways in case this doesn’t feel good.

I might be wrong about this but the quickest path I found to the logs is following:

Click the “Monitoring” tab in Lambda Function page.

![This is what you should see.](https://cdn-images-1.medium.com/max/2194/1*14xJHplCeKxJmk0k-x5GLA.png)*This is what you should see.*

Then press “View logs in CloudWatch”.

![If you haven’t done anything this might be empty](https://cdn-images-1.medium.com/max/2442/1*iXLQh4_58JFYL7ovG8kaqA.png)*If you haven’t done anything this might be empty*

Then you see a list of logs. Each log is matched to each event. So if you ask your Skill to do something it makes this kind of file that contains logs about that event.

![This is what your log might look](https://cdn-images-1.medium.com/max/2000/1*0c2dhugElGFZcOnoRrgsgw.png)*This is what your log might look*

What’s the idea of these logs? It’s like terminal when you run Python code. This is where your print functions put their output and this is where you can find errors. In my opinion, the easiest way to debug is to just print stuff. If you find some error you can read the message here (which often time tells you the problem) but in case you need to dig deeper just print different variables like people often do when debugging programs.

### Developing

Now everything is set up correctly so you can easily develop skills. The last thing I wanted to mention is what windows to keep open. First of all, have two windows (or tabs in one window) open from Alexa Developer console. Other is for **testing skills** (you can also use the real device and then you don’t need this) and other is for **creating new intents**. Obvious is to have a **code editor** where you have lambda_function.py file opened. Then I recommend having one **terminal where you can run bash deploy.sh** to push the code to the Lambda Function. And then, from the last part, keep your **CloudWatch** open so you can read the log whenever (almost every time you create something) things crash and cause errors. One thing I found useful is to have Python console open so I can run simple things to see how things work. For example just to make sure that I remember how to combine arrays correctly.

### Productive

[Productive](https://www.amazon.com/dp/B07TZWCHCS/) is the skill I created. The idea is to measure your time so you get a better understanding of where your time goes. I created this for need. Before this, I used stopwatch on my phone to see how long I work. It motivates me to stay focused and not watch Youtube when I know that I want to get as good time as possible (yeah, I’m a workaholic). Productive puts your time into one of three categories: work, free time, or other. This way end of the day you can understand better how much time you spent doing different things. Almost everyone doesn’t even realize that they don’t work the time they think but spend 2 hours commuting, eating, and doing other stuff. If you can’t measure it, you can’t improve it.

[Enable Productive for free](https://www.amazon.com/dp/B07TZWCHCS/)

**If you are interested of learning how to write code for Alexa I recommend to check links below:**

[https://www.youtube.com/watch?v=i8QCO0YWJRY&list=PLLlMDtI0KL5pK4-6N4hGK7_wwayF_naDG](https://www.youtube.com/watch?v=i8QCO0YWJRY&list=PLLlMDtI0KL5pK4-6N4hGK7_wwayF_naDG)

[https://www.youtube.com/watch?v=sj7NqS7yytw](https://www.youtube.com/watch?v=sj7NqS7yytw&t=12s)
[**Lankinen (@true_lankinen) | Twitter**
*The latest Tweets from Lankinen (@true_lankinen). Creator of TrimmedNews https://t.co/26GV8EvUHf*twitter.com](https://twitter.com/@true_lankinen)

~Lankinen
