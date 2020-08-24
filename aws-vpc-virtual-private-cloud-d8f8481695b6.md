
# AWS VPC(Virtual Private Cloud)

As per AWS official documentation
> *Amazon Virtual Private Cloud (Amazon VPC) enables you to launch Amazon Web Services (AWS) resources into a virtual network that you’ve defined. This virtual network closely resembles a traditional network that you’d operate in your own data center, with the benefits of using the scalable infrastructure of AWS.*

*VPC is nothing but it’s a virtual data center in the cloud.*

![](https://cdn-images-1.medium.com/max/2108/1*L8W8fKiR-16PxMDSFnH8aw.png)

*Ref: [https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/ExerciseOverview.html](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/ExerciseOverview.html)*

*Creating your own VPC*

*Go to AWS Console and under Networking & Content Delivery(click on VPC)*

![](https://cdn-images-1.medium.com/max/2000/1*4VhrCOVOZMt7oUPQh-h4Ug.png)

*Under Your VPCs, click on Create VPC*

![](https://cdn-images-1.medium.com/max/2000/1*iZ75n68WEHYtmiKQ9PncKw.png)

*Fill all the details(Name and IPv4 CIDR block, depending upon the network range you want, leave all the other details as default or depend upon your requirement, i.e whether you need IPv6 or Tenancy default or dedicated)and then click Yes,Create*

![](https://cdn-images-1.medium.com/max/2780/1*Td8wic5wI1773L3-UZ9VVQ.png)

*As soon as we create our default VPC, aws will create*

* *Route Table*

![](https://cdn-images-1.medium.com/max/4836/1*0CH_9TGaWkLc9_nH4bYVeQ.png)

* *Network ACLs*

![](https://cdn-images-1.medium.com/max/4860/1*hq8n25P5_jb2yWUy6KQwEQ.png)

* *Security Group*

![](https://cdn-images-1.medium.com/max/4848/1*4bVsL4l897IJSgIPoL8gJw.png)

* *Next step is to create subnet*

*Click on Subnet and then Create Subnet*

![](https://cdn-images-1.medium.com/max/5252/1*0VDAbyxQApqPeZhYfswhRg.png)

*Similarly create second subnet*

![](https://cdn-images-1.medium.com/max/3588/1*65oJbulUv5sMvEbNUkSR_Q.png)

*This is how it look like*

![](https://cdn-images-1.medium.com/max/4688/1*o_gOAuiIDFOEUlM5MUyGeQ.png)

* *Next create internet gateway so that we have some sort of internet connectivity*

*Go to Internet Gateways → Create internet gateway*

*Give it a Name tag*

![](https://cdn-images-1.medium.com/max/2516/1*3OlmOuBbPqEes2bAdnT-XA.png)

*By default it’s automatically detached*

![](https://cdn-images-1.medium.com/max/4776/1*ggKhky8BTZ_DIMvD51ZKqw.png)

*To attach it to VPC, go to action and click on Attach to VPC*

![](https://cdn-images-1.medium.com/max/2000/1*PJKaqz0mpdYWZ7BrJcet8A.png)

*It will ask you which VPC to attach*

![](https://cdn-images-1.medium.com/max/3752/1*J4-ahh_uw3rKJRxIoTQXJQ.png)

*Provide the VPC name you are building*

![](https://cdn-images-1.medium.com/max/2652/1*3qEfXVV_lQOgKPd7ZEBtWQ.png)

*P.S: Once again we can only have one IGW per VPC*

* *Next step to go to Route table*

*We only have one route table which allow local communication between subnet*

![](https://cdn-images-1.medium.com/max/3436/1*maZb4TSfkh8ARLUPgq5vuQ.png)

*Go to next tab,Subnet Associations under Route Table*

![](https://cdn-images-1.medium.com/max/3028/1*UbM4saQwtVoLEs7px4xiWw.png)

*As you can see these subnet are not associated with any route table(except with main route table), which is good as every-time we create a new subnet it will be associated with main route table and that’s why we don’t want our route table to have access to the internet.*

*So let’s create new Route Table, by clicking on Create Route Table*

![](https://cdn-images-1.medium.com/max/3824/1*HP6HPNITqT4OLjRgDnNXvw.png)

*For this route table let’s enable route access(Go to Routes and Add another route)*

* *Destination: 0.0.0.0/0*

* *Target: igw-b2d3a4ca(internet gateway)*

![](https://cdn-images-1.medium.com/max/2896/1*liC6t22IRPiD6zZTAMpOJg.png)

*This will give us internet accessibility*

*Now we can associate one subnet with this route table(Click on Subnet Association → choose one subnet)*

![](https://cdn-images-1.medium.com/max/3140/1*OjxcUsfOBpmqeoMdx2VIMA.png)

*So 10.0.1.0/24 is now our public subnet*

*Let spun up two servers one in Public Subnet(10.0.1.0/24) and one in Private Subnet(10.0.2.0/24). But before doing that we are missing one piece. Go back to subnet tab and as you can Auto-assign Public IP is set to No*

![](https://cdn-images-1.medium.com/max/5680/1*Bvqu1q61LkP5G6jDIEIJfA.png)

*For Public Subnet,Modify the auto-assign IP settings under Subnet Actions*

![](https://cdn-images-1.medium.com/max/4912/1*8X8ppbMAOoZlU75s11BERw.png)

![](https://cdn-images-1.medium.com/max/3940/1*Vtdlg20lR_9ErnnrLpCnvw.png)

*As you can see I spun up 2 instances one in Public Subnet(10.0.1.0/24) and one in Private Subnet(10.0.2.0/24). Public Subnet one got the Public IP*

![](https://cdn-images-1.medium.com/max/5020/1*l3En0cjvT0jzUyXRk9dFjA.png)

**NAT Gateway**
> Network address translation (NAT) gateway is used to enable instances in a private subnet to connect to the internet or other AWS services, but prevent the internet from initiating a connection with those instances

To create a NAT gateway, go back to VPC and click on NAT Gateways

![](https://cdn-images-1.medium.com/max/2884/1*OkgKP1VOW34V1soMzAUUIQ.png)

Click on create NAT Gateway

Make sure you select the Public Subnet

![](https://cdn-images-1.medium.com/max/5412/1*3O5U5rhyM2T86yDboWMmlA.png)

![](https://cdn-images-1.medium.com/max/4904/1*9yF_8h7N8zvX9YjeGo0Pug.png)

Once NAT gateway is available, go back to your Default Route table and add a route, with Target as NAT gateway.

![](https://cdn-images-1.medium.com/max/3268/1*sFooeXWpc2wnRWcF-DBXlw.png)

**Network ACL**
> A *network access control list (ACL)* is an optional layer of security for your VPC that acts as a firewall for controlling traffic in and out of one or more subnets. You might set up network ACLs with rules similar to your security groups in order to add an additional layer of security to your VPC

To create a NACL, go to Network ACLs under VPC and click on Create Network ACL

![](https://cdn-images-1.medium.com/max/4176/1*Rib6iVWnWn9rXyPqG4rJ7w.png)

NOTE: Under default NACL everything is allowed by default

![](https://cdn-images-1.medium.com/max/2844/1*PST3GGTl-hCEEyJ4HpZ6tg.png)

Now if we check the Inbound rule under this NACL, everything is denied by default

![](https://cdn-images-1.medium.com/max/3384/1*s2HBnqRRez6RN1_1zZhBIA.png)

To add a rule

![](https://cdn-images-1.medium.com/max/4620/1*Y9obfgH-4-0utxOmfAcRxA.png)
