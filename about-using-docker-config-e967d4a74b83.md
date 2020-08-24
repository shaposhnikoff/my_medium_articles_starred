Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m177[39m, end: [33m198[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m7[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m68[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m107[39m, end: [33m113[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m31[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m111[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m44[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m95[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m34[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m77[39m, end: [33m81[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m59[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m19[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m36[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m127[39m, end: [33m136[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m42[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m110[39m, end: [33m130[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m277[39m, end: [33m288[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m293[39m, end: [33m307[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m48[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m99[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m127[39m, end: [33m135[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m157[39m, end: [33m163[39m }

# Docker Tips: Using Docker Config

Photo by Alejandro Escamilla on Unsplash

## TL;DR

What about using a [Docker](http://docker.com) config instead of creating an image with an embedded configuration ?

## Embedding Configuration in an Image?

We often see Dockerfiles like the following one, where a new image is created only to add a configuration to a base image.

    $ cat Dockerfile
    FROM nginx:1.13.6
    COPY nginx.conf /etc/nginx/nginx.conf

In this example, the local nginx.conf configuration file is copied over to the [NGINX](https://www.nginx.com/) image’s filesystem in order to overwrite the default configuration file, the one shipped in /etc/nginx/nginx.conf.

One of the main drawbacks of this approach is that the image needs to be rebuilt if the configuration changes.

## Docker Config Enters in the Picture

Configs are available for services since Docker 17.06. Where secrets exist to store sensitive information, config allows to store non-sensitive information, such as configuration files, outside a service’s image.

As for the other Docker primitives (container, image, volume), config has its own set of commands in the CLI.

    $ docker config --help

    Usage: docker config COMMAND

    Manage Docker configs

    Options:

    Commands:
     create Create a configuration file from a file or STDIN as content
     inspect Display detailed information on one or more config files
     ls List configs
     rm Remove one or more configuration files

    Run ‘docker config COMMAND — help’ for more information on a command.

## **Create a Config**

Coming back to the previous example, instead of copying the NGINX configuration file in an image, we will create a Docker config.

In this example, the nginx.conf file contains the following content.

<iframe src="https://medium.com/media/e21e0a2c31671fd4a3e87ee435ab9a76" frameborder=0></iframe>

It basically defines a web server that listens on port 8000 and forwards all the HTTP requests arriving on the /api endpoint to the upstream API server.

Using the Docker CLI, we can create a config from this configuration file, we name this config proxy.

    $ docker config create proxy nginx.conf
    mdcfnxud53ve6jgcgjkhflg0s

We can then inspect the config as we would do with any other Docker primitives:

    $ docker config inspect proxy
    [
      {
        "ID": "x06uaozphg9kbnf8g4az4mucn",
        "Version": {
          "Index": 2723
        },
        "CreatedAt": "2017–11–21T07:49:09.553666064Z",
        "UpdatedAt": "2017–11–21T07:49:09.553666064Z",
        "Spec": {
          "Name": "proxy,
          "Labels": {},
          "Data": "dXNlciB3d3ctZGF0YTsKd29y...ogIgICAgIH0KICAgIH0KfQo="
        }
      }
    ]

The data are only Base64 encoded and can be easily decoded.

<iframe src="https://medium.com/media/76ed01a7bd6d978856fda69c77f58e44" frameborder=0></iframe>

Per definition, the data within a config is not confidential and thus not encrypted.

## Use a Config

Now that we have created the proxy config, we will see how we can use it in services. To do so, we will consider two approaches — using the command line and using a Docker stack.

In both cases, we define two services:

* **API**: a simple HTTP server listening on port 80 and sending back a suggestion of a city to visit each time it receives a Get request.

* A **proxy** using the config created above and sending all traffic targeting the /api endpoint to the upstream API service.

We’ll test this setup by sending a HTTP Get request on the /api endpoint on the port 8000 of the proxy and making sure we get a response coming from the API.

### **Using the command line**

We start by creating an overlay network. We will use this so both services, API and proxy, can communicate together.

    $ docker network create --driver overlay front

Then we create the api service.

    $ docker service create --name api --network front lucj/api

The last step is to create the proxy service.

    $ docker service create --name proxy \
      --name proxy \
      --network front \
      --config src=proxy,target=/etc/nginx/nginx.conf \
      --port 8000:8000 \
      nginx:1.13.6

Once everything is in place, let’s send an HTTP request on localhost, port 8000 (the port published on the host by the proxy service)

    $ curl localhost:8000/api
    {“msg”:”c249837f1f58 suggests to visit Emosiba”}

The request hits the proxy service which forwards it to the API. In other words, the NGINX configuration file provided as a Docker config was correctly taken into account.

### **Using a Docker Compose file**

Of course, using a Compose file to define the application and deploy it as a Docker stack is more convenient. We then create a stack.yml file with the following content.

<iframe src="https://medium.com/media/9497f942915cc5f375e948298930e458" frameborder=0></iframe>

Note: as the config is created prior to running the application, it is defined as external in this file.

The application can then be run with the following command:

    $ docker stack deploy -c stack.yml test
    Creating service test_proxy
    Creating service test_api

Let’s send an HTTP Get request to the /api endpoint of the proxy service.

    $ curl localhost:8000/api
    {“msg”:”f462d568c0b0 suggests to visit Onitufdu”}

As before, the request is forwarded to the API service.

## Service Update

When the content of a configuration needs to be modified, it’s a common pattern to create a new config (using docker config create), and then to update the service order to remove the access to the previous config, and to add the access to the new one. The service commands are--config-rm and -- config-add.

Let’s create a new config from the nginx-v2.conf file.

    $ docker config create proxy-v2 nginx-v2.conf
    xtd1s1g6b5zukjhvup5vi4jzd

We can then update the service with the following command. In doing so, we remove the** **config named proxy and add the one named proxy-v2**.**

    $ docker service update --config-rm proxy --config-add src=proxy-v2,target=/etc/nginx/nginx.conf proxy

Note: by default, when a config is attached to a service, it is available in the /config_name file. We then need to explicitly define the location using the target option.

## Summary

Config is a pretty neat thing which helps to decouple the application from its configuration. Do you use config in your application?
