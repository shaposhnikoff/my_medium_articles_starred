
# A beginner’s guide to creating your first Python website using Flask, AWS EC2 & Elastic Beanstalk

In this guide I will show you step by step how to deploy a simple Python Flask application to an Amazon EC2 instance using Elastic Beanstalk. I assume you have a Mac.

[If you are new to software development, I recommend reading Section 1 of the glossary first, where I explain the key terms that I will use in this guide.](https://medium.com/@miloharper/key-terms-513399c90d26#.ipw4uzjdq)

Ok let’s go!

1. You will need a text editor to write the code in. You can use any text editor. In this tutorial, we are going to use Sublime. Install it for free from [http://www.sublimetext.com/](http://www.sublimetext.com/).

1. In this tutorial, we are going to use SourceTree so we can visually track the changes to our code. If you are experienced at using the git command line interface, you are welcome to use that instead. Otherwise, install SourceTree for free from [https://www.sourcetreeapp.com/](https://www.sourcetreeapp.com/)

1. Open the Terminal. It is installed by default on your Mac. You will find it in Finder > Applications > Utilities > Terminal.

![](https://cdn-images-1.medium.com/max/2728/1*tSffMOxtDiMoUUa_HAYlcg.png)

4. Check you have pip installed, by typing the command ‘pip --version’ into the Terminal , then press Enter.

    pip --version

If the Terminal responds with the version number, this means it is installed, so go to the next step. If the Terminal says “bash: command not found” this [means you need to install pip first](https://pip.pypa.io/en/latest/installing/).

![](https://cdn-images-1.medium.com/max/2728/1*Rok6A_Xo-H14zT3ec4ly9Q.png)

5. Install virtualenv, so we can create virtual environments, by running this command in the Terminal.

    sudo pip install virtualenv

You should see a message in the Terminal saying virtualenv has successfully installed.

![](https://cdn-images-1.medium.com/max/2728/1*R9piaR20bkWO_ywY4APQDg.png)

6. Now let’s use the *mkdir* command to create a folder for our project, called myflaskapp. In the Terminal run this command:

    mkdir myflaskapp

7. Let’s use SourceTree to turn this folder into a local repository, so we can keep track of our code. Open SourceTree and click ‘+New Repository’ then select ‘Create local repository’ from the dropdown menu.

![](https://cdn-images-1.medium.com/max/2368/1*yJDcWk9tCvwm3gWTRZsoYQ.png)

8. Next click the “...” button and select the folder you’ve just created “myflaskapp”. Leave the name as “myflaskapp” and the type as “Git” then click “Create”.

![](https://cdn-images-1.medium.com/max/2368/1*k561ZRwoNvl1FtCRD8lC2w.png)

9. That’s great. You’ve just created your first git repository. This will allow you to see the changes you’ve made to your code, and go back if you made a mistake. It’s also important because Amazon Elastic Beanstalk uses your git commits to deploy your application. In SourceTree double click on ‘myflaskapp’.

![](https://cdn-images-1.medium.com/max/2368/1*o41QyskJ-pP86CuE1Fctyw.png)

10. SourceTree shows us that we haven’t written any code yet. We’re about to change that.

![](https://cdn-images-1.medium.com/max/4648/1*QBT9FTTn-moQtuj2gF5bEw.png)

11. We’re going to return to the Terminal. Let’s use the *cd *command to change the directory so we are inside the ‘myflaskapp’ folder. The *cd* command is equivalent to double clicking the folder, so you are inside the folder.

    cd myflaskapp

![](https://cdn-images-1.medium.com/max/2728/1*t-dJpyHbKDSO9nbgpzTxKw.png)

11. Let’s set up a virtual environment, by using the *virtualenv* command. We can name our environment anything, but we will call it, *env,* as this is customary.

    virtualenv env

![](https://cdn-images-1.medium.com/max/2728/1*S0dITzvxzPQYav2gEtZUEA.png)

This has created a folder inside *myflaskapp* called *env*, which contains the files to run your virtual environment. It’s made it’s own copy of Python, easy_install and pip.

![](https://cdn-images-1.medium.com/max/3528/1*nlsnmyJcZjWjkT5jNtQG-A.png)

12. When we write our Python code, we will put it directly inside the *myflaskapp* folder next to the *env* folder. To run Python or pip we would need to type *env/bin/python* or *env/bin/pip* respectively, but that’s quite a lot to type, so we will run a script that activates a shortcut, to make our lives easier. Run the command:

    source env/bin/activate

Now the shortcut has been activated, we can simply type *pip* or *python*. This shortcut will last for as long as the Terminal window is open. When you open the Terminal again, you will need to run source *env/bin/activate* again.

The virtual environment can be deactivated using the command ‘deactivate’.

    deactivate

However, I was just showing you how that works. We want our virtual environment to be active. So run the activate command again:

    source env/bin/activate. 

Notice how, your Terminal says (env) on the line where you type. This means your virtual environment named ‘env’ is active. This is important for the next steps to work.

![](https://cdn-images-1.medium.com/max/2728/1*wf6GTxrA9dJzzjuv7PeNZQ.png)

13. Now that we have activated our virtual environment, let’s install the Flask package. Make sure the Terminal confirms it’s been installed successfully before moving on to the next step.

    pip install flask

10. We need to keep track of the packages we’ve installed, in a *requirements.txt* file. This way Elastic Beanstalk will know which packages to install on the EC2 instance. Run this command:

    pip freeze > requirements.txt

![](https://cdn-images-1.medium.com/max/2728/1*20P2a3p55YPGzu0nO7WbJQ.png)

This will create a text file called *requirements.txt* listing all your dependencies.

![](https://cdn-images-1.medium.com/max/3528/1*_RQXVOfyNairmyXMf6zYqA.png)

If you open the file in Sublime, it will look like this.

![](https://cdn-images-1.medium.com/max/2180/1*YgTi5OCcNYGCVkPDiXnm3w.png)

14. Before we write our first code, let’s see what has happened in SourceTree. Notice the files listed under ‘Unstaged files’. You will see that SourceTree has detected the creation of the ‘requirements.txt’ file. That’s good! But it’s also listing all the files in the ‘env’ folder. It’s not necessary to track the contents of the ‘env’ folder. We don’t want that.

![](https://cdn-images-1.medium.com/max/4648/1*xSUdCA8FOorqmlOAZACRfA.png)

15. We need to instruct SourceTree to ignore the ‘env’ folder. Open the application Sublime and enter this text. Save the file inside the ‘myflaskapp’ folder as ‘.gitignore’.

    env/

13. Files which start with a ‘.’ will be hidden on a Mac. So ‘.gitignore’ is a hidden file. Sublime will display a message, warning you that the file will be hidden. Click ‘Use’.

![](https://cdn-images-1.medium.com/max/2128/1*sEdKq-ecjsb3_YqKurnelg.png)

16. Now go back to SourceTree. You will see that the files in the ‘env’ folder are no longer being tracked.

![](https://cdn-images-1.medium.com/max/4648/1*llWPbrMvYRw1v5KfX56wfw.png)

17. Drag the two files ‘.gitignore’ and ‘requirements.txt’ from ‘Unstaged files’ to ‘Staged files’. Then click in the box ‘Commit message’.

![](https://cdn-images-1.medium.com/max/4648/1*dbyAbaQ3QivqRRyzkvaPNg.png)

18. In the “Commit message” box type “Initial commit.” and click “Commit”.

![](https://cdn-images-1.medium.com/max/4648/1*0KoSi2pWNnTGWyYv0WGgjQ.png)

19. On the menu on the left hand side, click “master” under “Branches” to see your commit history. You will see that there has been just one commit so far.

20 .Let’s write our first code. Open the application Sublime and create a new file. Then copy in this code.

    from flask import Flask

    application = Flask(__name__)
    application.debug = True

    [@app](http://twitter.com/app)lication.route(‘/’, methods=[‘GET’])
    def hello():
     return ‘<p>Hello world</p>’

    if __name__ == “__main__”:
     application.run()

21. Save the file as ‘application.py’ and save it directly into the *myflaskapp* folder. After you save it, you will notice that Sublime has highlighted your code in colour. This is because Sublime detected the ‘.py’ file extension and recognised your file contains Python code.

![](https://cdn-images-1.medium.com/max/2700/1*bDSxKaX2t3oHrmuhQ-Ya8A.png)

Before we go any further. Let’s understand what we’ve written.

Line 1 imports the Flask package, which we installed to our virtual environment earlier in step 13. Flask is a web framework. It extends the capabilities of Python to allow you to create a website.

    from flask import Flask

Line 3 creates a Flask application, which we name ‘application’. This is an important requirement — Amazon Elastic Beanstalk will look for an object named ‘application’ in the file ‘application.py’. If you named your application ‘app’ it won’t work.

    application = Flask(__name__)

Line 4 sets debug to True, so if there is an error, the error will be displayed in the browser. We wouldn’t want to do this in production, as it could enable a hacker to discover the internal workings of our application. However, during development it’s useful to see the error messages.

    application.debug = True

Line 6 is a Flask decorator. It says, if the user goes to the “/” URL of your website (in other words the homepage) then run lines 7–8.

    [@app](http://twitter.com/app)lication.route(‘/’, methods=[‘GET’])

Lines 7–8 is a function, which we have named hello. It says return the text ‘<p>Hello world</p>’ to the user’s browser. The ‘<p>’ is an HTML tag which means paragraph. So this means a paragraph of text will be displayed in the browser which says ‘Hello world’.

    def hello():
     return ‘<p>Hello world</p>’

Lines 10–11 initialise the Flask application.

    if __name__ == “__main__”:
     application.run()

That’s quite a few new programming terms. [So I recommend referring to Section 2 of the glossary.](https://medium.com/@miloharper/key-terms-513399c90d26#.gsh9wktss)

22. Our folder *myflaskapp* should now look like this.

![](https://cdn-images-1.medium.com/max/3528/1*bmI0QCFzvdiRjdJfrToJIA.png)

23. Open SourceTree. You can see we have new uncommitted changes. We need to commit “application.py” to our repository. Click on “Uncommited changes”, then repeat the process by dragging “application.py” from “Unstaged files” to “Staged files”.

![](https://cdn-images-1.medium.com/max/4648/1*lq-Xaj1XG_Fp0WuVbxBz2Q.png)

24. Then click “Commit” on the menu at the top. Type “Simple Flask application” as the commit message and click “Commit”.

![](https://cdn-images-1.medium.com/max/4648/1*-oHFphJubewArHjI6ZH9rw.png)

25. Your commit history should now look like this.

![](https://cdn-images-1.medium.com/max/4648/1*UuIEfWCsxMtchj2kj5oWOg.png)

26. Let’s run the code locally on our computer. In the Terminal, make sure the virtual environment is active and it says ‘env’ in your prompt. Then run the command:

    python application.py

![](https://cdn-images-1.medium.com/max/2728/1*b1zrGPPyBzewUQCPIg1LrQ.png)

It should say it’s launched your application on [http://127.0.0.1:5000.](http://127.0.0.1:5000.) This means your Flask application is running locally on your computer. The website can only be viewed from your own computer.

*Hint: If you get the error message “ File “application.py”, line 6 SyntaxError: Non-ASCII character ‘\xe2’ in file application.py on line 6, but no encoding declared; see [http://python.org/dev/peps/pep-0263/](http://python.org/dev/peps/pep-0263/) for details” this may be because certain characters weren’t transferred correctly when you copied and pasted the code. Try deleting all the single and double quotes and replacing them with new ones using your keyboard.*

Copy the URL [http://127.0.0.1:5000](http://127.0.0.1:5000.) into the address bar of your Internet browser, and press Enter.

![](https://cdn-images-1.medium.com/max/5568/1*hwDF6up7aBLFFVaAYx7rIA.png)

Did it work? Pretty cool. You just created your first website.

However, your computer isn’t easily accessible from the Internet. In order for everyone else to see your amazing website, we need to use a hosting provider. We’re going to use Amazon Web Services and use their free tier.

27. Go to the website [https://aws.amazon.com](https://aws.amazon.com/)/.

28. Click ‘Create a Free AWS account’. If you already have an Amazon shopping account, you can login using your existing account.

![](https://cdn-images-1.medium.com/max/5568/1*-DY9wpVnIxW-4YpFgcdIMQ.png)

29. Enter your contact details.

![](https://cdn-images-1.medium.com/max/5568/1*flXLycpDEuNwA12A7G8SUQ.png)

30. We will be using the AWS free tier, so you won’t be charged anything. However, AWS still required your credit card information.

![](https://cdn-images-1.medium.com/max/5568/1*oJhT51_EVH59vGQ6c4VG2g.png)

31. Verify your identity with Amazon.

![](https://cdn-images-1.medium.com/max/5568/1*DclFnEn9yhxAWiR2t8ptXA.png)

32. Choose the free support plan.

![](https://cdn-images-1.medium.com/max/5568/1*pCvR-2P8w1xAcayBZC_lNw.png)

33. Once the sign up process is complete. Click ‘Sign in to the console’ and log in.

34. Let’s create an administrator account. In the AWS console, click Services. This will open up a huge dropdown listing all the services that are available. It looks overwhelming, but just choose IAM, which stands for Identity & Access & Management.

![](https://cdn-images-1.medium.com/max/5568/1*PulrDOu4Qfseg0F1NWQxmw.png)

35. From Identity & Access Management, click on ‘Users’.

![](https://cdn-images-1.medium.com/max/5568/1*_Z04Rl9KzyLTipCsH6Pi0w.png)

36. From Users click ‘Create New Users’.

![](https://cdn-images-1.medium.com/max/5568/1*zZwMa1_QqsvfK8YLU2Z0Xg.png)

37. Create a new user called admin. Type ‘admin’ into the first box. Keep ‘Generate an access key for each user’ ticked. Then click ‘Create’.

![](https://cdn-images-1.medium.com/max/5568/1*HJb65JvDApU5jzwCRbDMRQ.png)

38. You will be given your User security credentials — an Access Key ID and a Secret Access Key. Save these to your computer in a safe place. We will need to use them again

![](https://cdn-images-1.medium.com/max/5568/1*dkItttqFiT-hK5KrT63o2g.png)

39. Next we’ll create a security group, for admin to be a member of. From Identity & Access Management, click ‘Groups’, then ‘Create New Group’.

![](https://cdn-images-1.medium.com/max/5568/1*pGkNNeVa7glsWRd6n_ZbsQ.png)

40. We’ll call our group Administrators.

![](https://cdn-images-1.medium.com/max/5568/1*uJdA14jEbs6IFbi4HKBcxA.png)

41. Attach the security policy AdministratorsAccess.

![](https://cdn-images-1.medium.com/max/5568/1*VcFuJXC41Pr3wHyDwtCvUw.png)

42. Then click ‘Create Group’.

![](https://cdn-images-1.medium.com/max/5568/1*T5v6sFVepSpufJtEGKtW1w.png)

43. It’s a convoluted process, but we’re not quite done yet. We need to add the user we created, admin, to our Administrators group. To do this go back to Users, tick ‘admin’ and from the ‘User Actions’ dropdown menu select ‘Add User to Groups’.

![](https://cdn-images-1.medium.com/max/5568/1*jhfKmtMTtEvwr-tsbrJXKQ.png)

44. Tick ‘Administrators’, the group we just created, and click ‘Add to Groups’.

![](https://cdn-images-1.medium.com/max/5568/1*58a9i4HWLowjkSLpIXTyhw.png)

Whew! That was a lot of steps. Don’t ask me why Amazon decided to make it so complicated. But I guess their interface is designed with large companies in mind. We’re done with the AWS console. We can go back to our own computer now.

45. Go back to the Terminal. If your website is still running locally on your computer, you can terminate by pressing the Control and C keys together.

![](https://cdn-images-1.medium.com/max/2728/1*09aCzJ6vcOR8eNCo8GkoZg.png)

46. Deactivate your virtual environment.

    deactivate

47. Let’s install the Amazon Web Services Elastic Beanstalk tool, which simplifies deployment. We deactivated our virtual environment first, because we want to install Elastic Beanstalk globally on our Mac. Run the command:

    pip install awsebcli

*Hint: I experienced an error when I installed Elastic Beanstalk tool, because I’m running OS X El Captain. I found that this command fixed the error:*

    pip install awsebcli --upgrade --ignore-installed six

48. Great, now we’re ready to initialise Elastic Beanstalk. Run the command:

    eb init

49. Choose your nearest region, by typing in the corresponding number and pressing Enter.

![](https://cdn-images-1.medium.com/max/2728/1*Deyq13E6EBRb7BnGTb2GDA.png)

50. Enter *myflaskapp* as the default name of your application.

![](https://cdn-images-1.medium.com/max/2728/1*6D8NvSBLYozdBC_zTq05bg.png)

51. Answer yes to the question “It appears you are using Python.”. Type the letter y, then press Enter.

![](https://cdn-images-1.medium.com/max/2728/1*PqPBWqicgEBuqygBcYRL7w.png)

52. Enter the number 3, then press Enter, to select Python version 2.7.

53. It will ask you if you want to set up SSH. Choose no.

![](https://cdn-images-1.medium.com/max/2728/1*iyu_yA3WY_Zc8UZO0tuwKg.png)

This will create a hidden folder called ‘.elasticbeanstalk’, which will store your Elastic Beanstalk configuration settings for the project.

![](https://cdn-images-1.medium.com/max/3168/1*vXiKGiIiarIzfCJrCtCKnA.png)

54. Now that we’ve initialised Elastic Beanstalk, let’s create an instance of your application running on Amazon EC2. Elastic Beanstalk will look at your commits, so make sure your commit history is up to date. Then run the command:

    eb create

55. It will ask you to enter a environment name and a CNAME. [Refer to Section 3 of the glossary to understand these terms.](https://medium.com/@miloharper/key-terms-513399c90d26#.45cpp4p7g)

I chose to name the environment *myflaskapp-dev* and set the CNAME to *mysupercoolflaskapp*. You can choose your own.

![](https://cdn-images-1.medium.com/max/2728/1*Y0W52m14Gh3qKNRhyaN8UA.png)

56. The Terminal should output a message saying it has launched successfully.

![](https://cdn-images-1.medium.com/max/2728/1*Y0W52m14Gh3qKNRhyaN8UA.png)

57. Look for the CNAME in the text in the Terminal. This is the URL for your website. Mine is mysupercoolflaskapp.elasticbeanstalk.com. Copy this into your Internet browser address bar.

It worked!

![](https://cdn-images-1.medium.com/max/5552/1*7veETjTFm-HNYft2UR8vPQ.png)

**Extra tips**

You can check the status of your application by running the command:

    eb status

You can check the health of your application by running the command:

    eb health

And if you are experiencing an error when you navigate to the CNAME, you can find out what is going wrong by running the command:

    eb logs

Thanks for reading and good luck!
