
# A Practical Guide To HashiCorp Consul — Part 2



*This is part 2 of 2 part series on A Practical Guide to HashiCorp Consul. The [previous part](https://velotio.com/blog/2019/3/11/hashicorp-consul-guide-1) was primarily focused on understanding the problems that Consul solves and how it solves them. This part is focused on a practical application of Consul in a real-life example. Let’s get started.*

With most of the theory covered in the previous part, let’s move on to Consul’s practical example.

## What are we Building?

We are going to build a [Django Web Application](https://www.djangoproject.com/) that stores its persistent data in [MongoDB](https://www.mongodb.com/). We will [containerize](https://www.docker.com/resources/what-container) both of them using [Docker](https://www.docker.com/). Build and run them using [Docker Compose](https://docs.docker.com/compose/).

To show how our web app would scale in this context, we are going to run two instances of Django app. Also, to make this even more interesting, we will run MongoDB as a [Replica Set](https://docs.mongodb.com/manual/replication) with one primary node and two secondary nodes.

Given we have two instances of Django app, we will need a way to balance a load among those two instances, so we are going to use [Fabio](https://fabiolb.net/), a Consul aware load-balancer, to reach Django app instances.

This example will roughly help us simulate a real-world practical application.

![Example Application nodes and services deployed on them](https://cdn-images-1.medium.com/max/4000/1*lCaO-EHi-3NXpaMSfO5A6w.png)*Example Application nodes and services deployed on them*

The complete source code for this application is open-sourced and is available on GitHub — [pranavcode/consul-demo](https://github.com/pranavcode/consul-demo).
> Note: The architecture we are discussing here is not specifically constraint with any of the technologies used to create app or data layers. This example could very well be built using a combination of Ruby on Rails and Postgres, or Node.js and MongoDB, or Laravel and MySQL.

## How Does Consul Come into the Picture?

We are deploying both, the app and the data, layers with Docker containers. They are going to be built as services and will talk to each other over HTTP.

Thus, we will use [Consul for Service Discovery](https://velotio.com/blog/2019/3/11/hashicorp-consul-guide-1). This will allow Django servers to find [MongoDB Primary](https://docs.mongodb.com/manual/core/replica-set-primary) node. We are going to use Consul to resolve services via [Consul’s DNS interface](https://www.consul.io/docs/agent/dns.html) for this example.

Consul will also help us with the [auto-configuration of Fabio](https://github.com/fabiolb/fabio/wiki/Configuration) as load-balancer to reach instances of our Django app.

We are also using the health-check feature of Consul to monitor the health of each of our instances in the whole infrastructure.

Consul provides a beautiful user interface, as part of its Web UI, out of the box to show all the services on a single dashboard. We will use it to see how our services are laid out.

Let’s begin.

## Setup: MongoDB, Django, Consul, Fabio, and Dockerization

We will keep this as simple and minimal as possible to the extent it fulfills our need for a demonstration.

### MongoDB

The MongoDB setup we are targeting is in the form of MongoDB Replica Set. One primary node and two secondary nodes.

The primary node will manage all the [write operations](https://docs.mongodb.com/manual/core/replica-set-write-concern) and the [oplog](https://docs.mongodb.com/manual/core/replica-set-oplog) to maintain the sequence of writes, and replicate the data across [secondaries](https://docs.mongodb.com/manual/core/replica-set-secondary). We are also configuring the secondaries for the [read operations](https://docs.mongodb.com/manual/core/read-preference). You can learn more about [MongoDB Replica Set](https://docs.mongodb.com/manual/tutorial/deploy-replica-set) on their official documentation.

We will call our replication set as ‘consuldemo’.

We will run MongoDB on a [standard port 27017](https://docs.mongodb.com/manual/reference/default-mongodb-port) and supply the name of the replica set on the command line using the parameter ‘ — replSet’.

As you may read from the documentation MongoDB also allows configuring replica set name via [configuration file](https://docs.mongodb.com/manual/reference/configuration-options) with the parameter for replication as below:

<iframe src="https://medium.com/media/af86d14ebd9438210799680ac49d9a53" frameborder=0></iframe>

In our case, the replication set configuration that we will apply on one of the MongoDB nodes, once all the nodes are up and running is as given below:

<iframe src="https://medium.com/media/8afd2245d75cdfc9e1499405c8ac146f" frameborder=0></iframe>

This configuration will be applied to one of the pre-defined nodes and [MongoDB will decide which node will be primary and secondary](https://docs.mongodb.com/manual/core/replica-set-elections).
> Note: We are not forcing the set creation with any pre-defined designations on who becomes primary and secondary to allow the dynamism in service discovery. Normally, the nodes would be defined for a specific role.

We are allowing slave reads and reads from the nearest node as a [Read Preference](https://docs.mongodb.com/manual/core/read-preference).

We will start MongoDB on all nodes with the following command:

<iframe src="https://medium.com/media/76d5e7408cb4fd7e3a3d362aff88158e" frameborder=0></iframe>

This gives us a MongoDB Replica Set with one primary instance and two secondary instances, running and ready to accept connections.

We will discuss containerizing the MongoDB service in the latter part of this article.

## Django

We will create a simple [Django project](https://realpython.com/django-setup/) that represents Blog application and containerizes it with Docker.

[Building the Django app from scratch](https://docs.djangoproject.com/en/2.1/intro/tutorial01/) is beyond the scope of this tutorial, we recommend you to refer to [Django’s official documentation](https://docs.djangoproject.com/en/2.1/) to get started with Django project. But, we will still go through some important aspects.

As we need our Django app to talk to MongoDB, we will use a MongoDB connector for Django ORM, [Djongo](https://nesdis.github.io/djongo/). We will set up our Django settings to use Djongo and connect with our MongoDB. Djongo is pretty straightforward in configuration.

For a local MongoDB installation it would only take two lines of code:

<iframe src="https://medium.com/media/04fbaf6ae126e08f56d2e914fbd1a789" frameborder=0></iframe>

In our case, as we will need to access MongoDB over another container, our config would look like this:

<iframe src="https://medium.com/media/13d05072d9ef7305ad22cbd3479bda20" frameborder=0></iframe>

Details:

* ENGINE: The database connector to use for Django ORM.

* NAME: Name of the database.

* HOST: Host address that has MongoDB running on it.

* PORT: Which port is your MongoDB listening for requests.

Djongo internally talks to [PyMongo](https://api.mongodb.com/python/current/) and uses [MongoClient](http://api.mongodb.com/python/current/api/pymongo/mongo_client.html) for executing queries on Mongo. We can also use other MongoDB connectors available for Django to achieve this, like, for instance, [django-mongodb-engine](https://django-mongodb-engine.readthedocs.io/) or [pymongo](https://github.com/mongodb/mongo-python-driver) directly, based on our needs.
> Note: We are currently reading and writing via Django to a single MongoDB host, the primary one, but we can configure Djongo to also talk to secondary hosts for read-only operations. That is not in the scope of our discussion. You can refer to Djongo’s official documentation to achieve exactly this.

Continuing our Django app building process, we need to define our models. As we are building a blog-like application, our models would look like this:

<iframe src="https://medium.com/media/81b4866c19e747ca8f02b604036ad8dd" frameborder=0></iframe>

We can run a local MongoDB instance and create migrations for these models. Also, register these models into our Django Admin, like so:

<iframe src="https://medium.com/media/30388009b2f25faab2dbe872362062bc" frameborder=0></iframe>

We can play with the Entry model’s CRUD operations via Django Admin for this example.

Also, to realize the Django-MongoDB connectivity we will create a custom View and Template that displays information about MongoDB setup and currently connected MongoDB host.

Our Django views look like this:

<iframe src="https://medium.com/media/ddf0467f3c1a7760ac434dd9467a0b61" frameborder=0></iframe>

Our URLs or routes configuration for the app looks like this:

<iframe src="https://medium.com/media/b2c972bcf6d7c4bea10edff0e6c026ce" frameborder=0></iframe>

And for the project — the app URLs are included like so:

<iframe src="https://medium.com/media/d5aa43615f764e193c7d9e71a3c1e3f5" frameborder=0></iframe>

Our Django template, *‘*templates/home.html*’* looks like this:

<iframe src="https://medium.com/media/eba873617c1085c4ff959b0526b29344" frameborder=0></iframe>

To run the app we need to migrate the database first using the command below:

<iframe src="https://medium.com/media/ed8b6d751c4418360517d4e2b4c3cd15" frameborder=0></iframe>

And also collect all the static assets into static directory:

<iframe src="https://medium.com/media/ad0e2949604a7205a1ba96fa48f8947d" frameborder=0></iframe>

Now run the Django app with [Gunicorn](https://gunicorn.org/), a WSGI HTTP server, as given below:

<iframe src="https://medium.com/media/bb4b07ebc19e5cab1e67fa3b452a0358" frameborder=0></iframe>

This gives us a basic blog-like Django app that connects to MongoDB backend.

We will discuss containerizing this Django web application in the latter part of this article.

## Consul

We place a [Consul agent](https://www.consul.io/docs/agent/basics.html) on every service as part of our Consul setup.

The Consul agent is responsible for service discovery by [registering the service on the Consul cluster](https://velotio.com/blog/2019/3/11/hashicorp-consul-guide-1) and also monitors the health of every service instance.

## Consul on nodes running MongoDB Replica Set

We will discuss Consul setup in the context of MongoDB Replica Set first — as it solves an interesting problem. At any given point of time, one of the MongoDB instances can either be a Primary or a Secondary.

The Consul agent registering and monitoring our MongoDB instance within a Replica Set has a unique mechanism — dynamically registering and deregistering MongoDB service as a Primary instance or a Secondary instance based on what Replica Set has designated it.

We achieve this dynamism by writing and running a shell script after an interval that toggles the Consul service definition for MongoDB Primary and MongoDB Secondary on the instance node’s Consul Agent.

The service definitions for MongoDB services are stored as JSON files on the Consul’s config directory ‘/etc/config.d’.

Service definition for MongoDB Primary instance:

<iframe src="https://medium.com/media/be9cdb36915ae38d8f14d9f9bece482d" frameborder=0></iframe>

If you look closely, the service definition allows us to get a DNS entry specific to MongoDB Primary, rather than a generic MongoDB instance. This allows us to send the database writes to a specific MongoDB instance. In the case of Replica Set, the writes are maintained by MongoDB Primary.

Thus, we are able to achieve both service discovery as well as health monitoring for Primary instance of MongoDB.

Similarly, with a slight change the service definition for MongoDB Secondary instance goes like this:

<iframe src="https://medium.com/media/85d8c030c36d5e0ddfdab7682966c9f6" frameborder=0></iframe>

Given all this context, can you think of the way we can dynamically switch these service definitions?

We can identify if the given [MongoDB instance is primary](https://docs.mongodb.com/manual/reference/command/isMaster) or not by running command `db.isMaster()` on MongoDB shell.

The check can we drafted as a shell script as:

<iframe src="https://medium.com/media/2db0c0a5d3e90f895e54631eb9cbb647" frameborder=0></iframe>

Similarly, the non-master or non-primary instances of MongoDB can also be checked against the same command, by checking a `secondary` value:

<iframe src="https://medium.com/media/9d446858dcb7445886db76f9c3033035" frameborder=0></iframe>

Note: We are using [jq](https://stedolan.github.io/jq/) — a lightweight and flexible command-line JSON processor — to process the JSON encoded output of MongoDB shell commands.

One way of writing a script that does this dynamic switch looks like this:

<iframe src="https://medium.com/media/f2be22670905cab55f363488419fe8b6" frameborder=0></iframe>

Note: This is an example script, but we can be more creative and optimize the script further.

Once we are done with our service definitions we can run the Consul agent on each MongoDB nodes. To run an agent we will use the following command:

<iframe src="https://medium.com/media/746576ed97025a4c38ee522e7b1de89b" frameborder=0></iframe>

Here, ‘consul_server’ represents the Consul Server running host. Similarly, we can run such agents on each of the other MongoDB instance nodes.

Note: If we have multiple MongoDB instances running on the same host, the service definition would change to reflect the different ports used by each instance to uniquely identify, discover and monitor individual MongoDB instance.

## Consul on nodes running Django App

For the Django application, Consul setup will be very simple. We only need to monitor Django app’s port on which Gunicorn is listening for requests.

The Consul service definition would look like this:

<iframe src="https://medium.com/media/04ea8af1a15a9dd982486bbb34bc1e46" frameborder=0></iframe>

Once we have the Consul service definition for Django app in place, we can run the Consul agent sitting on the node Django app is running as a service. To run the Consul agent we would fire the following command:

<iframe src="https://medium.com/media/999379553e0bacb40483d7e538502ba2" frameborder=0></iframe>

## Consul Server

We are running the Consul cluster with a dedicated Consul server node. The Consul server node can easily host, discover and monitor services running on it, exactly the same way as we did in the above sections for MongoDB and Django app.

To run Consul in server mode and allow agents to connect to it, we will fire the following command on the node that we want to run our Consul server:

<iframe src="https://medium.com/media/0274fa737abf29e987ddb4d056ccdc38" frameborder=0></iframe>

There are no services on our Consul server node for now, so there are no service definitions associated with this Consul agent configuration.

## Fabio

We are using the power of Fabio to be [auto-configurable](https://github.com/fabiolb/fabio) and being Consul-aware.

This makes our task of load-balancing the traffic to our Django app instances very easy.

To allow Fabio to auto-detect the services via Consul, one of the ways is to add a tag or update a [tag in the service definition](https://github.com/fabiolb/fabio/wiki/Service-Configuration) with a prefix and a service identifier `urlprefix-/<service>`. Our Consul’s service definition for Django app would now look like this:

<iframe src="https://medium.com/media/1e4a3be257bbb17f2ef2c550398a1581" frameborder=0></iframe>

In our case, the Django app or service is the only service that will need load-balancing, thus this Consul service definition change completes the requirement on Fabio setup.

## Dockerization

Our whole app is going to be deployed as a set of Docker containers. Let’s talk about how we are achieving it in the context of Consul.

## Dockerizing MongoDB Replica Set along with Consul Agent

We need to run a Consul agent as described above alongside MongoDB on the same Docker container, so we will need to run a custom ENTRYPOINT on the container to allow running two processes.

Note: This can also be achieved using Docker container level checks in Consul. So, you will be free to run a Consul agent on the host and check across service running in Docker container. Which, will essentially exec into the container to monitor the service.

To achieve this we will use a tool similar to [Foreman](https://www.theforeman.org/). It is a lifecycle management tool for physical and virtual servers — including provisioning, monitoring and configuring.

To be precise, we will use the Golang adoption of Foreman, [Goreman](https://github.com/mattn/goreman). It takes the configuration in the form of [Heroku’s Procfile](https://devcenter.heroku.com/articles/procfile) to maintain which processes to be kept alive on the host.

In our case, the Procfile looks like this:

<iframe src="https://medium.com/media/5b4874466342aa26522b3424f1e3710a" frameborder=0></iframe>

The `consul_check` at the end of the Profile maintains the dynamism between both Primary and Secondary MongoDB node checks, based on who is voted for which role within MongoDB Replica Set.

The shell scripts that are executed by the respective keys on the Procfile are as defined previously in this discussion.

Our [Dockerfile](https://docs.docker.com/engine/reference/builder/), with some additional tools for debug and diagnostics, would look like:

<iframe src="https://medium.com/media/a868d15e9c4565e683fb1786ad85d2a3" frameborder=0></iframe>
> *Note: We have used bare Ubuntu 18.04 image here for our purposes, but you can use [official MongoDB image](https://hub.docker.com/_/mongo) and adapt it to run Consul alongside MongoDB or even do Consul checks on Docker container level as mentioned in the official documentation.*

## Dockerizing Django Web Application along with Consul Agent

We also need to run a Consul agent alongside our Django App on the same Docker container as we had with MongoDB container.

<iframe src="https://medium.com/media/21d296edf716a5fe675e7e3ef1104cbd" frameborder=0></iframe>

Similarly, we will have the Dockerfile for Django Web Application as we had for our MongoDB containers.

<iframe src="https://medium.com/media/8650d91457ba274c5fa0523803f8f0cd" frameborder=0></iframe>

## Dockerizing Consul Server

We are maintaining the same flow with Consul server node to run it with custom ENTRYPOINT. It is not a requirement, but we are maintaining a consistent view of different Consul run files.

Also, we are using Ubuntu 18.04 image for the demonstration. You can very well use[ Consul’s official image](https://hub.docker.com/_/consul) for this, that accepts all the custom parameters as are mentioned here.

<iframe src="https://medium.com/media/36a8ed6b643eedc41215c78bb631535f" frameborder=0></iframe>

## Docker Compose

We are using [Compose](https://github.com/docker/compose) to run all our Docker containers in a desired, repeatable form.

Our [Compose file](https://docs.docker.com/compose/compose-file/) is written to denote all the aspects that we mentioned above and utilize the power of Docker Compose tool to achieve those in a seamless fashion.

Docker Compose file would look like the one given below:

<iframe src="https://medium.com/media/7c8384a8861e3c210fa5b9e408ad8051" frameborder=0></iframe>

That brings us to the end of the whole environment setup. We can now run Docker Compose to build and run the containers.

## Service Discovery using Consul

When all the services are up and running the Consul Web UI gives us a nice glance at our overall setup.

![Consul Web UI showing the set of services we are running and their current state](https://cdn-images-1.medium.com/max/4000/1*_dLrLx9cZtkzsOae51jL4Q.png)*Consul Web UI showing the set of services we are running and their current state*

The MongoDB service is available for Django app to discover by virtue of Consul’s DNS interface.

<iframe src="https://medium.com/media/11418e853ffa5f706cb4f7a1200c4d73" frameborder=0></iframe>

Django App can now connect MongoDB Primary instance and start writing data to it.

We can use Fabio [load-balancer](https://velotio.com/blog/2017/7/5/http-load-balancing-in-kubernetes-with-ingress) to connect to Django App instance by auto-discovering it via Consul registry using specialized service tags and render the page with all the database connection information we are talking about.

Our load-balancer is sitting at ‘33.10.0.100’ and ‘/web’ is configured to be routed to one of our Django application instances running behind the load-balancer.

![Fabio auto-detecting the Django Web Application end-points](https://cdn-images-1.medium.com/max/4000/1*CA5mnHUsF8CEvBO_P6T97A.png)*Fabio auto-detecting the Django Web Application end-points*

As you can see from the auto-detection and configuration of Fabio load-balancer from its UI above, it has weighted the Django Web Application end-points equally. This will help balance the request or traffic load on the Django application instances.

When we visit our Fabio URL ‘33.10.0.100:9999’ and use the source route as ‘/web’ we are routed to one of the Django instances. So, visiting ‘33.10.0.100:9999/web’ gives us the following output.

![Django Web Application renders the MongoDB connection status on the home page](https://cdn-images-1.medium.com/max/4000/1*xTWsEhzx3MZ3tClB2plIgg.png)*Django Web Application renders the MongoDB connection status on the home page*

We are able to restrict Fabio to only load-balance Django app instances by only adding required tags to Consul’s service definitions of Django app services.

This MongoDB Primary instance discovery helps Django app to do database migration and app deployment.

One can explore Consul Web UI to see all the instances of Django web application services.

![Django Web Application services as seen on Consul’s Web UI](https://cdn-images-1.medium.com/max/4000/1*FbVyRmSrR4U5nIHlTTPSNg.png)*Django Web Application services as seen on Consul’s Web UI*

Similarly, see how MongoDB Replica Set instances are laid out.

![MongoDB Replica Set Primary service as seen on Consul’s Web UI](https://cdn-images-1.medium.com/max/4000/1*vCG24fB3kPR5mcANC8aIXg.png)*MongoDB Replica Set Primary service as seen on Consul’s Web UI*

![MongoDB Replica Set Secondary services as seen on Consul’s Web UI](https://cdn-images-1.medium.com/max/4000/1*Va1owv0JHT4MhqPhR91YFg.png)*MongoDB Replica Set Secondary services as seen on Consul’s Web UI*

Let’s see how Consul helps with health-checking services and discovering only the alive services.

We will stop the current MongoDB Replica Set Primary (‘mongo_2’) container, to see what happens.

![MongoDB Primary service being swapped with one of the MongoDB Secondary instances](https://cdn-images-1.medium.com/max/4000/1*5Sc66X6dg058dYspGaDf-w.png)*MongoDB Primary service being swapped with one of the MongoDB Secondary instances*

![MongoDB Secondary instance set is now left with only one service instance](https://cdn-images-1.medium.com/max/4000/1*NKuSVxrnL8uhc5jGH3LncQ.png)*MongoDB Secondary instance set is now left with only one service instance*

Consul has started failing the health-check for previous MongoDB Primary service. MongoDB Replica Set has also detected that the node is down and the re-election of Primary node needs to be done. Thus, getting us a new MongoDB Primary (‘mongo_3’) automatically.

Our checks toggle has kicked-in and swapped the check on ‘mongo_3’ from MongoDB Secondary check to MongoDB Primary check.

When we take a look at the view from the Django app, we see it is now connected to a new MongoDB Primary service (‘mongo_3’).

![Switching off the MongoDB Primary is also reflected in the Django Web Application](https://cdn-images-1.medium.com/max/4000/1*UniePbab61Mbc3gDYreLTg.png)*Switching off the MongoDB Primary is also reflected in the Django Web Application*

Let’s see how this plays out when we bring back the stopped MongoDB instance.

![Failing MongoDB Primary service instance is now cleared out from service instances as it is now healthy MongoDB Secondary service instance](https://cdn-images-1.medium.com/max/4000/1*s8r74QpN1YJc-rj6d5DzkQ.png)*Failing MongoDB Primary service instance is now cleared out from service instances as it is now healthy MongoDB Secondary service instance*

![Previously failed MongoDB Primary service instance is now re-adopted as MongoDB Secondary service instance as it has become healthy again](https://cdn-images-1.medium.com/max/4000/1*CU9hN3WN1JpHj_YQ65QlpA.png)*Previously failed MongoDB Primary service instance is now re-adopted as MongoDB Secondary service instance as it has become healthy again*

Similarly, if we stop the service instances of Django application, Fabio would now be able to detect only a healthy instance and would only route the traffic to that instance.

![Fabio is able to auto-configure itself using Consul’s service registry and detecting alive service instances](https://cdn-images-1.medium.com/max/4000/1*PMnZkcBb7KOkjiYnRn299g.png)*Fabio is able to auto-configure itself using Consul’s service registry and detecting alive service instances*

This is how one can use Consul’s service discovery capability to discover, monitor and health-check services.

## Service Configuration using Consul

Currently, we are configuring Django application instances directly either from[ environment variables](https://docs.djangoproject.com/en/2.1/topics/settings/) [set within the containers by Docker Compose](https://docs.docker.com/compose/environment-variables/) and consuming them in Django project settings or by [hard-coding the configuration parameters](https://docs.djangoproject.com/en/2.1/ref/settings/) directly.

We can use [Consul’s Key/Value](https://www.consul.io/api/kv.html) store to share configuration across both the instances of Django app.

We can use [Consul’s HTTP interface](https://www.consul.io/api/index.html) to store key/value pair and retrieve them within the app using the open-source Python client for Consul, called [python-consul](https://python-consul.readthedocs.io/en/latest/). You may also use any other Python library that can interact with Consul’s KV store if you want.

Let’s begin by looking at how we can set a key/value pair in Consul using its HTTP interface.

<iframe src="https://medium.com/media/ad1aaa999610a73f8be460ded1ede6f2" frameborder=0></iframe>

Once we set the KV store we can consume it on Django app instances to configure it with these values.

Let’s [install python-consul](https://pypi.org/project/python-consul/) and add it as a project dependency.

<iframe src="https://medium.com/media/1a66f06eb1ede25e329bf7fe42d82e0b" frameborder=0></iframe>

We will need to connect our app to Consul using python-consul.

<iframe src="https://medium.com/media/49c572af03c1ff7b7dcb0fa25d2d8e0c" frameborder=0></iframe>

We can capture and configure our Django app accordingly using the ‘python-consul’ library.

<iframe src="https://medium.com/media/014a0501f5439fcae13350ac4f408f00" frameborder=0></iframe>

These key/value pair from Consul’s KV store can also be viewed and updated from its Web UI.

![Consul KV store as seen on Consul Web UI with Django app configuration parameters](https://cdn-images-1.medium.com/max/4000/1*7ILgATz5jd-7eEZFGLFvlg.png)*Consul KV store as seen on Consul Web UI with Django app configuration parameters*

The code used as part of this guide for Consul’s service configuration section is available on ‘service-configuration’ branch of [pranavcode/consul-demo ](https://github.com/pranavcode/consul-demo)project.

That is how one can use Consul’s KV store and configure individual services in their architecture with ease.

## Service Segmentation using Consul

As part of Consul’s [Service Segmentation](https://www.consul.io/segmentation.html) we are going to look at [Consul Connect intentions](https://www.consul.io/docs/connect/intentions.html) and [data center distribution](https://www.consul.io/docs/guides/datacenters.html).

Connect provides service-to-service connection authorization and encryption using [mutual TLS](https://en.wikipedia.org/wiki/Mutual_authentication).

To use Consul you need to enable it in the server configuration. Connect needs to be enabled across the Consul cluster for proper functioning of the cluster.

<iframe src="https://medium.com/media/cf021614bed7ff276231341e275f3a11" frameborder=0></iframe>

In our context, we can define that the communication is to be TLS identified and secured we will define an upstream sidecar service with a proxy on Django app for its communication with MongoDB Primary instance.

<iframe src="https://medium.com/media/18a5a109ce95c9d0cc7d42d4351f205d" frameborder=0></iframe>

Along with Connect configuration of sidecar proxy, we will also need to run the Connect proxy for Django app as well. This could be achieved by running the following command.

<iframe src="https://medium.com/media/18a5a109ce95c9d0cc7d42d4351f205d" frameborder=0></iframe>

We can add Consul Connect Intentions to create [a service graph](https://velotio.com/blog/2019/3/11/hashicorp-consul-guide-1) across all the services and define traffic patterns. We can create intentions as shown below:

<iframe src="https://medium.com/media/49925ce84711b7c27cf81a81c4c22064" frameborder=0></iframe>

Intentions for service graph can also be managed from Consul Web UI.

![Define access control for services via Connect and service connection restrictions](https://cdn-images-1.medium.com/max/4000/1*s3C9kuDlVpADZIfy34lF2A.png)*Define access control for services via Connect and service connection restrictions*

This defines the service connection restrictions to allow or deny them to talk via Connect.

We have also added ability on Consul agents to denote which datacenters they belong to and be accessible via one or more Consul servers in a given datacenter.

The code used as part of this guide for Consul’s service segmentation section is available on ‘service-segmentation’ branch of [velotiotech/consul-demo](https://github.com/velotiotech/consul-demo) project.

That is how one can use Consul’s service segmentation feature and configure service level connection access control.

## Conclusion

Having an ability to seamlessly control the service mesh that Consul provides makes the life of an operator very easy. We hope you have learned how Consul can be used for service discovery, configuration, and segmentation with its practical implementation.

As usual, we hope it was an informative ride on the journey of Consul. This was the final piece of this two-part series. This part tries to cover most of the aspects of Consul architecture and how it fits into your current project. In case you miss the first part, find it [here](https://velotio.com/blog/2019/3/11/hashicorp-consul-guide-1).

We will continue our endeavors with different technologies and get you the most valuable information that we possibly can in every interaction. Let’s us know what you would like to hear from us more or if you have any questions around the topic, we will be more than happy to answer those.

## References

* [Consul Demo that complements this guide](https://github.com/pranavcode/consul-demo)

* [HashiCorp Consul](https://www.consul.io/) and[ its repo on GitHub](https://github.com/hashicorp/consul)

* [HashiCorp Consul Guides](https://www.consul.io/docs/guides/index.html) and[ Code](https://github.com/hashicorp/consul-guides)

*******************************************************************

*This post was originally published on [**Velotio Blog](https://velotio.com/blog/2019/4/5/hashicorp-consul-guide-2).***

[**Velotio Technologies](https://velotio.com/)** is an outsourced software product development partner for technology startups and enterprises. We specialize in enterprise B2B and SaaS product development with a focus on artificial intelligence and machine learning, DevOps, and test engineering.

*Interested in learning more about us? We would love to connect with you on our [**Website](https://velotio.com)**, [**LinkedIn](https://www.linkedin.com/company/velotio-technologies/)** or [**Twitter](https://twitter.com/velotiotech)**.*

*******************************************************************
