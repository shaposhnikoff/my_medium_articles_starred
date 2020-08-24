Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m41[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m99[39m, end: [33m109[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m130[39m, end: [33m153[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m171[39m, end: [33m219[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m227[39m, end: [33m236[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m303[39m, end: [33m310[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m333[39m, end: [33m342[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m103[39m, end: [33m106[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m132[39m, end: [33m153[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m123[39m, end: [33m134[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m149[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m381[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m3[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m63[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m89[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m118[39m, end: [33m132[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m185[39m, end: [33m195[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m3[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m69[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m126[39m, end: [33m135[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m3[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m69[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m113[39m, end: [33m118[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m183[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m3[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m79[39m, end: [33m80[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m104[39m, end: [33m121[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m168[39m, end: [33m183[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m10[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m175[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m3[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m75[39m, end: [33m108[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m16[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m41[39m, end: [33m46[39m }

# Let’s Make Kubernetes CLI Easier

Similar to Docker CLI, the kubectl is also not very user friendly, you have to be explicit about lots of options which actually make sense as defaults in everyday usage and debugging when we are working on multi-namespaced Kubernetes cluster(s). The aim of this post is to define some functions, aliases, and JSON templates to make the CLI easier for us.

*The below screenshot shows a glimpse of what it would enable you to do:*

![](https://cdn-images-1.medium.com/max/6648/1*l4F2d91P4-Ex0QfhKtI1GA.png)

*Let’s first look at** most frequent kubectl** commands and their frequently used options (make sure to pay attention to comments):*

<iframe src="https://medium.com/media/e09f06c10df7ce0d51f50fc7ecd07567" frameborder=0></iframe>

Below shortcuts can make your life damn easier. You can source these scripts in your profile or rc file. But, to get everything pre-setup, you can use [**goyalmunish/devenv](https://hub.docker.com/r/goyalmunish/devenv)** Docker image as your development environment.

<iframe src="https://medium.com/media/fce310d33258c045c109f70126e83ec9" frameborder=0></iframe>

It uses below helper file ([**goyalmunish/devenv](https://hub.docker.com/r/goyalmunish/devenv)** image takes care of all setup for you):

![](https://cdn-images-1.medium.com/max/3520/1*WMRcWOtT5U2dS2XDlphslw.png)

<iframe src="https://medium.com/media/d11c63e1556e61433131867d80384819" frameborder=0></iframe>

*Note that output of kubectl get pod <pod_name> -o json is a JSON object representing pod with name <pod_name>. But, the output of kubectl get pod -o json is in the format {"apiVersion": "v1", items: POD_MANIFESTS_ARRAY}. Check [**kc_plural](http://0.0.0.0:8000/conf/mgoyal_bashrc_zshrc_common)** alias which makes both the outputs uniform. So, when dealing with kubectl, unless you are using kc_plural:*

* Use jq '. | {foo: bar}' on single JSON input.

* Use jq '.items[] | {foo: bar}' on multiple JSON inputs.

Note that kubectl get all does not list everything. Refer [Rules for extending special resource alias - all](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-cli/kubectl-conventions.md#rules-for-extending-special-resource-alias---all) to know these rules. Use kubectl api-resources for a complete list of supported resources.

The kubectl api-resources enumerates the resource types available in your cluster. This means that you can combine it with kubectl get to actually list every instance of every resource type in a namespace:

    kubectl api-resources --verbs=list --namespaced -o name \
      | xargs -n 1 kubectl get --show-kind --ignore-not-found -l <label>=<value> -n <namespace>

***Output of kubectl:***

*Output of kubectl get pod looks similar to:*

    NAME                                                       READY   STATUS      RESTARTS   AGE
    guide-graph-app-server-7c98655497-gp4hx                    2/2     Running     0          5h22m
    help-center-migrations                                     0/2     Completed   0          5h21m
    api-tests-tksd9-3094769849                                 0/2     Error       0          5h10m

Here,

* 2/2 in READY status for guide-graph-app-server-7c98655497-gp4hx means that two containers (doesn’t say anything about initContainers, but they would have already completed as then only containers can run) and both are in running status.

* 0/2 in READY status for help-center-migrations means that it has two containers and non-of them is running. Further status is Completed, which means that both the containers have finished there (and so ended) successfully.

* 0/2 in READY status for api-tests-tksd9-3094769849 means none of the containers is running and the STATUS status Error suggests that something went wrong.

*Output of kubectl get deployment looks similar to:*

    NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
    zendesk-worker                                        4/4     4            4           5h56m

Here,

* 4/4 in READY status should be read as status.readyReplicas/status.replicas, as 4 as AVAILABLE status is availableReplicas (number of available pods (ready for at least **minReadySeconds**) targeted by this deployment).

*Output of kubectl get job looks similar to:*

    NAME                         COMPLETIONS   DURATION   AGE
    account-service-migrations   1/1           3m25s      6h8m
    kafka-bootstrap              1/1           41s        6h8m

Here,

* 1/1 in COMPLETIONS status for account-service-migrations should be read as spec.completions/status.succeeded.

***The kubectl debug:***

The addition of kubectl debug command to [v1.18](https://kubernetes.io/docs/setup/release/notes/#v1-18-0) allows developers to easily debug their Pods inside the cluster. This command allows one to create an [ephemeral container](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/) that runs next to the Pod one is trying to examine, but also attaches to the console for interactive troubleshooting.

Check [sig-clie/20190805-kubectl-debug.md](https://github.com/kubernetes/enhancements/blob/master/keps/sig-cli/20190805-kubectl-debug.md) for details.

**Here are some related interesting stories that you might find helpful:**

* [Why and How to set Probes in Kubernetes? Design a robust K8s cluster](https://medium.com/@goyalmunish/why-and-how-to-set-probes-in-kubernetes-d7da39e94e64)

* [Host vs. Container Environment Variables in docker exec command](https://medium.com/@goyalmunish/passing-host-vs-container-environment-variables-to-docker-exec-5c1b18e6de8e)

* [Let’s Make Docker CLI Easier](https://medium.com/@goyalmunish/lets-make-docker-cli-easier-75009d00830e)

* [Docker simplified!](https://medium.com/@goyalmunish/docker-simplified-ad1f8a7350bf)

* [Kubernetes simplified!](https://medium.com/@goyalmunish/kubernetes-simplified-300fef5fb0e6)
