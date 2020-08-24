
# Don’t Memorize Kubernetes API Resources. Use These Two kubectl Commands Instead!

Photo by John Schnobrich on Unsplash

How many times have you forgotten the name of a Kubernetes API resource you intended to create, view, or modify? Maybe you knew the name of the resource but forgot what the proper schema should be? Both of these scenarios happen to me all the time, especially for resources with longer names or for those I may not interact with daily.

For example, let’s pretend that you needed to create a new validating webhook. The ValidatingWebhookConfiguration resource would allow you to register this, but you may not know about this resource or remember its full name. Luckily, using the **kubectl api-resources** command, you could search through all of the registered API resources to find the right one.

Once you find the right resource, you would need to know the correct YAML schema so its configuration can be applied. Schemas can be lengthy and difficult to memorize. Luckily, you could use the **kubectl explain** command to find the full schema, which would help you write the declarative YAML without memorizing every detail.

In this post, I’ll explain the workflow I use to create Kubernetes API resources that involves as little memorization as possible. I first use **kubectl api-resources** to find the name of the resource if I’m unsure of its name. Then, I use **kubectl explain** to brush up on the resource’s schema so that I can create it.

Let’s first focus on kubectl api-resources.

## Discovering API Resources with kubectl api-resources

The **kubectl api-resources** command prints each registered API resource. Below is a snippet from the output of this command.

<iframe src="https://medium.com/media/7868d2ca99e39249c499cd3723ee9b1d" frameborder=0></iframe>

Scrolling through to the middle of the output shows the ValidatingWebhookConfiguration resource mentioned earlier.

<iframe src="https://medium.com/media/468f56f7e9bd65d1db81ee6e2e46738b" frameborder=0></iframe>

While searching through the full contents of this output is one way of interacting with this command, I prefer to pipe this output to **grep** if I know what I’m looking for to limit the number of results returned. For example, let’s say I’m searching for the ValidatingWebhookConfiguration resource, and I know the resource’s name includes the word “validating” but I don’t remember the rest. I can pipe the kubectl api-resources output to grep using the command **kubectl api-resources | grep validating**, shown below:

<iframe src="https://medium.com/media/7e64a312683ec7a8960d31529ace3dfe" frameborder=0></iframe>

Since I grepped this output, I don’t need to search through the entire list of resources to determine the full name.

You can use this trick for any resource. Another good use case is for when you want to recall all of the API resources under a particular API group. For example, let’s say that you have Gatekeeper installed in your cluster. You can reveal each of the API resources required to interact with Gatekeeper by piping the kubectl api-resources output to grep, searching for the word “gatekeeper”:

<iframe src="https://medium.com/media/8c156a15a3d63f6fe9ae3c6b8483abae" frameborder=0></iframe>

Given this output, you would know that you need to work with Config, K8sRequiredLabels, and ConstraintTemplate resources to interact with your Gatekeeper installation.

While finding resources with kubectl api-resources will likely be a useful tool in your toolbox, you must also know their schemas to create them. The **kubectl explain** command can help with this, “explained” next.

## Recalling Resource Schemas with kubectl explain

There are many different resources and schemas you must interact with when working in Kubernetes. Luckily, you don’t have to commit these to memory with the kubectl explain command.

Following the example from before, let’s say that after I found the ValidatingWebhookConfiguration resource, I wanted to start writing the YAML for it. I may know part of the YAML and just need a refresher on the remaining bits, or I may be seeing it for the first time. Either way, I can begin discovering the schema by running **kubectl explain validatingwebhookconfiguration**, shown below:

<iframe src="https://medium.com/media/ec50bfbecb275cbbf0da5cdc45e024a0" frameborder=0></iframe>

This output displays the top-level fields of the schema. You can see that I need to provide the apiVersion, kind, metadata, and webhooks to configure this resource. To step into each of these fields, I need to pass in the **recursive** flag. Below is a snippet of output after using running **kubectl explain validatingwebhookconfiguration --recursive**.

<iframe src="https://medium.com/media/20eb0b90b7ce2facc50e70587a5c3def" frameborder=0></iframe>

Using this command, you can see the full schema down to the last child level. Scrolling to the end of this output is the complete “webhooks” field, which configures the webhook in a ValidatingWebhookConfiguration resource.

<iframe src="https://medium.com/media/fbb60115129c431cd80860bc91ec39df" frameborder=0></iframe>

I use this all the time for both simple and complex resources. I can’t always remember the exact schemas, so I use kubectl explain with the --recursive flag to provide a quick reference from the command line.

You probably noticed that the kubectl explain --recursive command does not define each schema field — it simply outputs what the field is and its corresponding type. If you are unsure about how to configure a particular field, you’ll have to look through the Kubernetes documentation to learn more (or you can pass the field to kubectl explain — see my update below). The good thing, however, is that the schema itself will not be a mystery. You’ll be more familiar with what to look for once you get to the documentation.

## Thanks for Reading!

I hope that you found this article to be helpful. I use the **kubectl api-resources** and **kubectl explain** commands all the time to recall resources that I haven’t memorized. Using these commands will save you a lot of time configuring your Kubernetes API resources.

## Update: 7/31/2020

I previously mentioned that **kubectl explain --recursive** doesn’t provide any detailed descriptions of the output fields. Well, I recently learned a new trick that might save you a trip to documentation. After inspecting the output of kubectl explain validatingwebhookconfigurations --recursive, let’s say that you wanted to learn more about the webhooks.matchPolicy setting. It turns out you can pass this straight to kubectl explain and get a detailed description of the field by running this command:

    kubectl explain validatingwebhookconfiguration.webhooks.matchPolicy

Here’s an example below:

<iframe src="https://medium.com/media/e529cbadc76c61eee0a0914283a8de14" frameborder=0></iframe>

If you want to learn more about a specific field, don’t run off to the documentation quite yet. Pass it to kubectl explain!

*Originally published at [https://austindewey.com](https://austindewey.com/2020/06/21/dont-memorize-kubernetes-api-resources-use-these-two-kubectl-commands-instead/) on June 21, 2020.*
