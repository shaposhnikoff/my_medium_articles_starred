
# Using Terraform to Manage DNS Records

For the last ~2 years, I’ve been using Terraform to manage mostly AWS infrastructure. It has allowed me to know exactly what resources I’ve provisioned, save time by using modules for common configurations, and have a (probably false) sense of security that I can replicate everything in another region with a single variable change. Terraform is pretty great.

Last week, I started transferring some domains to a new registrar. I had been keeping DNS records with my old registrar. Manually copying and pasting records between domain registrars would be tedious and I’d have to do it all over again if (and when) I transfer to another registrar. Moving those records to a dedicated provider and defining them with Terraform felt logical since I was already bought in to “infrastructure as code” and familiar with [AWS Route 53](https://aws.amazon.com/route53/).

First, I defined a zone, which is about as simple as it gets.

    resource "aws_route53_zone" "example" {
      name = "example.domain"
    }

Next, I added some MX records for G Suite by Google.

    resource "aws_route53_record" "example_mx" {
      zone_id = "${aws_route53_zone.example.zone_id}"
      name    = ""
      type    = "MX"
      
      records = [
        "1 ASPMX.L.GOOGLE.COM",
        "5 ALT1.ASPMX.L.GOOGLE.COM",
        "5 ALT2.ASPMX.L.GOOGLE.COM",
        "10 ASPMX2.GOOGLEMAIL.COM",
        "10 ASPMX3.GOOGLEMAIL.COM",  
      ]
      
      ttl = "${var.ttl}"
    }

You’ll notice a reference to the previously defined zone(${aws_route53_zone.example.zone_id}) and a global variable for time to live (${var.ttl}). This variable is an example of how Terraform makes it super easy to change lots of configuration at once.

    variable "ttl" {
      default = "3600"
    }

Finally, I can get the name servers for my zone to put in my domain registrar.

    output "example_name_servers" {  
      value = "${aws_route53_zone.example.name_servers}"
    }

When I run terraform output I’ll get something like this:

    example_name_servers = [
        ns-1306.awsdns-35.org,
        ns-1678.awsdns-17.co.uk,
        ns-423.awsdns-52.com,
        ns-536.awsdns-03.net
    ]

I could continue defining records one by one but this can get repetitive. Take, for example, defining common G Suite aliases for mail, calendar, and documents.

    resource "aws_route53_record" "example_mail" {
      zone_id = "${aws_route53_zone.example.zone_id}"
      name    = "mail"
      type    = "CNAME"
      records = ["ghs.google.com"]
      ttl     = "${var.ttl}"
    }

    resource "aws_route53_record" "example_cal" {
      zone_id = "${aws_route53_zone.example.zone_id}"
      name    = "cal"
      type    = "CNAME"
      records = ["ghs.google.com"]
      ttl     = "${var.ttl}"
    }

    resource "aws_route53_record" "example_docs" {
      zone_id = "${aws_route53_zone.example.zone_id}"
      name    = "docs"
      type    = "CNAME"
      records = ["ghs.google.com"]
      ttl     = "${var.ttl}"
    }

Terraform comes with a lot of [built-in functions for interpolation](https://www.terraform.io/docs/configuration/interpolation.html) which can make these records a lot easier to define.

    variable "example_gsuite" {  
      default = ["mail", "cal", "docs"]
    }

    resource "aws_route53_record" "example_gsuite" {  
      count   = "${length(var.example_gsuite)}"  
      zone_id = "${aws_route53_zone.example.zone_id}"  
      name    = "${element(var.example_gsuite, count.index)}"  
      type    = "CNAME"  
      records = ["ghs.google.com"]  
      ttl     = "${var.ttl}"
    }

The count [meta-parameter](https://www.terraform.io/docs/configuration/resources.html#count) allows us to create identical resources for each item in the example_gsuite list. The element function gets the value from the list based on the count index like a condensed for loop in JavaScript.

    var gsuite = ["mail", "cal", "docs"]

    for (var i = 0; i < gsuite.length; i++) {
      console.log(gsuite[i])
    }

Terraform has *a lot* of [DNS providers](https://www.terraform.io/docs/providers/index.html) built in as of version 0.9 and will be making it [easier to add them](https://www.hashicorp.com/blog/upcoming-provider-changes-in-terraform-0-10/) in 0.10 onward. I could see a module in the future that provides configurable basics for G Suite like [gnarea/terraform-gmail](https://github.com/gnarea/terraform-gmail) has started.

    module "example_gsuite" {
      source = "github.com/example/tf-mod-gsuite//route53"
    
      domain = "example.domain"
      aliases = ["mail", "cal", "docs"]
      ttl = "${var.ttl}"
    }

You can find all of these example configurations on [GitHub](https://github.com/maxbeatty/example-terraform-dns-route53). It *is* overkill to use Terraform for only these simple records, but this foundation will set you up for more advanced usage like aliasing an application load balancer or [CloudFront](https://aws.amazon.com/cloudfront/) distribution to a record.
