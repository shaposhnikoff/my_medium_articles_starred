
# Running cron jobs on Amazon Web Services (AWS) Elastic Beanstalk

I love Amazon’s Elastic Beanstalk (EB) product on Amazon Web Services (AWS). It is essentially the perfect balance of Infrastructure-as-a-Service and Platform-as-a-Service. It gives you the ability to deploy applications with ease, auto-scale as you need to, but effectively just wraps this all up in plain old Elastic Compute Cloud (EC2) containers so you can tune and tweak to your heart’s content should you ever want (or need) to.

We use EB extensively at [Vearsa](https://www.vearsa.com), and one issue that comes up frequently is how to run scheduled background tasks or jobs in an EB enviornment. There are plenty of options of varying complexity —[ OpsWorks](http://aws.amazon.com/opsworks/), [Data Pipeline](http://aws.amazon.com/datapipeline/), a dedicated EC2 micro instance for running cron (!!!) or using **cron.yaml**. In this post, we’ll explore how to set up scheduled tasks using the cron.yaml configuration file in an EB Worker Tier environment.

### Elastic Beanstalk Worker Tier

In order to set up a cron job on EB, you’ll need an [Elastic Beanstalk Worker Tier](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features-managing-env-tiers.html) application. This is effectively an environment for running long-running tasks in the background, and is typically used with Amazon’s [Simple Queue Service](http://aws.amazon.com/sqs/) (SQS) product when these tasks are fired from the front-end of an application. You configure your EB Worker to listen for items placed on an SQS queue. When an item is picked up from the queue, a POST request is sent to a given endpoint in your EB Worker app, with the request body containing the queue item’s message body.

### Cron jobs in an EB Worker app

Cron jobs in EB work similarly, but without relying on SQS. Instead, you set up a POST endpoint in your EB Worker app, and in this place the code you want to execute on a scheduled basis. At the root of your project, you create a file named **cron.yaml**, which allows you to set up the scheduling. The format of this file is as follows:

    version: 1
    cron:
     — name: "schedule"
       url: "/schedule"
       schedule: "0 */12 * * *"

The main part of this file to focus on is the **cron** section. This comprises three fields:

* **name:** A unique label for your cron job

* **url:** The URL endpoint to send a POST request to each time the job runs. You can have multiple jobs that point to the same URL.

* **schedule:** The cron expression that defines how frequently the job should run.

In this example, I have created a cron job named **schedule** that will fire a POST request to the **/schedule** endpoint in my EB Worker app **on the hour every 12 hours**.

If you need multiple cron jobs, you can do this by having multiple blocks in the **cron** section of this file, for example:

    version: 1
    cron:
     — name: "schedule"
       url: "/schedule"
       schedule: "0 */12 * * *"
     - name: "backup"
       url: "/backup"
       schedule: "0 0 * * 0"

The **backup** job will fire a POST to **/backup** at midnight every Sunday.

### Cron Expression Syntax

The third field in each cron job uses a *cron expression* to define when the job should run. This expression contains five space-separated items:

1. minutes

1. hours

1. day of month

1. month

1. day of week

You can use the wildcard character ***** to denote that the job should run on every occurrence. You can also set ranges using hyphens, multiple values using commas and special combinators such as my **schedule** example job above ***/12**, which will run every 12 hours. Here are some examples:

    * * * * *      // Run every minute
    0 * * * *      // Run on the hour, every hour
    0 23 * * *     // Run at 11 p.m. every day
    0 0,12 * * 6   // Run at midnight and midday every Saturday
    30 2 */2 * *   // Run every second day at 2.30 a.m.

### Checking the cron job has been scheduled

When you’ve added your cron job(s) to **cron.yaml**, you need to deploy your EB Worker app for the jobs to be loaded and scheduled. When you deploy the app, the EB console will log an **INFO** event with a message similar to the following to confirm jobs have been scheduled successfully:
> Successfully loaded 1 scheduled tasks from cron.yaml.

In the web-based AWS console for EB, this looks like the following:

![AWS web console for EB showing confirmation message for loading of scheduled jobs](https://cdn-images-1.medium.com/max/4736/1*h7W4SxXqk1FfHknmM3qn4A.png)*AWS web console for EB showing confirmation message for loading of scheduled jobs*

If you deploy your app via the command line, you’ll see the same message appear in the output from the **eb deploy** command. A message will also be logged in the **/var/log/eb-commandprocessor.log** file.

Note that the cron configuration will be reset on each deployment, so if you need to change the frequency of the job, you can simply change the **cron.yaml** file and re-deploy your app.

Thanks for reading.
