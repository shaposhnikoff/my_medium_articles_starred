
# Securing Secrets for your IaC using Jenkins, Terraform and Ansible Vault

Image from FullStackPython

The following describes how I used 3 of my favorite tools to build a secure CI/CD code pipeline for building and maintaining infrastructure across different environments. I will only be showing code from the Jenkins Pipeline since the majority of the magic happens from within Jenkins. The use of Terraform and Ansible will become clear once we can see what is executed in the pipeline. I also use Gitlab as my repo which is obviously also integrated with Jenkins but you could also use other git repos as well like github etc. Please keep in mind that this uses exclusively the open source versions of these tools which is why we needed to incorporate all of these tools together. This may not be a good solution for everyone but served me well as a quick solution on a very tight deadline with no budget and extremely low man power.

Before getting too deep, I would like to take a moment to describe what each tool provides in this scenario:

**Terraform**

If you are a cloud user and haven’t heard of or have used Terraform, I would highly recommend that you do. I have been using for the better part of two years now and would have to say that I have become a super fan boy of its many benefits. My love for Terraform would be better served in a separate post but in simple terms Terraform is the work horse that provides us with a provides us with the ability codify APIs for planning, creating and managing infrastructure. It is highly customizable and flexible while also keeping track of a desired state that allows for versioning or recovery if needed.

**Ansible**

Despite the known power and automation ability that Ansible provides through playbooks and even cli, Ansible Vault is the tool that I will be referencing here. Ansible Vault is going to provide us with securing secrets variables in our .tfvars file(s) and also keeps from storing our terraform.state files in clear text since we are using Gitlab as back end storage. They will be stored in the Gitlab repo in an encrypted format and will be decrypted upon pull and re-encrypted upon push.

**Jenkins**

Our automation engine that is going to make everything a non manual process. This gives us a button click ability for a pipeline of stages and also allows us to store secrets securely outside of our code for AWS and Ansible Vault.

Jenkins Requirements

Typically, I prefer to run Jenkins in a Master/Slave environment and on DC/OS but for simplicity purposes this uses a single Jenkins node for executing all the things. Jenkins requires the following in order to make this all work:

* Ansible and Terraform installed

* Git Authentication

* Groovy Env variables and a password file in Jenkins home for ansible-vault

* Secrets for AWS Keys in Jenkins

Each of the above will be explained more when breaking down each stage of the Jenkinsfile below.

**Gitlab**

Our repo and the backend store for the terraform state files. Gitlab is tied to our Jenkins instance by a user account and credentials so that our Jenkins Pipeline is able to pull and push code changes without any manual interaction. Gitlab has become my favorite code repo over the past few years with so many out of the box integrations and additional features such as Docker Registry and its own built in Pipeline tool.

Now that we have a better understanding of each component, let’s go ahead and take a look at the **Jenkinsfile:**

<iframe src="https://medium.com/media/55a3360877df2fe870c41582cfa14804" frameborder=0></iframe>

**First 8 Lines**

As mentioned, you can see that all the stagess are executed from a single Jenkins node and has a few important variables. The most important one being line 8:

    load “$JENKINS_HOME/.envvars/.env.groovy”

This line could be considered a bit unnecessary but is used to set the “VAULT_LOCATION” variable in the Pipeline which is the directory where your password(s) for Ansible Vault is stored. This is an additional step to keep from storing the location of the password file location in plain text in your code repo. “$JENKINS_HOME/.envvars/.env.groovy” will need to be created manually on Jenkins as well as the $VAULT_LOCATION directory.

As Jenkins user:

    mkdir -pv $JENKINS_HOME/.envvars/ && \
    mkdir -pv $JENKINS_HOME/.vault && \
    echo "env.VAULT_LOCATION="$JENKINS_HOME/.vault"" >> “$JENKINS_HOME/.envvars/.env.groovy”

The other important variable set here is “environment”. This is used as not only to assign the environment of in the infrastructure but also as the filename of the plain text password file in the $VAULT_LOCATION directory on the Jenkins node and also the name of the secrets file for terraform. The ${envionment}.txt and ${environment}-secrets.tfvars gets passed to Ansible Vault as cmd line arguments, which we will see later, for encrypting and decrypting secrets.

As Jenkins user:

    echo "ThisIsOurPasswordForEncrypt&Decrypt" >> $JENKINS_HOME/.vault/Development.txt 

**Lines 11-13 (checkout stage)**

These lines are pretty self explanatory. You are checking out the code repo to the Jenkins workspace. You need to have atleast the Git plugin installed and configured on Jenkins. In Gitlab you will need to have a user created with an ssh key that has high enough permissions on the repo to push to master (Yes I know. I said master). Add the ssh key used to from Gitlab to the Jenkins node under $JENKINS_HOME/.ssh/id_rsa.pub. I had issues with pushing code until I did this (which is one of the last stages in pipeline).

See Jenkins [Git Plugin](https://wiki.jenkins.io/display/JENKINS/Git+Plugin) for info on setting up Git auth between your code repo and Jenkins.
[**Git Plugin - Jenkins - Jenkins Wiki**
*The git-client-plugin provides both command line and JGit implementations for the GitClient interface. Using command…*wiki.jenkins.io](https://wiki.jenkins.io/display/JENKINS/Git+Plugin)

**Lines 15-21 (decrypt secrets stage)**

This is where things start to get a little more interesting. You need to ensure that you have installed [Ansible](http://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) and [Terraform](https://www.terraform.io/intro/getting-started/install.html) on your Jenkins node. In this stage, your terraform secrets file (${environtment}-secrets.tfvars) and your terraform state files (terrform.tfstate and terraform.tfstate.backup) get decrypted using the password file on the Jenkins node that we created above.

    ansible-vault decrypt --vault-password-file=${env.VAULT_LOCATION}/${environment}.txt ${environment}-secrets.tfvars

In order for this stage to complete successfully you would have had to initially had a terraform state and these files would have already had to have been encrypted. Normally, my first terraform apply or two is done locally just to ensure it is working as expected. You also need to ensure that you are encrypting the .tfvars and state files prior to each commit so that it is stored in the repo in encrypted format. (Remember in Terraform open source, terraform state files are stored in plain text which is why we are encrypting those.)

Initial encryption happens with Ansible Vault as well. You need to either pass the same password in a password file that we used above or just pass encrypt to the files by typing the same password above twice. Below are steps for initial creation:

    ansible-vault encrypt ${envionment}-secrets.tfvars terraform.state*
    New Vault password:
    Confirm New Vault password:
    Encryption successful

    git add ${envionment}-secrets.tfvars terraform.state*
    git commit -m 'blah'
    git push

For more information on using Ansible Vault:
[**Ansible Vault - Ansible Documentation**
*Ansible Vault can also encrypt arbitrary files, even binary files. If a vault-encrypted file is given as the argument…*docs.ansible.com](http://docs.ansible.com/ansible/latest/user_guide/vault.html)

For more information about Terraform State:
[**State - Terraform by HashiCorp**
*Terraform must store state about your managed infrastructure and configuration. This state is used by Terraform to map…*www.terraform.io](https://www.terraform.io/docs/state/)

**Lines 23–32 (init and validate stages)**

Nothing too special going on in these stages so not a whole lot of time will be spent here. In the init stage, we call a terraform init which is used to download and install all the necessary modules needed to run on our provider. In the validate stage, we call on the terraform validate to validate our terraform code. If there is any syntax errors the pipeline will fail because terraform could not validate the code and there are errors. You can see the errors from the Console in the Pipeline on Jenkins.

More info on terraform init:
[**Command: init - Terraform by HashiCorp**
*The `terraform init` command is used to initialize a Terraform configuration. This is the first command that should be…*www.terraform.io](https://www.terraform.io/docs/commands/init.html)

More info on terraform validate:
[**Command: validate - Terraform by HashiCorp**
*The `terraform validate` command is used to validate the syntax of the terraform files.*www.terraform.io](https://www.terraform.io/docs/commands/validate.html)

**Lines 34–45 (plan stage)**

A couple of important things going on here. The first thing you will need to do, if you are using AWS is place your keys in Jenkins Credentials as secrets text with the ids of ‘aws-access-key’ and ‘aws-secret-key’. You will then pass them as ‘AWS_ACCESS_KEY_ID’ and ‘AWS_SECRET_ACCESS_KEY’ from Jenkins credentials when you run ‘terraform plan’. These are the default variables needed to auth to AWS API with Terraform.

See Jenkins Credentials for more info:
[**Using credentials**
*There are numerous 3rd-party sites and applications that can interact with Jenkins, for example, artifact repositories…*jenkins.io](https://jenkins.io/doc/book/using/using-credentials/)

Once you can auth to AWS via Terraform, you will be able to successfully run the command for the terraform plan. The terraform plan is used to create an execution plan against your current state of the infrastructure. If something has been changed in AWS, it will be changed back via settings in the state file. If you are making changes to the code, terraform will let you know what resources are being affected with either a change, add and/or destroy. Seeing the execution plan for what is going to eventually be applied is extremely helpful and it allows for you to see what resources might be affected during the apply. It helps you decide if you agree or disagree with the changes being proposed.

During this phase that we are passing the decrypted secrets file to the plan and outputting the plan to a file so that we persist the execution plan. We will pass the plan file to the apply in next stage.

    terraform plan -var-file=${environment}-secrets.tfvars -out=create.tfplan

You will need to go to the Console of the current pipeline run in Jenkins to see the output of the execution plan. And in line 45, you actually need to tell Jenkins whether or not you agree with the proposed execution plan and whether or not to continue or to exit.

    // wait for approval. If Plan checks out.    
    input 'Deploy stack?'

This was added as a safeguard against the pipeline doing anything destructive to your resources. If we allowed it to continue here without this input there is a chance that it could do something to interrupt service and/or destroy something that should not have been. This allows us to choose the fate of our proposed changes.

For more info on terraform plan:
[**Command: plan - Terraform by HashiCorp**
*The `terraform plan` command is used to create an execution plan. Terraform performs a refresh, unless explicitly…*www.terraform.io](https://www.terraform.io/docs/commands/plan.html)

**Lines 47–55 (apply stage)**

So you are happy with the execution plan being proposed and you gave Jenkins to go ahead to move to next stage. This phase takes the create.tfplan file and actually “applies” those changes to your resources. This stage also uses the same credentials as the plan stage for authentication to the AWS API.

    terraform apply create.tfplan

Please note, that if something does not go to according to the execution plan and the apply fails, there are no safeguards or recovery methods in the pipeline. You will actually need to go into the Jenkins Workspace and recover your state files, encrypt and commit to ensure no data loss. There was no time to implement a fail safe for this step unfortunately and to note we were not doing anything super complex to have much of a chance of apply failing. It did however happen a time or two during initial setup so just something to be aware of. Perhaps, you could come up with something for this?

More info on terraform apply:
[**Command: apply - Terraform by HashiCorp**
*The `terraform apply` command is used to apply the changes required to reach the desired state of the configuration, or…*www.terraform.io](https://www.terraform.io/docs/commands/apply.html)

**Lines 58–64 (re-encrypt stage)**

This stage does the same steps as the decrypt stage (lines 15–21) but instead re-encrypts the files before they get committed and pushed. We pass the same password file to the secrets file and the state files so that the password does not change. Not going to spend a whole lot of time discussing since we spent a solid amount of time on it above.

    ansible-vault **encrypt** --vault-password-file=${env.VAULT_LOCATION}/${environment}.txt ${environment}-secrets.tfvars 
    ansible-vault **encrypt** --vault-password-file=${env.VAULT_LOCATION}/${environment}.txt terraform.tfstate*

**Lines 66–74 (push and merge stage)**

This stage is an extremely important stage to complete successful as we are using Gitlab as our backend store for our terraform state. If this state file gets lost, corrupted and/or out of sync with changes, it becomes extremely difficult to recover and/or continue to use terraform to manage your infra resources. This is one of the main reasons that I commit directly to master. It is not preferred practice, I know, but as I said before, this served me fine because my deployments were not overly complex and I work in an extremely small team.

Im not going to go into much detail about what is being executed as this is not a post about using git and the steps are pretty self explanatory. (If you are a novice to git or have never used, chances are this entire post is likely irrelevant to you anyways.)

    git add terraform/aws/terraform.tfstate* terraform/aws/*-secrets.tfvars          
    git commit -am 'Commit Terraform State - Jenkins Job ${env.JOB_NAME} - build  ${env.BUILD_NUMBER} for ${environment}'          
    git push origin HEAD:master

The important key for this stage is to ensure that you are committing the encrypted secrets file as well as the latest encrypted state files. I also commit to the repo with the Job Name and Build number to track them in commit messages.

**Lastly, Lines 76–82 (notification stage)**

Modify these lines to meet your needs to receive an email notification when pipeline runs successfully. Would prefer this to be a Slack notification but unfortunately we use a corporate Slack and I don’t have access to integrate. Email was much easier in the time frame given. Also, would be nice to have had time to work on setting up a way to notify on build fail as well. Perhaps that is also something you could incorporate into your Pipeline?

This was a fun little problem for me to solve. As mentioned, it is probably not an ideal solution if you have a larger DevOps Team with more time to dedicate. Ideally I would prefer to use something like Hashicorp Vault for securing secrets but this has actually worked out great for me with no issues on doing a few daily deployments for code changes.

Happy Hacking! Cheers!
