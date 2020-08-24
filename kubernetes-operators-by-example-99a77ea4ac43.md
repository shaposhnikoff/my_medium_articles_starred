
# Kubernetes Operators by Example

A gentle introduction to Kubernetes Operators through a simplified example.

![](https://cdn-images-1.medium.com/max/2000/1*NLf18i4U3LQpWJ4Z5FUpqQ.jpeg)

**Operators**
> Operators are software extensions to Kubernetes that make use of custom resources to manage applications and their components. Operators follow Kubernetes principles, notably the control loop.
> The Operator pattern aims to capture the key aim of a human operator who is managing a service or set of services. Human operators who look after specific applications and services have deep knowledge of how the system ought to behave, how to deploy it, and how to react if there are problems.
> The most common way to deploy an Operator is to add the Custom Resource Definition and its associated Controller to your cluster. The Controller will normally run outside of the control plane, much as you would run any containerized application. For example, you can run the controller in your cluster as a Deployment.

*— Kubernetes — [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)*

For additional information, including motivation, read the sentinel article on Operators;* [Introducing Operators: Putting Operational Knowledge into Software](https://coreos.com/blog/introducing-operators.html)*.

**Creating Operators**

Creating Custom Resource Definitions (CRDs) is fairly straightforward; described in [*Extend the Kubernetes API with CustomResourceDefinitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)*. However, creating the controller is a non-trivial task; see the Kubernetes’ reference [example](https://github.com/kubernetes/sample-controller).

The good news, however, is that there are a number of frameworks for creating Operators that simplify the task, e.g.,

* [KUDO](https://kudo.dev/)

* [kubebuilder](https://book.kubebuilder.io/)

* [Metacontroller](https://metacontroller.app/)

* [Operator SDK](https://sdk.operatorframework.io/) (aka. Operator Framework)

In this article, we will be using Operator SDK; it has the most GitHub stars and supports a variety of languages.

**Operator SDK**
> This project is a component of the Operator Framework, an open source toolkit to manage Kubernetes native applications, called Operators, in an effective, automated, and scalable way.
> The Operator SDK provides the tools to build, test, and package Operators. Initially, the SDK facilitates the marriage of an application’s business logic (for example, how to scale, upgrade, or backup) with the Kubernetes API to execute those operations.

*— Operator SDK — [Operator SDK](https://sdk.operatorframework.io/)*

The OpenShift documentation has an excellent way to categorize Operators in relation to Operator SDK’s supported languages.
> The level of sophistication of the management logic encapsulated within an Operator can vary. This logic is also in general highly dependent on the type of the service represented by the Operator.
> One can however generalize the scale of the maturity of an Operator’s encapsulated operations for certain set of capabilities that most Operators can include. To this end, the following Operator Maturity model defines five phases of maturity for generic day two operations of an Operator:

![](https://cdn-images-1.medium.com/max/4880/1*GiyPdAjCyOxAle5i0b0j_Q.png)

*— OpenShift — [Understanding Operators](https://docs.openshift.com/container-platform/4.2/operators/olm-what-operators-are.html)*

In this article, we will use the simplest supported language, Helm, to create an Operator.

**Helm**
> Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application.

*— Helm — [Helm](https://helm.sh/)*

What makes Helm particularly easy to use is that one simply creates Kubernetes object configuration files enhanced with template directives.

**Prerequisites**

If you are interested in following along, you will need to have:

* Administrator access to a Kubernetes cluster

* [docker](https://www.docker.com/) installed

* [Docker Hub](https://hub.docker.com/) account

* [Operator SDK](https://sdk.operatorframework.io/) installed

* (Optional) [Helm](https://helm.sh/) installed

**The Hello Operator**

In this article, we will create a simple Operator that defines a CRD with:

* *example.com* as the Kubernetes [API group](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-groups)

* *v1alpha1* as the [API version](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-versioning)

* *Hello* as the [API resource kind](https://kubernetes.io/docs/reference/using-api/api-concepts/#standard-api-terminology)

* The *spec* accepts a single optional string property; *favoriteDrink; *defaults to *coffee*

An example Custom Resource (CR) configuration file:

<iframe src="https://medium.com/media/335e127bb39395ff04bf96b7a8fa742e" frameborder=0></iframe>

When such a CR is created, the Operator’s controller will create a ConfigMap with the same *name* as the CR and with key *drink* and value populated from the CR’s *favoriteDrink* property, i.e.:

<iframe src="https://medium.com/media/c891196a2562bab09a191959e4c2354e" frameborder=0></iframe>

**note**: Yes, this example is simple to the point of being frivolous; but the goal here is to learn the basic concepts so that one can build more meaningful Operators.

The Operator will be namespace-scoped; to the *default* Namespace:
> A namespace-scoped operator watches and manages resources in a single namespace, whereas a cluster-scoped operator watches and manages resources cluster-wide. Namespace-scoped operators are preferred because of their flexibility. They enable decoupled upgrades, namespace isolation for failures and monitoring, and differing API definitions.

*— Operator SDK — [Operators and CRD scope with Operator SDK](https://sdk.operatorframework.io/docs/operator-scope/)*

**Scaffold the Project Folder**

We begin by scaffolding the project folder, *hello-operator*:

    **$ operator-sdk new hello-operator \
      --api-version=example.com/v1alpha1 \
      --kind=Hello \
      --type=helm**

The scaffold implements a sample Operator; a comparatively complex one utilizing several kinds of resources. Much of our effort will be to modify various elements of the project to deliver our simple *Hello* Operator.

**The Custom Resource Definition (CRD)**

We first update the [OpenAPI schema](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#schemaObject) in the CRD configuration file, *deploy/crds/example.com_hellos_crd.yaml, *for our optional *favoriteDrink* property:

<iframe src="https://medium.com/media/48898793c56c73ef28a1e77d63690569" frameborder=0></iframe>

**note**: Kubernetes will validate associated CRs against the OpenAPI schema, e.g., it will reject CRs with a spec containing anything other than *favoriteDrink*.

We then update the example CR configuration file; *deploy/crds/example.com_v1alpha1_hello_cr.yaml *to adhere to the CRD specification.

<iframe src="https://medium.com/media/53579b2435088b5d8c0ba1fffec1e2c0" frameborder=0></iframe>

We can now create the CRD:

    **$ kubectl apply -f deploy/crds/example.com_hellos_crd.yaml
    **customresourcedefinition.apiextensions.k8s.io/hellos.example.com created

and validate the creation:

    **$ kubectl api-resources | grep hello**
    hellos                                         example.com                    true         Hello

We can create the CR:

    **$ kubectl apply -f deploy/crds/example.com_v1alpha1_hello_cr.yaml**
    hello.example.com/example-hello created

and validate the creation:

    **$ kubectl describe hello example-hello**
    Name:         example-hello
    Namespace:    default
    Labels:       <none>
    Annotations:  API Version:  example.com/v1alpha1
    Kind:         Hello
    Metadata:
     ...
    Spec:
      Favorite Drink:  tea
    Events:            <none>

Before we continue, let us delete the CR:

    **$ kubectl delete -f deploy/crds/example.com_v1alpha1_hello_cr.yaml
    **hello.example.com "example-hello" deleted

**The Controller Code**

First, a bit of housecleaning; we update the *appVersion* in *helm-charts/hello/Chart.yaml *to a sensible value, e.g., *0.1.0*.

We set the default value for the *favoriteDrinks* property by replacing the content in *helm-charts/hello/values.yaml* with *favoriteDrink: coffee.*

Next we delete all the files in the *helm-charts/hello/templates* folder and replace them with *configmap.yaml*:

<iframe src="https://medium.com/media/2af4e65f0a5c67629c6eb74f8f3f357f" frameborder=0></iframe>

Observations:

* With the Operator SDK, the Chart release name is the triggering CR name

* Likewise, the Chart values are the CR *spec* properties

During development, we can run the controller locally (but still operates on the Cluster):

    operator-sdk run local

We can now, as before, create the CR:

    **$ kubectl apply -f deploy/crds/example.com_v1alpha1_hello_cr.yaml **
    hello.example.com/example-hello created

This time, due to the creation of the CR, the controller creates the expected ConfigMap:

    **kubectl describe configmap example-hello**
    Name:         example-hello
    Namespace:    default
    Labels:       app.kubernetes.io/managed-by=Helm
    Annotations:  meta.helm.sh/release-name: example-hello
                  meta.helm.sh/release-namespace: default

    Data
    ====
    drink:
    ----
    tea
    Events:  <none>

**note**: If you have Helm installed, you can observe the Helm release.

    **$ helm list**
    NAME          NAMESPACE REVISION UPDATED                                 STATUS   CHART       APP VERSION
    example-hello default   1        2020-07-07 06:21:45.798088403 -0400 EDT deployed hello-0.1.0 1.16.0

As expected, deleting the CR will also delete the ConfigMap, e.g.,

    **$ kubectl delete -f deploy/crds/example.com_v1alpha1_hello_cr.yaml
    **hello.example.com "example-hello" deleted

and

    **$ kubectl get configmap**
    No resources found in default namespace.

**Controller Deployment**

In your Docker Hub account, create a public repository; in my case I named it *hello-operator*; thus available as *sckmkny/hello-operator.*

We next build and tag the Docker image:

    **$ operator-sdk build sckmkny/hello-operator:0.1.0**

We push it to Docker Hub:

    **$ docker push sckmkny/hello-operator:0.1.0**

We next update the *deploy/operator.yaml *file; replacing *REPLACE_IMAGE* with the Docker image, e.g., *sckmkny/hello-operator:0.1.0*.

**note**: During the project scaffolding, we got a warning that we should properly configure *deploy/role_binding.yaml* to only allow the controller the minimal level of access; in our case only being able to operate on ConfigMaps. But, when I went to limit the controller, I got errors that the controller needed read access to Pods. Thinking determining the minimal level of access would take a fair amount of trial and error (so did not end up doing).

We can now create the controller (and supporting resources):

    **$ kubectl apply -f deploy**
    deployment.apps/hello-operator created
    role.rbac.authorization.k8s.io/hello-operator created
    rolebinding.rbac.authorization.k8s.io/hello-operator created
    serviceaccount/hello-operator created

We confirm that the controller Pod is running:

    **$ kubectl get all**
    NAME                                  READY   STATUS    RESTARTS   AGE
    pod/hello-operator-66587f9bc9-95chm   1/1     Running   0          54m

    NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
    service/hello-operator-metrics   ClusterIP   10.105.83.191   <none>        8383/TCP,8686/TCP   54m
    service/kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP             37d

    NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/hello-operator   1/1     1            1           54m

    NAME                                        DESIRED   CURRENT   READY   AGE
    replicaset.apps/hello-operator-66587f9bc9   1         1         1       54m

With this in place we can create our CR:

    **$ kubectl apply -f deploy/crds/example.com_v1alpha1_hello_cr.yaml **
    hello.example.com/example-hello created

and verify the controller is indeed working by observing the created ConfigMap as we did earlier.

    **$ kubectl describe configmap example-hello**
    Name:         example-hello
    Namespace:    default
    Labels:       app.kubernetes.io/managed-by=Helm
    Annotations:  meta.helm.sh/release-name: example-hello
                  meta.helm.sh/release-namespace: default

    Data
    ====
    drink:
    ----
    tea
    Events:  <none>

**Wrap Up**

That is about it, a simple Operator using Helm. Of course, one can create more complex Operators using Operator SDK with either the Ansible or Go languages.
