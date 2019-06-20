Unknown markup type 10 { type: [33m10[39m, start: [33m171[39m, end: [33m180[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m42[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m97[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m42[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m78[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m108[39m, end: [33m112[39m }

# AWS CLI(Command Line Interface)

As per AWS Official documentation
> *The AWS Command Line Interface (AWS CLI) is a unified tool that provides a consistent interface for interacting with all parts of AWS.*
> *The AWS CLI is an open source tool built on top of the AWS SDK for Python (Boto) that provides commands for interacting with AWS services. With minimal configuration, you can start using all of the functionality provided by the AWS Management Console from your favorite terminal program.*

*Install Command line interface on Linux*

***Requirements***

* *Python 2 version 2.6.5+ or Python 3 version 3.3+*

* *Windows, Linux, macOS, or Unix*

* *We can install aws command line interface and its dependencies with the help of pip(package manager for python)*

* *To install pip, we first need to install EPEL(Extra Package for Enterprise Linux) [https://fedoraproject.org/wiki/EPEL](https://fedoraproject.org/wiki/EPEL)*

    *# yum -y install epel-release*

* *Followed by python-pip installation*

    *# yum -y install python2-pip*

* *Now install awscli*

    *# pip install awscli*

    *Collecting awscli*

    *Downloading https://files.pythonhosted.org/packages/c2/1e/f70d1125f5bf6383d2ee7a76aea72bed6ba103c1bb9dd4ca051787552d2b/awscli-1.15.24-py2.py3-none-any.whl (1.3MB)*

    *100% |████████████████████████████████| 1.3MB 1.1MB/s*

    *Requirement already satisfied (use --upgrade to upgrade): docutils>=0.10 in /usr/lib/python2.7/site-packages (from awscli)*

    *Requirement already satisfied (use --upgrade to upgrade): PyYAML<=3.12,>=3.10 in /usr/lib64/python2.7/site-packages (from awscli)*

    *Collecting botocore==1.10.24 (from awscli)*

    *Downloading https://files.pythonhosted.org/packages/65/98/12aa979ca3215d69111026405a9812d7bb0c9ae49e2800b00d3bd794705b/botocore-1.10.24-py2.py3-none-any.whl (4.2MB)*

    *100% |████████████████████████████████| 4.2MB 339kB/s*

    *Requirement already satisfied (use --upgrade to upgrade): s3transfer<0.2.0,>=0.1.12 in /usr/lib/python2.7/site-packages (from awscli)*

    *Requirement already satisfied (use --upgrade to upgrade): rsa<=3.5.0,>=3.1.2 in /usr/lib/python2.7/site-packages (from awscli)*

    *Requirement already satisfied (use --upgrade to upgrade): colorama<=0.3.9,>=0.2.5 in /usr/lib/python2.7/site-packages (from awscli)*

    *Requirement already satisfied (use --upgrade to upgrade): jmespath<1.0.0,>=0.7.1 in /usr/lib/python2.7/site-packages (from botocore==1.10.24->awscli)*

    *Requirement already satisfied (use --upgrade to upgrade): python-dateutil<3.0.0,>=2.1; python_version >= "2.7" in /usr/lib/python2.7/site-packages (from botocore==1.10.24->awscli)*

    *Requirement already satisfied (use --upgrade to upgrade): futures<4.0.0,>=2.2.0; python_version == "2.6" or python_version == "2.7" in /usr/lib/python2.7/site-packages (from s3transfer<0.2.0,>=0.1.12->awscli)*

    *Requirement already satisfied (use --upgrade to upgrade): pyasn1>=0.1.3 in /usr/lib/python2.7/site-packages (from rsa<=3.5.0,>=3.1.2->awscli)*

    *Requirement already satisfied (use --upgrade to upgrade): six>=1.5 in /usr/lib/python2.7/site-packages (from python-dateutil<3.0.0,>=2.1; python_version >= "2.7"->botocore==1.10.24->awscli)*

    *Installing collected packages: botocore, awscli*

    *Found existing installation: botocore 1.10.11*

    *Uninstalling botocore-1.10.11:*

    *Successfully uninstalled botocore-1.10.11*

    *Successfully installed awscli-1.15.24 botocore-1.10.24*

* *To upgrade cli*

    *# pip install awscli --upgrade*

* *To verify aws cli installed sucessfully*

    *# aws --version*

    *aws-cli/1.15.24 Python/2.7.5 Linux/3.10.0-693.el7.x86_64 botocore/1.10.24*

* *Let start our journey to AWS world using command line tool and the first command we are going to use is **aws configure** which is going to configure settings that aws command line interface uses when interacting with aws(this include security credentials and default region)*

* *Before using aws configure we must need to sign up for aws account and download security credentials. If we don’t have access keys we can generate it from aws management console, go to IAM(under Security, Identity & Compliance)*

![](https://cdn-images-1.medium.com/max/2000/1*y0vhP-YugD-JRiaY-OvsVw.png)

* *Click on Users tab*

![](https://cdn-images-1.medium.com/max/2000/1*cnN86ZNm4GWfLW1_mdkrsA.png)

* *Click on Add user and fill all the details(make sure Programmatic access is clicked)*

![](https://cdn-images-1.medium.com/max/4480/1*ckND_l9jd2qW02J4ePz_jQ.png)

* *In the next screen choose(Attach existing policies directly) and Policy name(Choose Administrator access)(Not the best choice for security purpose but this is just a testing env)*

![](https://cdn-images-1.medium.com/max/5460/1*gdQ78i1boDk61ST7vYWKWQ.png)

* *Review and hit create user*

![](https://cdn-images-1.medium.com/max/5492/1*aV87iI1HPA_HTWDHAJdSAA.png)

* *As mentioned on the final screen, This is the last time these credentials will be available to download. However, you can create new credentials at any time.*

* *Now pass these values to aws configure command*

    *# aws configure*

    *AWS Access Key ID [****************QSQQ]:*

    *AWS Secret Access Key [****************64vL]:*

    *Default region name [us-west-2]: us-west-2*

    *Default output format [json]:*

* *Default region is the name of the region you want to make calls against by default. This is usually the region closest to you, but it can be any region. For example, type us-west-2 to use US West (Oregon).*

* *Default output format can be either json, text, or table. If you don't specify an output format, json is used.*

*We can also change these output format at the command line*

    *# aws <command> <option> --output text
    eg:
    # aws ec2 describe-instances --output text*

*OR*

    *# aws <command> <option> --output table
    eg:
    # aws ec2 describe-instances --output table*
> *The AWS CLI signs requests on your behalf, and includes a date in the signature. Ensure that your computer’s date and time are set correctly; if not, the date in the signature may not match the date of the request, and AWS rejects the request.*

* *The CLI stores credentials specified with aws configure in a local file named credentials in a folder named .aws in your home directory*

    *# ls -l ~/.aws*

    *total 8*

    *-rw-------. 1 root root  43 May 19 09:18 config*

    *-rw-------. 1 root root 116 May 19 09:18 credentials*

*NOTE: Storing aws credentials in a file is not a secure way, the best practice is to create IAM role and assigned it to an instance.*

* *We can also specify these variables with the help of environment variables*

* *Order of precedence(Configuration Settings and Precedence)*

    *AWS Command Line Option --> Environment Variables --> CLI configuration file*

*One of the features of Linux Shell that I was badly missing in AWS env is the auto- completion feature, i.e when I type below mentioned command, I want AWS cli/bash will auto-complete this partially typed command*

    *# aws ec*

*This is how typical aws command look like*

    *aws <service(command)> <operation(subcommand)
    aws      ec2              describe-instances*

*Lucky AWS provides a feature called aws_completer*

*To enable this feature it needs to set off information*

* *Name of the shell, you are using*

* *Location of auto-completer script*

    *# echo $SHELL*

    */bin/bash*

    *# which aws_completer*

    */usr/bin/aws_completer*

*To enable command completion, this connect aws_completer to aws cli*

    *# complete -C '/usr/bin/aws_completer' aws*

*To test it*

    *# aws e*

    *ec2                 ecs                 elasticache         elastictranscoder   elbv2               es*

    *ecr                 efs                 elasticbeanstalk    elb                 emr                 events*

*AWS Cli provide bunch of command/subcommand and it’s impossible to remember all these command, so the best approach is*

    *# aws ec2 help*

* *It give you the man page*

* *Syntax and examples*

*We can even run it on sub-command*

    *aws ec2 describe-instances help*

* *There is one more tool I would like to point out here called aws shell, it’s an integrated shell for working with the aws cli*

* *Installation*

    *# pip install aws-shell --upgrade --ignore-installed six*

* *Use*

    *# aws-shell*

![](https://cdn-images-1.medium.com/max/2012/1*QnhsewqYsu4hAtcst6cJdg.png)

* *As you can see aws-shell provides auto completion of commands and options as we type*

* *For more info*
[**awslabs/aws-shell**
*aws-shell - An integrated shell for working with the AWS CLI.*github.com](https://github.com/awslabs/aws-shell)

* *Generate CLI Skelton(It shows all the possible parameter of the sub-command)*

    *aws ec2 run-instances --generate-cli-skeleton > ec2instance.json*

*and then we can keep the parameters we want and pass it to aws cli*

    *aws ec2 run-instances --cli-input-json file://ec2instance.json*

*NOTE: By default it turn “**Dry**Run”: true, please make turn it to false*

    *"**Dry**Run": false*
