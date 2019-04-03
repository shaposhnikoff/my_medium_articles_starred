
# Testing Kubernetes RBAC

“blue locked door” by Chris Barbalis on Unsplash

Securing your [Kubernetes](https://www.yld.io/speciality/kubernetes/) cluster is one thing, keeping it secure is a continuous uphill struggle. However, with the introduction of new features to Kubernetes it is becoming much easier to do both.

Kubernetes (as of version 1.6) has introduced the concept of **R**ole-**B**ased **A**ccess **C**ontrol (**RBAC**), allows administrators to define policies to restrict the actions of users of your cluster. This means it is possible to create a user with limited access, allowing you to restrict access to resources such as Secrets, or by limiting access of that user to a specific Namespace.

This blog post will not look at how to implement RBAC, as there are many decent sources of information that cover it in vast detail:

* [https://medium.com/containerum/configuring-permissions-in-kubernetes-with-rbac-a456a9717d5d](https://medium.com/containerum/configuring-permissions-in-kubernetes-with-rbac-a456a9717d5d)

* [https://www.cncf.io/blog/2018/08/01/demystifying-rbac-in-kubernetes/](https://www.cncf.io/blog/2018/08/01/demystifying-rbac-in-kubernetes/)

* [https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/](https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/)

* [https://kubernetes.io/docs/reference/access-authn-authz/rbac/](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

Instead, this post will focus on how to ensure your business’s compliance and requirements are actually being adhered to and to ensure that we need to test our applied RBAC objects, to ensure they do what we intend them to do.

### Our Scenario

An Organisation has just started on-boarding multiple teams onto their new Kubernetes cluster, a requirement for these teams to be on-boarded onto this cluster was that each team wouldn’t be allowed to modify another teams deployment causing unforeseen cross-team issues or downtime.

The Platform team who owns the Kubernetes deployment has recently deployed RBAC to the cluster, restricting access for team members to a specific namespace, on first inspection teams cannot view pods in any of the other team’s namespace.

Fast forwarding one week, the Platform team starts to notice that a user in a restricted namespace has been reading secrets from other namespaces. But How? Didn’t they Implement RBAC?

Well, they did, but as with code, we need to test our system’s against the desired outcome. Thankfully Kubernetes’ CLI tool kubectl provides us with tooling that allows us to test our RBAC configuration. kubectl auth can-i

## Can I?

can-i simply checks that with the API to see if an action can be performed. It can take the following options kubectl auth can-i VERB [TYPE | TYPE/NAME | NONRESOURCEURL] . And now it is possible for the current user to check whether to not they can perform an action. Let's give this a go:

    kubectl auth can-i create pods

This should return a “yes” or a “no” with a corresponding exit code.

But as soon as we try to test the authorisation for another user, we hit a stumbling block, with the command above we can only test using the currently loaded ./kube/config , it is quite unreasonable to have a file per user type! But thankfully Kubernetes once again comes to the rescue with the ability to impersonate users with the --as= and the --as-group= flags.

Let’s update our command to make use of impersonating a different user:

    kubectl auth can-i create pods --as=me

We should see that we now get a “no” returned with an exit code of 1 being returned.

This is great news, we now have a set of commands that allow us to test if a specific set of users or an individual user has access to any number of Kubernetes resources from listing pods to deleting secrets.

## Automation

We shouldn’t stop here though, we have now paved the way to implement a test suite that can describe the list of the requirements and have the ability to run this as part of our Continuous Delivery pipeline, so let us!

We have so many choices and languages to choose to implement this, from Ava and Mocha in JavaScript through to Rspec. In this case, I am going to be using a pure bash implementation of a testing framework named [Bats](https://github.com/bats-core/bats-core).

Below is an example testing that a user in a team namespace can scale deployments in their own namespace. It can be executed like any shell script if the executable bit has been set, or by using bats filename .

    #!/usr/bin/env bats

    @test "Team namespaces can scale deployments within their own namespace" {
        run kubectl auth can-i update deployments.apps --subresource="scale" --as-group="$group" --as="$user" -n $ns 
        [ "$status" -eq 0 ]
        [ "$output" == "yes" ]
      done
    }

Sidebar; the --as and --as-group commands require the following RBAC rules to be used:

    rules:
    - apiGroups:
      - authorization.k8s.io
      resources:
      - selfsubjectaccessreviews
      - selfsubjectrulesreviews
      verbs:
      - create

And like that, you have an easy method of implementing testing for your RBAC rules in Kubernetes. With this as a part of your Kubernetes’ Continuous Delivery Pipeline we should now have a much more robust RBAC policy enforcement that has been verified to be working. Allowing changes that break policies to be caught much earlier!

Thanks for taking the time for reading!

Written by [Tom Gallacher](https://twitter.com/tomgco) — Staff Engineer at [YLD.](https://www.yld.io)

### Interested in Kubernetes? Read more about it:
[**Kubernetes: Core Concepts**
*It doesn't need to be hard, learn the core concepts of Kubernetes.*medium.com](https://medium.com/yld-engineering-blog/kubernetes-core-concepts-324ea7028c29)
[**Kubernetes: Authentication**
*Providing access to Kubernetes with multiple teams of developers, handling new joiners or people doesn’t have to be…*medium.com](https://medium.com/yld-engineering-blog/kubernetes-auth-380e57d19da0)
