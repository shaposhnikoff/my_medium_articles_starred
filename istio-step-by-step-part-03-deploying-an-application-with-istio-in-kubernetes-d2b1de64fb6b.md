
# Istio step-by-step Part 03 — Deploying an application with Istio in Kubernetes

Hi! Welcome back to my Istio step-by-step tutorial series. Through this tutorial, I am going to deploy a small application with Istio in Kubernetes.

![Photo credits: [https://www.pexels.com/photo/background-blank-business-composition-583848/](https://www.pexels.com/photo/background-blank-business-composition-583848/)](https://cdn-images-1.medium.com/max/9216/1*Xa4XxF_WAUuFAHqV0J7EGg.jpeg)*Photo credits: [https://www.pexels.com/photo/background-blank-business-composition-583848/](https://www.pexels.com/photo/background-blank-business-composition-583848/)*
> Please refer to my 12th article on [deploying-istio-bookinfo-application-in-minikub](https://medium.com/@nethminiromina/istio-step-by-step-part-12-deploying-istio-bookinfo-application-in-minikube-501c90cfeab4)e for more information on Bookinfo application.

First I have to mention that Istio has released a new version as [Istio 1.0.3](https://preliminary.istio.io/about/notes/1.0.3/) and you can check for more details about that version from their website. Also in this tutorial, I am using Istio 1.0.3 to deploy the application.

As a quick tour, first, we are going to create a small service-mesh with two services. One is a small telnet file and the other one is a simple Ballerina Hello-World service. Then inject istio-proxy for both services and invoke them. Invoking services are done in two parts as ***invoking the service from inside the service-mesh*** and** *invoke the service from outside of the service mesh***.

Here I am using Ballerina is because it is very easy when working with Docker, Kubernetes and other stuff. So, if you are new to [Ballerina](https://ballerina.io/), you can go through their website. There are many [examples](https://ballerina.io/learn/by-example/) for you. Second is you have to install Istio correctly and make sure that all the pods with “Istio-system” namespace are running. So that’s all, let’s move to work.

![](https://cdn-images-1.medium.com/max/2000/1*7u3LQZl6nkDL5i2-6W5HjQ.png)

Before starting the work you have to enable the *istio-injection* by executing the command,

    kubectl label namespace default istio-injection=enabled --overwrite

Well now let’s move to work.

## First, create a Docker container image and deploy

Use the following code to create a Docker container image and save the file with the name **Dockerfile**.

    FROM alpine:latest
    RUN apk update && apk add curl busybox-extras
    ENTRYPOINT [**"tail"**,**"-f"**,**"/dev/null"**]

Normally when you build a Dockerfile, Docker will create images and store them in the local machine’s Docker registry. But, in this tutorial, I am not using the local machine’s Docker registry; I am using the Docker registry of the Docker daemon running *inside* Minikube’s VM instance. So, to point the ‘docker’ command to your Minikube’s Docker daemon use the command,

    eval $(sudo minikube docker-env)

Make sure that you are in the directory where you Dockerfile is. Now build the docker image using a name and a tag what you wish. Here I am using hellodemo as the name and v1 as the tag. ***Remember to use small letters for the name.***

    docker build -t hellodemo:v1 .

Hope you have something like this in your terminal.

![](https://cdn-images-1.medium.com/max/2150/1*PYnns5a6wDJNUFPFHtAYxQ.png)

If you want you to check whether your image is in Minikube’s Docker registry, you can do it by,

    sudo minikube ssh docker images

Great, you have completed the first part of deploying a Docker container image. Now will create a deployment.

    kubectl run **hellodemo** --image=hellodemo:v1 --port=9095 --image-pull-policy=IfNotPresent

Here **hellodemo **is the name of the deployment and you can check your deployment, you can run the command,

    kubectl get deployments

![](https://cdn-images-1.medium.com/max/3374/1*pNlCZQvUh2v7KRhGztNFDA.png)

Also, you can check your pod by running the command

    kubectl get pods

![](https://cdn-images-1.medium.com/max/2000/1*cn05FWMeV8HIp4pucK53dA.png)

You can see under the **READY** column, there is mentioned as **2/2**. That means in your pod there are two containers. Yes, of course, there must be two containers because one is the service container and the other one is the istio-proxy. For more observation, you can check the pod description by executing the command,

    kubectl describe pod <pod_name>

Awesome, you have completed the second part of deploying Docker container image. Now as the final part you have to create a service. To this run the command,

    kubectl expose deployment hellodemo --type=NodePort

Manual load balancers don’t communicate with the cluster to find out where the backing pods are running, and we must expose the Service with type: NodePort and they are only available on high ports, 30000-32767.

By default, a Pod is only accessible by its internal IP address within the Kubernetes cluster. But, to make the container accessible from outside the Kubernetes virtual network, you have to expose the Pod as a Kubernetes **Service**.

You can check your service by running the command,

    kubectl get svc

![](https://cdn-images-1.medium.com/max/2574/1*2hUU9etCQBmc3TgSwOWaTw.png)

Great you have completed the first task.

## Now, let's deploy the Ballerina Hello World service.

In this task, I am using IntelliJ as the IDE. If you are using IntelliJ, you have to install the ballerina plugin. You can refer this [document](https://medium.com/@shan1024/setting-up-ballerina-in-intellij-idea-e49e7e150a54) to install the plugin.
> FYI: This ballerina service is not valid for the new version of Ballerina. Please refer this [link](https://github.com/Nethminiromina/istio-helloWorld) for related artifacts.

Well first create a ballerina service. *Remember to choose **Ballerina Service **for the “Kind” when creating the ballerina file. *Next, add Kubernetes annotations that are required to generate the Kubernetes deployment artifacts.

![](https://cdn-images-1.medium.com/max/2000/1*sQCk9cNal-h_l_jsd4ZlZQ.png)

Here is my ballerina code with all Kubernetes annotations.

    **import **ballerina/http;
    **import **ballerina/log;
    **import **ballerinax/kubernetes;
    *
    *@kubernetes:Service {
        serviceType:**"NodePort"**,
        name:**"helloworldservice"
    **}
    
    **endpoint **http:Listener listener {
        port:9095
    };
    
    @kubernetes:Deployment {
        image: **"helloworldservice"**,
        name: **"helloworldservice"**,
        dockerHost:**"tcp://<minikube ip>:2376"**, *// IP can be obtained via `sudo minikube ip` command
        *dockerCertPath:**"<Home Dir>/.minikube/certs"**,
        singleYAML:**true
    **}

    @http:ServiceConfig {basePath:**"/helloworld"**}
    **service**<http:Service> hello **bind **listener {
        @http:ResourceConfig{
            path: **"/"**,
            methods: [**"GET"**]
        }
        sayHello(**endpoint **caller, http:Request req) {
            http:Response res = **new**;
    
            res.setPayload(**"Hello, World!\n"**);
    
            caller**->**respond(res) **but **{ error e **=> **log:printError(
               **"Error sending response"**, err = e) };
        }
    }

Note that if you are using **minikube**, you need to specify the *dockerHost* and *dockerCertPath*.

That's all for creating the document and now will build the code. For this run the command,

    ballerina build <.bal file name>

![](https://cdn-images-1.medium.com/max/2174/1*-vyMePQVf0vHDgtrBla00A.png)

Now if you check the project structure you will find out a generated YAML file.

![](https://cdn-images-1.medium.com/max/2000/1*1lZFOj8_VHMEkfMtqlElyw.png)

It contains all the configurations of the service and deployment.

Then run the commands in the terminal to deploy the Kubernetes artifacts and check whether the service, deployment and pods are deployed correctly.

![](https://cdn-images-1.medium.com/max/2000/1*B2BHdtbPpX_juTNWZ8MLYw.jpeg)

Excellent, we have completed two tasks of the tutorial. Now we have two pods with two containers in each and two services along with two deployments.

## Invoke the hello-world service from inside the mesh

Okay so let’s try to invoke the hello-world service from the hellodemo container. To do this, we have to go into the hellodemo container and run the curl command.

    kubectl exec -it hellodemo-6889b97ff-g4cf7 -c hellodemo -- curl http://helloworldservice:9095/helloworld/

![](https://cdn-images-1.medium.com/max/3778/1*-oHEdv4KYydIM1vQtY343A.png)

You can see in the terminal there is printed as “*Hello, World!*”. That means the service is invoked.

## Invoke the service from outside of the mesh

This is the final stage of our tutorial. To do this task we need a *Gateway* and a *Virtual service*. You know Kubernetes has used an *Ingress *controller to handle the traffic that enters the cluster from the outside. But Istio has replaced it with two components named Gateway and Virtualservice.

The Gateways are used to configure the ports for Envoy, while the VirtualServices are used to enable the [intelligent routing](https://istio.io/docs/examples/intelligent-routing/) that is one of the very reasons we want to use Istio in the first place. These two work in concert to configure the Envoy.

In our case, we need a gateway and a Virtualservice for our hello-world service. Here are the Gateway and the Virtualservice of our service.

### Gateway.yaml

    **apiVersion**: networking.istio.io/v1alpha3
    **kind**: Gateway
    **metadata**:
      **name**: helloworld-gateway
    **spec**:
      **selector**:
        **istio**: ingressgateway *# use istio default controller
      ***servers**:
      - **port**:
          **number**: 80
          **name**: http
          **protocol**: HTTP
        **hosts**:
        - **"*"**

This Gateway configuration sets up a proxy to act as a loadbalancer exposing port 80 for HTTP ingress.

Another important point is,

    **spec**:
      **selector**:
        **istio**: ingressgateway

in here what is done is, as the istio-ingresgateway pods are tagged with the label “istio=ingressgateway”, this pod will be the one that receives this gateway configuration and ultimately expose the port because it matches the Istio Gateway, label selector.

### Virtualservice.yaml

    **apiVersion**: networking.istio.io/v1alpha3
    **kind**: VirtualService
    **metadata**:
      **name**: **"helloworldservice"
    spec**:
      **hosts**:
      - **"*"
      gateways**:
      - helloworld-gateway
      **http**:
      - **match**:
        - **uri**:
            **exact**: /helloworld
        **route**:
        - **destination**:
            **host**: **"helloworldservice"
            port**:
              **number**: 9095

A VirtualService can then be bound to a gateway to control the forwarding of traffic arriving at a particular host or gateway port. Like in above Virtualservice under Specifications, gateways are assigned to “helloworld-gateway” and it forwards traffic arriving from URI exact “/helloworld” to internal helloworldservice on port 9095.

Here is the project structure.

![](https://cdn-images-1.medium.com/max/2000/1*_yU5Ay986z0E1DDHZjdofA.jpeg)

Great, now will apply these to components to the service and try to invoke them from outside.

To apply these for the service, run the command

    kubectl apply -f Gateway.yaml

    kubectl apply -f Virtualservice.yaml

![](https://cdn-images-1.medium.com/max/2000/1*JOmbEP5bwj8oIOeBpYmVng.png)

So, to invoke the service we have to determine the ingress IP and port. As we are using NodePort, use following commands to set Ingress Port.

    export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

    export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')

Also as we are using minikube use the following command to set the Ingress IP.

    export INGRESS_HOST=$(sudo minikube ip)

All set and let’s invoke the service from outside. To do this run the command,

    curl http://$INGRESS_HOST:$INGRESS_PORT/{basepath}

According to my program,

    curl http://$INGRESS_HOST:$INGRESS_PORT/helloworld

Here is my output.

![](https://cdn-images-1.medium.com/max/3588/1*Up-mXLGIBS52K0E4vXqRpA.png)

So, you can see the service can be invoked from outside of the service-mesh too.

To your knowledge:

* Why we use port 9095 when invoking the service from other services? *The reason is, you know in the gateway inside the service-mesh services are listening to port 9095 according to our configurations.*

This is all for this tutorial and hope you could understand the basic scenario happens. Comment below if you need a video tutorial for this session. See you soon with the next tutorial. Keep in touch to learn more. Thank you.

<<[Previous tutorial](https://medium.com/@nethminiromina/istio-step-by-step-part-02-getting-started-with-istio-c24ed8137741)

[Next tutorial](https://medium.com/@nethminiromina/istio-step-by-step-part-04-traffic-routing-path-of-istio-service-mesh-part-a-ingress-routing-28e03cdaa048) >>

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
