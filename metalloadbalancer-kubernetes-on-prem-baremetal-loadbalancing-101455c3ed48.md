
# MetalLoadBalancer: Kubernetes On-Prem /BareMetal LoadBalancing

If you are looking to ramp up on Kubernetes before diving into load balancing, please check out some of these guides, articles and tutorials to get you started… https://www.lynda.com/Kubernetes-tutorials/Kubernetes-Microservices/691061-2.html, https://github.com/kelseyhightower/kubernetes-the-hard-way, https://medium.com/@JockDaRock/minikube-on-windows-10-with-hyper-v-6ef0f4dc158c

If you have ever deployed applications to Kubernetes (k8s) on Google Cloud, AWS, or Azure, you notice how quickly you can deploy your app and have it reachable, externally (public IP), from the k8s cluster within a matter of a couple minutes. You merely have to set the service type of your application to LoadBalancer on your service manifest and k8s contacts an Elastic / Cloud Load Balancer and procures an IP address for your app.

Now, if you are looking to see the same functionality for your on-prem k8s clusters, you are typically disappointed. K8s uses glue code to automagically work with Cloud Providers’ LoadBalancers. This means if you assign a LoadBalancer type to your services for bare metal or on-prem deployments, you will literally be waiting forever for your external IP address.

The options for the service types you end up with for on-prem k8s deployments are NodePort or externalIPs. You can read more about service types here [https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types).

NodePort is workable, but you are confined to using the IP address of your k8s nodes, and defaults to ports 30000–32767 for access to outside of the k8s cluster. You can use externalIPs but the IP address already has to be pre-configured to route directly to one of your cluster nodes.

Both NodePort and externalIP addresses are limiting in different ways and are really not an ideal solution for on-prem or internal k8s deployments. Not to mention, if you would like to test applications with LoadBalancing without having to use up expensive IP addresses on your cloud provider, you are kind of stuck.

**Enter Metal LB**

Metal LB ([https://metallb.universe.tf/](https://metallb.universe.tf/), [https://github.com/google/metallb](https://github.com/google/metallb)) is a load balancer for your bare metal / on-prem deployments. It uses standard networking appliances (real or virtual) to distribute IP addresses that are external to your cluster and assigns those IP addresses to your app services that specify the LoadBalancer type. The implementation is easy to apply on your k8s cluster and you could be up and running within 5 minutes.

Ok, great, thanks for telling me about this, but how do I use it? Good question, I have a put together a couple of tutorials to get you started depending on your setup of Kubernetes.

For your Dev Environment on DockerCE: [https://medium.com/@JockDaRock/kubernetes-metal-lb-for-docker-for-mac-windows-in-10-minutes-23e22f54d1c8](https://medium.com/@JockDaRock/kubernetes-metal-lb-for-docker-for-mac-windows-in-10-minutes-23e22f54d1c8)

For On-Prem Kubernetes Deployment:

[https://medium.com/@JockDaRock/kubernetes-metal-lb-for-on-prem-baremetal-cluster-in-10-minutes-c2eaeb3fe813](https://medium.com/@JockDaRock/kubernetes-metal-lb-for-on-prem-baremetal-cluster-in-10-minutes-c2eaeb3fe813)
