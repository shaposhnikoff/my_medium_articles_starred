
# Istio step-by-step Part 11 — Customize Istio Installation

Hey everyone, welcome back to my Istio step-by-step tutorial series. This time I will be going to introduce you to how to customize Istio installation.

![Photo credits: [https://www.pexels.com/photo/person-touching-open-macbook-on-table-839465/](https://www.pexels.com/photo/person-touching-open-macbook-on-table-839465/)](https://cdn-images-1.medium.com/max/10058/1*Uo04kfDLAkaNx_HAkPrqQA.jpeg)*Photo credits: [https://www.pexels.com/photo/person-touching-open-macbook-on-table-839465/](https://www.pexels.com/photo/person-touching-open-macbook-on-table-839465/)*

We need Istio latest version (1.4 or above) for this task.
> Refer to my previous article for installation Istio 1.4.

Istio new version allows us to customize the Istio control plane and the sidecar Envoy proxies in the data plane using istoclt command-line. It has user input validation to help prevent installation errors and customization options to override any aspect of the configuration. This tutorial will explain to you how to customize istio installation.

You can check for existing profiles by executing the following command.

    istioctl profile list

![](https://cdn-images-1.medium.com/max/2344/1*Um4pzGI8874vBDO5hHDcGQ.png)
> Refer to my [previous article](https://medium.com/@nethminiromina/istio-step-by-step-part-10-installing-istio-1-4-in-minikube-ebce9a4e99c) for information on these profiles.

You can set your configurations to any profile with the below command

    istioctl manifest apply — set profile=<profile_name>

To check the profile configuration you can use the command;

    istioctl profile dump <profile_name>
> All Istio profile configurations can be found in the istio-1.x.x/install/kubernetes/operator/profiles directory.

## Let’s create a custom Istio profile.

### Step 01

Create myProfile.yaml file.

### Step 02

You can add your configurations in this file. For testing, you can add the following configuration snippet to the file.

    apiVersion: install.istio.io/v1alpha2
    kind: IstioControlPlane
    spec:
      trafficManagement:
        enabled: false

      policy:
        enabled: false

      telemetry:
        enabled: false

      configManagement:
        enabled: false

      autoInjection:
        enabled: true

      gateways:
        enabled: true
        components:
          egressGateway:
            enabled: false
          ingressGateway:
            enabled: true

      values:
        pilot:
          configSource:
            subscribedResources:

        security:
          createMeshPolicy: false

        prometheus:
          enabled: false

### Step 03

Now apply this new profile configuration.

    istioctl manifest apply -f myProfile.yaml

![](https://cdn-images-1.medium.com/max/2892/1*_iaABJPIj6BNfntVujcgnw.png)

You can see the components we configured have been updated.

If you set the demo profile, you will receive and output as follow.

![](https://cdn-images-1.medium.com/max/2916/1*V376r5OUkwz4CL3GqZwnQg.png)

Likewise, you can add your own configurations and apply them.

You can verify the installation by generating a manifest file.

    istioctl manifest generate > $HOME/generated-manifest.yaml
    istioctl verify-install -f $HOME/generated-manifest.yaml

Now you have customized Istio. So, that's all for this tutorial. Follow up my next article [deploying-istio-bookinfo-application-in-minikube](https://medium.com/@nethminiromina/istio-step-by-step-part-12-deploying-istio-bookinfo-application-in-minikube-501c90cfeab4).

<< [previous article](https://medium.com/@nethminiromina/istio-step-by-step-part-10-installing-istio-1-4-in-minikube-ebce9a4e99c)

[next article](https://medium.com/@nethminiromina/istio-step-by-step-part-12-deploying-istio-bookinfo-application-in-minikube-501c90cfeab4) >>

### References

1. [https://istio.io/docs/setup/install/istioctl/](https://istio.io/docs/setup/install/istioctl/)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
