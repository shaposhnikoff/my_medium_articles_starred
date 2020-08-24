Unknown markup type 10 { type: [33m10[39m, start: [33m223[39m, end: [33m237[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m119[39m, end: [33m123[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m21[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m90[39m, end: [33m102[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m141[39m, end: [33m155[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m102[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m174[39m, end: [33m185[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m92[39m, end: [33m104[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m179[39m, end: [33m192[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m184[39m, end: [33m193[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m198[39m, end: [33m204[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m66[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m154[39m, end: [33m166[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m209[39m, end: [33m221[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m243[39m, end: [33m255[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m55[39m, end: [33m63[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m40[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m94[39m, end: [33m102[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m147[39m, end: [33m155[39m }

# Supercharging Helm

Levelling up your Helm practices

![](https://cdn-images-1.medium.com/max/3840/1*Jqa33nXxSSoYmzqjzUsKZg.jpeg)

If you’re taking Kubernetes seriously, I’ll bet you’re using Helm. It’s a no-brainer. Consistency, additional functionality and a great CLI, all wrapped up in one very useful package. Out of the box, Helm is very functionality focused and you’ll see the benefit almost immediately.

There are, however, some catches. As with many Kubernetes tools, Helm is insecure by default. It will install its server side component, Tiller, with almost unlimited power. This presents quite a serious attack vector for a malicious actor.

In this article, I’ll begin by discussing methods for mitigating the risk that Tiller poses to your cluster. Secondly, I’ll be offering some tips and lesser known features that helm offers. Let’s get started.

## Tiller

![Tiller has God tier powers out of the box](https://cdn-images-1.medium.com/max/2000/0*85RRq9Gtr9uDLjGV.jpg)*Tiller has God tier powers out of the box*

Tiller is *the *problem with Helm. If it breaks, it’s compromised or it gets deleted, you’re in trouble. That’s the reason it has been [removed](https://helm.sh/blog/helm-3-preview-pt2/) in Helm 3. While we’re stuck with Helm 2, however, we aren’t completely helpless.

### Namespace Specific Tiller

Our first mitigation is to install a single Tiller for a namespace. This greatly limits the blast radius of a breach. How do you do this? Well first, Tiller is going to need a service account. If you’re not familiar with a ServiceAccount resource in Kubernetes, you can read about them [here](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/).

### **Configuring RBAC**

Tiller will need a lot of permissions within your namespace. To add permissions to a service account, you first need a Role.

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:  
      name: tiller-role 
      namespace: my-namespace
          rules:
          - apiGroups: ["", "extensions", "apps"]  
            resources: ["*"]  
            verbs: ["*"]

As you can see, this Role is permitted to do whatever it likes, but *only *in the namespace my-namespace*. *So now we have our role, we need our ServiceAccount. This one is simple.

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tiller-service-account
      namespace: my-namespace

Finally, you need a little glue. At the moment your Role and your ServiceAccount are disconnected. You need something to couple the two together. This comes in the form of a RoleBinding.

    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: tiller-role-binding
      namespace: my-namespace
    subjects:
    - kind: ServiceAccount  
      name: tiller-service-account
      namespace: my-namespace
      roleRef:  
        kind: Role  
        name: tiller-role 
        apiGroup: rbac.authorization.k8s.io

So we’ve created a service account that can do whatever it likes, but only in the namespace my-namespace. How do we tell helm to use this? Well, the CLI has got you covered.

    helm init --service-account tiller-service-account --tiller-namespace my-namespace

And that’s it! Your tiller will work beautifully for deployments into that namespace. If it is compromised, it can wreak havoc on your namespace but it isn’t a cluster wide breach (assuming RBAC security has no vulnerabilities…).

### No Tiller

Kelsey Hightower [phrased it](https://github.com/kelseyhightower/nocode) beautifully.

    No code is the best way to write secure and reliable applications. Write nothing; deploy nowhere.

Just don’t deploy tiller. This is easy to do and requires the least possible configuration to get going.

    helm init --client-only

However, in Helm 2, this means you aren’t going to be able to use some of the more awesome features that we cover later on in this article. You’ll need to install your yaml using helm template, but you still get much of the [DRY](https://dzone.com/articles/software-design-principles-dry-and-kiss) benefits that Helm offers.

### Wait for Helm 3

Lots of people have opted to avoid Helm until Helm 3 is released. As I mentioned earlier, Helm 3 ships without any tiller at all and simply interfaces directly with k8s resources like ConfigMap and Secret to keep track of your releases.

Helm 3 is greatly simplified, but [maintains much](https://medium.com/better-programming/helm-3-fun-with-the-new-beta-8f91c70891ff) of the existing functionality that you might be used to in Helm 2. The beta has recently been released, so the production release is right around the corner. If you can wait, it might be wise.

## Helm Tips

The Helm CLI comes packaged with a bunch of brilliant little features and gotchas. These are well documented, but I rarely catch them all in the same place. My aim in this section is to give you some good pointers to avoid the same pitfalls that I fell into.

### Favour helm upgrade over helm install

Typically, we might run something like this…

    helm install -f my-values.yaml ./path/to/my/chart --name something

This is a great way to **install **something into your Kubernetes cluster, but what happens if you try to run it again? An error, that’s what! That’s because helm install won’t run upgrades too. For that, we have helm upgrade. Alas, you can’t run helm upgrade when you don’t have anything installed! Or… can you?

    helm upgrade something ./path/to/my/chart -f my-values.yaml --install

When you run this command, if nothing exists, it will simply install it for you. This is incredibly useful for CI/CD pipelines, because you don’t need to keep manually installing the first time. Simply run and rerun, as many times as you like!

### Automatic rollbacks with atomic upgrades

![The sounds I make when applying helm commands.](https://cdn-images-1.medium.com/max/2254/0*k3fCQwP9kgsay5E2.jpg)*The sounds I make when applying helm commands.*

A few months back, helm added a new flag to their cli: --atomic. This flag will automatically run a helm rollback if your upgrade fails. This is great for a couple of reasons.

* If your rollout has failed, you wanna get back to 100% working as fast as possible.

* If you leave your deployment in a FAILED state, the behaviour from helm can be [unpredictable](https://github.com/helm/helm/issues/1193). --atomic will ensure your application is always in a DEPLOYED state.

So our command starts to look a little something like this:

    helm upgrade something ./path/to/my/chart -f my-values.yaml --install --atomic

### Helm Test

Tests in helm are very, very underutilised. This feature allows you to ensure that your application is running correctly. It can be anything, from a simple smoke test, to a comprehensive set of end to end tests.

Some feel it’s a mixing of concerns. Should our package manager also be our smoke tester? Well, the feature is there if you are a fan and it’s optional if you’re not keen. Using it is simple, and in the interests of not repeating myself, I’ll simply [provide a link](https://github.com/helm/helm/blob/master/docs/chart_tests.md) to the excellent helm documentation on github.

## Helm away!

Helm provides some great features and when used properly, it is a cornerstone of any successful kubernetes operation.

I regularly talk about Kubernetes, DevOps and good engineering practices on [twitter](https://twitter.com/chris_cooney).
