
# Getting to Know and Love AWS Elastic Beanstalk Configuration Files (.ebextensions)

How to use .ebextensions to configure the EB environment for your Rails App

![](https://cdn-images-1.medium.com/max/5710/1*bnHhvA0rzKb-ZBw0XE8qJQ.png)

[AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) is the service offered by Amazon to make your life easier when you need to deploy applications.
> You can simply upload your code and Elastic Beanstalk automatically handles the deployment, from capacity provisioning, load balancing, auto-scaling to application health monitoring.

And, my favorite part:
> There is no additional charge for Elastic Beanstalk — you pay only for the AWS resources needed to store and run your applications.

I've been using AWS Elastic Beanstalk to host a rails app I'm working on and I have learned a few things along the way. In this guide I'll share with you how to:

1. Setup your .ebextensions

1. Configure your Nginx server

1. Run Rake Tasks Automatically

1. Get Files from your AWS S3 Bucket

1. Keep Things Clean

## 1. Setup your .ebextensions

You can configure your EB environment through [Elastic Beanstalk configuration files (or .ebextensions)](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html).
> Configuration files are YAML formatted documents with a .config file extension that you place in a folder named .ebextensions and deploy in your application source bundle.

EB configuration files are so powerful that you don't have to connect to your instance through SSH to issue configuration commands. You can configure your environment entirely from your project source by using .ebextensions.

To start using them, create a folder called .ebextensions within the root of your project and start adding files with the .config extension to it.

![In a rails application, your **.ebextensions** folder would be at the same level as the **app** and **config** folders.](https://cdn-images-1.medium.com/max/2000/1*bycoinYBWZEosjD4NpuUzw.png)*In a rails application, your **.ebextensions** folder would be at the same level as the **app** and **config** folders.*

The name of the file doesn’t matter as long as the extension is .config, but keep in mind that your EB configuration files will be processed in alphabetical order. This is why it is a good idea to use numbers when naming your .config files. By naming your files appropriately, you can split your configuration activities into multiple stages.

### 1.1. Writing EB Configuration Files

In each EB configuration file, you'll specify one or more keys that will determine what will be done in your EB environment. You'll find most of the keys you need here: [Customizing Software on Linux Servers](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html). There are also other keys like the [Option Settings](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions-optionsettings.html) key and the [Resources](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environment-resources.html) key to further customize your EB configuration files. We’ll be seeing how to use some of these keys in the following sections. Finally, when you write your EB configuration files, you have to keep in mind the following:
> YAML relies on consistent indentation. Match the indentation level when replacing content in an example configuration file and make sure that your text editor uses spaces, not tab characters, to indent.

## 2. Configure your Nginx Server

We will use .ebextensions to create Nginx configuration files that will be included in the main Nginx configuration. Let's see how this works.

Your Elastic Beanstalk Nginx server is using the configuration file located at /etc/nginx/nginx.conf. You can verify this is in fact the location of your Nginx configuration file by connecting to your EB environment through SSH and running the service nginx configtest command:

    $ eb ssh environment-name
    $ sudo service nginx configtest
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful

The service nginx configtest command tests that the configuration file is valid and exists. Luckily for us, it also prints where the configuration file is located.

Now you have seen where the configuration file used by your Nginx server is located, but you will not modify this file directly through your .ebextensions. Instead, you'll use your .ebextensions to create Nginx configuration files that will be included in the main nginx.conf file.

To understand how this works, let's take a look at the /etc/nginx/nginx.conf file:

<iframe src="https://medium.com/media/f368b2d7ff6f70d795f3b2cdd8409ff9" frameborder=0></iframe>

Pay attention to line **32**, where the include directive is used:

    include /etc/nginx/conf.d/*.conf;

Thanks to this line, all the .conf files we place in the /etc/nginx/conf.d/ directory will be included in the Nginx configuration. But there is one caveat, the include directive is inside the http block. This means that our .conf files can only contain Nginx directives that are valid in the http context.

Which ones are the valid directives in the http context? You can find out here: [Alphabetical index of directives](http://nginx.org/en/docs/dirindex.html).

### 2.1. Set the 'client_max_body_size' Directive

If you have found the **413 Request Entity Too Large** error, you'll want to increase the allowed size of the client request body by using the client_max_body_size directive. Let's use an EB configuration file to add that directive to our Nginx configuration. Add the following file in your .ebextensions directory:

<iframe src="https://medium.com/media/8af6f43f2caa0405d722af61db72dffa" frameborder=0></iframe>

There are a few of things to note here:

* We are using the files key to create a file called 01_proxy.conf in the /etc/nginx/conf.d/ directory. You can read more about it [here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html#linux-files).

* This file has the mode set to 000644 and the owner and group set to root, which means when we do a ls -l for this file, the result will be: -rw-r--r--1 root root 26 Feb 4 23:40 01_proxy.conf

* The content of the file is a single line: client_max_body_size 10M;

* Since client_max_body_size is a valid directive in the http context, it will be included in the /etc/nginx/nginx.conf file without problems.

You would have to connect to your server through SSH and run service nginx reload for this change to take effect, but you can tell your EB configuration file to do that for you:

<iframe src="https://medium.com/media/9bdf5aa642c41e4431f9603c62a1232f" frameborder=0></iframe>

New things added:

* We are using the container_commands key to run the service nginx reload command. You can read more about it [here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html#linux-container-commands).

* Note that this implies that the container_commands key is executed after the files key. The configuration keys are processed in the order specified in [Customizing Software on Linux Servers](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html#linux-container-commands).

### 2.2. Add a 'location' Directive

Maybe you are in need to tell Nginx that when a user types http://www.mydomain.com/files/awesome_image.png, you actually want that image to be served from app/public/ridiculously/long/path/to/files/. You will need something more sophisticated than the previous solution, since the location directive is not a valid directive in the http context of the Nginx configuration file.

As you know, the location directive goes inside the server block, and the server block your Nginx is using is located in the /etc/nginx/conf.d/webapp_healthd.conf file. This file was generated by your EB environment when you ran the eb create command. As you can imagine, this file is also included by the include directive in the /etc/nginx/nginx.conf file we saw earlier. Let's introduce you to it, so you are no longer strangers:

<iframe src="https://medium.com/media/8d9c96aad333064d66e314207e63bb42" frameborder=0></iframe>

This file creates the upstream block to send requests to your app, defines the log format to be used and adds some basic location blocks to serve static content. Now that you are aware of the existence of webapp_healthd.conf, let's see your options to add the location directive you need. You could:

1. Overwrite the webapp_healthd.conf file

1. Create a new .conf file that is processed before the webapp_healthd.conf file

1. Create a new .conf file and delete the webapp_healthd.conf file

I'll tell you how to do the second option since it's my favorite one because you keep the original webapp_healthd.conf. I don't like to mess with generated files.

Let's create a new .conf file within our existing EB configuration file. In this new file, we will have the same content as the webapp_healthd.conf file plus our custom location.

<iframe src="https://medium.com/media/2c0016e8b84b18c4a98cdcf9d4a368d1" frameborder=0></iframe>

Things to notice:

* We are creating two different files with a single files key.

* We named our second file 02_app_server.conf. Yes, you guessed correctly, files included by the include directive in the nginx.conf file are also processed in alphabetical order ([since Nginx 1.3.10](http://nginx.org/en/CHANGES) at least).

* The name of the upstream block was changed to new_upstream_name because we can't have two upstream blocks with the same name.

* The name of the log_format directive was changed to new_log_name_healthd because we can't have two log_format directives with the same name.

The naming becomes extremely important at this point, because now we have two server blocks defined with the same values for the listen and server_name directives: one in 02_app_server.conf and one in webapp_healthd.conf. Since Nginx relies on those directives to decide which one to use, how will it decide if both of them have the same values? Nginx will choose the first one that matches. Therefore it is really important that the file where our custom server is defined is listed before the webapp_healthd.conf file.

As an alternative, if you are uncomfortable with having to be careful with the .conf file names, you can add the following command to your 01_nginx.config file to remove the webapp_healthd.conf files:

    container_commands:                         
      01_reload_nginx:                           
        command: "sudo service nginx reload"
      02_remove_webapp_healthd:
        command: "rm -f /opt/elasticbeanstalk/support/conf/webapp_healthd.conf /etc/nginx/conf.d/webapp_healthd.conf"

### 2.3. Further Customization of your Nginx Server

If you want to know more about tuning your Nginx server for your specific needs, I recommend you read: [Understanding the Nginx Configuration File Structure and Configuration Contexts](https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts), [Nginx: A Practical Guide to High Performance ](http://shop.oreilly.com/product/0636920039426.do)and [Nginx Essentials](https://www.packtpub.com/networking-and-servers/nginx-essentials).

## 3. Run Rake Tasks Automatically

You can instruct your EB environment to run a rake task every time you make a deploy.

### 3.1. With the 'container_commands' Key

Let's tell it to run rake db:seed:

<iframe src="https://medium.com/media/66f17b684c070a4ea365aca01df2779d" frameborder=0></iframe>

Very simple, right? Just use the container_commands key to issue the rake command. You can even set some environment variables before running the rake command if you need to, just as explained in the [AWS documentation](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Ruby.container.html#create_deploy_Ruby_custom_container):

<iframe src="https://medium.com/media/9a51a25898f9cd7e310f29fdfc1ac47b" frameborder=0></iframe>

### 3.2. With a Shell Script in a Hook Directory

If you need to execute a more complex rake task, you can use a script similar to the one used by your EB environment to run the rake db:migrate task. Your EB environment uses a shell script located in /opt/elasticbeanstalk/hooks/appdeploy/pre/12_db_migration.sh to execute the database migrations. /opt/elasticbeanstalk/hooks/appdeploy/ is a special location where EB places three hook directories: pre, enact and post. All .sh files located inside those directories will be processed by the EB environment in alphabetical order before deployment, during deployment and after deployment, respectively.

Here is an example where we create a shell script inside the pre hook directory to execute a rake task before our app is deployed:

<iframe src="https://medium.com/media/7270f4072fddf83a2aaf7b629dd565b3" frameborder=0></iframe>

Things to notice:

* We are creating a single file called 13_db_seed.sh. This file has the same contents as the /opt/elasticbeanstalk/hooks/appdeploy/pre/12_db_migration.sh file created by your EB environment. The only difference is that our file executes a rake db:seed command instead of a rake db:migrate command.

* The file we are creating has the number 13 in its name, so it's executed after the migration task, which has the number 12 in its name.

* The file will be created with execute permissions because the mode is 000755.

* We are placing our file inside pre deployment hook directory.

* The .sh file we are creating loads the environment variables defined in /opt/elasticbeanstalk/support/envvars, which includes the environment variables you have defined for your EB environment. It also verifies if the rake task exists before trying to execute it. If it exists, the rake task is executed.

Be careful with this method, as the structure of the hook directories or the files inside them could change in the future.

## 4. Get Files from your AWS S3 Bucket

So far, all our files have been created with the content specified inline inside the files key of our EB configuration file. Maybe this is not good enough for you. Maybe you want to have a single Nginx configuration file to be used by all your EB applications. In this case, it is a better approach to host your shared .conf file in your AWS S3 bucket and instruct EB to download the file and put it in the appropriate directory. Let's see how to do this.

### 4.1. With the 'files' Key and the 'source' Option

We'll keep using the files key, but instead of using the content option, we'll use the source option to specify the contents of the file. This method is presented as an example in the example snippets of the [files key documentation](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html#linux-files):

<iframe src="https://medium.com/media/7481c0705edb4e1867671fd80116114c" frameborder=0></iframe>

Things to notice:

* We are using the source option instead of the content option to specify the file contents.

* When you use the files key with the source option to download files from AWS S3, you must also add the authentication option to specify the name of the authentication method to be used.

* We define the authentication method using the Resources key. You can copy/paste that resource definition, but make sure you change your-bucket-name for the actual name of your S3 bucket. Your bucket name will start with elaticbeanstalk to indicate that it was the bucket created by your EB application and will end with a series of numbers. You can find your bucket name in your [AWS Console](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwin1aeSh__RAhXD4iYKHVoBAUQQFggZMAA&url=https%3A%2F%2Faws.amazon.com%2Fconsole%2F&usg=AFQjCNEoh07uBpRXLFqdNMkE61u37M18qA&sig2=CV0XTTnc8Ho7yJGPoPBGaA&bvm=bv.146496531,d.eWE), under the S3 service.

* Basically, what we are doing with the Resources key is that we are using the [AWS::CloudFormation::Authentication](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-authentication.html) resource to create an authentication method named S3Auth (you can change this name if you want). We define three properties for this authentication method: S3 as the type, your-bucket-name as one of the buckets and the IAM instance profile to be used for authentication as the roleName. We are using an [AWS CloudFormation Function](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions-functions.html) to get the [IamInstanceProfile configuration option](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options.html), since the EC2 instance is associated to an IAM role through an [instance profile](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts-roles.html).

* Once the file is created, we reload the nginx server with a container command and we are done.

### 4.2. With the 'commands' Key

I learned this method in [this StackOverflow answer by Matt Holtzman](http://stackoverflow.com/questions/23709841/how-to-change-nginx-config-in-amazon-elastic-beanstalk-running-a-docker-instance/30790732#30790732). Here we'll use the aws s3 command to download the file from our AWS S3 bucket:

<iframe src="https://medium.com/media/e1f8bf227e52909a86ff96cebb83e4f8" frameborder=0></iframe>

Things to notice:

* We use the commands key to issue a command to the AWS CLI and download a file from our S3 bucket.

* After downloading the file, we use the container_commands key to move the downloaded file to our desired location and reload the nginx server.

## 5. Keep Things Clean

### 5.1. Rename your EB Configuration Files to Keep Them in Order

You can rename your .ebextension files to reorder them whenever you add a new configuration file and it won’t have any impact on the contents of the server. So don't be afraid to rename all of them whenever you need to so you can have a continuous numbering (01, 02, 03, …).

### 5.2. Be Careful When Renaming Generated Files

Be careful when you rename the files generated with your .ebextensions. For example, if you rename your /etc/nginx/conf.d/01_proxy.conf to /etc/nginx/conf.d/proxy.conf, the first 01_proxy.conf file you generated won't go away.

The files key only creates files, it doesn't remove them when you remove the file from your .ebextensions. In order to remove the first file, you would need to use the commands or container_commands key to issue a remove command, or you need to connect through SSH and remove the file manually.

Also, keep in mind that if you make a deploy with the new proxy.conf file and you haven't removed the first 01_proxy.conf file, your deploy will fail because the new file will be created while the other one will still exits. This would mean that now you have two client_max_body_size directives in your http block and you'll get the error:

    ERROR: [Instance: i-123456789012345678] Command failed on instance. Return code: 6 Output: nginx: [emerg] “client_max_body_size” directive is duplicate in /etc/nginx/conf.d/proxy.conf:1

### 5.3. Avoid modifying files directly through SSH

A lot of things go on when you run eb deploy and eb upgrade. Avoid modifying files directly through SSH because your changes could be overwritten when you run one of those two commands.

## Final Words

Now you know how to use your the EB configuration files to customize your EB environment entirely from your source bundle. All the examples in this guide have been tested with:

* EB CLI version: 3.9.0 (Python 2.7.1)

* Platform version: 64bit Amazon Linux 2016.09 v2.3.1 running Ruby 2.3 (Puma)

Thank you for reading and leave your comments if you have a suggestion.
