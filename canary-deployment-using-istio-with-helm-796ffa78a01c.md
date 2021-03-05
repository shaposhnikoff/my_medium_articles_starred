
# Canary Deployment using Istio with Helm

Open source platform Istio by no doubt eases the microservice communication, management and security. In this post we are going to deal with the management part and especially canary deployment which seems a small scope but one of the very important step in effective and efficient handling of microservices which impacts end user experience. Also, we will dive further into streamlining istio service mesh traffic with Helm, a package manager for Kubernetes.

I have **Istio version 1.0.2** installed while writing this post but all of the configs should work with version 1 sub releases.

![Helm + Istio (src: qiita.com)](https://cdn-images-1.medium.com/max/2272/1*BuRrFf9pK6IQbep1ECQrGQ.png)*Helm + Istio (src: qiita.com)*

### **So, what is canary deployment?**
> Canary release is a technique to reduce the risk of introducing a new software version in production by slowly rolling out the change to a small subset of users before rolling it out to the entire infrastructure and making it available to everybody. — Danilo Sato on martinfowler.com

We can use canary deployment mainly with two ways. One is gradually release new version to group of users so that we can figure out errors, get feedback & be confidence in new version or rollback in case. The rollout could be to users of certain target group(pro, free users etc.), demographic region or random sampling.

Next use case could be to test the new version of service without affecting or exposing to general users but use it for QA, bug testing.

**Let’s Dive…**

If you are kind of guy who prefers “Just show me code”, here it is on [github](https://github.com/dwdraju/helmis).

The Istio installation instruction, ssl configuration, creating TLS certificates are on [this README](https://github.com/dwdraju/helmis/blob/master/README.md) as we are going to focus more on our primary topic of canary deployment.

Let’s first start with creating new helm chart

$ helm create explore

The initial file structure will be like this:

![](https://cdn-images-1.medium.com/max/2000/1*S73zm389pIbFYpo6CL-TSg.png)

We are going to give label for our stable and canary release as track: stable and track: canary . Also add new deployment file for canary but this time we append -canary to our deployment name which goes like [this commit](https://github.com/dwdraju/helmis/commit/4033bd15fa3e20eecaa6fb5e79c0b38f9aca85d1).

![deployment-canary.yml](https://cdn-images-1.medium.com/max/2648/1*paBIRSS2bGA-ftvkglUuAQ.png)*deployment-canary.yml*

Now, we need separate image tag for stable and canary release for which we organize values.yaml file like this:

![](https://cdn-images-1.medium.com/max/2000/1*SSYW1MdhL8e3u2JEDLdXlg.png)

We can add more values as per need but for simplicity we are focusing only on image tag.

You can change the image repository and tag in which place I am using image repo dwdraju/go-istio and tags v1 for stable release and v2 for canary release. Also, tuning container port 80 to 8080.

### Time for Istio

We have Istio Gateway config which uses ingressgateway and SSL certificate we created earlier of name istio-ingressgateway-certs . The gateway file is on [istio/gateway.yml](https://github.com/dwdraju/helmis/blob/master/istio/gateway.yml).

Now, we need Isito VirtualService and DestinationRule which we keep in helm/templates/istio.yml file that goes like this:

![Istio VirtualService](https://cdn-images-1.medium.com/max/2492/1*MvDp7iNdOoTAa1O2QD0DQQ.png)*Istio VirtualService*

![Istio DestinationRule](https://cdn-images-1.medium.com/max/2184/1*xHMT4kNwgFUbfu9V8S3AEQ.png)*Istio DestinationRule*

We add values for {{ .Values.hostName }} ,{{ .Values.ReleaseWeight }} and {{ .Values.CanaryWeight }} on top ofvalues.yaml file.

Apply the helm chart

$ helm install --name canary-explore .

Now, we can easily change the weight to canary and stable release as well as image tag from values.yaml file and let the remaining templates intact.

Likewise, we can add separate files of HPA, ConfigMap, Secret with canary suffix and adjust them on deployment-canary.yaml file accordingly.

Cheers Canarying with Helm and Istio !!!
