Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m30[39m }

# Ingress Controller Kubernetes

Image by Arek Socha from Pixabay

## **What’s Ingress ?**

Services are exposed as HTTP and HTTPS to be accessed from outside the cluster. Ingress defines a collection of rules to allow inbound traffic to access services within the cluster.

Ingress is configured to expose services through URL which is externally reachable , Loadbalancers, name based virtual routing & SSL/TLS Terminated endpoints

## **Ingress on Minikube**

In this tutorial , we will go through the Ingress setup on minikube using two simple hello whale web applications and route traffic based on uri that’s requested.

    hello.whale.info/green/ -->Green Whale Pod Exposed on Port 31114
    hello.whale.info/blue/ --> Blue Whale Pod exposed on Port 31113

![Ingress -Minikube Cluster](https://cdn-images-1.medium.com/max/2000/1*BoooARt8vFUatULILAkt3A.png)*Ingress -Minikube Cluster*

By default ingress controller is not automatically started within the minikube cluster. It has to be enabled through an add-on.

**Step-1 : Enable Ingress Controller**

Enable the ingress add-on using the below command

    minikube addons enable ingress

Verify the Ingress controller is running in the cluster

![Ingress Controller pod](https://cdn-images-1.medium.com/max/2336/1*MmMH0JtQp0fLE53RYP_Cfw.png)*Ingress Controller pod*

**Step-2: Deploy two web applications and expose the services through NodePort.**

    kubectl create -f hello-green-whale-deployment.yml
    kubectl create -f hello-blue-whale-deployment.yml

![Deployment Manifest](https://cdn-images-1.medium.com/max/2000/1*ICwiFAtupK0ujTmylybAJA.jpeg)*Deployment Manifest*

    kubectl create -f hello-green-whale-svc.yml
    kubectl create -f hello-blue-whale-svc.yml

![Service Manifest](https://cdn-images-1.medium.com/max/2000/1*KLdAgDktLIuAsGduuVTr4Q.jpeg)*Service Manifest*

![2 Pods exposed via NodePort](https://cdn-images-1.medium.com/max/2000/1*Wg0ZvE2ZJs0ILLdy-vjhtw.png)*2 Pods exposed via NodePort*

Access Services on 31114 and 31113 .

![](https://cdn-images-1.medium.com/max/2382/1*jIsdQexu_pnNIHGUNSQCIg.jpeg)

**Step-3 — Deploy the Ingress**

    kubectl create -f hello-whale-ingress.yml

![Ingress Manifest](https://cdn-images-1.medium.com/max/2000/1*6TOrdROz6tIVTZcO55DmxA.png)*Ingress Manifest*

![](https://cdn-images-1.medium.com/max/2000/1*M5b_WN1EygwHadCiZT1QTg.png)

![Ingress Description](https://cdn-images-1.medium.com/max/2262/1*wiNzR2owCqPhbVj0aCB3aw.png)*Ingress Description*

Update /etc/hosts file with the entry with minikube ip

    *192.168.99.104 hello.whale.info *

**Step-4 Route to Green and Blue will display respective web pages using simple fanout.**

    hello.whale.info/green --> SVC Port --> 31114
    hello.whale.info/blue  --> SVC Port --> 31113

![](https://cdn-images-1.medium.com/max/2458/1*MzXot8eD6dB9eHbT-31Bgw.jpeg)

**Source GitHub:[https://github.com/vamsijakkula/Ingress](https://github.com/vamsijakkula/Ingress)**

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
