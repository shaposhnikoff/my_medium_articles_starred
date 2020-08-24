Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m83[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }

# Deploy nginx-ingress and retain full control of your AWS Load Balancer

Kubernetes Ingress resources allow you to define how to route traffic to pods in your cluster, via an ingress controller. One of the more common ingress controllers is the NGINX Ingress Controller, maintained by the Kubernetes project. When using an ingress controller, one of the first questions you have to address is how will traffic reach the controller.

If you are in a bare-metal environment, you’ll typically wire up your own load balancer to an exposed NodePort and there is some documentation on that [here](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/).

If you are in a cloud environment there are some differences because, for instance, you can’t generally share an IP address and you cannot use layer 2 capabilities either. You are expected to use a layer 4 (TCP/UDP) or layer 7 (HTTP/HTTPS) load balancer, this can be your own dedicated instance or cluster, or it can be a cloud-provider load balancer.

Taking AWS as an example, you have three options:

1. Classic Elastic Load Balancer (Layer 4 or Layer 7), **ELB**

1. Application Load Balancer (Layer 7), **ALB**

1. Network Load Balancer (Layer 4), **NLB**

The ELB provides support for a single target, which means you can’t reuse it for other purposes in your environment. It can also only support a single certificate for SSL termination. In contrast, an ALB supports condition-based routing, higher throughput, authentication integration (such as with AWS Cognito) and Lambda targets. It also has more extensive logging capabilities and support for AWS Web Application Firewall (WAF) access control lists (ACLs).

The NLB is an altogether different beast. It is generally transparent to the client (which means you see the originating IP address, subject to some constraints) and is designed for much higher throughput.

When you want to take use of some of the extra capabilities of an ALB, using the NGINX Ingress Controller to manage the load balancer just isn’t going to cut it.

![](https://cdn-images-1.medium.com/max/2182/1*PoKwin1fOT_oDzlbDsdjRA.png)

The diagram above shows an outline of how you might want to deploy your ingress capability. CloudFront or another content distribution network to provide fast local access to some resources and further limit exposure of your VPC, then an ALB in a public subnet with your Kubernetes cluster fully isolated in a private subnet. AWS WAF integrated to provide some healthy protections, such as blocking access to debug information (such as the [Java Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html)) that the developers didn’t remember to switch off, or perhaps that you need but only from internal network addresses or within the cluster. Perhaps you also want Amazon Cognito to be applied to some websites to enforce a SAML-based login, such as via Azure Active Directory, Okta, OneLogin or similar.

You might also deploy a second ALB for internal access with a different set of rules but still targeting the same ingress controller or a second one.

For the rest of this discussion, I’m going to focus on a single ALB design and ignore WAF and Cognito. However, the ALB will not be managed by nginx-ingress and can be configured as needed. I recommend [Terraform](https://terraform.io) for this.

I’m basing this discussion on release [0.24.1](https://github.com/kubernetes/ingress-nginx/releases/tag/nginx-0.24.1), but you should deploy a more recent version to benefit from some vulnerability fixes that have since been implemented.

The [mandatory.yaml](https://github.com/kubernetes/ingress-nginx/blob/nginx-0.24.1/deploy/mandatory.yaml) file contains the resources needed to deploy the controller:

* 1x namespace (ingress-nginx)

* 3x config maps (http, tcp and udp configurations)

* 1x service account

* 1x cluster role (to monitor services and endpoints and update ingress resources)

* 1x role (to manage its own configuration data in the ingress-nginx namespace)

* 1x cluster role binding

* 1x role binding

* 1x deployment (the ingress controller itself)

Note that this doesn’t deploy any inbound access method. Usually, this would be added following the instructions [here](https://kubernetes.github.io/ingress-nginx/deploy/#aws). These would normally construct the ELB or ALB as needed, but since we don’t want Kubernetes to manage the load balancer, I’ll create a service that exposes ingress-nginx on a NodePort:

    apiVersion: v1
    kind: Service
    metadata:
      name: ingress-nginx
      namespace: ingress-nginx
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
    spec:
      type: NodePort
      selector:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      ports:
      - name: http
        nodePort: 32323 # You can choose your own port!
        port: 80
        targetPort: http
        protocol: TCP

Personally, I prefer to offload SSL termination to the ALB and ensure security of my network internally. If you want to have a second TLS hop to nginx-ingress, you can of course do that too.

This alone will give you a working ingress, however as the Service isn’t responsible for a load balancer, your ingress objects won’t advertise an address. They are waiting for a load balancer to be defined. We can fix this.

Assigning a load balancer address can be achieved by pointing the --publish-service argument to an ExternalName service. First we need to create such a service:

    apiVersion: v1
    kind: Service
    metadata:
      name: ingress-nginx-external
      namespace: ingress-nginx
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
    spec:
      type: ExternalName
      externalName: <<EXTERNAL_NAME>>

<<EXTERNAL_NAME>> can be any address you want. I typically use the ALB FQDN, but it can be anything. It just ties up an annoyance in the Kubernetes state but I don’t think it makes any meaningful behavioural difference.

To apply this, you also need to patch the ingress-nginx Deployment. I use this script:

    JQ_CMD="$(cat - <<EOF
    .spec.template.spec.containers |= (
      map(
        if .name=="nginx-ingress-controller" then
          .args |= map(
            if (. | startswith("--publish-service")) then
              "--publish-service=\$(POD_NAMESPACE)/ingress-nginx-external"
            else
              .
            end
          )
        else
          .
        end
      )
    )
    EOF
    )"

    kubectl get deployment --namespace ingress-nginx nginx-ingress-controller -o json \
    | jq "$JQ_CMD" -Mc \
    | kubectl apply -f -

A final note. The default deployment does not create a PodDisruptionBudget, any podAntiAffinity and runs with a single pod replica. These are all things that you should change in a production estate, to ensure that you don’t have outages due to evictions, failures or network segmentation.

This post is also published on my blog at [https://stevehorsfield.wordpress.com/2019/10/02/deploy-nginx-ingress-and-retain-full-control-of-your-aws-load-balancer/](https://stevehorsfield.wordpress.com/2019/10/02/deploy-nginx-ingress-and-retain-full-control-of-your-aws-load-balancer/).
