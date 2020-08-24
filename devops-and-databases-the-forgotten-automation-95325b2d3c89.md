Unknown markup type 10 { type: [33m10[39m, start: [33m178[39m, end: [33m200[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m120[39m, end: [33m125[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m43[39m }

# DevOps and Databases — The forgotten automation

How to bring automation into managing stateful databases

![Source: [https://www.xenonstack.com/wp-content/uploads/xenonstack-devops-for-databases.png](https://www.xenonstack.com/wp-content/uploads/xenonstack-devops-for-databases.png)](https://cdn-images-1.medium.com/max/2000/0*7EM_33uoR9UdASR2.png)*Source: [https://www.xenonstack.com/wp-content/uploads/xenonstack-devops-for-databases.png](https://www.xenonstack.com/wp-content/uploads/xenonstack-devops-for-databases.png)*

Do you see that guys? Are we really not going to talk about it? That gigantic elephant is literally standing right in the middle of our conference room and everyone is just going to pretend like it doesn’t exist?

I’m of course referring to the **Database** aspect of your application. *That part that no one wants to bring up when we talk about DevOps*. I know guys, it’s scary. Databases hold all this data in some form of **state** and well, *the entire DevOps ecosystem has built around the premise of stateless applications*.
> A [2019 State of Database Deployments in Application Delivery report](https://www.datical.com/whitepapers/survey-the-state-of-database-deployments-in-application-delivery-2019/) found that for the second year in a row, database deployments are a bottleneck. 92% of respondents reported difficulty in accelerating database deployments.

Have no fear my fellow technologist, it’s 2020 and we can automate the database! Let’s dig into some of the fun that is to be had when **DOing the DB!**

### First — What are the issues with Automating the Database?

![Photo by [Campaign Creators](https://unsplash.com/@campaign_creators?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/9216/0*AYGSxIUZ9PwzWESF)*Photo by [Campaign Creators](https://unsplash.com/@campaign_creators?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

For many DevOps enthusiast such as myself, database stuff is probably the piece of technology you are least familiar with. What the heck is a SQL statement? Why can’t we just do select * from <table>; for every query? We need to know the areas of concern so we can even begin to address them.

Some of the areas in which database automation gets tricky are:

* Installation and Configuration of a Highly-Available Database cluster in every environment

* Managing and securing the actual data itself (we can’t just export/import financial or user data from prod)

* The risk associated with schema changes, particularly data destruction

* Size of data that needs to move from 1 environment to the next

* Secrets management for connectivity and permission to databases

* The potential for conflicting changes introduced from different sources (any developer could need a new change to the DB for what they are working on)

Wow, those are actually some big problems to tackle. Well let’s take those concerns and figure out what our options are one by one, shall we?

### Installation and Configuration Management

![Photo by [Obi Onyeador](https://unsplash.com/@thenewmalcolm?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/12146/0*XP6cmD7iR6B9JNBI)*Photo by [Obi Onyeador](https://unsplash.com/@thenewmalcolm?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

This is actually probably the simplest issue to address as it relates to automating the database because it is no different than what we do for middleware.

**Traditional On-Prem Solutions:**

* I would suggest using Chef to write the configuration of my database server, especially if you are using Windows SQL Server. Chef works great in a stateful environment, which is the very definition of a database.

* **DO NOT **concern yourself with automating in-place upgrades. In-place upgrades you’re going to back up the database first before attempting right? Which is time-consuming and pretty risky. Instead, focus on your upgrade logic attaching a shared data mount from the old server, and stand up a new server with a fresh installation of the new version. This allows for a much quicker cutover on traffic and maintains the current running infrastructure in case you need to back out the upgrade.

**Cloud Solutions:**

* Cloud is great because a lot is managed for you. Write your Terraform or CloudFormation for standing up your RDS instance with the appropriate security groups and authentication details.

* Auto-Generate a random string during database provisioning and store the credential in a secrets management/vault solution as to never hardcode a password anywhere. For tooling to access the database, have them lookup in the vault for the specific value.

**Local Dev Environments:**

* Regardless of which database technology you use, they can all pretty much be configured in a docker container. Make sure your HA configuration works in a containerized version so devs can make their mistakes locally before touching your static database environments.

* Have 2 versions of your database containers. 1 that is empty and free of data but a current version of the production schema. 2 that has the production version of your schema but populated with faker data.

You can use a tool like [Faker](https://github.com/faker-ruby/faker) for creating mock data in your database.

Here’s a Javascript example of this:

<iframe src="https://medium.com/media/21344cc590f8403bc66f207fffcfc47d" frameborder=0></iframe>

### Data Lifecycle Management

![Source: [https://spirion.com/wp-content/uploads/2019/10/SPIRION-Data_Lifecycle-2k-1024x520.jpg](https://spirion.com/wp-content/uploads/2019/10/SPIRION-Data_Lifecycle-2k-1024x520.jpg)](https://cdn-images-1.medium.com/max/2048/0*SjgpUdVVhp2JeO5Q.jpg)*Source: [https://spirion.com/wp-content/uploads/2019/10/SPIRION-Data_Lifecycle-2k-1024x520.jpg](https://spirion.com/wp-content/uploads/2019/10/SPIRION-Data_Lifecycle-2k-1024x520.jpg)*

One of the top security concerns relating to databases is the information getting leaked or misused. This is why I am absolutely against backup and restore processes that run with production data. However, it is vital to have similar data and volume of data in a pre-production environment, especially for proper load testing.

*Choose one of these approaches:*

* Export and sanative production data into a QA database. The downside to this is for every new schema change you must also ensure sanitization scripts are updated which makes it hard to bundle those changes together.

* Have a job that runs a diff in the database size every 24 hours to see how much new data has been generated. Then use a Faker script to populate QA with the same amount of records.

### Preventing destructive changes

![[Source](https://tr1.cbsistatic.com/hub/i/r/2019/08/05/7c395d36-5c3f-40a4-bee6-dc054d98f726/thumbnail/770x578/8036f66c20cbffb5a46be8730a3a3a2b/istock-942607134-1.jpg)](https://cdn-images-1.medium.com/max/2000/0*0H-oS-4XSJJqqK4v.jpg)*[Source](https://tr1.cbsistatic.com/hub/i/r/2019/08/05/7c395d36-5c3f-40a4-bee6-dc054d98f726/thumbnail/770x578/8036f66c20cbffb5a46be8730a3a3a2b/istock-942607134-1.jpg)*

This is the most important and vital thing to get right when Automating your database. There is actually one simple strategy to getting this right and it’s very easy to implement with a tool such as [*Flyway](https://flywaydb.org/)*.

1. You absolutely have to version your schema. Every change is a new version. This allows your automation to know what state the database is in and then run each appropriate script from there in order to upgrade it to the most current version.

1. Never do a straight rename or delete a column. Always create the new column, copy the values, and then delete an old column. This ensures you can run the script backward and forwards without losing your data.

Flyway is also cool because you can embed it in the application. As such, when a new version of the application is deployed the Flyway logic will query the database to see what state it is in and run any migrations necessary to get it to match the same version of the application.

*A sample of what that looks like is below. *Here is my sample migration package:

<iframe src="https://medium.com/media/ebdad134134cf0eda90b64548641d113" frameborder=0></iframe>

I can compile my project

    bar$ mvn compile

This is now the status

    bar$ mvn flyway:**info**
    
    [INFO] Database: jdbc:h2:file:./target/foobar (H2 1.4)
    [INFO]
    +-----------+---------+---------------------+------+---------------------+---------+
    | Category  | Version | Description         | Type | Installed On        | State   |
    +-----------+---------+---------------------+------+---------------------+---------+
    | Versioned | 1       | Create person table | SQL  | 2017-12-22 15:26:39 | Success |
    | Versioned | 2       | Add people          | SQL  | 2017-12-22 15:28:17 | Success |
    | Versioned | 3       | Anonymize           | JDBC |                     | Pending |
    +-----------+---------+---------------------+------+---------------------+---------+

Note the new pending migration of type JDBC. It’s time to execute our new migration. I can run:

    bar$ mvn flyway:**migrate**

This will give you the following result:

    [INFO] Database: jdbc:h2:file:./target/foobar (H2 1.4)
    [INFO] Successfully validated 3 migrations (execution time 00:00.022s)
    [INFO] Current version of schema "PUBLIC": 2
    [INFO] Migrating schema "PUBLIC" to version 3 - Anonymize
    [INFO] Successfully applied 1 migration to schema "PUBLIC" (execution time 00:00.011s)

And you can check that this is indeed the new status:

    bar$ mvn flyway:**info**
    
    [INFO] Database: jdbc:h2:file:./target/foobar (H2 1.4)
    [INFO]
    +-----------+---------+---------------------+------+---------------------+---------+
    | Category  | Version | Description         | Type | Installed On        | State   |
    +-----------+---------+---------------------+------+---------------------+---------+
    | Versioned | 1       | Create person table | SQL  | 2017-12-22 15:26:39 | Success |
    | Versioned | 2       | Add people          | SQL  | 2017-12-22 15:28:17 | Success |
    | Versioned | 3       | Anonymize           | JDBC | 2017-12-22 16:03:37 | Success |
    +-----------+---------+---------------------+------+---------------------+---------+

As expected we can see that the Java-based migration was applied successfully!

### The Size of Data in Different Environments

![Photo by [Kolar.io](https://unsplash.com/@jankolar?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/7718/0*sU5u37DO5rINFu0b)*Photo by [Kolar.io](https://unsplash.com/@jankolar?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

This is another one of those problems that can be tricky to solve. Performance testing an application and evaluating a system that has a large subset of data is something you can definitely automate but not typically done in the standard CI/CD release workflow.

Different Approaches:

* One way to approach this would be to ensure your database is configured in such a way that the actual data is mounted to a different filesystem than the installation of the database. This would allow you to backup/copy/restore a filesystem to another running database executable without the need to mess with the runtime database itself.

* My recommendation is to again use a tool like Faker for generating significant amounts of data for performance testing runs. It may take a bit longer to set up but you can ensure good, clean data is in your database and your tests are evaluating against the real scenario and not also having to troubleshoot data integrity issues.

### Managing Database User Secrets

![Photo by [Kristina Flour](https://unsplash.com/@tinaflour?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)](https://cdn-images-1.medium.com/max/9640/0*OJCzdnFiU-wNnoKX)*Photo by [Kristina Flour](https://unsplash.com/@tinaflour?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)*

This is actually not that hard to solve if you have a solution like [Hashicorp Vault](https://www.vaultproject.io/) or working in AWS RDS. We use terraform to generate a randomly generated string that populates a secrets manager store which takes the need for humans to ever know or touch the secret out of the equation.

Here’s an example of what that looks like with Terraform:

<iframe src="https://medium.com/media/253b77cbe870eec849ef76c882a1df46" frameborder=0></iframe>

### Conclusion

There are lots of additional complexities that need to be thought through when automating the database in a DevOps world. Too many organizations leave the database as an afterthought and allow DBA’s to continue to manage them the same way they have for decades now.

It’s 2020 guys, we have the technology and we have the people. Challenge your team to think about the most critical and valuable part of any application, the data!

Happy Automating!

## By the way, 👏🏻 *clap* 👏🏻 your hands (up to 50x) if you enjoyed this post. It encourages me to keep writing and help other people finding it :)

Please follow 👉 [Tj Blogumas](undefined) for more Awesome DevOps Stories!

### This story is published in [DevOps Dudes](https://medium.com/devops-dudes), Medium’s only publication dedicated strictly to DevOps! Subscribe for the latest DevOps news!

Like our [Facebook](https://www.facebook.com/devopsdudes) or [Twitter](https://twitter.com/DevopsDudes) page as well!
