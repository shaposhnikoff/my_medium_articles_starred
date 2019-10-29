
# SSO on AWS in 5 Minutes



Like everything on AWS, there are multiple ways to achieve similar outcomes. Recently I wrote an article about setting up [SSO into your AWS Accounts using SAML](https://medium.com/swlh/securing-your-aws-accounts-with-saml-authentication-2c1435758d8c) — specifically ADFS. Although most larger or established companies utilize Active Directory for authentication, smaller companies or startups may choose not to so they don’t have to worry about yet another tool or system to manage. So what options are there if you still want to utilize a centralized login system? One of those options is [AWS SSO](https://aws.amazon.com/single-sign-on/), which also happens to be a free service.

### Getting Setup

Before you get started, note down [AWS SSO’s Prerequisites](https://docs.aws.amazon.com/singlesignon/latest/userguide/prereqs.html) as these are important. All of the following steps will also need to be performed in the organization’s master account.

In the AWS Console, search for and open the SSO service and you should see the option to Enable AWS SSO. Similar to implementing SAML on AWS, AWS SSO also uses the [Secure Token Service](https://docs.aws.amazon.com/STS/latest/APIReference/Welcome.html) to provide access to each account. As part of the setup process, a number of IAM roles will be created in each member account to allow SSO authentication to occur.

![](https://cdn-images-1.medium.com/max/5012/1*5NByxDOkwlF7GvCnZXNT-w.png)

![](https://cdn-images-1.medium.com/max/5012/1*Jb28r428dQa8pzWrck8ClA.png)

After SSO is enabled, we need to create a user (or users) who will be able to access the AWS accounts within the organization. The SSO service can also use Active Directory as the source for your user information if you wanted, but I’m going to use the standalone version here as you won’t need to worry about setting up connectors to make that work. Security across all accounts is also granular, with the ability to grant different levels of access to different users, groups and accounts. In my example, I’m only going to add a single user and an Administrators group which has access to all accounts within my organization.

![](https://cdn-images-1.medium.com/max/5020/1*fcx_16NE1N4QQmmS_oU0ZQ.png)

![](https://cdn-images-1.medium.com/max/5008/1*1v19EF5s0--jrOGs06s7Uw.png)

![](https://cdn-images-1.medium.com/max/5012/1*vE4jiN7bPbDHIsxMMZ-QLA.png)

Next, we select the AWS Accounts tab which will show a list of the accounts under my organization. I want to assign my Administrators group to all of these accounts so I can gain access as an administrator to them all, but you could also choose to assign individual users also. In the assign users interface, select the Groups tab and the Administrators group we want to assign to all of the accounts.

![](https://cdn-images-1.medium.com/max/5016/1*IugPPACgmrFWwrmvFDL5fA.png)

![](https://cdn-images-1.medium.com/max/5012/1*sZz4fDLWyx18IbfUnGbEHQ.png)

The next step is to add Permission Sets. Permission Sets control the level of access a user or group will have when authenticated. There are two ways of assigning permissions, you can either use pre-defined levels of access or create a custom policy in the same way you would in the IAM console. One thing to note with using the predefined roles is the STS token expires after 1 hour by default, which can be annoying when interacting with the API. I usually extend the expiration period to something less annoying after creating a permission set or a custom role with an extended expiry time.

![](https://cdn-images-1.medium.com/max/5020/1*_zIwcKNrS9oMUY5M6MLuWw.png)

![](https://cdn-images-1.medium.com/max/5016/1*muefzfiSqu8PT5gM84kHuw.png)

![](https://cdn-images-1.medium.com/max/5012/1*DvnZzRXOMYzwd_1uI965Pg.png)

We’re all set! Let’s try logging in to the URL for your organization, this will generally be something like https://<name or id>.awsapps.com/start. This URL can also be customized to be something more user-friendly for your team to remember.

![](https://cdn-images-1.medium.com/max/5016/1*wuOr4T_xjkafyK1JjLcUUg.png)

After logging in, there will be an AWS Account icon which will show a list of AWS accounts your user has access to. Each account listed can authenticate into the console or gain programmatic access with API credentials.

![](https://cdn-images-1.medium.com/max/5012/1*FH_FYxPrUj8385v46YAcFA.png)

Authenticating with your API credentials is similar to creating an IAM user and utilizing an Access and Secret Access Key, however, because you’re inheriting a role (which generates a temporary set of credentials) you also have a third item, a session token. The session token will only be valid for a specific period, after that time it will cease to work and you’ll need to re-authenticate to get a new set of credentials.

![](https://cdn-images-1.medium.com/max/5016/1*hJstmx6bkEy27f3F36xzCA.png)

Console access works as you’d expect — you can see the role you’ve assumed in the header bar of the console and will look something like *AWSReservedSSO_AdministratorAccess_xxxxxxxxx/gavin@domain.com*

![](https://cdn-images-1.medium.com/max/5016/1*L3_5DO_hTMM8PrrBlNiA4g.png)

### Conclusion

AWS SSO is a quick and easy way to get SSO up and running across lots of accounts and removes the need to utilize apps like saml2aws for programmatic access with assumed user roles. If centralized management of user accounts is important to you (and is often required for several compliance programs), I’d recommend integrating the SSO service with Active Directory, or at the very least, use AWS’s APIs to automate the on-boarding and off-boarding of accounts.
