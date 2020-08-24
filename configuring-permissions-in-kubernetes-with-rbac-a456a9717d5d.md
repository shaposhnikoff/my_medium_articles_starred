Unknown markup type 10 { type: [33m10[39m, start: [33m188[39m, end: [33m213[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m296[39m, end: [33m316[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m368[39m, end: [33m397[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m57[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m103[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m122[39m, end: [33m137[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m103[39m, end: [33m114[39m }

# Configuring permissions in Kubernetes with RBAC

Kai Pilger via unsplash.com

*by Nikita Mazur*

Ensuring the control of who has access to your Information System and what users have access to is the objective of an Identity and Access management system. It is one of the fundamental processes in the Security Management and it should be thoroughly taken care of.

In Kubernetes, Identity and User management are not integrated in the platform and should be managed by external IAM platforms like Keycloak, Active Directory, Google’s IAM, etc. However, authentication and authorization are handled by Kubernetes.

In this article we will be focusing on the authorization aspects of the IAM in Kubernetes, and more specifically on how to make sure a user has the right permissions towards the right resources using the Role-Based Access Control model.

## Prerequisites

RBAC is a stable feature from Kubernetes 1.8. In this article we will assume that you are using Kubernetes 1.9+. We will also assume that RBAC has been enabled in your cluster through the --authorization-mode=RBAC option in your Kubernetes API server. You can check this by executing the command kubectl api-versions; if RBAC is enabled you should see the API version .rbac.authorization.k8s.io/v1.

## Overview of RBAC concepts in Kubernetes

The RBAC model in Kubernetes is based on three elements:

* Roles: definition of the permissions for each Kubernetes resource type

* Subjects: users (human or machine users) or groups of users

* RoleBindings: definition of what Subjects have which Roles

Let’s explore how these elements work.

In the example below you can see a Role which allows to get, list and watch pods in the namespace “mynamespace”:

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: mynamespace
      name: example-role
    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "watch", "list"]

To give a user the permissions described in the previous Role it is necessary to create a RoleBinding. In the example below, the RoleBinding “example-rolebinding” binds the Role “example-role” to the user “example-user”:

    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: example-rolebinding
      namespace: mynamespace
    subjects:
    - kind: User
      name: example-user
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: example-role
      apiGroup: rbac.authorization.k8s.io

It should be noted that Roles and RoleBindings are namespaced, which means that the permissions can only be given for the Kubernetes resources that are in the same namespace as the Role and the RoleBinding. Also, it is without saying that a RoleBinding can only reference a Role that exists in its namespace.

## Roles, ClusterRoles, RoleBindings and ClusterRoleBindings

In the previous example we have used Roles and RoleBindings. But, there is also the possibility to use ClusterRoles and ClusterRoleBindings which are useful in the following cases:

* Give permissions for non-namespaced resources like nodes

* Give permissions for resources in all the namespaces of a cluster

* Give permissions for non-resource endpoints like /healthz

The definitions of the cluster scoped versions of Roles and RoleBindings are very similar to the non-cluster scoped ones. If we take the previous example and adapt it, we will have the following definitions. ClusterRole:

    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: example-clusterrole
    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "watch", "list"]

ClusterRoleBinding:

    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: example-clusterrolebinding
    subjects:
    - kind: User
      name: example-user
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: example-clusterrole
      apiGroup: rbac.authorization.k8s.io

## How to enable permissions in Roles and ClusterRoles

In the first example we granted basic permissions to a user to get, watch and list pods. Let’s explore other possibilities we have for the different resources and permissions.

In the example below the Role allows to perform any actions with a Deployment resource:

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: mynamespace
      name: example-role-2
    rules:
    - apiGroups: ["extensions", "apps"]
      resources: ["deployments"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

Notice that in this case the apiGroups field has been filled in with the API group of the Deployment. Depending on the Kubernetes version, the Deployment resource is available in the API apps/v1 or extensions/v1beta2; the API group is the part before the slash.

We can have multiple rules defined in a Role, like in the example below:

    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "list", "watch"]
    - apiGroups: ["batch", "extensions"]
      resources: ["jobs"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

In this case we grant different permissions depending on whether the targeted resource is a Pod or a Job.

We can also target a resource by its name, like in the following example:

    rules:
    - apiGroups: [""]
      resources: ["configmaps"]
      resourceNames: ["my-config"]
      verbs: ["get"]

## How to bind a Subject to a Role or a ClusterRole

In the first example we have seen how to bind a human user to a Role. However, there is also the possibility to bind a service account (non-human user) or a group of human users and/or service accounts.

In the example below, the RoleBinding example-rolebindingbinds the ServiceAccount example-sto the Role example-role:

    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: example-rolebinding
      namespace: mynamespace
    subjects:
    - kind: ServiceAccount
      name: example-sa
      namespace: mynamespace
    roleRef:
      kind: Role
      name: example-role
      apiGroup: rbac.authorization.k8s.io

You can create a ServiceAccount with the following command:

    kubectl create serviceaccount example-sa --namespace mynamespace

In the previous RoleBinding definition we can also replace the subject to bind a group. In the example below, we bind the frontend-adminsgroup:

    subjects:
    - kind: Group
      name: "frontend-admins"
      apiGroup: rbac.authorization.k8s.io

Another possibility is to bind group of service accounts. Here we bind all the service accounts in the mynamespacenamespace:

    subjects:
    - kind: Group
      name: system:serviceaccounts:mynamespace
      apiGroup: rbac.authorization.k8s.io

Or all the service accounts in the cluster:

    subjects:
    - kind: Group
      name: system:serviceaccounts
      apiGroup: rbac.authorization.k8s.io

## One last thing about RBAC in Kubernetes

We have seen how to grant permissions to a user or a service account using the Role-based Access Control model. It is an efficient way to implement authorization in Kubernetes, and it is probably the most popular one, but it is not the only model available: you can also use other models like ABAC (Attribute-based access control), Node Authorization model and the Webhook mode. We will be describing these in further articles, along with another IAM feature in Kubernetes: authentication.

See you soon, protect your cluster and stay safe!

Feel free to leave your feedback. Don’t forget to [follow](https://twitter.com/@containerumcom) us on Twitter and [join](https://t.me/joinchat/DHMHbEb8Q3EUpb3qWDzc5A) our Telegram chat to stay tuned!

You might also want to check our Containerum project** **on [GitHub](https://github.com/containerum/containerum). We really need your feedback to make it stronger — you can submit an issue, or just support the project by giving it a ⭐. Your support matters to us!

Containerum is your source of knowledge on Kubernetes.
