Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m59[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m78[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m251[39m, end: [33m259[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m92[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m63[39m, end: [33m70[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m23[39m, end: [33m30[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m121[39m, end: [33m131[39m }

# Bringing your k8s cluster back from death — K8s recovery series

Credit for the image goes to Ashley McNamara’s awesome Gopher collection

With the increase in usage and acceptability of **Kubernetes** for production, it becomes important to know how to recover the cluster if its core components starts to misbehave. Let’s take an instance, what happens if the all schedulers on the clusters are down, who will schedule the scheduler itself. 🤔

I will be covering some of the cluster recovery scenarios in this series of articles were I will be killing/shooting some of the core components and then recovering back the cluster from the situation.

### **Scenario: K8s scheduler is DOWN**

K8s [scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) is responsible for scheduling a pod on the cluster, and if it goes down nothing new on the cluster will be able to schedule, including the k8s scheduler itself.

### **Impact:**

Your existing services will continue to run but any new pods will not be able to schedule, for instance your deployments will be impacted, any pod if dies because of Liveliness probes will not be able to come up and so on.

### **Mock Scenario:**

**Disclaimer:** DO NOT TRY ON PROD/LIVE clusters.

1. Take backup of the kube-scheduler deployment

    kubectl get deployments -n kube-system kube-scheduler -o yaml > ~/scheduler_backup.yaml

2. Delete the `kube-scheduler` deployment:

    kubectl delete deployments kube-scheduler -n kube-system

3. Verify all scheduler pods are deleted:

    kubectl get pods -n kube-system | grep scheduler

![All scheduler pods are deleted](https://cdn-images-1.medium.com/max/5760/1*Pc6Wbnw4pj6UdmC8lRjrNg.png)*All scheduler pods are deleted*

4. Create back the deployment using the backup created earlier:

    kubectl create -f ~/scheduler_backup.yaml

5. Verify all scheduler pods are in `Pending` state

    kubectl get pods -n kube-system | grep scheduler

![Scheduler Pods in Pending State](https://cdn-images-1.medium.com/max/5760/1*3z1NpUVGrb1Y1bIfU7DiDg.png)*Scheduler Pods in Pending State*

## **Recovery Procedure:**

There are two ways for recovering from this situation:

### **Method-1: Create a recovery scheduler pod:**

Run the below command for generating a POD spec for the rescue scheduler pod from the existing deployment spec of the k8s scheduler.

    label=kube-scheduler ;

    namespace=kube-system ;

    kubectl get deploy — namespace=$namespace -l k8s-app=${label} -o json | jq — arg namespace $namespace — arg name ${label}-rescue — arg node $(kubectl get node -l node-role.kubernetes.io/master -o jsonpath=’{.items[0].metadata.name}’) ‘.items[0].spec.template | .kind = “Pod” | .apiVersion = “v1” | del(.metadata, .spec.nodeSelector) | .metadata.namespace = $namespace | .metadata.name = $name | .spec.containers[0].name = $name | .spec.nodeName = $node | .spec.serviceAccount = “default” | .spec.serviceAccountName = “default” ‘ > ~/rescue-scheduler.yml

    kubectl apply -f ~/rescue-scheduler.yml

![Rescue scheduler creation](https://cdn-images-1.medium.com/max/5760/1*W5bOESUBvgnzxkbslTfzMA.png)*Rescue scheduler creation*

This will create a recovery pod named *kube-scheduler-rescue* in the *kube-system* and get it schedule on one of the master nodes. You would be wondering ***how it got** **scheduled??*** The catch is the above command added the ip of one of the master nodes in the nodeName and explicitly scheduled the pod on the master node.

![Rescue scheduler in action](https://cdn-images-1.medium.com/max/4760/1*lRzEePBH4_ErtqOwv_Ei1A.png)*Rescue scheduler in action*

You will notice that the other scheduler pods gets scheduled.

![Scheduler pods getting scheduled](https://cdn-images-1.medium.com/max/3892/1*UZMhOq2dsthrDk5JnEgjkA.png)*Scheduler pods getting scheduled*
> **Note:** You might see one pod in pending state, depending on whether you have [antiAffinity](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity) enabled or not. (I have it enabled)

Now delete the recovery/rescue pod.

    kubectl delete pod kube-scheduler-rescue -n kube-system

Now the last `Pending` scheduler will also get scheduled.

![All scheduler pods recovered](https://cdn-images-1.medium.com/max/4356/1*Fi4KYY6uDj9ZcCNux2LxBQ.png)*All scheduler pods recovered*

### Method-2: Edit the scheduler deployment:

You can directly edit the deployment file of the kube-scheduler and add your master name as *nodeName*

    kubectl edit deployment -n kube-system kube-scheduler

Add the following above/below the `nodeSelector` in the file (Note: It can be anywhere under template spec, giving the location for easy finding. Be cautious for the indent). Replace the <IP/Name> with any of the master’s (follow screenshots in case of doubts)

    nodeName: <IP>

![Adding nodeName in scheduler deployment](https://cdn-images-1.medium.com/max/2356/1*GciAVZQfPFm7tKcEPgkVgA.png)*Adding nodeName in scheduler deployment*

Once saved, you will see the scheduler pods transitioning into *Running* state, but all are getting scheduled on the same node. (Because that is what we instructed it to with the above edit)

![Scheduler pods on the same node](https://cdn-images-1.medium.com/max/5760/1*wOKcErPIs_MJ25p32dj71g.png)*Scheduler pods on the same node*

Once the pods are stable, again edit the deployment and remove the above made changes. This will let the scheduler go to the other nodes based on the selector.

![](https://cdn-images-1.medium.com/max/5760/1*quFyiGN2GJHNfDGaApoiLw.png)

You might see a pod in *Pending* state again depending on the affinity configurations, if yes, you will need to delete the *replicaset* having one pod.

    kubectl get replicasets -n kube-system | grep scheduler

![](https://cdn-images-1.medium.com/max/5760/1*Vap2-3dZH1lB_Ke_4bb4Mw.png)

***Congratulations!!*** You have successfully brought back your scheduler and cluster to life.

Keep looking for more in the series. ***Happy Learning!!!***

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
