Unknown markup type 10 { type: [33m10[39m, start: [33m208[39m, end: [33m305[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m191[39m, end: [33m210[39m }

# Kubernetes -Horizontal Pod AutoScaler(HPA)

Image by David Mark from Pixabay

Controller Manager compare the metrics received from Metrics server API in regular intervals and scales the deployment once the defined threshold is crossed. HPA is a control loop as defined by the parameter --horizontal-pod-autoscaler-sync-period( in our case it's 10 seconds, by default it's 15 seconds)

![HPA Architecture](https://cdn-images-1.medium.com/max/2000/1*pMsAE6aM1utl5YDBbutDgw.png)*HPA Architecture*

In this post , we will see as how we can scale Kubernetes pods using Horizontal Pod Autoscaler(HPA) based on CPU and Memory. Support for scaling on memory and custom metrics, can be found in autoscaling/v2beta2

We will see as how HPA can be implemented on Minikube .

**Step-1 : Enable Minikube with the following settings**

*minikube start — extra-config=controller-manager.horizontal-pod-autoscaler-upscale-delay=1m — extra-config=controller-manager.horizontal-pod-autoscaler-downscale-delay=1m — extra-config=controller-manager.horizontal-pod-autoscaler-sync-period=10s — extra-config=controller-manager.horizontal-pod-autoscaler-downscale-stabilization=1m*

In our deployment we will override the default values as above , so we can observe the autoscale and downscale bit early as compared to the default values.

![Minikube with settings enabled](https://cdn-images-1.medium.com/max/2582/1*bu1S4D0ky0U97CEtPU_jWw.png)*Minikube with settings enabled*

*minikube add-ons enable metrics-server*

![Minikube with metrics-server enabled](https://cdn-images-1.medium.com/max/2000/1*eblbSZCPQ4w09yn696mNww.png)*Minikube with metrics-server enabled*

**Step-2 : Build the docker image and push it to the repo**

*sudo docker build -t vamsijakkula/php-hpa .*

*sudo docker push vamsijakkula/php-hpa*

This image is a sample php deployment which has some basic compute intensive tasks. For details, please refer to the source code given below.

<iframe src="https://medium.com/media/12813702bf9ff203bbda051c99bf31cf" frameborder=0></iframe>

**Step-3 : Deploy the manifest *kubectl create -f php-apache.yml***

![PHP Deployment](https://cdn-images-1.medium.com/max/2000/1*U0Hx419Ua1a9DuS6u13n4w.png)*PHP Deployment*

**Step-4: Create HPA( CPU Utilization)— kubectl create -f hpa.yml**

<iframe src="https://medium.com/media/8b04fce6c2aa5b9bb0388b842c58b1c9" frameborder=0></iframe>

We can even define a HPA as *kubectl autoscale deployment php-apache — cpu-percent=50 — min=1 — max=10*

For HPA we are defining to auto-scale based on CPU Utilization of 50 % and max replicas to be 10 pods and min to be 1 post downscale. Pods will be downscaled after a stabilization period of 1m.

![HPA Deployment](https://cdn-images-1.medium.com/max/2000/1*jqIiCs19XGDuxmzzLRWagw.png)*HPA Deployment*

**Step-5: Generate Load , so the pods will scale based on threshold and down-scale after a min**.

*kubectl run — generator=run-pod/v1 -it — rm load-generator — image=busybox /bin/sh
while true; do wget -q -O- [http://php-apache](http://php-apache); done*

![Load Generation](https://cdn-images-1.medium.com/max/2582/1*2ph5I_vLLoaFQaL5jN9JcA.png)*Load Generation*

**Step-6 : Verify HPA and Deployment**

![HPA Threshold Increase](https://cdn-images-1.medium.com/max/2000/1*hDCWOfEC0W1PuP2QAj6IVw.png)*HPA Threshold Increase*

Pods Auto-scaled based on CPU Utilization from 1 to 4 and it’s in increasing mode.

![](https://cdn-images-1.medium.com/max/2000/1*ftu9oHJ-q368ZmRZRFIIpw.png)

**Step:7 Stop the load generation process and watch the down-scale of the pods**.

All the pods got downscaled to 1 and CPU Utilization came down as well.

![CPU Utilization Came Down and Rest of the Pods Terminated](https://cdn-images-1.medium.com/max/2000/1*MSdgiSZXV2Jj9RsKRKXU3Q.png)*CPU Utilization Came Down and Rest of the Pods Terminated*

**Step-8: Create a new HPA based on Memory Utilization.**

Create HPA , mention the avgUtilization value as 10Mi as each Pod is currently running at 12–13 Mi.

![Current Pod Memory Utilization](https://cdn-images-1.medium.com/max/2000/1*gnju1GrH68YFFSFGgE6Z5A.png)*Current Pod Memory Utilization*

<iframe src="https://medium.com/media/9bb9d2de9f7e8f9cc9866609f08a59ac" frameborder=0></iframe>

**Step 9 : Generate load as above and watch the HPA**

HPA will scale the pods

![Pods Scaling](https://cdn-images-1.medium.com/max/2000/1*cMCLkqlbP3v3JyL_4OMRpQ.png)*Pods Scaling*

![HPA Based on Memory](https://cdn-images-1.medium.com/max/2000/1*CQ34Lv2QjEtUcScNYEN7ww.png)*HPA Based on Memory*

Source Code:[https://github.com/vamsijakkula/hpa](https://github.com/vamsijakkula/hpa)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬**.**

**To join our community Slack team chat **🗣️ **read our weekly Faun topics **🗞️,** and connect with the community **📣** click here⬇**

![](https://cdn-images-1.medium.com/max/3000/1*6P3WpLjGv5v1ucm5dgkucg.png)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
