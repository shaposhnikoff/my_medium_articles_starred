Unknown markup type 10 { type: [33m10[39m, start: [33m194[39m, end: [33m203[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m207[39m, end: [33m215[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m7[39m, end: [33m145[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m121[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m36[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m128[39m, end: [33m140[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m37[39m }

# AWS CLI with jq and Bash

The CLI is utilitarian, but a little jq sauce makes it beautiful

Transitioning from using the AWS console UI to the command line isn’t easy. The CLI is holds the same power as the APIs, and the dump trucks of JSON. When working in code that isn’t a problem, but it is a little painful when presented as the response to CLI commands.

That’s where jq shines — taking globs of JSON data and filtering summarizing it to get to the good stuff. And it gets even better under Bash (sorry Powershell fans, I’m a Windows user but Windows is so much better with WSL).

Many of my interactions with the CLI begin with a question, so I’ve compiled the list below as examples of how to make the best of this powerful combination of tools. Note that I’m not using the--profile or --region options, which you may want to add. I hope you find some of them useful!

### When was my AWS user account created?

Silly, but sometimes that’s justification enough:

    aws iam get-user | jq -r ".User.CreateDate[:4]"

FYI, if you get 2010 or earlier, consider using something other than your root credentials (doh!). IAM was [introduced as a beta](https://aws.amazon.com/about-aws/whats-new/2010/09/02/announcing-aws-identity-and-access-management-iam-preview-beta/) in September of 2010, so you may be “doing it wrong”.

### Which Services am I using?

This question comes up during security reviews a lot, and while it *can* be answered in the Config console UI (given enough clicks), or using Cost Explorer (fewer clicks), this is much faster and easy to automate:

    aws ce get-cost-and-usage --time-period Start=$(date "+%Y-%m-01" -d "-1 Month"),End=$(date --date="$(date +'%Y-%m-01') - 1 second" -I) --granularity MONTHLY --metrics UsageQuantity --group-by Type=DIMENSION,Key=SERVICE | jq '.ResultsByTime[].Groups[] | select(.Metrics.UsageQuantity.Amount > 0) | .Keys[0]'

Bash (well, the date command) provides the dates for the previous month, but obviously you can make it what makes sense to you (perhaps the current month). That command will produce output like this:

    [
      "AWS Budgets",
      "AWS CloudTrail",
      "AWS CodeCommit",
      "AWS Config", 
      *...snipped for length...*
      "Amazon WorkSpaces",
      "Amazon WorkSpaces Application Manager",
      "AmazonCloudWatch",
      "CodeBuild"
    ]

### What is each service costing me?

The AWS services you’re using can be different from the ones costing you the most. It’s easy enough to keep an eye on the high-level with this command (this time for the current month):

    aws ce get-cost-and-usage --time-period Start=$(date "+%Y-%m-01"),End=$(date --date="$(date +'%Y-%m-01') + 1 month  - 1 second" -I) --granularity MONTHLY --metrics USAGE_QUANTITY BLENDED_COST  --group-by Type=DIMENSION,Key=SERVICE | jq '[ .ResultsByTime[].Groups[] | select(.Metrics.BlendedCost.Amount > "0") | { (.Keys[0]): .Metrics.BlendedCost } ] | sort_by(.Amount) | add'

I can’t guarantee you’ll like the numbers.

### How many instances of each type do I have, and in what states?

Considering buying reserved instances or thinking about migrating to a newly introduced class?

    aws ec2 describe-instances | jq -r "[[.Reservations[].Instances[]|{ state: .State.Name, type: .InstanceType }]|group_by(.state)|.[]|{state: .[0].state, types: [.[].type]|[group_by(.)|.[]|{type: .[0], count: ([.[]]|length)}] }]"

Here is some sample output:

    [
      {
        "state": "running",
        "types": [
          {
            "type": "x1e.8xlarge",
            "count": 3
          },
          {
            "type": "m4.xlarge",
            "count": 1
          },
          {
            "type": "t2.medium",
            "count": 1
          },
          {
            "type": "t2.micro",
            "count": 112
          }
        ]
      },
      {
        "state": "stopped",
        "types": [
          {
            "type": "m4.large",
            "count": 1
          }
        ]
      }
    ]

So much easier to see this big picture this way.

### What CIDRs have Ingress Access to which Ports?

This is helpful when you need to perform a survey or audit of your system boundaries. While such a task isn’t ever “easy”, it can go more smoothly with with a summary:

    aws ec2 describe-security-groups | jq '[ .SecurityGroups[].IpPermissions[] as $a | { "ports": [($a.FromPort|tostring),($a.ToPort|tostring)]|unique, "cidr": $a.IpRanges[].CidrIp } ] | [group_by(.cidr)[] | { (.[0].cidr): [.[].ports|join("-")]|unique }] | add'

Hopefully you don’t see too many 0.0.0.0/0’s or 0–65535’s in there…

### How do I Create a CodeCommit Repository and Clone It?

    $ export REPO_URL=$(aws codecommit create-repository --repository-name **<name>** | jq -r ".repositoryMetadata.cloneUrlHttp")
    $ git clone $REPO_URL **<name>** && cd **<name>**

Since the name repeats, this one is a better as an alias:

    .bashrc:
    function make-and-clone-code-commit-repo()
    {
    export REPO_URL=$(aws codecommit create-repository --repository-name $1 | jq -r ".repositoryMetadata.cloneUrlHttp")
    git clone $REPO_URL $1 && cd $1
    }
    alias mcc="make-and-clone-code-commit-repo"

Which makes it much easier to use:

    $ mcc **<name>**

### Which Lambda Functions Runtimes am I Using?

And, are they all using the current runtime version or is someone going rogue?

    aws lambda list-functions | jq ".Functions | group_by(.Runtime)|[.[]|{ runtime:.[0].Runtime, functions:[.[]|.FunctionName] }
    ]"

Is everyone taking the time to set memory size and the time out appropriately?

    aws lambda list-functions | jq ".Functions | group_by(.Runtime)|[.[]|{ (.[0].Runtime): [.[]|{ name: .FunctionName, timeout: .Timeout, memory: .MemorySize }] }]"

### Lambda Function Environment Variables

This is a simple one, but helpful. Are you exposing secrets in variables? Have a typo in a key?

    aws lambda list-functions | jq -r '[.Functions[]|{name: .FunctionName, env: .Environment.Variables}]|.[]|select(.env|length > 0)'

### Creating EC2 Instances…

Step 1: Find the right AMI (this is slow, ’cause there are a *lot* of AMIs) and hold it in an environment variable:

    export AMI_ID=$(aws ec2 describe-images --owners amazon | jq -r ".Images[] | { id: .ImageId, desc: .Description } | select(.desc?) | select(.desc | contains(\"Amazon Linux 2\")) | select(.desc | contains(\".NET Core 2.1\")) | .id")

Step 2: Create a key pair, and hold on to it in a file:

    aws ec2 create-key-pair --key-name aurora-test-keypair > keypair.pem

Step 3: Create the instance using the AMI and the key pair, and hold onto the result in a file:

    aws ec2 run-instances --instance-type t2.micro --image-id $AMI_ID --region us-east-1 --subnet-id <your_subnet_id> --key-name keypair --count 1 > instance.json

Step 4: Grab the instance Id from the file:

    export INSTANCE_ID=$(jq -r .Instances[].InstanceId instance.json)

Step 5: Wait for the instance to spin-up, then grab it’s IP address and hold onto it in an environment variable:

    export INSTANCE_IP=$(aws ec2 describe-instances --instance-ids $INSTANCE_ID --output text --query 'Reservations[*].Instances[*].PublicIpAddress')

Step 6: SSH and profit:

    ssh -i keypair.pem ec2-user@$INSTANCE_IP

### What are my RDS Instance Endpoints?

    aws rds describe-db-instances | jq -r '.DBInstances[] | { (.DBInstanceIdentifier):(.Endpoint.Address + ":" + (.Endpoint.Port|tostring))}'

### How Many Services does AWS Have?

Okay, not strictly a CLI thing but still a fun number to track.

    curl -s https://raw.githubusercontent.com/boto/botocore/develop/botocore/data/endpoints.json | jq -r '.partitions[0].services | keys[]' | wc -
    l

### How Many CloudFormation Stacks do I have in each Status?

    aws cloudformation list-stacks | jq  '.StackSummaries | [ group_by(.StackStatus)[] | { "status": .[0].StackStatus, "count": (. | length) }
    ]'

### Which EC2 Instances were created by Stacks?

Hopefully none, but if you have them you know how important it is to be aware of their parentage.

    for stack in $(aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE | jq -r '.StackSummaries[].StackName'); do aws cloudformation describe-stack-resources --stack-name $stack | jq -r '.StackResources[] | select (.ResourceType=="AWS::EC2::Instance")|.PhysicalResourceId'; done;

### How Many Gigabytes of Volumes do I have, by Status?

    aws ec2 describe-volumes | jq -r '.Volumes | [ group_by(.State)[] | { (.[0].State): ([.[].Size] | add) } ] | add'

Yeah. Maybe those should be cleaned up. ;)

### How many Snapshots do I have?

Perhaps related to the number of volumes. Beware that the list is very, very, very long if you look at the ones owned by amazon.

    aws ec2 describe-snapshots --owner-ids self | jq '.Snapshots | length'

And, how large are they in total?

    aws ec2 describe-snapshots --owner-ids self | jq '[.Snapshots[].VolumeSize] | add'

And, how do they breakdown by the volume used to create them?

    aws ec2 describe-snapshots --owner-ids self | jq '.Snapshots | [ group_by(.VolumeId)[] | { (.[0].VolumeId): { "count": (.[] | length), "size": ([.[].VolumeSize] | add) } } ] | add'

Note, when a Snapshot is copied the VolumeId of the copy does not reflect the volume of the original (it gets the special value vol-ffffffff).

### What’s happening in my Log Streams?

It’d be cool if the CLI allowed me to get log events using the log stream ARN, but it doesn’t so it starts with getting the log group names:

    logs=$(aws logs describe-log-groups | jq -r '.logGroups[].logGroupName')

And, then maybe the first log stream for each:

    for group in $logs; do echo $(aws logs describe-log-streams --log-group-name $group --order-by LastEventTime --descending --max-items 1 | jq -r '.logStreams[0].logStreamName + " "'); done

Or, loop through the groups and streams and get the last 10 messages since midnight:

    for group in $logs; do for stream in $(aws logs describe-log-streams --log-group-name $group --order-by LastEventTime --descending --max-items 1 | jq -r '[ .logStreams[0].logStreamName + " "] | add'); do echo ">>>"; echo GROUP: $group; echo STREAM: $stream; aws logs get-log-events --limit 10 --log-group-name $group --log-stream-name $stream --start-time $(date -d 'today 00:00:00' '+%s%N' | cut -b1-13) | jq -r ".events[].message"; done; done

Finally, since log groups default retention period is “Never Expire” they can start to build up after a few years. I don’t use CloudWatch logs for long-term storage (and neither should you) but since AWS doesn’t provide a way to set the default retention to something I’d prefer, I run the following command from time to time to make sure they’re all set to 30 days:

    for group in $(aws logs describe-log-groups --query "logGroups[].[logGroupName]" --output text --no-paginate); do aws logs put-retention-policy --log-group-name $group --retention-in-days 30; done;

### What logs does my Lambda Function generate when I run it?

Simple, but necessary since Lambda base64 encodes the logs in the response.

In jq version 1.5 (you’ll get 1.4 unless you go out of your way to get 1.5 since it’s still in RC at this time):

    aws lambda invoke --function-name **<function name>** --payload '{}' --log-type Tail - | jq -r '{ "StatusCode": .StatusCode, "LogResult": (.LogResult|@base64d)}'

In 1.4, you have to just rely on bash:

    aws lambda invoke --function-name **<function name>** --payload '{}' --log-type Tail - | jq -r '.LogResult' | base64 --decode

### How Much Data is in Each of my Buckets?

It’s remarkably difficult get the breakdown of data volume by bucket in the AWS console. CloudWatch contains the data, but if your account has more than a few buckets it’s very tedious to use.

This little command gives your the total size of the objects in each bucket, one per line, with human-friendly numbers:

    for bucket in $(aws s3api list-buckets --query "Buckets[].Name" --output text); do aws cloudwatch get-metric-statistics --namespace AWS/S3 --metric-name BucketSizeBytes --dimensions Name=BucketName,Value=$bucket Name=StorageType,Value=StandardStorage --start-time $(date --iso-8601)T00:00 --end-time $(date --iso-8601)T23:59 --period 86400 --statistic Maximum | echo $bucket: $(numfmt --to si $(jq -r ".Datapoints[0].Maximum // 0")); done;

Prefer to have that is dollars per month? Just a little math ( based on the current standard tier price of $0.023 per GB per month):

    for bucket in $(aws s3api list-buckets --query "Buckets[].Name" --output text --profile eleven-prod); do aws cloudwatch get-metric-statistics --namespace AWS/S3 --metric-name BucketSizeBytes --dimensions Name=BucketName,Value=$bucket Name=StorageType,Value=StandardStorage --start-time $(date --iso-8601)T00:00 --end-time $(date --iso-8601)T23:59 --period 86400 --statistic Maximum --profile eleven-prod | echo $bucket: \$$(jq -r "(.Datapoints[0].Maximum //
     0) * .023 / (1024*1024*1024) * 100.0 | floor / 100.0"); done;

Warning, this one is slow.

### Suggestions?

Have suggestions for more interesting questions and/or examples? Put ’em in the comments, below.
