
# Automated deployments via Jenkins & AWS Codedeploy



1. Create a sample repository of your code in GitHub or you can use the already existing one. Put files as below structure:

![](https://cdn-images-1.medium.com/max/2074/0*yqtbU7IrTkRSbu-H.)

2. Click on the script folder and make the structure as shown below, keep your code inside the script folder. As here *newcode* is our code.

* All the file structure should be like the appspec.yml file as here we have given below:

    install_dependencies.sh
    install_dependencies01.sh
    start_server.sh

![](https://cdn-images-1.medium.com/max/2178/1*PJuc8Tw6qyxzPvSDn9UfaQ.png)

* Under *appspec.yml* file write as given below:
> appspec.yml: The application specification file (AppSpec file) is a [YAML](http://www.yaml.org/)-formatted or JSON-formatted file used by AWS Codedeploy to manage a deployment.

    version: 0.0

    os: linux

    files:

     - source: script/newcode

       destination: /var/www/html

    hooks:

     BeforeInstall:

       - location: script/install_dependencies.sh

         timeout: 300

         runas: root

     AfterInstall:

       - location: script/install_dependencies01.sh

         timeout: 300

         runas: root

     ApplicationStart:

       - location: script/start_server.sh

         timeout: 300

    runas: root

In above yaml file the** source** and **destination **we have to give as our code name.

3. Go to console and attach the role to EC2 instance having permissions as shown below:

![](https://cdn-images-1.medium.com/max/2408/0*xCVuf6pFnOiYgifQ.)

4. Install Codedeploy agent to the server with below command(Ubuntu).

    apt-get -y update

    apt-get -y install ruby

    apt-get -y install wget

    cd /home/ubuntu

    wget [https://aws-codedeploy-us-east-1.s3.amazonaws.com/latest/install](https://aws-codedeploy-us-east-1.s3.amazonaws.com/latest/install)

    chmod +x ./install

    ./install auto

Note: Give the region as per your account like we have given here: *us-east-1*

In case of AutoScaling group we can put the command in the User data to install Codedeploy agent automatically on every new instance launched by AutoScaling group.

5. Now we will setup the Jenkins for automation build trigger. Follow below steps to configure:

* Under GitHub go to settings and in the left pane we can get **Integration &** **services , **under that go to Jenkins hook url and paste as below link.

*http://jenkinsserverip:8080/github-webhook/*

![](https://cdn-images-1.medium.com/max/2000/0*LtmOSNBIT4EcJXAS.png)

6. Now save it and we can see the Services are updated with the Jenkins(GitHub plugin) as shown below.

![](https://cdn-images-1.medium.com/max/NaN/0*Db6HPyfWuTkw7xm2.)

7.Next under Deploy keys, we need to add the key of the Jenkins server.We can get the key from the server with the below command:

* login to server then go to Jenkins user first-

*$su jenkins*

*$jenkins@ip:/home/ubuntu$ cat /var/lib/jenkins/.ssh/id_rsa.pub*

* Copy the key got from above command and paste it in the key section as shown below:

![](https://cdn-images-1.medium.com/max/2312/1*mEUB0kmcS9K-iCI9H40fDg.png)

* Give write access to it and add the key.

![](https://cdn-images-1.medium.com/max/2000/0*1p5IQRR6-eZdbCAN.)

8. Lets create a Jenkins job now, follow the steps below:

![](https://cdn-images-1.medium.com/max/2000/0*TXwuRtIo5zyOsmre.)

9. In Source code management, choose Git and write the URL like :

*git@github.com:<username>/<codename>.git*

![](https://cdn-images-1.medium.com/max/2000/0*MPEOJIfOFTXipEIE.)

10. In Build triggers tick GITScm polling which means If Jenkins will receive PUSH GitHub hook from repository defined in Git SCM section it will trigger Git SCM polling logic. So polling logic in fact belongs to Git SCM.

![](https://cdn-images-1.medium.com/max/2000/0*ojAalzQUjVMYZY5o.)

11. In Post build actions, choose AWS Codedeploy in list and then give the Codedeploy credentials: Application name, Deployment group name, AWS region and S3 bucket at which we can get the deployment output.

![](https://cdn-images-1.medium.com/max/2000/0*eJbymMmm-g1MxNvJ.)

12. Next go to your Jenkins and uncheck the CSRF Protection and also install the [**GitHub Integration Plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+Integration+Plugin).**

Now we are all set to see the magic here. As we do any changes in the code that will trigger the Jenkins jobs to build automatically and then the AWS Codedeploy will start deploying the code just after Jenkins build completes and then code changes will be reflected within seconds to the output as here I observed.
> **Link: **http://<jenkinsIP:8080>/newcode

**Happy Automation Deployment!**
