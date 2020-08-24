Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m308[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m1577[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m650[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m943[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m206[39m }

# A Detailed Look at AWS Cognito

Access control is often a significant challenge in every application. We use access control to identify who the user is, grant access to the information they are allowed to see and prevent access to everything else.

Application developers have better things to do than figure out how to implement access control, which is why there are frameworks and services to perform this task. In this article, I am going to explore configuring user pools in AWS Cognito, and administrator management of the user accounts in the user pool.

### AWS Cognito: The What and Why

I have pretty much explained what AWS Cognito is; a reliable, scalable, user sign-up and authentication service. AWS Cognito ties itself to an authentication directory, where user account data is stored. Alternatively, you can federate AWS Cognito so the actual authentication is performed by social media authentication engines like Google, Facebook, and Amazon, or by a Security Assertion Markup Language (SAML) complaint enterprise authentication service or exiting Microsoft Active Directory.

[AWS](https://aws.amazon.com/cognito/) defines the benefits of AWS Cognito as:

* Secure and Scalable User Directory — A fully managed directory service capable of scaling to millions of users.

* Social and Enterprise Identify Federation — The ability to connect the user authentication to services such as Google, Facebook, Amazon or an enterprise service, removing the need to maintain the user authentication data outside of the federated relationship.

* Standards Compliant — AWS Cognito supports OAuth 2.0, SAML and OpenID Connect authentication standards, so you don’t have to implement these in your solution.

* Security for your applications and users — AWS Cognito supports multi-factor authentication (MFA), and encryption of data both in-transit and in-storage. AWS Cognito is HIPAA eligible and PCI DSS compliant.

* Access Control for AWS Resources — Whatever resources you create in AWS, access to those resources can be restricted based upon the roles and user mapping you provide for that resource.

* Customizable UI and Easy Integration — AWS Cognito has a customizable UI, so you don’t even have to write the code to collect the user’s credentials — you can use and customize the one provided as part of AWS Cognito. Integration within your application can be done in minutes.

When using AWS Cognito with a *User Pool*, the directory storing the user authentication data is managed for you, without needing to monitor or manage the underlying infrastructure. This means application developers and SRE teams can focus on their primary function and not the deployment and management of the user pool directory.

Alternatively, *federation* allows you to tie the authentication to exiting directory, so your application relies upon the authentication infrastructure provided by that service and you don’t need to worry about it all. This effectively means if the other service is experiencing an outage or other failure, your application won’t be able to authenticate the users. There are cases for both solutions.

The Software Development Kit lets you integrate AWS Cognito with

* iOS Objective C applications;

* IOS Swift applications;

* Android applications;

* REACT native applications and,

* Web applications.

### Setting Up AWS Cognito

In this section, we are going to look at configuring a user pool. After logging in to the console and navigating to the AWS Cognito, we have this view.

![](https://cdn-images-1.medium.com/max/NaN/1*cAphD1s5Dc_NxFqW2wi1HQ.jpeg)

Since we have no user pools now, we will create one. There are two options: (a) to use the defaults, and (b) step through every option. For our example here, we will take the default approach.

**NOTE** Once you create the user pool, many values, particularly attributes, **cannot** be changed. If you made a mistake and need to change an attribute setting, the user pool must be recreated. Consequently, some care should be taken to select the correct configuration before users are added.

Even during the research and writing of this article, I re-created the sample AWS Cognito User Pool multiple times.

Let’s examine each of the groups of attributes for our user pool. Once we define the name of the user pool, it cannot be changed. In each of the sections, there is a pencil icon, allowing you to see all of the details for the items in the section.

![](https://cdn-images-1.medium.com/max/NaN/1*Q7NN0GPMGSycOPuQ_zMteQ.jpeg)

Let’s look at each of these sections and discuss what they mean. Let’s allow the user to login using their email address.

![](https://cdn-images-1.medium.com/max/NaN/1*IAxZLEqZ0QvzdVv2pIAhug.jpeg)

**WARNING:** If you allow users to login using a user name, you are responsible for creating unique user names. If you are planning to have a very large AWS Cognito user pool, you need to collect sufficient attributes to create a unique user name.

Alternatively, we can allow users to login with a username. With the username option, we can allow users to also log in with their email address, phone number or preferred username. However, you **must** make this selection when creating the user pool, as it cannot be changed later.

The username or email address/phone number options are mutually exclusive. If you choose to allow the users to use their email address or phone number for their username, they cannot have a “username” per se.

![](https://cdn-images-1.medium.com/max/NaN/1*pZdzfw4EKnvUsIPQSH55YQ.jpeg)

The standard attributes identify the information needed for user profiles, but if specified here, they form the required information needed when the user signs up. It is also possible to add custom attributes, where you specify the type of attribute, it’s name, the minimum and maximum length of the attribute and whether or not the attribute value can be changed later.

**WARNING:** Selecting any of the standard attributes will cause problems if you are going to add users through the console. This is because the console add user form only has username, invitation method, password, phone number, and email address. If any other attributes are selected, you will not be able to add users through the console, as there are no provisions in the form for the other attributes.

For example, if the user pool is being used to provide logins for users at other companies, you may want to add a custom attribute for Company Name. This would allow you to easily find users belonging to a specific company. An alternative would be to use groups. Groups may be an easier solution for this example, as you can associate an IAM role which each group.

Let’s move on using the configuration and attributes selected above.

### Password Policies

The second step is to define the password policy. If you want to allow users to be able to sign themselves up, and how quickly temporary passwords should expire.

![](https://cdn-images-1.medium.com/max/NaN/1*TJa277WDcJBIr4ZJ9zwQRg.jpeg)

The default password policy is fairly consistent with industry norms. I would not recommend making the password length shorter or disabling the other options for passwords. The second option defines if users can sign-up directly to AWS Cognito, or if they must be added by an administrator.

There is flexibility in allowing the user to sign up themselves, but if you want control over who can join the user pool, this is not a good selection. By requiring an administrator to join the user to the user pool, you control who can join and under what conditions. If you opt to have an administrator create the users, then you will need to provide some automation to perform this task. For this discussion, I am choosing to have users added by an administrator.

### Multi-Factor Authentication

The next step is to configure multi-factor (MFA) or two-step authentication. This option provides better security for the end users, as it requires a second piece of information — the security challenge — to be entered to log in. The security challenge is sent via Short Message Service (SMS) to the user, who must provide the security code to complete the login.

![](https://cdn-images-1.medium.com/max/NaN/1*ABWn7s9SedbKTlyNC75voQ.jpeg)

There are three options for Multi-Factor authentication: (a) off; where MFA is disabled., (b); optional; where MFA can be on for certain users and off for others, and © required; where every user has MFA enabled.

Selecting “optional” for MFA is a valid option if you only want certain users to have MFA enabled, or if you are going to enable risk-based or adaptive authentication. Adaptive authentication will require MFA if multiple failed login attempts occur, or the login request is from a new device or location. AWS Cognito associates a risk score using failed logins, location and device to determine if MFA will be needed for this authentication attempt.

When selecting optional MFA, you must choose if the second factor will be an SMS message or a time based One Time Password value. AWS recommends the decision be a time-based one time password, which allows SMS to be used as a password recovery option.

If SMS is being used for MFA, then you will be charged for the SMS messages sent to the user. SMS pricing varies from region to region. The cost of an SMS message in the United States is $0.00645. This is a cost that needs to be factored into the budget.

With MFA selected as either optional or required, the user must be verified using either their email address or phone number or both. Any of these options is a valid selection, and the choice is yours. For this example, I am going to select verify the phone number.

Finally, AWS Cognito needs to have an IAM role configured giving AWS Cognito permission to send SMS messages. A default role name is selected and populated in the field. Modify the default IAM role name if you choose, and click “Create Role”.

**NOTE:** If you already have an IAM role for SMS verification, it cannot be used here. You must create a new role.

### Email Configuration

If AWS Cognito will be sending email messages on your behalf, you may need to configure AWS Simple Email Service. AWS Cognito is limited to 50 email messages per day for user pool, and 500 email messages per day per AWS account. If you anticipate exceeding these hard limits, you will need to configure Simple Email Service.

The message customization section in the user pool configuration allows you to modify the text sent to the user when they are added to the user pool. When the user clicks the link in the invitation email, they can complete the sign-up process by providing the other attributes specified when the user pool was created. You can modify these messages as you see fit.

When you are creating the user pool and click on the “Next Step” button, you will be shown the “Review” page. Let’s cover the remaining pages for completeness, as you will visit them after the user pool is created.

### Devices

AWS Cognito can track the devices user’s access. You can opt to *always* track every device the user logs in from; *opt-in*, where the user can choose to remember the device they are logging in from, or *never* track the device. Any of the options are valid and the decision is yours,

### App Clients

Every application wishing to authenticate through the user pool must have the pool ID, a unique ID for the application and an optional secret key. When the user pool is being created, we wouldn’t normally add an app client.

### Triggers

Triggers allow more advanced configurations using AWS Lambda functions to provide additional, dynamic actions when the trigger is executed. Your user pool may never have any of these triggers defined.

### Review

After you have completed all of the configuration, the “Review” page is where you validate your responses and click “Create” to deploy the user pool. Once created, a message is displayed indicating the user pool has been created.

![](https://cdn-images-1.medium.com/max/NaN/1*A02ddr8x68brPwyIWYJ99w.jpeg)

### Users and Groups

With the user pool created, a new option is shown on the navigation menu allowing us to use the console to create groups, create users and associate users with groups.

![](https://cdn-images-1.medium.com/max/NaN/1*O_dwX8lacZgfKt6XHBYnRw.jpeg)

Before looking at some automation to perform this type of work, let’s review adding users and groups through the console.

### Adding a Group

For this example, we are going to add two groups to our user pool. After selecting Groups and clicking on “Create Group”, we are prompted to provide the group name, a description, select a role, and set a precedence value.

![](https://cdn-images-1.medium.com/max/NaN/1*NlqWJL2-NvAoB2lBg4asOA.jpeg)

The group name can contain letters and characters; spaces are not allowed. (The image above shows a space in the group name which was flagged an error when I attempted to create the group.) A description defines what the group is for. When creating a group, you can associate an IAM role, meaning members of this group will automatically have that role applied.

In this example, our policy is S3 read-only, which we could extend to only allow read-only access to specific S3 buckets by applying a resource to the policy.

The precedence value helps AWS Cognito decide which group should be applied when the user belongs to more than one group. In this case, the group with the lowest precedence value is the group the user will be associated with when they log in.

![](https://cdn-images-1.medium.com/max/NaN/1*9ICQOx5GcTE8EYm-OInw3g.jpeg)

With our two groups created, let’s add a user to each group through the console.

### Adding a User

The process to add a user follows a similar process as adding a group, but the workflow after the user is added differs.

![](https://cdn-images-1.medium.com/max/NaN/1*qVwz_TDl2CvJlZ4dLFAvKw.jpeg)

From this view, we can choose to add the users one at a time or import a file with the new user information. If we click on “Add User”, we are presented a form to complete with the information necessary to add the user to the pool.

![](https://cdn-images-1.medium.com/max/NaN/1*XxL6E4jO1YKBEsSeqfeXRg.jpeg)

Recall we specified when defining the pool that the username was to be an email address. Therefore, when adding the user, the username must follow the email format of user@domain. If the username does not, an error message is displayed indicating the username doesn’t match the required format.

It is important to note the phone number format must also follow the international dialing format of ‘+ countrycode-phonenumber’. Once all of the fields are entered, click the “Create User” button and the user is added to the pool.

![](https://cdn-images-1.medium.com/max/NaN/1*TvKZtNO-s_ouW1aDnE0CiA.jpeg)

Notice that creating the user did not automatically associate the user with a group. This is done in the console after the user is created. To add the user to the group, click on the desired username, which displays the details for the user.

![](https://cdn-images-1.medium.com/max/NaN/1*96js446dabioBBJfu1RAcg.jpeg)

Once the user details are displayed, click on “Add to Group”, which displays a dialog where you can select the group the user is to be associated with.

![](https://cdn-images-1.medium.com/max/NaN/1*XljgjJCSW58SdtUxPzxuvA.jpeg)

Once selected, click the “Add to Group” button. Once added, a response message is displayed indicating the user has been added to the group.

### A Sample Implementation

Now that we have our user pool created, let’s look at using the AWS SDK to interact with the user pool.

### List Users

We may want to list the users in the pool. This sample function illustrates listing the users in the sample “Testme” pool created earlier in the article.

    import boto3
    import os
    import datetime
    import json
    import time
    __POOL_ID__ = "pool_id"
    
    def main():
        global __POOL_ID__
       
        client = boto3.client('cognito-idp')
        response = client.list_users(
        UserPoolId=__POOL_ID__
        )
        print(response)
        return
    if __name__ == "__main__":
        main()

When we run this against our sample pool, we see (formatting corrected for readability):

    $ python3 listuser.py
    {
        'Users': [
            {
                'Username': 'ea6bca08-156e-4c8a-8338-f7fc7ccba16e', 
                'Attributes': [
                    {
                        'Name': 'sub', 
                        'Value': 'ea6bca08-156e-4c8a-8338-f7fc7ccba16e'
                    }, 
                    {
                        'Name': 'email_verified', 
                        'Value': 'true'
                    }, 
                    {
                        'Name': 'phone_number_verified', 
                        'Value': 'true'
                    }, 
                    {
                        'Name': 'phone_number', 
                        'Value': '+1903npaXXXX'
                    }, 
                    { 'Name': 'email', 
                        'Value': 'user@example.com'
                    }
                ],
                'UserCreateDate': datetime.datetime(2019, 10, 24, 8, 56, 17, 922000, tzinfo=tzlocal()),
                'UserLastModifiedDate':  datetime.datetime(2019, 10, 24, 8, 56, 17, 922000, tzinfo=tzlocal()),
                'Enabled': True, 
                'UserStatus': 'FORCE_CHANGE_PASSWORD'}
            ], 
        'ResponseMetadata': 
            {
                'RequestId': '565cf314-4a07-4271-895f-984f34f41d34',
                'HTTPStatusCode': 200,
                'HTTPHeaders': 
                {
                    'date': 'Thu, 24 Oct 2019 21:25:23 GMT',
                    'content-type': 'application/x-amz-json-1.1',
                    'content-length': '444', 'connection': 'keep-alive',
                    'x-amzn-requestid': '565cf314-4a07-4271-895f-984f34f41d34'},
                    'RetryAttempts': 0
                }
            }

This wouldn’t be practical when there are a lot of users, but it serves to demonstrate using SDK to retrieve the users in the pool.

### Adding Bulk Users

Whether we want to add one user or a thousand, automation is important. It is easy to assume that you create a user with the standard attributes identified when you create the pool, when in fact you do not. No additional information other than what is needed to create the user in the console is needed when using the *admin_create_user* API call.

When creating users using the *admin_create_user* API, you only need to specify the username, email address, phone number and whether to consider the email address and phone number verified. When the API call is made, a temporary password is automatically generated. All of the other standard attributes would be added later, including adding the user to a group.

Here is the API call (in Python 3) to create the users:

    client = boto3.client('cognito-idp')
    
        response = client.admin_create_user(
            UserPoolId=__POOL_ID__,
            Username=userData['email'],
            UserAttributes=[
            {
                'Name': 'phone_number',
                'Value': userData['phone']
            },
            {
                'Name': 'email',
                'Value': userData['email']
            },
            {
                'Name': 'email_verified',
                'Value': 'True'
            },
            {
                'Name': 'phone_number_verified',
                'Value': 'True'
            }
            ],
            ForceAliasCreation=True,
            DesiredDeliveryMediums=[
                'EMAIL',
            ]
        )

The value, *__POOL_ID__* is a global variable identifying the user pool to create the user in. Once created, the verification email is sent to the user, along with the user’s temporary password.

If we want to add additional user attributes, we would use the admin_update_user_attributes* API call, and to add the user to a group, the *admin_add_user_to_group* API.

### Updating user attributes

Once the user account has been added, we can add additional standard and custom attributes to the user record. In the code sample below, we have added user attributes for name, first name, middle name, given name, birthdate, address, and gender. Adding these attributes to the user account requires using the *admin_update_user_attributes* API call, which needs the user pool ID and the username the attributes should be applied to.

    response = client.admin_update_user_attributes(
            UserPoolId=__POOL_ID__,
            Username=userData['email'],
            UserAttributes=[
                {
                    'Name': 'birthdate',
                    'Value': userData['birthdate'],
                },
                {
                    'Name': 'address',
                    'Value': userData['address'],
                },
                {
                    'Name': 'family_name',
                    'Value': userData['family_name'],
                },
                {
                    'Name': 'given_name',
                    'Value': userData['given_name'],
                },
                {
                    'Name': 'middle_name',
                    'Value': userData['middle_name'],
                },
                {
                    'Name': 'name',
                    'Value': userData['name'],
                },
                {
                    'Name': 'gender',
                    'Value': userData['gender'],
                }
            ]
        )

This code segment from our sample code is executed after the user is created, and used to update the standard attributes for the user. If custom attributes were created when the user pool was defined, those custom attributes are prefixed with the term ‘custom:’ as in ‘custom: Company’ in the User attributes definition. With the user attributes applied, this is what the user record looks like in the AWS console.

![](https://cdn-images-1.medium.com/max/NaN/1*lscIPCb1ih9oyJwscuqedw.jpeg)

### Add users to Groups

We can extend our sample program to also add the user to a group if one is specified in the CSV source file.

For example, consider this code:

    if userData['group'] is not None:
            response = client.admin_add_user_to_group(
                UserPoolId=__POOL_ID__,
                Username=userData['email'],
                GroupName=userData['group']
        )

If there is a group in the CSV source file, then execute the API to add the user to the specified group.

Now if we execute our sample program, we can see the newly created users and their corresponding entry in the groups.

![](https://cdn-images-1.medium.com/max/NaN/1*lscIPCb1ih9oyJwscuqedw.jpeg)

### An Important Note about Removing Users

If you want to remove a user from your user pool through the console, you must first disable the user account. Then and only then is it possible to delete the account. This same restriction does not apply to the API. It may be desirable, however, to disable access for a user before deleting the user account.

### Cognito Sync

I am mentioning AWS Cognito Sync here only because some of the current AWS Cognito documentation still references it. AWS Cognito sync provides a mechanism for synchronizing application-related user data across devices. If you need this service, check out [AWS AppSync](https://aws.amazon.com/appsync/) which provides this capability and more.

### Automation

Like many things in AWS, Cognito User and Identity Pools can be created using CloudFormation. Indeed, if you are going to deploy AWS Cognito, it may be better to define the CloudFormation to create the pool, as it is entirely possible you won’t get the pool created to meet your needs on the first attempt. Additionally, if you create one AWS Cognito pool, it is possible you may need another, so again, defining the CloudFormation first.

Other possible automation opportunities include:

* bulk user creation, modification, and deletion requests;

* group management; and,

* inactivating users who have not logged in.

There are likely other possible automation opportunities that I have not mentioned here.

The sample code discussed in this article is available from my website. Click the links below to get a copy of that file.

[adduser.py](https://labrlearningweb.s3.amazonaws.com/assets/tools/cognito/adduser.py) [listuser.py](https://labrlearningweb.s3.amazonaws.com/assets/tools/cognito/listuser.py) [users.csv](https://labrlearningweb.s3.amazonaws.com/assets/tools/cognito/users.csv) [cloudformation.json](https://labrlearningweb.s3.amazonaws.com/assets/tools/cognito/cognito_userpool_cfn.json)

This sample code is by no means complete. It has no error checking nor exception handling. It is purely for demonstration purposes only as related to this article.

### Pricing

One of the best features of AWS is paying for what you use. This is the same for AWS Cognito, in that you pay for the number of Monthly Active Users (MAUs), and not the total configured users in the directory. The price varies based upon volume. An MAU is a user who has signed-in, signed-up, refreshed their authentication token or changed their password.

![](https://cdn-images-1.medium.com/max/NaN/1*H_j9T9EEP0fBMoKdISuH2w.jpeg)

This pricing structure makes AWS Cognito quite affordable. Even if you have 10,000,001 MAUs, the monthly fee is $25,000. That sounds like a lot, but if you have 10,000,001 MAUs, you likely have many more users configured, and the relatability of the authentication service becomes as important as the application operation itself.

What I am getting at here is that while $25,000 buys a lot of 20 oz bottles of Dr. Pepper, it pales in comparison to the failure of a “homegrown” authentication solution that fails to authenticate users or that doesn’t include all of the features found in AWS Cognito.

Remember, there may be additional charges for SMS and SES messages if those facilities are used. Configuring SES is beyond the scope of this article.

### Recommendation

Like all AWS services, as the service is being designed, the rationale for making various design and architecture decisions should be documented. By creating and maintaining the documentation and decisions, when at some future point you wonder why someone made the decision they did, there is an explanation as to why. This is very important for services like AWS Cognito, where many of the decisions are immutable.

### Conclusion

To *User Pool* or *Federate*, that is the question.

Admittedly, there is a lot more to AWS Cognito than what I covered here. This article focused on setting up the user pool and applying automation to add users in bulk to the user pool. This article didn’t discuss connecting the application to the user pool.

While the application side is important, there are issues with getting the user pool set up and properly configured. As mentioned previously, if you are going to be dealing with thousands of users and a bulk load of those users is needed, automation is required. We saw it is not possible to add the user, user attributes and assign a group all at the same time: there are three separate API calls involved to accomplish these three actions.

While I didn’t discuss using CloudFormation to create the user pool, this is an option worth considering, especially since it is likely that if you create one user_pool, you will want to create another.

### References

[AWS AppSync](https://aws.amazon.com/appsync/)

[AWS Cognito](https://aws.amazon.com/cognito/)

[Configuring Phone Number or Email Validation](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-email-phone-verification.html)

[Getting Started with AWS Cognito](https://aws.amazon.com/cognito/getting-started/)

[Limits in AWS Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/limits.html)

[Managing Security for AWS Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/managing-security.html)

[Simple Email Service](https://aws.amazon.com/ses/)

[Worldwide SMS Pricing](https://aws.amazon.com/sns/sms-pricing/)

### About the Author

Chris is a highly-skilled Information Technology AWS Cloud, Training and Security Professional bringing cloud, security, training and process engineering leadership to simplify and deliver high-quality products. He is the co-author of more than seven books and author of more than 70 articles and book chapters in technical, management and information security publications. His extensive technology, information security, and training experience makes him a key resource who can help companies through technical challenges.

### Copyright

This article is Copyright © 2019, Chris Hare.
