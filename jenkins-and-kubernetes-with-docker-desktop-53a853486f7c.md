
# Jenkins and Kubernetes with Docker Desktop

Jenkins and Kubernetes with Docker Desktop

With the full release of Kubernetes support for Docker Desktop, I decided to update [my guide of developing locally with Jenkins](https://medium.com/@garunski/local-development-with-kubernetes-and-jenkins-49cfb826ef65). The new offering makes it much easier to have a local Kubernetes cluster running locally.

Configure your [Docker Desktop](https://www.docker.com/products/docker-desktop) for [Windows](https://docs.docker.com/docker-for-windows/#kubernetes) to run Kubernetes. Remember to modify the [Settings](https://docs.docker.com/docker-for-windows/#docker-settings-dialog) to give Docker a bit more resources and share your drives.

To make it easier to debug the deployments onto the cluster, I will deploy [Kubernetes dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) using a [helm](https://docs.helm.sh/using_helm/#installing-helm) command.

    helm install --wait --name k8s-dash --set service.type=NodePort,service.nodePort=31111 stable/kubernetes-dashboard

I ran into a few issues while installing the Kubernetes dashboard. First, helm caches the charts locally, and since I installed a different version of the chart more than a year ago, I ran into the problem of trying to use the latest parameters for an old chart. To fix this issue, run helm repo update to get the latest charts. The second issue was accessing the UI in Chrome which could not decipher the SSL certificate with the following error.

    You cannot visit localhost right now because the website sent scrambled credentials that Chrome cannot process.

To fix this, I followed [https://stackoverflow.com/a/31900210/5732682](https://stackoverflow.com/a/31900210/5732682) answer. I would never enable this on a production server, and neither should you.

Once everything is installed and setup you should be able to visit [https://localhost:31111](https://localhost:31111) and access the [Dashboard UI](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/).

Next, we want to deploy Jenkins onto the cluster and have it configured for local development.

<iframe src="https://medium.com/media/87d87f43738715311301a9fa69035111" frameborder=0></iframe>

Previously, I used the [File System SCM ](https://wiki.jenkins.io/display/JENKINS/File+System+SCM)plugin to monitor and use the local folder for builds. But there is a simpler solution that works almost as well that does not require an extra plugin to be installed. We can use the folder path to the local git repository as if it were a remote repository. The only issue with this way is that Jenkins will only use the git server for changes not local file changes, so you will have to check in the changes locally before the build can pick them up. But I have seen quite better performance from using just git and not relying on the file system to watch for changes.

Docker Desktop with Kubernetes mounts the local shared volumes under /host_mount/{DRIVE_LETTER}/ in Windows environments. The Agent configuration is a separate pod configuration that needs its own path to the source code.

There are some other changes that make life easier that I have implemented since the last time I wrote the configuration. First is disabling security for local development, on line 6. This removes the login part of Jenkins and has the bonus of not having to look up the admin password every time. I have also increased the default Agent resources, line 52. There was an issue with Kubernetes terminating the pod once the limits of the Agent resources were reached and failing builds. This was fixed by increasing the limits.

To avoid the spin up time and needing to download of all the plugins to the storage, I am using a PersistentVolumeClaim that is allocated ahead of time. This way, each time I delete recreate the Jenkins instance, there is not as much lag time of it being brought up. Sometimes it is necessary to delete and recreate the volume and that can be done with a simple kubectl command.

I sometimes ran into the issue of Persistent Volume not able to be bound to the Pod, which I suspect is because I had an old Persistent Volume on the cluster from a previous configuration of Jenkins that may have been interfering with a new volume to be bound to the pod. I just cleared my Persistent Volumes by deleting them from the UI, or you can use kubectl to delete them. After that I deleted the Jenkins deployment and redeployed both the Persistent Volume and Jenkins.

This concludes the quick and simple setup of Jenkins on Kubernetes locally.
