
# Create an AWS VPC and Subnet using the AWS CLI and Bash

For those learning AWS/AWS CLI this is a quick document on how create an AWS VPC and Subnet using the AWS CLI (AWS Command Line Interface) from a Linux/Bash script.

The below bash script performs the following:

1. adds a new VPC

1. names the VPC

1. adds dns support

1. adds a dns hostname

1. creates an internet gateway

1. names the internet gateway

1. creates the subnet

1. names the subnet

1. enables public ip on the subnet

1. creates the security group for the subnet

1. names the security group

1. enables port 22 for ssh

1. creates the route table

1. names the route table

1. adds route to the internet gateway

1. adds the route table to the subnet

I am using the jq Command-line JSON processor to capture JSON output from each AWS CLI command. If using a RHEL/CentOS 7 jq is found in EPEL.

The private ip subnet will be created in the bash script and will be mapped to a public ip that is provided by AWS. After you build an AWS Linux instance with the vpc, subnet and security group you will be able to access the instance through:

ssh -i “yourPrivateKeyFile.pem” ec2-user@ec2-public-ip-address.compute-1.amazonaws.com

Here is the create-aws-vpc bash script:

    #!/bin/bash
    # create-aws-vpc

    #variables used in script:
    availabilityZone="us-east-1a"
    name="your VPC/network name"
    vpcName="$name VPC"
    subnetName="$name Subnet"
    gatewayName="$name Gateway"
    routeTableName="$name Route Table"
    securityGroupName="$name Security Group"
    vpcCidrBlock="10.0.0.0/16"
    subNetCidrBlock="10.0.1.0/24"
    port22CidrBlock="0.0.0.0/0"
    destinationCidrBlock="0.0.0.0/0"

    echo "Creating VPC..."

    #create vpc with cidr block /16
    aws_response=$(aws ec2 create-vpc \
     --cidr-block "$vpcCidrBlock" \
     --output json)
    vpcId=$(echo -e "$aws_response" |  /usr/bin/jq '.Vpc.VpcId' | tr -d '"')

    #name the vpc
    aws ec2 create-tags \
      --resources "$vpcId" \
      --tags Key=Name,Value="$vpcName"

    #add dns support
    modify_response=$(aws ec2 modify-vpc-attribute \
     --vpc-id "$vpcId" \
     --enable-dns-support "{\"Value\":true}")

    #add dns hostnames
    modify_response=$(aws ec2 modify-vpc-attribute \
      --vpc-id "$vpcId" \
      --enable-dns-hostnames "{\"Value\":true}")

    #create internet gateway
    gateway_response=$(aws ec2 create-internet-gateway \
     --output json)
    gatewayId=$(echo -e "$gateway_response" |  /usr/bin/jq '.InternetGateway.InternetGatewayId' | tr -d '"')

    #name the internet gateway
    aws ec2 create-tags \
      --resources "$gatewayId" \
      --tags Key=Name,Value="$gatewayName"

    #attach gateway to vpc
    attach_response=$(aws ec2 attach-internet-gateway \
     --internet-gateway-id "$gatewayId"  \
     --vpc-id "$vpcId")

    #create subnet for vpc with /24 cidr block
    subnet_response=$(aws ec2 create-subnet \
     --cidr-block "$subNetCidrBlock" \
     --availability-zone "$availabilityZone" \
     --vpc-id "$vpcId" \
     --output json)
    subnetId=$(echo -e "$subnet_response" |  /usr/bin/jq '.Subnet.SubnetId' | tr -d '"')

    #name the subnet
    aws ec2 create-tags \
      --resources "$subnetId" \
      --tags Key=Name,Value="$subnetName"

    #enable public ip on subnet
    modify_response=$(aws ec2 modify-subnet-attribute \
     --subnet-id "$subnetId" \
     --map-public-ip-on-launch)

    #create security group
    security_response=$(aws ec2 create-security-group \
     --group-name "$securityGroupName" \
     --description "Private: $securityGroupName" \
     --vpc-id "$vpcId" --output json)
    groupId=$(echo -e "$security_response" |  /usr/bin/jq '.GroupId' | tr -d '"')

    #name the security group
    aws ec2 create-tags \
      --resources "$groupId" \
      --tags Key=Name,Value="$securityGroupName"

    #enable port 22
    security_response2=$(aws ec2 authorize-security-group-ingress \
     --group-id "$groupId" \
     --protocol tcp --port 22 \
     --cidr "$port22CidrBlock")

    #create route table for vpc
    route_table_response=$(aws ec2 create-route-table \
     --vpc-id "$vpcId" \
     --output json)
    routeTableId=$(echo -e "$route_table_response" |  /usr/bin/jq '.RouteTable.RouteTableId' | tr -d '"')

    #name the route table
    aws ec2 create-tags \
      --resources "$routeTableId" \
      --tags Key=Name,Value="$routeTableName"

    #add route for the internet gateway
    route_response=$(aws ec2 create-route \
     --route-table-id "$routeTableId" \
     --destination-cidr-block "$destinationCidrBlock" \
     --gateway-id "$gatewayId")

    #add route to subnet
    associate_response=$(aws ec2 associate-route-table \
     --subnet-id "$subnetId" \
     --route-table-id "$routeTableId")

    echo " "
    echo "VPC created:"
    echo "Use subnet id $subnetId and security group id $groupId"
    echo "To create your AWS instances"

    # end of create-aws-vpc
