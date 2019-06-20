
# 100 Days of DevOps — Day 21- MySQL RDS Database Creation using Terraform

Welcome to Day 21 of 100 Days of DevOps, Let continue our journey with terraform and today we are going to create MySql database using terraform
> *What Is Amazon Relational Database Service (Amazon RDS)?*

*Amazon Relational Database Service (Amazon RDS) is a web service that makes it easier to set up, operate, and scale a relational database in the cloud. It provides cost-efficient, resizable capacity for an industry-standard relational database and manages common database administration tasks.*

***Step1:** Create a DB subnet group*

* *In order to create a new MySql database we first need to create a subnet group and assign at least two subnets to it.*

<iframe src="https://medium.com/media/4ff0dd3babc1aaff91bfc5a9c5412bd0" frameborder=0></iframe>

***Step2:** Create a Security Group to allow mysql port 3306*

<iframe src="https://medium.com/media/6bc774e85398da7467ef517119d13dc2" frameborder=0></iframe>

***Step3:** Next step is to create MySQL resource*

<iframe src="https://medium.com/media/91ac968aa0e9efb8fa5e6e35ba6e2a61" frameborder=0></iframe>

    ** **allocated_storage:** This is the amount in GB
    * **storage_type:** Type of storage we want to allocate(options avilable "standard" (magnetic), "gp2" (general purpose SSD), or "io1" (provisioned IOPS SSD)
    * **engine:** Database engine(for supported values check [https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_CreateDBInstance.html)) eg: Oracle, Amazon Aurora,Postgres 
    * **engine_version: **engine version to use
    * **instance_class:** instance type for rds instance
    * **name:** The name of the database to create when the DB instance is created.
    * **username:** Username for the master DB user.
    * **password:** Password for the master DB user
    * **db_subnet_group_name:**  DB instance will be created in the VPC associated with the DB subnet group. If unspecified, will be created in the default VPC
    * **vpc_security_group_ids:** List of VPC security groups to associate.
    * **allows_major_version_upgrade:** Indicates that major version upgrades are allowed. Changing this parameter does not result in an outage and the change is asynchronously applied as soon as possible.
    * **auto_minor_version_upgrade:**Indicates that minor engine upgrades will be applied automatically to the DB instance during the maintenance window. Defaults to true.
    * **backup_retention_period: **The days to retain backups for. Must be between 0 and 35. When creating a Read Replica the value must be greater than 0
    * **backup_window:** The daily time range **(in UTC)** during which automated backups are created if they are enabled. Must not overlap with maintenance_window
    * **maintainence_window:** The window to perform maintenance in. Syntax: "ddd:hh24:mi-ddd:hh24:mi".
    * **multi_az:** Specifies if the RDS instance is multi-AZ
    * **skip_final_snapshot:** Determines whether a final DB snapshot is created before the DB instance is deleted. If true is specified, no DBSnapshot is created. If false is specified, a DB snapshot is created before the DB instance is deleted, using the value from final_snapshot_identifier. Default is false*

***GitHub Link for Complete Code***
[**100daysofdevops/100daysofdevops**
*Contribute to 100daysofdevops/100daysofdevops development by creating an account on GitHub.*github.com](https://github.com/100daysofdevops/100daysofdevops/tree/master/two-tier-environment/rds_mysql)

* *One of the clear issues I see in the above code is that we are passing the password in the plain text inside the terraform code*

* *Now to encrypt that password we can use KMS*

***Step1:** First Create KMS Keys*

<iframe src="https://medium.com/media/c1bcf0147417835e7899d990c210bb02" frameborder=0></iframe>

***Step2:** Now use that key to encrypt a secret on a command line*

    *aws kms encrypt --key-id <kms key id> --plaintext admin123 --output text --query CiphertextBlob*

***Step3:** Now the encoded string we got pass it as a payload in your terraform code*

<iframe src="https://medium.com/media/83e62b017bd9d2e0e10b24aa91172b29" frameborder=0></iframe>

* *Now why I didn’t put this solution in first place and the reason for that, because of the below-mentioned error and I want to present a working solution*

<iframe src="https://medium.com/media/90d75af29d5afd79ded11fc9c439bf38" frameborder=0></iframe>

* *There is already a bug opened for this issue*
[**data-source/aws_kms_secret: Soft remove data source type with removal message by bflad · Pull…**
*References: #5144 #5195 The aws_kms_secret data source uses dynamic attribute functionality which is not supported in…*github.com](https://github.com/terraform-providers/terraform-provider-aws/pull/7657)

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
