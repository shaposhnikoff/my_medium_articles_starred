Unknown markup type 10 { type: [33m10[39m, start: [33m255[39m, end: [33m259[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m270[39m, end: [33m273[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m278[39m, end: [33m281[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m176[39m, end: [33m183[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m189[39m, end: [33m200[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m235[39m, end: [33m246[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m286[39m, end: [33m293[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m292[39m, end: [33m303[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m70[39m, end: [33m84[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m173[39m, end: [33m187[39m }

# Simplifying Kubernetes RBAC Management

Photo by Daniel von Appen on Unsplash

At [ReactiveOps](https://reactiveops.com/) we get to help some great companies with Kubernetes. Working with so many different organizations gives us a unique perspective on some of the challenges involved with maintaining Kubernetes at scale.

One of the most common areas we see organizations struggling with Kubernetes involves managing RBAC. For many organizations, it’s so difficult and confusing that RBAC is often not even implemented, or only implemented halfway. For the organizations that do manage to properly lock down RBAC configuration, they often find the configuration difficult to maintain.
> The lack of proper RBAC configuration generally results in too many people ending up with too much access to too many Kubernetes clusters.

After seeing first hand how challenging RBAC configuration can be to scale, we built an open source Kubernetes operator to try to help. Today we’re introducing [RBAC Manager](https://github.com/reactiveops/rbac-manager). We hope this project will help organizations that are struggling to manage their current RBAC configuration, and that it will encourage more organizations to fully embrace RBAC in their Kubernetes implementations.

## Current Challenges with RBAC Management

To fully understand our goals with RBAC Manager, it’s worth taking some time to understand how RBAC configuration currently works with Kubernetes. Let’s start with a very simple example with a single user. We’ll call this user “A”, and say that they need edit access to web and api namespaces in a cluster. To accomplish that, we’d create 2 new role bindings:

<iframe src="https://medium.com/media/d28950c224f93c8ab3a0cab662e66cbf" frameborder=0></iframe>

Of course that’s not particularly complicated, but this is a very simple bit of configuration for a single user. This becomes significantly more complex when we introduce more users, namespaces, or clusters.

With all the YAML configuration that this increased scale will inevitably require, it quickly becomes difficult to keep track of it all. It becomes a daunting process to take a list of users and see what level of access they each have across a cluster.

Additionally, if we ever wanted to change the level of access for this user in one of these namespaces, it would not be as simple as you might hope. There’s no way to change a roleRef in a RoleBinding. The approach involves deleting a RoleBinding and creating a new one with an updated roleRef instead.

In workflows that are increasingly driven by automation, and where infrastructure as code is often seen as the gold standard, this doesn’t fit in very well. That’s not even including how difficult it might get to manage deleting role bindings at any kind of scale. You couldn’t just delete a RoleBinding from a repo and hope for some kind of automated task to manage the change for you.

Now of course it is possible to modify Kubernetes roles and cluster roles. To get the most granular results, one would create unique roles and cluster roles for each role binding, and then modify the roles instead of the bindings to them. Although that’s certainly a great solution in some cases, it can become quite complex to manage these roles at any scale. Our goal here was to simplify RBAC management, and as such, we wanted to take advantage of the [excellent default roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#default-roles-and-role-bindings) provided by Kubernetes with our workflows. In my experience, these default cluster roles (view, edit, admin, and cluster-admin) cover the vast majority of use cases for most organizations when it comes to RBAC user management.

## An Attempt to Simplify this with RBAC Manager

Our goal with RBAC Manager has always been to make RBAC configuration just a bit simpler to work with. We wanted an approach that would allow for simpler configuration along with a more straightforward path to automation.

### Simpler Configuration

RBAC Manager is a Kubernetes operator that watches for new or updated RBACDefinition custom resources. To create the two role bindings for the scenario described above, the RBACDefinition requires approximately half as much YAML configuration:

<iframe src="https://medium.com/media/999e9b65760ab2dc291455b4c33196e6" frameborder=0></iframe>

The reduction in configuration only gets more significant as more users, namespaces, or clusters are added to the picture. Of course there’s a lot more to RBAC than this example shows. RBAC Manager supports binding users, groups, or service accounts to any combination of roles or cluster roles, at either a namespace or cluster level.

This ability to assign multiple role bindings with a single custom resource has been quite powerful in our experience. No longer do you have to look for role bindings across multiple namespaces to understand the current state of your RBAC configuration, instead it’s neatly summarized in a single RBAC Definition.

### A Path to Automation

RBAC Manager was built with the idea that we wanted to automate RBAC management in Kubernetes. Making these kind of changes manually was error prone and simply did not scale well. Unfortunately the existing Kubernetes resources here did not lend themselves to automated updates in the same way that something like a Deployment would.

With that in mind, a RBAC Definition functions much more like a Deployment than a Role Binding. Just like a Deployment simplifies updating pods, a RBAC Definition is intended to simplify updating Role Bindings and Cluster Role Bindings. Changes to a RBAC Definition trigger changes in resources it owns, in this case Role Bindings and Cluster Role bindings. If a role binding is no longer referenced in a RBAC Definition, RBAC Manager deletes it. Similarly, if a requested role in a RBAC Definition changes, RBAC Manager will delete and recreate the appropriate Role Bindings. And finally, if an RBAC Definition is deleted, all the associated Role Bindings and Cluster Role Bindings will also automatically be deleted.

This new approach opens the doors to automation, and allows you to manage RBAC Configuration with a CI workflow. Similar to how Deployments simplify updating Pods in a CI workflow, a RBAC Definition can simplify updating Role Bindings and Cluster Role Bindings.

We’ve found RBAC Manager especially useful for our workflows at ReactiveOps over the past couple months, we hope you find this just as helpful as we have.

This is a fully open source project, and we’d love any ideas or contributions you might have. For more technical information, or to install it, visit our [GitHub](https://github.com/reactiveops/rbac-manager). Stay tuned for some follow up posts covering how we use RBAC Manager to manage RBAC configuration on AWS and GKE clusters.

*If you enjoyed this story, clap it up!*

[*uptime 99](https://medium.com/uptime-99) is a [ReactiveOps](https://www.reactiveops.com/?utm_source=medium&utm_medium=uptime99&utm_campaign=uptime99_footer) publication about DevOps, containers, and everything cloud. If you have an article you’d like to submit, [email us](mailto:scott@reactiveops.com)!*
