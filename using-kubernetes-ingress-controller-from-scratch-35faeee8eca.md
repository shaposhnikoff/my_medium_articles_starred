
# Using Kubernetes Ingress Controller from scratch

Kubernetes(k8s) provides great feature to help you maintain your docker images. with k8s ingress controller, allow you to handle the income traffic easily, let’s get started to see its magic.

### prerequisite

In order to make thing works, you need docker, kubectl and a cluster ready, you may want to follow [this article ](http://kubernetes.io/docs/hellonode/)to set up.

you can find the project used to build the docker image in this tutorial [here](https://github.com/samwalker505/test-ingress)

### Goal

Create two services(namely node1, node2)and a ingress controller. the ingress controller will route /gg1 to node1, /gg2 to node2

### step1: Create docker image

build the docker image and push it to container engine

    docker build -t test-ingress .
    gcloud docker push gcr.io/*$YOUR_PROJECT*/test-ingress

### step2: Create deployment

create a deployment with the following config

<iframe src="https://medium.com/media/8dea32da738c86ad10ec9c0f01384209" frameborder=0></iframe>

use kubectl to create a deployment

    kebectl create -f deployment1-config.yaml

repeat the above procedure and replace test-ingress-node1 with test-ingress-node-2

use command **kubectl get pods** to check , it should be like this

    NAME                                  READY     STATUS    RESTARTS   AGE
    test-ingress-node-1-379438432-gv98l   1/1       Running   0          18h
    test-ingress-node-2-564118882-mzjx1   1/1       Running   0          17h

### step3: Create service connecting deployment

we need to expose the deployment by creating a service so that they can communicate with each other. we can achieve it by

    kubectl expose deployment test-ingress-node-1 --target-port=8080 --type=NodePort

repeat the above procedure and replace test-ingress-node1 with test-ingress-node-2

use **kubectl get services** to check, it should look like this

    NAME                  CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
    test-ingress-node-1   10.123.245.45    <nodes>       8080/TCP   16h
    test-ingress-node-2   10.123.245.144   <nodes>       8080/TCP   15h

### **step4: Create Ingress controller**

Ingress resources needed a Ingress Controller to handle it. you can find a implementation you need. In this tutorial, I will use the implementation of nginx as example

    kubectl run nginx --image=nginx --port=80

next, expose it to each node

    kubectl expose deployment nginx --target-port=80 --type=NodePort

check all pods is working

    NAME                                  READY     STATUS    RESTARTS   AGE
    nginx-2032906785-fcjgz                1/1       Running   0          17h
    test-ingress-node-1-379438432-gv98l   1/1       Running   0          18h
    test-ingress-node-2-564118882-mzjx1   1/1       Running   0          18h

### Step5: Setup Ingress Resource

create ingress resource with following config

<iframe src="https://medium.com/media/47e4f8a7f643c97a153b4eeac12b7edf" frameborder=0></iframe>

you can specify the default node in backend, and route to different node by the path in rules, for more please check the [doc](http://kubernetes.io/docs/user-guide/ingress/).

    kubectl create -f ingress-config.yaml

it would expose an external ip, you can check the resource by

**kubectl get ing,** it should look like this

    NAME                    HOSTS     ADDRESS           PORTS     AGE
    test-node-adv-ingress   *         107.178.249.214   80        16h

from here 107.178.249.214 will be the external ip which is public accessible.

and you can use** kubectl describe ingress test-node-adv-ingress** to view its behavior, like this:

    Name:   test-node-adv-ingress
    Namespace:  default
    Address:  107.178.249.214
    Default backend: test-ingress-node-1:8080 (10.120.0.7:8080)
    Rules:
      Host Path Backends
      ---- ---- --------
      *
         /gg1  test-ingress-node-1:8080 (10.120.0.7:8080)
         /gg2  test-ingress-node-2:8080 (10.120.0.9:8080)
    Annotations:
      backends:  {"k8s-be-30113--618acb1b4e2fe823":"HEALTHY","k8s-be-31931--618acb1b4e2fe823":"HEALTHY"}
      forwarding-rule: k8s-fw-default-test-node-adv-ingress--618acb1b4e2fe823
      target-proxy:  k8s-tp-default-test-node-adv-ingress--618acb1b4e2fe823
      url-map:  k8s-um-default-test-node-adv-ingress--618acb1b4e2fe823
    No events.

you can check /gg1 and /gg2 is route to different node

***PLEASE NOTE THAT: it may take 5–8 minutes for the node to work with ingress, please be patient. and also you may need to set up firewall rules in gcloud to allow traffic, please refer to the google cloud doc.***

***NOTE 2: the ingress needed a root url (‘/’) send 200 ok to work***

### **Conclusion**

That’s all, you set up all the things, hope you can enjoy it :)

here are some references:
[**HTTP Load Balancing**
*Note that these instructions are currently quite manual. They will improve as Google Container Engine adds more…*cloud.google.com](https://cloud.google.com/container-engine/docs/tutorials/http-balancer)
[**Ingress Resources**
*Edit This Page Terminology Throughout this doc you will see a few terms that are sometimes used interchangeably…*kubernetes.io](http://kubernetes.io/docs/user-guide/ingress/)

Thank you XOXO
