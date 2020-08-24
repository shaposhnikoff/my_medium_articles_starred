
# How I use Terraform to restore the latest snapshot from production’s DB to staging’s DB ( AWS RDS )

Our system is using PostgreSQL RDS. One day, my client asked me that he want to restore the latest data from production DB to staging DB for testing.

After for a while, I’m thinking about some solutions

1. We can dump the data from production DB, then restore the data into the staging DB. And the downside is that he need to access the servers and run the scripts.

1. We can use Ansible to write the playbook for dump and restore the data. Or we can use Ansible to find the latest snapshot from production DB and create new DB RDS for staging, then terminate the old DB staging.

1. We can use Terraform to check the latest snapshot from production DB . If there is new snapshot, then create new instance RDS for staging.

And I found that the Option 3 is very easy to accomplish and It did the right thing for small task. When Terraform discover there is a new snapshot of production DB, they will create a new RDS instance for staging and delete the old instance automatically. That’s awesome.

So I decided to write the Terraform script.

    provider "aws" {
      region = "us-east-1"
    }

    
    # Get latest snapshot from production DB

    data "aws_db_snapshot" "db_snapshot" {
        most_recent = true
        db_instance_identifier = "db-prod"
    }

    # Create new staging DB

    resource "aws_db_instance" "db_uat" {
      instance_class       = "db.t2.micro"
      identifier           = "db-uat"
      username             = "xxx"
      password             = "xxx"
      db_subnet_group_name = "db-private-subnet"
      snapshot_identifier  = "${data.aws_db_snapshot.db_snapshot.id}"
      vpc_security_group_ids = ["sg-4fd43532"]
      skip_final_snapshot = true
    }

So every time he want to get latest data from production, just run the Terraform script.
