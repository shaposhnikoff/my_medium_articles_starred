
# AWS App Mesh — a big win!

AWS recently released a new service App Mesh during the 2019 summit which has generated a lot of interest from developers world-wide. This service is a great example of how Amazon is highly customer-focused in delivery of products/features to the market. Besides that, there is no additional charge for using the service! :-)

With the advent of cloud, the importance of microservices has increased tremendously. In microservices architecture, large monolithic code-bases/architectures are broken down into smaller, more independent modules. Responsible for highly defined and discrete tasks, these individual modules communicate with each other using APIs. To name a few, the most significant benefits of a microservice architecture are as follows:

* Software built as microservices can be broken down into multiple component services, so that each of these services can be deployed and then redeployed independently without compromising the integrity of an application.

* Better fault isolation; if one microservice fails, the others will continue to work.

* Code for different services can be written in different languages, and maintained in different repositories.

* Better, and more contained CI-CD flows; each service can be built and deployed in its own separate pipeline, without affecting others.

* Increase the autonomy of individual development teams within an organization, as each service can be architected and managed in isolation. This leads to faster delivery, as the effort in co-ordination is reduced significantly.

Though the benefits sound *pretty cool*, the complexity increases exponentially in highly distributed microservice architecture. Mismanagement of these services is as much a problem, as problems faced during the initial stages of the transformation from monolithic applications. App Mesh is aimed to solve these problems from day-one.

[AWS App Mesh](https://aws.amazon.com/app-mesh/) is a managed service mesh(a service mesh a logical boundary for network traffic between services that reside in it) control plane. It provides application-level networking support using service discovery naming, standardizing how you control and monitor your services across multiple types of compute infrastructure EC2, ECS, Fargate, Kubernetes. App Mesh uses the Envoy proxy. For now, you must use the App Mesh [Envoy container image](https://docs.aws.amazon.com/app-mesh/latest/userguide/envoy.html) until the Envoy project team [merges changes](https://github.com/aws/aws-app-mesh-roadmap/issues/10) that support App Mesh.

![AWS App Mesh uses Envoy sidecar proxy](https://cdn-images-1.medium.com/max/2014/1*4HUF9eQdaiyTOf5TMv-IVg.jpeg)*AWS App Mesh uses Envoy sidecar proxy*

As the number of services grow within an application, it becomes difficult to pinpoint the exact location of errors, re-route traffic after failures, and safely deploy code changes. Previously, this has required you to build monitoring and control logic directly into your code and redeploy your service every time there are changes. Lets go through the benefits of App Mesh and how it addresses these important challenges.

## **Logging**

You can configure App Mesh to log the traffic to/from your service. *Virtual node* can be configured to instruct Envoy to dump the HTTP access logs to a location on your instance. An agent in your application can pull logs from there to export to the destination of choice, which could be a log storage and processing service like CloudWatch Logs using standard Docker log drivers (such as [awslogs](https://docs.docker.com/config/containers/logging/awslogs/)). This ensures that all services, immaterial of their implementation will have a very consistent logging mechanism.

In absence of App Mesh, if you wanted to ensure consistent logging during the development of a distributed microservices architecture, you would have to ensure all the development teams use the same SDK. Say, your different development teams want to use Java, C++, Golang. But, say the logging SDK of your choice does not support Golang, then basically either you cannot use that SDK, or the development team that wants to use Golang would have to choose a different programming language. Besides that, you would also need to ensure that every application generates and dumps the logs in a consistent fashion. This is important as you can appropriately use monitoring tools to poll streams, configure events, etc. With App Mesh, the application does not have to worry about generating access logs, so it saves the management and maintenance overhead in development lifecycle.

## Traffic Control

With App Mesh, you create a mesh first, which is now your infrastructure for the services that you deploy. You can configure the mesh to control the routing of your service East-West traffic. App Mesh lets you configure services to connect directly to each other instead of requiring code within the application or using a load balancer. When each service starts, its proxies connect to App Mesh and receives configuration data about the locations of other services in the mesh. You can use controls in App Mesh to dynamically update traffic routing between services with no changes to your application code.

In absence of App Mesh, the services and the information of endpoints that they communicate with is tightly coupled. Endpoints are initialized at the start of service, and to update dynamically, you would need to implement API in your service to achieve that. This becomes exceedingly complex in an environment with many services, and for managing canary deployments. With App Mesh, the service does not need to have any knowledge of the endpoints, as all the traffic control plane configuration is handled by the mesh.

There is simplified high-level example on the [AWS blog](https://aws.amazon.com/blogs/compute/introducing-aws-app-mesh-service-mesh-for-microservices-on-aws/) which describes very well how the mesh deployment of service looks like.

![AWS App Mesh example from AWS blog](https://cdn-images-1.medium.com/max/2048/1*vuMWqG4RlMrrjugmtc-Ugw.png)*AWS App Mesh example from AWS blog*

## Load balancing

App Mesh *virtual router* allows you configure target endpoints with specified weights. Virtual routers handle traffic for one or more virtual services within your mesh. After you create a virtual router, you can create and associate routes for your virtual router that direct incoming requests to different *virtual nodes*. This feature is particularly useful for canary deployments, when you want to test a new service deployment in production environments. For a new service, you can start with lower weights, and once you have confidence, you can update the targets to remove the older service. Since App Mesh uses Envoy, the control plane policy configuration can be updated to dynamically change the routing, which is quite difficult to do otherwise.

## Visibility

Since with App Mesh, the traffic propagates through the Envoy proxy, you can use the logging mechanism to view the end-to-end flow of your traffic in a very consistent fashion. The HTTP access logs can be configured to dump at location of choice on the instance, using the virtual node configuration. These logs can then be pushed to destination of choice using the methodology of choice. You can leverage quite a bit by leveraging compatible AWS services CloudWatch and X-Ray.

For example, say you are preparing for a new deployment, and want to see the effect of your mesh configuration changes in the traffic flow. If you update the virtual router targets, you will be able to see your changes update the traffic flow almost instantly using X-Ray. This is very helpful to quickly get a feel of how your deployment is going. Again, achieving this is not impossible without App Mesh, but this service makes it quite easy to deploy and visualize.

Apps Mesh currently only has the ability to manage the internal East-West traffic, and cannot be used to manage the external North-South traffic. To manage the *North-South ingress*, you may use [in conjunction with API gateway](https://github.com/aws/aws-app-mesh-examples/tree/master/examples/apps/colorapp) to handle authentication, edge routing, etc. while the service mesh provides fine-grained control of your microservice architecture. For *North-South egress*, you have to model the destination [egress endpoint as a virtual node](https://github.com/aws/aws-app-mesh-roadmap/issues/2) and set it to be the backend of an existing node in the mesh.

An obvious question that comes to the mind is ***what about the latency?*** We now have one more component that runs alongside each service for incoming and outgoing traffic both. Of course, there is latency, but there are many benefits that offset the cost. App Mesh enables to focus ‘only’ on the application code without worrying about the infrastructure and the routing. Besides that, the latency added is quite small. Enabling any kind of logging or tracing/instrumentation in itself will also add to some latency itself. So, overall, the impact will not be significant. Besides that, since you only pay for the core infrastructure, you get all the benefits of the service for free.

Though, in itself, its a big win, just evaluating it first-hand, I have the following asks, which I think will be extremely useful:

* Integration with AWS Lambda

* HTTP header and cookie based target routing

* Support for more regions :-)
