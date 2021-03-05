
# How to test a Python AWS Lambda function locally with PyCharm Run Configurations

I was developing an Alexa skill using Python in AWS Lambda and wanted to make my local development workflow as seamless as possible. One thing I wanted to do was to test the lambda locally right from PyCharm rather than using terminal or the Lambda console. It wasn’t as easy and required quite a bit of research and trial-and-error.
> Disclaimer: I am a Sr. Product Manager for AWS but this post is not endorsed by AWS in any way.

### Problem statement

* I have an AWS Lambda function in Python 3.6

* I use PyCharm Community 2017.1

* I want to test the code locally before pushing it to AWS Lambda by simply pressing *Ctrl-R* in PyCharm to speed up development time

Here are the implementation steps.

### 1. Running Lambda locally

First, you need to be able to run Lambda locally. For this purpose you can use the [python-lambda-local](https://pypi.python.org/pypi/python-lambda-local) package, which supports Python 2.7 and 3.6.

**Install the package** by running
> pip install python-lambda-local

**You can test if it works** by going to your project directory and running
> python-lambda-local -f lambda_handler lambda_function.py event.json

where

* *lambda_handler* is the name of your handler function

* *lambda_function.py* is the name of your file with Python code

* *event.json* is the test event data

The script will provide the exact same output as the test feature in the Lambda console.

### 2. Enable PyCharm to run bash scripts

Now we want to run python-lambda-local from PyCharm rather than the terminal. Unfortunately, PyCharm 2017.1 doesn’t support running bash scripts out of the box, but luckily there’s a plugin called [BashSupport](https://www.plugin-dev.com/project/bashsupport/).

Install the plugin by going to
> PyCharm main menu > Preferences > Plugins (left pane) > Browse Repositories…

and search for *BashSupport*.

![](https://cdn-images-1.medium.com/max/4424/1*bSC5DVqNkqnAPhJAswmBCg.png)

Click *Install* and restart PyCharm.

### 3. Create a bash script

Create a file named *test.sh* (or however you want it named) with the following contents:
> #!/bin/bash
echo “Testing lambda”
python-lambda-local -f $1 $2 $3

$1, $2 and $3 are command line arguments to make the script generic.

### 4. Edit Run Configurations in PyCharm

The last step is to make PyCharm run your bash script instead of the default Python interpreter.

In PyCharm, go to *Run > Edit Configurations…*

![](https://cdn-images-1.medium.com/max/2000/1*_gTPbiuxHPs1BGaoKpiktQ.png)

Click on the + sign at the top and choose *Bash*.

![](https://cdn-images-1.medium.com/max/2000/1*DmYdCSu_gIUvFjgxybXbsQ.png)

* **Name: **type “python-lambda-local”

* **Script:** type or select the path to test.sh

* **Program arguments:** type “lambda_handler lambda_function.py event.json” or whatever your handler function, Python file name and test data file name are.

* **Working directory:** type or select the project directory

![](https://cdn-images-1.medium.com/max/3828/1*PbswgLQPcNWDhlGWyhb4xA.png)

Click *OK.*

### 5. Enjoy!

Now go to *Run > Run ‘python-lambda-local’.*

![](https://cdn-images-1.medium.com/max/2000/1*OZAgOqdOddfv0G04aGpPfw.png)

PyCharm will run the script and output the result in the bottom pane.

![](https://cdn-images-1.medium.com/max/5032/1*nmfV2r8eR2zbFTWAFevcDw.png)

### If this was helpful, hit the ❤️ and follow me. I tend to write about tech and music with occasional philosophical ramblings.
