
# How to setup and deploy a Rails 5 app on AWS ElasticBeanstalk with PostgreSQL, Redis and more…



Note: This tutorial is an excerpt for the deployment chapter in my book [Building a SaaS Ruby on Rails 5](https://BuildASaaSAppinRails.com). The book will guide you from humble beginnings by deploying an app to production. If you find this type of content valuable, you can grab a free chapter right now!

Also, the beta for my new project [Pull Manager](https://pullmanager.com) is almost ready. If you’re losing track of pull requests, have old ones lingering around or would just love a dashboard that aggregates these requests over multiple services (Github, Gitlab, and Bitbucket), [check it out](https://pullmanager.com).

Deploying a Rails app can be a somewhat daunting task to get set up right on new applications, even for seasoned Rails developers. While Capistrano has been the main player in Rails app deployments, it seems to have fallen somewhat out of favor as load balancers and computing servers have gained popularity.

Thankfully, [AWS(Amazon Web Services)](https://aws.amazon.com/elasticbeanstalk/) has created a tool for deploying and scaling Rails web application in their eco-system. Here is an explanation from their landing page:
> AWS Elastic Beanstalk is an easy-to-use service for deploying and scaling web applications and services developed with Java, [.NET](https://aws.amazon.com/net/), PHP, Node.js, Python, Ruby, Go, and [Docker](https://aws.amazon.com/docker/) on familiar servers such as Apache, Nginx, Passenger, and [IIS](https://aws.amazon.com/windows/).
> You can simply upload your code and Elastic Beanstalk automatically handles the deployment, from capacity provisioning, load balancing, auto-scaling to application health monitoring. At the same time, you retain full control over the AWS resources powering your application and can access the underlying resources at any time.
> There is no additional charge for Elastic Beanstalk — you pay only for the AWS resources needed to store and run your applications.

## Getting Started

If you do not have an AWS account yet, go sign up, and you get a bunch of free services for a year on specific levels(i.e t1.micro server, etc). The tutorial is going to focus on MacOS, and I can expand if enough interest is shown for windows, though most rails based commands should work on any platform.

After your set up your user, get AWS’s CLI(here via brew):

    $ brew update
    $ brew install aws-elasticbeanstalk

Now create the rails app if you have not already

    $ **rvm or rbenv optional setup here**
    $ gem install rails -v 5.0.1
    $ rails new rails5app
    $ cd rails5app
    $ git init && git add -A && git commit -m "Init Commit Message"

…and now we’ll create an example scaffold

    $ rails g scaffold message subject body:text
    $ rails db:migrate
    $ git add .
    $ git commit -am "Add Messages"
    $ git push

Alright, we have a basic Rails app needed to continue with the Elastic BeanStalk deploy. Lets initialize ElasticBeanstalk

    $ eb init

    Select a default region
    1) us-east-1 : US East (N. Virginia)

    Select an application to use
    1)[ Create new Application ]

    Enter Application Name
    (default is "rails5app"):
    Application rails5app has been created.

    It appears you are using Ruby. Is this correct?
    (y/n): y

    Select a platform version.
    1) Ruby 2.3 (Puma)

    Do you want to set up SSH for your instances?
    (y/n): y

    Select a keypair.
    1) [ Create new KeyPair ]
    (default is 1): 1

That sets up our environment, uses US East(preferable if you want to use AWS free SSL services), Ruby 2.3 and lets you access your EC2 when it is up and running by typing ‘eb ssh’.

Now we will create the environment. This is basically how Elastic Beanstalk lets you switch between different versions of your application. Most will use this to set up different rails environments to be deployed(prod, staging, etc)

    $ git commit -am "Post EB init"
    $ eb create production

..and wait. Right now Elastic Beanstalk is setting *everything* up for you, loadbalancers, security groups, EC2 server and more. Seriously, you should probably go grab a coffee. Also, I have had the eb-cli log time out on me before, disconnect, but still finish the environment creation.

A quick status check afterwards will get you environment information and your ElasticBeanstalk URL for your application:

    $ eb status
    Environment details for: production
     Application name: rails5app
     Region: us-east-1
     Deployed Version: app-5cbc-170110_150338
     Environment ID: e-krea6tdvjx
     Platform: 64bit Amazon Linux 2016.09 v2.3.0 running Ruby 2.3 (Puma)
     Tier: WebServer-Standard
     CNAME: production.a3rjg2n9mm.us-east-1.elasticbeanstalk.com
     Updated: 2017–01–10 22:10:57.815000+00:00
     Status: Ready
     Health: Green

Though, going there now would give you errors. To save you any ulcers, it’s because there is no Rails secret setup. So, here’s how you do that AND add an ENV variable to Elastic Beanstalk instances:

    $ eb setenv SECRET_KEY_BASE=$(rails secret)
    INFO: Environment update is starting.
    INFO: Updating environment production's configuration settings.
    INFO: Environment health has transitioned from Ok to Info. Configuration update in progress. 1 out of 1 instance completed (running for 51 seconds).
    INFO: Successfully deployed new configuration to environment.

BOOM

From here on out, pushing code to your instance(s) is just an ‘eb deploy’ away.

![Your default rails chariot awaits](https://cdn-images-1.medium.com/max/2148/1*dEF3kF16SH_Jm37i4qvl7A.png)*Your default rails chariot awaits*

<iframe src="https://medium.com/media/dc9c58a2792e9075bf84b0fdc4e709d5" frameborder=0></iframe>

## The rest of the good stuff

I bet you’re saying, that’s great and all, but that’s not a production ready app. You’re right, let’s get postgresql, redis and sidekiq going! First up, Postgres. You’ll want to add the ‘pg’ gem to your gemfile and add the following to your database.yml:

    production:
        <<: *default
        adapter: postgresql
        encoding: unicode
        database: <%= ENV['RDS_DB_NAME'] %>
        username: <%= ENV['RDS_USERNAME'] %>
        password: <%= ENV['RDS_PASSWORD'] %>
        host: <%= ENV['RDS_HOSTNAME'] %>
        port: <%= ENV['RDS_PORT'] %>

What is all that magic? AWS Elastic Beanstalk will throw those ENV variables on the your ec2 instances as soon as you have an RDS instance setup. Lets do that:

* Go to [https://console.aws.amazon.com/elasticbeanstalk/home?region=us-east-1#/applications](https://console.aws.amazon.com/elasticbeanstalk/home?region=us-east-1#/applications)

* Click on the box for your application

* Click on ‘Configuration’, scroll down to the bottom to ‘Data Tier’ and click ‘Setup RDS instance’

* Setup RDS to your liking(postgres, t2.micro(if you want to stay in free tier, username, etc)

* Wait for AWS to do it’s thing. You may acquire another cup of coffee now.
> Note: A few readers have brought it to my attention that adding an RDS instance through the EBS interface binds it to the existence or destruction of your EBS environment. Setting up an RDS separately and manually adding the environment variable may have greater RDS assurance for some.

Now, another ‘eb deploy’ for your database config should get everything up and running through postgresql.

Now, lets add two configuration files most apps will need. First, adding git to the deploy. Add a folder .ebextensions, and a file ruby.config

    packages:
     yum:
      git: []

Now, when you get a few more gems in your app, especially those that insall native extensions, you may get failed deploys from running out of memory. Since, most apps can run on the free tier, and the only time memory is an issue is during a deploy, we will enable swap. Add a file 0001_setup_swap.config in the .ebextensions folder with the following text:

    commands:
      000_dd:
        command: echo “noswap”#dd if=/dev/zero of=/swapfile bs=1M count=3072
      001_mkswap:
        command: echo “noswap”#mkswap /swapfile
      002_swapon:
        command: echo “noswap”#swapon /swapfile

Sweet, we’re almost done. The last piece of almost any production Rails app will be sidekiq(with Redis…which means we can also tie ActionCable into this Redis as well). Let’s start by adding Redis in AWS:

* Go to [https://console.aws.amazon.com/elasticache/home?region=us-east-1#redis](https://console.aws.amazon.com/elasticache/home?region=us-east-1#redis):

* Click create, and fill out the pretty straightforward form(picking t2.micro for free tier again).

* Click Save and go for a water this time while you wait…three coffees are a lot.

* Once created, clicking on the instance on the list will take you to a page with an endpoint listed in the table(it should look like your-name.iz6wli.0001.use1.cache.amazonaws.com)

Almost done! We just need to add a couple config files for sidekiq, redis and elastic’s deploy! First here is how I like to set up Sidekiq/Redis config:

In config/initializers/sidekiq.rb

    rails_root = Rails.root || File.dirname(__FILE__) + ‘/../..’
    rails_env = Rails.env || ‘development’
    redis_config = YAML.load_file(rails_root.to_s + ‘/config/redis.yml’)
    redis_config.merge! redis_config.fetch(Rails.env, {})
    redis_config.symbolize_keys!
    Sidekiq.configure_server do |config|
     config.redis = { url: “redis://#{redis_config[:host]}:#{redis_config[:port]}/12” }
    end
    Sidekiq.configure_client do |config|
     config.redis = { url: “redis://#{redis_config[:host]}:#{redis_config[:port]}/12” }
    end

…and config/redis.yml

    development:
     host: ‘localhost’
     port: ‘6379’
    test:
     host: ‘localhost’
     port: ‘6379’
    production:
     host: ‘your-name.iz6wli.0001.use1.cache.amazonaws.com’
     port: ‘6379’

*optional* config/cable.yml

    development:
     adapter: async
     
    test:
     adapter: async
     
    production:
     adapter: redis
     url: redis://your-name.iz6wli.0001.use1.cache.amazonaws.com:6379

…lastly, for Elastic Beanstalk .ebextenstions/0002_sidekiq.config([gist](https://gist.github.com/polysaturate/98dbca462e25604d6b18f38dc8380757))

    # Sidekiq interaction and startup script
    commands:
      create_post_dir:
        command: "mkdir -p /opt/elasticbeanstalk/hooks/appdeploy/post"
        ignoreErrors: true
    files:
      "/opt/elasticbeanstalk/hooks/appdeploy/post/50_restart_sidekiq.sh":
        mode: "000755"
        owner: root
        group: root
        content: |
          #!/usr/bin/env bash
          . /opt/elasticbeanstalk/support/envvars

    EB_APP_DEPLOY_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k app_deploy_dir)
          EB_APP_PID_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k app_pid_dir)
          EB_APP_USER=$(/opt/elasticbeanstalk/bin/get-config container -k app_user)
          EB_SCRIPT_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k script_dir)
          EB_SUPPORT_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k support_dir)

    . $EB_SUPPORT_DIR/envvars
          . $EB_SCRIPT_DIR/use-app-ruby.sh

    SIDEKIQ_PID=$EB_APP_PID_DIR/sidekiq.pid
          SIDEKIQ_CONFIG=$EB_APP_DEPLOY_DIR/config/sidekiq.yml
          SIDEKIQ_LOG=$EB_APP_DEPLOY_DIR/log/sidekiq.log

    cd $EB_APP_DEPLOY_DIR

    if [ -f $SIDEKIQ_PID ]
          then
            su -s /bin/bash -c "kill -TERM `cat $SIDEKIQ_PID`" $EB_APP_USER
            su -s /bin/bash -c "rm -rf $SIDEKIQ_PID" $EB_APP_USER
          fi

    . /opt/elasticbeanstalk/support/envvars.d/sysenv

    sleep 10

    su -s /bin/bash -c "bundle exec sidekiq \
            -e $RACK_ENV \
            -P $SIDEKIQ_PID \
            -C $SIDEKIQ_CONFIG \
            -L $SIDEKIQ_LOG \
            -d" $EB_APP_USER

    "/opt/elasticbeanstalk/hooks/appdeploy/pre/03_mute_sidekiq.sh":
        mode: "000755"
        owner: root
        group: root
        content: |
          #!/usr/bin/env bash
          . /opt/elasticbeanstalk/support/envvars

    EB_APP_USER=$(/opt/elasticbeanstalk/bin/get-config container -k app_user)
          EB_SCRIPT_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k script_dir)
          EB_SUPPORT_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k support_dir)

    . $EB_SUPPORT_DIR/envvars
          . $EB_SCRIPT_DIR/use-app-ruby.sh

    SIDEKIQ_PID=$EB_APP_PID_DIR/sidekiq.pid
          if [ -f $SIDEKIQ_PID ]
          then
            su -s /bin/bash -c "kill -USR1 `cat $SIDEKIQ_PID`" $EB_APP_USER
          fi

One more ‘git commit’ and ‘eb deploy’ and you are good to go. Aw yeah!

![](https://cdn-images-1.medium.com/max/2272/1*0hqOaABQ7XGPT-OYNgiUBg.png)

![](https://cdn-images-1.medium.com/max/2272/1*Vgw1jkA6hgnvwzTsfMlnpg.png)

![](https://cdn-images-1.medium.com/max/2272/1*gKBpq1ruUi0FVK2UM_I4tQ.png)
> [Hacker Noon](http://bit.ly/Hackernoon) is how hackers start their afternoons. We’re a part of the [@AMI](http://bit.ly/atAMIatAMI) family. We are now [accepting submissions](http://bit.ly/hackernoonsubmission) and happy to [discuss advertising & sponsorship](mailto:partners@amipublications.com) opportunities.
> If you enjoyed this story, we recommend reading our [latest tech stories](http://bit.ly/hackernoonlatestt) and [trending tech stories](https://hackernoon.com/trending). Until next time, don’t take the realities of the world for granted!

![](https://cdn-images-1.medium.com/max/30000/1*35tCjoPcvq6LbB3I6Wegqw.jpeg)
