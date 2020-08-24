
# NFS Persistent Volume in Kubernetes

Image by ArtTower from Pixabay

In this tutorial, let’s see as to how to present the NFS file mount to the Kubernetes cluster running on Ubuntu.

![Architecture](https://cdn-images-1.medium.com/max/2000/1*QRBsPfOnA3c7M7MjSUjsrA.png)*Architecture*

**Step-1:** **Installation of NFS Server** -Setup the host machine i.e. Ubuntu as NFS Server

*sudo apt install nfs-kernel-server*

![Install NFS Server](https://cdn-images-1.medium.com/max/2000/1*nqw-m9plntpoeNOqpaMxqw.png)*Install NFS Server*

*sudo mkdir -p /pv/nfs/test-volume
sudo chmod 777 /pv/nfs/test-volume*

![Create NFS Volume and Change Permissions](https://cdn-images-1.medium.com/max/2000/1*xRPDv7MNB95cfda5myJAdw.png)*Create NFS Volume and Change Permissions*

Restart NFS Server and Check the Status

![NFS Restart and Status Check](https://cdn-images-1.medium.com/max/2000/1*ehlaZhRSgYm9YkV2eiLFfQ.png)*NFS Restart and Status Check*

**Step2: Export the Shared Directory**

Update */etc/exports* with a single line as below

*/pv/nfs/test-volume *(rw,sync,no_subtree_check,insecure)*

![Update /etc/exports](https://cdn-images-1.medium.com/max/2000/1*zM0clDHNUEJL_cq1RQCvbA.png)*Update /etc/exports*

*sudo exportfs -a
sudo exportfs -v*

![Export FS](https://cdn-images-1.medium.com/max/2136/1*Qbj2ImrvXYSy_EBmqCs6Dw.png)*Export FS*

Create an Index.html in /pv/nfs/test-volume as below

![File creation in NFS Volume](https://cdn-images-1.medium.com/max/2000/1*cjkSNan6z1GiVYAeFjUkog.png)*File creation in NFS Volume*

**Step3**: **Create Persistent Volume and Persistent Volume Claim**

<iframe src="https://medium.com/media/64f39435ae40db6d64515962b8138b3f" frameborder=0></iframe>

<iframe src="https://medium.com/media/189575a701f51b852071891543b63a62" frameborder=0></iframe>

![PV and PVC Created in Kuberenetes Cluster](https://cdn-images-1.medium.com/max/2000/1*ECI0rk-l6Khbn9V8fyo5Eg.png)*PV and PVC Created in Kuberenetes Cluster*

**Step 4: Deploy Nginx Pod and Expose it as service for access from outside the Kubernetes cluster**

In the Nginx deployment, we have to mention the mount path and as well as the PVC claim details which we have created already, this is an import step. Please refer to the nginx manifest file.

<iframe src="https://medium.com/media/8b1f1b838f3eedaf1f3d67dead04aae9" frameborder=0></iframe>

![](https://cdn-images-1.medium.com/max/2000/1*QarxKDRbsuBz499kTXWclA.png)

**Expose the Service through NodePort**

*kubectl expose deploy nginx-deploy — port 80 — type NodePort*

![Expose the Service](https://cdn-images-1.medium.com/max/2000/1*BMeC5d6lai8i2v4HouCcjQ.png)*Expose the Service*

![](https://cdn-images-1.medium.com/max/2000/1*IRbYLESsM5rX8vXGdC8P7A.png)

Access the Service on Minikube IP and NodePort(30628)

![Access the Service through NodePort](https://cdn-images-1.medium.com/max/2000/1*ePbvFo5uAdig4BeLQQVvew.png)*Access the Service through NodePort*

![Check in the container if the html file is present!!](https://cdn-images-1.medium.com/max/2000/1*hqJKxAJI9qf5dMFGljmKVw.png)*Check in the container if the html file is present!!*

**Step 5: Final Step is to check if we can edit the file in the shared location**

![Edit index.html file](https://cdn-images-1.medium.com/max/2000/1*YmwnqMrcWB9TH29mj3hglw.png)*Edit index.html file*

![Check if the changes are reflecting](https://cdn-images-1.medium.com/max/2000/1*ZWktmcbSJzhnDRekoWSr4w.png)*Check if the changes are reflecting*

*Source Code: [https://github.com/vamsijakkula/nfs-pv-storage](https://github.com/vamsijakkula/nfs-pv-storage)*

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬**.**

**To join our community Slack team chat **🗣️ **read our weekly Faun topics **🗞️,** and connect with the community **📣** click here⬇**

![](https://cdn-images-1.medium.com/max/3000/1*6P3WpLjGv5v1ucm5dgkucg.png)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
