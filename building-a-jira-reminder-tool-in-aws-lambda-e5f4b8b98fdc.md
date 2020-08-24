
# Building a Jira Reminder Tool in AWS Lambda

Here at Neighborhoods.com we make extensive use of Jira as an engineering task management platform. However, like many developers, I’m not as diligent at updating my tickets as I should be. To solve that problem, I built a simple reminder tool using AWS’s serverless Lambda functions that sends personalized reminders to each member of the team. A late afternoon email gives each developer a snapshot of their current ticket statuses and remaining hours, linked to the tickets in Jira. Let’s walk through the steps to set up such a tool on Lamba.

![A sample reminder email linking to your tickets](https://cdn-images-1.medium.com/max/3268/1*ZTAan6csT2s1qA1NfiZMHg.png)*A sample reminder email linking to your tickets*

## Why Lambda?

A serverless function is the perfect platform for such a lightweight tool as we only need our code to be run for about ten seconds each day. AWS Lambda lets you write your function in Node, Python, Java, .Net Core or Go.

For this tool I used Python, if only to be consistent with our other Lambda functions, but any of the supported languages should work just a well. It’s important to note that Lambda has a substantial but limited choice of Python libraries that are included out of the box. Boto libraries, which include most of what one might want when interacting with the AWS ecosystem, come standard. For production use, consider packaging your own copy of Boto to avoid any unpredictable results — something we have learned from experience. In fact, almost any library can be packaged in your code, as long as our total zip is under 50MB and you built your packages in a flavor of Linux compatible with AWS Linux.

## Setting up a role

First things first, you will need to create an [IAM role](https://console.aws.amazon.com/iam/home#/roles). At a minimum the role will need Lambda execution permissions, as well as access to any other AWS utilities your function will use, which in our case is just SES.

![Setting your IAM role permissions](https://cdn-images-1.medium.com/max/2504/1*jp3nziLT5Un3L3cxAKfuVw.png)*Setting your IAM role permissions*

## Creating a Lambda function

To begin, head to the [AWS Lambda](https://console.aws.amazon.com/lambda/home) service and create a new function. Name your function and select the appropriate runtime and role.

By default, a lambda_handler function, with event and context arguments, is the initial method run when the Lambda function is triggered. Information about the triggering event is passed in, though in this case you don’t need event data for the reminder tool. The function just needs to do a few discrete tasks using data fetched from Jira.

<iframe src="https://medium.com/media/087d2f0888e72eadafa7cdb251758fe4" frameborder=0></iframe>

**Task 1:** Reach out over the [Jira API](https://docs.atlassian.com/jira-software/REST/latest) and fetch all of the ticket data for our team. For now it will just fetch assigned tickets that are ‘In Progress’ or in ‘Code Review’ as these are the tickets that are likely to need updating.

<iframe src="https://medium.com/media/cf6c75ea55d2c2acb5ed991ecf17c921" frameborder=0></iframe>

**Task 2:** As the method loops through the Jira data (snippet 3 below), the get_ticket_data method (snippet 4) will parse out the ticket assignee, key (e.g. PROJX-1234), status and time remaining (note: the Jira API uses seconds), then arrange tickets by assignee.

<iframe src="https://medium.com/media/43daa183b4bca89c44a671549dbc9ad8" frameborder=0></iframe>

<iframe src="https://medium.com/media/e9e1fe429cf3e70e3dbe494ea949de67" frameborder=0></iframe>

**Task 3.** For each assignee, generate html and plain text emails based on their ticket assignments, then send out the email using SES. I don’t think there is anyone on our team that uses a text-only email client, but including a text only version can reduce the likelihood of the email being treated as spam.

<iframe src="https://medium.com/media/04deaa93d1454b8484c5cf7d54684efb" frameborder=0></iframe>

That’s most of the code you’ll need. You can find the complete version on [Github](https://github.com/neighborhoods/JiraReminder).

## Setting triggers

Lambda functions can be triggered by any number of a wide array of events and conditions occurring within the AWS infrastructure. However, for this little tool we just want it to be run on a schedule, once a day like a cron. Cron-like behavior is set using CloudWatch Events, which can be added and configured in the Lambda console. However, the UI is far better when using the [CloudWatch Event rule creation interface](https://console.aws.amazon.com/cloudwatch/home#rules:action=create), so I’d recommend clicking save on your Lambda function and opening [CloudWatch](https://console.aws.amazon.com/cloudwatch/home#rules:action=create) in a new tab. If it’s your first time creating a CloudWatch Event rules, click around and check out the variety of event and scheduling options. Our Jira reminder tool runs on a schedule using a cron expression. If your cron syntax is correct you should see the next ten trigger dates. On the right hand side we can then attach the event rule to our Lambda function.

![CloudWatch Event Rules setup](https://cdn-images-1.medium.com/max/6264/1*vlJ9Yiq_PtHg4EOk7P0chg.png)*CloudWatch Event Rules setup*

## Setting environment variables

Values that we don’t want committed in the function code, such as our Jira username and password, can be passed in as environment variables. These can be can be set in the Lambda console.

![Environment vars are pulled into the code using os.environ[{var_name}] (see Snippet 2 lines 5–6)](https://cdn-images-1.medium.com/max/3852/1*Nb4_aH54_ky2kR0Arb5giA.png)*Environment vars are pulled into the code using os.environ[{var_name}] (see Snippet 2 lines 5–6)*

A quick disclaimer before testing. In the interests of simplicity we’ve taken the shortest path to deploying live code, which involves using the AWS console. For a controlled, scripted implementation, a tool such as [CloudFormation](https://console.aws.amazon.com/cloudformation/home), [Terraform](https://www.terraform.io/) or [Serverless Framework](https://serverless.com/) is recommended.

## Testing

We will want to test any Lambda function before it is active in production. In the top-right of the Lambda console there is a ‘test’ button, which wont activate until a test configuration is created. If the Lambda function was event-driven we’d want to set up test configurations to simulate any pertinent events. Our Jira tool is run on a schedule, so just create a dummy event, or scroll down on the event template list and use the ‘hello world’ setup. Once saved we can trigger our function with a click. Lambda functions will write to CloudWatch logs, but when testing the execution output will be displayed in the console, right below the function code. Note: in order not to spam your teammates, don’t forget to override the email_address param that’s passed into send_reminder_email.

## Extra credit

We can easily extend this tool by adjusting the Jira API search parameters. For instance, by including tickets with other statuses, or perhaps only including tickets that have not been updated within the last 24 hours. What about tickets in the sprint that are unassigned? Perhaps they get included in the email to the Jira board owner.

## Conclusion and source code

There you have it. You can see how easy it is to start running serverless code on AWS Lambda. Feel free to use or improve on this lightweight but [useful tool](https://github.com/neighborhoods/JiraReminder).
