
# How to Build an End to End Production Grade Architecture on AWS Part 3

gruntwork.io

## Part 3: Bootstrap Your Production-Grade Infrastructure in a Day

This is part 3 of a guided tour of a modern production-grade architecture for AWS. In the last 2 parts, we talked about [setting up Kubernetes with a network configuration to run your applications](https://blog.gruntwork.io/how-to-build-an-end-to-end-production-grade-architecture-on-aws-part-1-eae8eeb41fec) and [all the different components you can use to make the infrastructure robust across various concerns](https://blog.gruntwork.io/how-to-build-an-end-to-end-production-grade-architecture-on-aws-part-2-4f6e5dc30100). But how long would it take to build this from scratch?

Building production-grade infrastructure is hard. There are a lot of concerns to cover, the table stakes are high, and it takes a long time to build. Here is how long we expect infrastructure projects to take, based on our empirical data we’ve gathered while working with hundreds of different companies:

![](https://cdn-images-1.medium.com/max/NaN/1*3sBV0ISBkG00eJ1LD505pA.png)

Note that the numbers above are optimistic. They assume a best-case scenario where your team already knows what they’re doing and has uninterrupted time to focus on infrastructure work. If you’re new to the DevOps space, or have a thousand concerns competing for your time, it’ll take even longer.

What can you do to bootstrap the process? In this post we want to cover how you can leverage Gruntwork’s product offerings to speed up the process of getting up and running with your application on production-grade infrastructure.

* [Infrastructure as Code Library](#f718)

* [Reference Architecture](#46cd)

* [CIS Compliance](#6f7c)

## Infrastructure as Code Library

![[https://gruntwork.io/infrastructure-as-code-library/#infra-modules](https://gruntwork.io/infrastructure-as-code-library/#infra-modules)](https://cdn-images-1.medium.com/max/2000/1*zBZm_89-PpMtaWcg4tB4SQ.png)*[https://gruntwork.io/infrastructure-as-code-library/#infra-modules](https://gruntwork.io/infrastructure-as-code-library/#infra-modules)*

Infrastructure as code is a recent phenomena in the industry driven by the cloud and improvements in DevOps tools that allows managing infrastructure components (servers, disks, network interfaces, firewalls, routers, etc) using code.

There are many reasons why you want to manage your infrastructure as code:

* Review changes before they are applied

* Document your processes and infrastructure

* Componentize your infrastructure into reusable pieces

* Continuously test your infrastructure using automated testing

One of the biggest benefit of adopting Infrastructure as Code is the ability to reuse code. By leveraging code that is already written, you are able to replicate the infrastructure that is already tested across various different scenarios and environments to bootstrap your development process.

Based on this principle, we have developed [a library of battle-tested, reusable, production-grade infrastructure code](https://gruntwork.io/infrastructure-as-code-library/) for the most commonly used infrastructure in AWS and GCP written using a combination of Terraform, Go, Bash, and Python. We already have reusable modules available for most of the infrastructure you saw in this series, including VPCs, K8S, RDS databases, CI / CD pipelines, monitoring/alerting, etc. Each component is built with the production-grade infrastructure checklist in mind, is heavily tested, and is used in production by hundreds of customers.

For a taste of some of our modules, check out the following open source modules that we have built and maintain:

* [terraform-aws-influx](https://github.com/gruntwork-io/terraform-aws-influx)

* [terraform-google-gke](https://github.com/gruntwork-io/terraform-google-gke)

* [terraform-kubernetes-helm](https://github.com/gruntwork-io/terraform-kubernetes-helm)

When you use our library, you get a production-grade version of that component with little effort. Like using the AWS managed services, you are able to build on a foundation that handles all the concerns of running production-grade infrastructure for you. This vastly reduce the amount of code you have to write to get a version of the production-grade architecture covered in this series. Additionally, you also don’t have to maintain that code over time. This means that in most cases, you can get new features in AWS, or compatibility with new versions of the underlying tool (e.g Terraform) with a simple version bump in your code.

## Reference Architecture

![](https://cdn-images-1.medium.com/max/6336/1*DKPJAX65K6o0btCk11C4hQ.png)

The library provides the building blocks for implementing the architecture described in this blog post series. However, you still have to do the work to combine the pieces together to build up the architecture. To help ease the transition, we also offer [the Reference Architecture](https://gruntwork.io/reference-architecture/).

The Reference Architecture is an implementation of the production-grade architecture covered in this series that utilizes the IaC library under the hood. This is an opinionated way to use IaC Library, while the IaC Library is fairly unopionated. We deploy a version of the architecture into your AWS accounts and provide 100% of the code with it, all within a day. Because you have all the code for the infrastructure, you are free to customize the infrastructure to fit your needs, including adding your own applications onto the infrastructure.

The Reference Architecture comes with:

* Managing infrastructure across multiple accounts

* Management and application VPCs

* EKS cluster

* Namespaces with Tiller deployments

* Kubernetes Service deployments

* CI/CD pipelines for sample applications on Jenkins, CircleCI, and TravisCI

* Relational databases supported by RDS

* Cache systems supported by ElastiCache

* Additional stateful data stores (Kafka, Zookeeper, Elasticsearch, SQS)

* Serverless application deployments

* CDN deployments

* OpenVPN server

* Centralized logging and monitoring on Cloudwatch

* Cloudwatch alarms and dashboards

* CloudTrail audit logging

* DNS management on Route53

* Support for end to end encryption, including disk encryption and private TLS certificate management

* Server hardening, fail2ban, auto-updates, SSH access management, and much more

The Reference Architecture does 80–90% of what most companies need; getting that in 1 day and adding in the last 10–20% is much faster than building all 100% from scratch. This provides a solid foundation that can be used as a starting point for your application infrastructure. However, if the implemented components do not meet the needs of your specific deployment, you have 100% of the code. You are free to modify, add, or remove components from the infrastructure, customizing it to fit your needs.

## CIS Compliance

![](https://cdn-images-1.medium.com/max/3840/1*GmqrcvtQndIqunZvU7pkDA.png)

The [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services/) is an objective, consensus-driven guideline for establishing secure infrastructure on AWS. Organizations of any size looking to strengthen their security posture can turn to the Benchmark for guidance on best practice configurations for the AWS cloud. While the Infrastructure as Code Library and Reference Architecture are compatible with the benchmark, special configuration needs to be done to be fully compliant. For those that want an out-of-the-box experience, we offer [Gruntwork Compliance](https://gruntwork.io/achieve-compliance/).

Gruntwork Compliance extends the Infrastructure as Code Library and Reference Architecture with the exact configurations necessary to meet the requirements of the AWS Foundations Benchmark. When you combine Gruntwork Compliance with the Reference Architecture, you get a version that is certified by Gruntwork and by the [Center for Internet Security](https://www.cisecurity.org/) to comply with requirements of the AWS Foundations Benchmark. Additionally, we deploy the Reference Architecture with [AWS Security Hub](https://aws.amazon.com/security-hub/) to help measure compliance on an ongoing basis as well.

Note that nothing changes regarding the standard Reference Architecture offering. This means that you still get a full, end-to-end, production-grade application architecture in one day with 100% access to the code.

*If you are interested in our Reference Architecture or our IaC library in general,* [*let us know!](https://gruntwork.io/contact/)* *Or, if you are interested in other components that we currently do not support, we can* [*build it for you for a one-time fee](https://gruntwork.io/custom-module-development/).*
