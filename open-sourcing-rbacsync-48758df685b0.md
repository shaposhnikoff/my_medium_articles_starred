
# Managing Kubernetes RBAC Groups

github.com/cruise-automation/rbacsync

At Cruise, we use [Kubernetes](https://kubernetes.io/) for many workloads across the organization. In the past, we had several clusters across different teams, deployed and managed in a variety of ways. While this worked early on, it quickly became an operational burden. To address this, we introduced a Kubernetes-based Platform as a Service (PaaS) . This effort has allowed us to consolidate workloads into just a few clusters, deployed in a common way, to allow teams to focus on building and shipping their software rather than operational concerns. Teams previously required a full time Site Reliability Engineer to setup and manage a cluster. Now they only need to get access to the PaaS cluster and can get started running their workload. This frees up our SREs to solve other problems and allows us to have more engineering resources focusing on our core problems.

When consolidating our users onto a single cluster, we recognized that controlling access to specific resources would become a problem. In the very least, we wanted to ensure that users on different teams couldn’t tamper with each other’s software and with services provided as part of the PaaS. Our plan was to use [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) to manage this. It allows cluster operators to define *roles* for operations in Kubernetes. These *roles* can then be bound to specific resources, using *role binding, *to name one or more *subjects* that get those *roles*. These subjects can be *users, groups* or *service accounts*.

Because we’re using Google Kubernetes Engine (GKE), we have a strong login identity. This makes it easy to identify who is using the cluster, but not the groups to which a user belongs. This meant that for each user we added to the cluster, we needed to create *role bindings* to link them to a group. While you can add several users to a single role binding, keeping this updated became difficult. Not only did we want to assign specific roles to large numbers of engineers, we also wanted to define groups make it even easier.

## What did we do?

With an understanding of the problem above, we put together a configuration example that would allow us to map *roles* to *groups*. Speaking in Kubernetes YAML, we were hoping for something like this:

![](https://cdn-images-1.medium.com/max/2816/0*_1R7vYtQblvhQeeI)

The above simply says “bind cluster role ‘cluster-admin’ to the group ‘mygroup-admin@getcruise.com’”. If that group has the members “a@getcruise.com”, “b@getcruise.com” and “c@getcruise.com”, and it is honored by Kubernetes authorization system, it is equivalent to the following:

![](https://cdn-images-1.medium.com/max/2824/0*tLAezVPKPVVfx9ed)

Unfortunately, GKE does not make the group memberships of users available to the Kubernetes authorization system, and we couldn’t replace the component or change it to make it work as we needed. Because translating “groups” to a list of “subjects” is relatively straightforward, we wrote [RBACSync](https://github.com/cruise-automation/rbacsync) to automate the process.

## How does it work?

[RBACSync](https://github.com/cruise-automation/rbacsync) is what is called a *Kubernetes controller*. A *controller* watches the Kubernetes API for changes and takes action on those changes. In the case of RBACSync, it watches a configuration object, defined by a [Custom Resource Definition](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/), that states a *group* and *role reference*. When the controller sees a new configuration or gets an update on an existing configuration, it looks up the group in an *upstream* and creates a *role binding* with all of the members of the group.

The above is a mouthful, so hopefully an example will help to illustrate it more clearly. The following is a minimal ClusterRBACSyncConfig that binds the *group* “mygroup-admin@getcruise.com” to the *role reference* “cluster-admin”:

![](https://cdn-images-1.medium.com/max/2796/0*h36xPw6FOe3qBbfC)

When the RBACSync controller see this configuration, it calls into the Google Groups API, looks up all the members of the group, then synthesizes the appropriate *role binding. *Let’s say that “mygroup-admin@getcruise.com” has the members “a@getcruise.com”, “b@getcruise.com” and “c@getcruise.com”. The following would be created, binding the members to the role “cluster-admin”:

![](https://cdn-images-1.medium.com/max/2836/0*g1t8pzj5J2ZfsAK1)

If you’re squinting at this YAML and asking yourself what happened, notice that the “subjects” field has all the members that we had called out above. Also notice that this structure that was automatically created matches the “equivalent” structure from the example in the section above. While this seems simple, this configuration makes Kubernetes RBAC significantly more powerful. Instead of deploying a large number of *role bindings*, we now just deploy a few RBACSyncConfig objects and let the controller do the rest. If the members of the group change, RBACSync will poll the upstream API and update the members on that group.

Hopefully, that provides enough of an overview to get the idea. If you want to understand it in more depth or deploy it in your cluster, please check out the [README](https://github.com/cruise-automation/rbacsync).

## In Production

We’ve been running RBACSync in production for about four months and have discovered a few things in the process.

The first problem we ran into was that Google Cloud Platform (GCP) service accounts could not be added to Google Groups, causing us to split these out into separate *role bindings*. We fixed this by allowing one to augment groups in the configuration with extra members, explicitly called out. This feature also allows you to simply define the group directly in the RBACSyncConfig, such that the controller will automatically create role bindings for each reference.

Deploying RBACSync into our environment made it easier to map users to roles, but we quickly found that the existing roles were insufficient and had to create new roles to meet our requirements. After we deployed the new roles, we had to be responsive to user feedback and tune them to the right level of access. Whether or not you’re planning to use RBACSync, make sure to leave time when implementing RBAC to come up with a solid set of roles that implement your policy.

Another issue that we ran into was Google Group API availability. If the upstream for a group becomes unavailable, it currently removes all of the members from the *role binding*. To defend against a complete loss of access, critical group members can have cluster administration bindings through static configuration, ensuring that a group of users can always have access. We have found that this isn’t quite enough and would like to introduce group membership caching into RBACSync to prevent loss of access on loss of connectivity.

To avoid privilege escalation, we separated cluster configurations and namespace configurations, using ClusterRBACSyncConfig and RBACSyncConfig, allowing each to *only *reference ClusterRole and Role respectively. Don’t worry if you didn’t understand that on the first read through. If you’re curious, the details are [here](https://github.com/cruise-automation/rbacsync#custom-resources). The problem with this approach is that it requires one to deploy the same roles to every namespace. Since then, we’ve found that having common cluster roles that bind to a namespace is sufficient and does not risk privilege escalation as long as access to the configuration is also limited. We’ll relax this in the future.

## Conclusion

In the future, we intend to continue hardening RBACSync based on our experience with it in production. While we don’t need it at this time, we’d also like to see support for other group up streams, such as Okta or Active Directory, as well as verifying it can work on Kubernetes clusters that aren’t on GKE.

The biggest thing we learned with RBACSync is that creating controllers is a reasonable path to make Kubernetes work in our organization. There are many steps required to make Kubernetes production ready, but the flexible API and the ability to extend it with new types and behavior means that we were able to deploy sooner and didn’t have to wait for upstream features.

If this project interests you and you want to work on our Kubernetes-based platform to support self-driving car development, [we’re currently hiring in SF & Seattle](https://getcruise.com/careers)!

GitHub: [https://github.com/cruise-automation/rbacsync](https://github.com/cruise-automation/rbacsync)
