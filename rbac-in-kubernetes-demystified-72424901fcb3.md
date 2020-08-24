
# RBAC in Kubernetes: Demystified

Role Based Access Control(RBAC) is a very crucial concept in Kubernetes yet at times hard to understand. So here I am to demystify it in simple words. In this article, we will talk about

1. API Access Stages

1. Role of **RBAC**

1. Example

## **API access stages**

Before talking about **RBAC**, it’s important to understand the complete picture where a user or application wants access to Kubernetes objects and then we will talk about where does RBAC fit in these stages. Overall it’s a 3 stage process. We will briefly talk about authentication and admission control while our main focus would be Authorization where RBAC comes into the picture.

![](https://cdn-images-1.medium.com/max/2560/1*phQdmw2wG2qM9ZB-jVxrlA.png)

1. Authentication

1. Authorization(**RBAC**)

1. Admission Control

**Authentication**

In a Kubernetes cluster, we may want to have different users with different roles and privileges. **Ex**: consider cluster A will be used by the developer, tester and administrator(3 different users).

*Developer* — create, view, get permission.

*Tester* — create,delete,update,get permission.

*Administrator* — All permission.

In Kubernetes, we have APIs to create all Objects but we don’t have API to create a user. So Authentication is the first step that happens when you create a user or group to make sure the user or application is authenticated to access K8s objects.

### Authorization(Role of RBAC)

Authorization starts once the user or application has been authenticated. After the request is authenticated as coming from a specific user, the request must be authorized. There can be multiple authorization modules. But we will talk about RBAC.

Q. Does the user or application is authorised or have permissions to access the K8s objects?

First Lets talk about some jargons

***Subjects***: *Subject is the entity that needs access. It could be user or group or a service account(a process in a pod)*

***Resources**: Resource is the K8s object that a subject wants to access. It could be pods, deployments, services etc*

***Verbs**: Verbs are the actions that a subject can do on resources. It could be the list, delete, create, watch etc*

***Roles and RoleBindings***: We need to define some rules to perform *verbs* on a *resource.* Those rules are called **Roles** or **ClusterRoles**

Roles are for a specific namespace while ClusterRoles are for the entire cluster.

And for a particular subject to be able to access a resource according to a rule, we need to define bindings between the two, those are **RoleBinding** and **ClusterRoleBinding**.

**Rolebinding** binds the Role to a Subject to access the Resources within a namespace while **ClusterRoleBinding** binds the ClusterRole to a Subject to access the resources cluster-wide.

![](https://cdn-images-1.medium.com/max/2732/1*vlM_26kMbA443eA0g4tcew.jpeg)

## Example

**For a specific namespace:**

Let's say we have a User John who wants to be able to *list* the pods in a namespace *foo*. Then the steps we need to follow

1. Assuming we have a user *John* who is already authenticated. We will create a Role **listpods** for namespace **foo** and verb **list**

    **kind**: Role

    **apiVersion:rbac**.authorization.k8s.io/v1beta1

    **metadata**:

       **name**: listpods

       **namespace**: foo

    **rules**:

     - **apiGroups**: [""]

       **resources**: ["pods"]

       **verbs**: ["list"]

2. We will create a RoleBinding to bind the user **John** and Role **listpods**

    **kind**: RoleBinding

    **apiVersion**: rbac.authorization.k8s.io/v1beta1

    **metadata**:

       **name**: demobinding

       **namespace**: foo

    **subjects**:

     - **kind**: User

       **name**: John

       **apiGroup**: rbac.authorization.k8s.io

    **roleRef**:

       **kind**: Role

       **name**: listpods

       **apiGroup**: rbac.authorization.k8s.io

3. Once we have that in place, user John can list the pods in namespace “foo”

    kubectl list pods -n foo

**For across the cluster:**

Let’s say we have a User ***sa*** who wants to be able to *list and get *the pods across the cluster. Then the steps we need to follow

1. Assuming we have a user ***sa*** who is already authenticated. We will create a ClusterRole **democlusterrole** for verb **get** and **list**

    **kind**: ClusterRole

    **apiVersion:rbac**.authorization.k8s.io/v1beta1

    **metadata**:

       **name**: democlusterrole

    **rules**:

     - **apiGroups**: ["*"]
    **   
       resources**: ["pods"]

    **   verbs**: ["get", "list"]

2. We will create a ClusterRoleBinding to bind the user ***sa*** and ClusterRole democlusterrole

    **kind**: ClusterRoleBinding

    **apiVersion**: rbac.authorization.k8s.io/v1beta1

    **metadata**:

       **name**: democlusterbinding

    **subjects**:

     - **kind: User**

       **name**: sa

       **apiGroup**: rbac.authorization.k8s.io

    **roleRef**:

       **kind**: ClusterRole

       **name**: democlusterrole

       **apiGroup**: rbac.authorization.k8s.io

3. Once we have that in place, user ***sa*** can list the pods cluster wide

    kubectl list pods --all-namespaces

    kubectl get pods --all-namespaces

## **Admission Control**

Admission Control Modules are software modules that can modify or reject requests. Admission control role comes after authentication and authorization stages are completed.

For more information Please visit [https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/)
