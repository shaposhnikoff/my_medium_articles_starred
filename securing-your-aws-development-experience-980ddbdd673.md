
# Securing Your AWS Development Experience

Silly security graphic. Why not.
> # AWS Nomads #1**. **This is the first article in an ongoing series on AWS development for scrappy developers.

AWS can be a maze of services and options. This is a simple guide to developing securely on AWS. Future essays will discuss various aspects of developing secure applications on AWS, which has many more variations for different application and service types.

## Step 1: Secure Your Root Account

Your “Root Account” is the login that has authoritative access to everything. Until you explore the mysteries of IAM, all you have is a root account. You need to keep a root account. Here’s how to secure it.

First of all, good old email and login are simply not secure enough. You can get away with it for years, and never have a problem: but it’s a matter of when, not if.

So, from your AWS console, go to your account name in the top bar, and select “My Security Credentials.”

You may need to get past a notice about IAM users. Don’t worry about that for now, we’ll come back to that.

You should land on a page that looks something like this:

![](https://cdn-images-1.medium.com/max/5904/1*FF1MkAFT14tpqPPJQUIIkA.png)

The important configuration here is Multi-factor authentication (MFA). Everything below that section is more useful for developing secure applications, rather than simply securing your account. We’ll come to those in future essays.

You are already familiar with MFA. Anytime a login sends a code to your phone or email to make sure you are really you, that is MFA. However, Amazon goes with a considerably stronger MFA than that.

Let’s turn it on:

![](https://cdn-images-1.medium.com/max/4780/1*pjDEVmRuC-CgIBnSkTpaYg.png)

![](https://cdn-images-1.medium.com/max/4904/1*Dhnkoo2o1C71iUYv75xkNQ.png)

Virtual MFA device is sufficient. Yubikey and gemalto tokens are hardware devices that you need physical possession of to verify your identity. The virtual MFA device is an application you can run on your machine (or on a secondary device, such as a phone). All of these work in essentially the same way: they are aligned with Amazon’s systems during the process to follow, and then they have a time-based code that you will provide during login. For someone to abscond with your account they would need your login, password, AND physical access to the code-generator. It’s still not impossible to beat, so if you ever lose control of your code-generator, the first thing you want to do is reset your AWS account.

For a virtual MFA device, I prefer Google Authenticator.

![](https://cdn-images-1.medium.com/max/4860/1*N563p76NhDr3cfoy025MQg.png)

If your app can scan that code, go for it. Otherwise cut and paste (or god forbid, type) that secret key into your authenticator app.

Then, put two sequential MFA codes in the two fields. If you have done this correctly, you will be all set up!

![](https://cdn-images-1.medium.com/max/4928/1*A8Ldh83gRcr3AhLouxyieg.png)

Now that you have secured your root account, it’s time to stop using it!

## Step 2: Using IAM Credentials

At a minimum, I prefer to use three separate login credentials: one for managing billing concerns, one for development, and one for operations (deployment, monitoring, etc.). In a team context, you could have these different logins handled by different people, but even though I am just one person, I still prefer to use these different contexts.

The rationale is, even if one account were somehow compromised, let’s say because I am all logged in and I get up from my spot in the cafe without securing my screen, and some sneaky hacker slips into my chair.

Well, at least the damage is constrained!

With the IAM system, we create groups (associated with policies) and users. Let’s set up our login for managing account billing.

1. *Go to IAM*. On the AWS console, as the root user, under the “Security, Identity, and Compliance” heading, select IAM.

1. *Check your Security Status*. The dashboard at IAM will give you a little report on your security. If there’s anything that’s not green, you will want to take care of that in due course.

1. *Create a Group*. It is most convenient to associate the access policies with a group, rather than a user, although it can work either way. By associating it with a group, if you *do* add users in the future, you already have clean organization of your policies ready to roll. For this example, I am going to create a “Finance” group:
 — Select Groups in the left menu;
 — Press the “Create New Group” button to start the wizard;
 — Name the group (mine is “Finance”), and select Next Step;
 — Associate a policy (mine is “Billing”), and select Next Step (See also the subsection: “A Word About Policies” below);
 — Create Group.

1. *Create the User. *This is the user you will log in as to view costs and manage any billing details.
 — Select Users from the left menu;
 — Press the “Add User” button to start the wizard;
 — Name your user. Mine is, inventively, ‘billing’;
 — Specify access type, programmatic or console. This is interesting: for our use case, we are going to give the user access to the AWS Management Console. But you can see that when it comes to running applications, we can specify what user that application will run as and generate access keys for inter-service authentication. Another essay will cover this.
 — Create a password. If this is just for you, you might want to create the password here and uncheck the box for forcing a password reset on first login.
 — Press the “Next: Permissions” button;
 — Here we can add the ‘billing’ user to the ‘Finance’ group. As you can see, there are other ways to set permissions.
 — “Next: Tags” — tags might be useful in extremely large and complicated collections of users, but for our purposes, we’ll skip this.
 — “Next: Review”
 — “Create User”
Voila!

1. *The Billing User is a Little Special*. Unlike just about everything else you will ever use IAM for, the billing use case has one additional wrinkle. Even though we aligned this user with the “Billing” policy that gives access to all the billing services, if you were to log in with this user now, you would not have those permissions!

1. *Enable IAM Access to Billing Controls*.
 — Go to the account drop down on the top bar;
 — Select “My Account”;
 — Scroll down to the section: “IAM User and Role Access to Billing Information”;
 — Edit and enable this access.

### A Word About Policies

The heavy listing in permissions and restrictions is, naturally, in the policies, and as you may have noticed when selecting the policy, there is an absolute maze of choices:

![](https://cdn-images-1.medium.com/max/4684/1*zycwYMFiIRySsF2zH-rAQg.png)

For some use cases, there is one handy subset which you can view by selecting the “Policy Type” filter, and changing it to “AWS Managed — Job Function,” which offers this more manageable selection of choices:

![](https://cdn-images-1.medium.com/max/4776/1*bAKrMc0V4FIsaSPSIDMPUA.png)

When you go to create a your developer account, the “PowerUserAccess” is reasonably appropriate, unless you really want to restrict which services you can access as a developer. In that case, it becomes simpler to simply create policies, rather than scrounge through these options seeking one that matches your specific use case.

## Where To Go From Here?

Now that you have a secure account for developing on Amazon, future essays will be linked here to provide some next steps for mastering the Amazon tools.

![My bits are in here somewhere. Has anyone seen my bits?](https://cdn-images-1.medium.com/max/2048/1*2G65vQVIJUkToMveNVyRXw.jpeg)*My bits are in here somewhere. Has anyone seen my bits?*
