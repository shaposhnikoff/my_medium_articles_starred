
# Installing Istio On Google Kubernetes Engine (GKE)

Installing Istio On Google Kubernetes Engine (GKE)

[Istio](https://istio.io/) is the new cool kid on the block. Well, it is the new cool kid on the block over on the street with all the houses just starting to be built. It’s new, really new. And even better, it just hit 1.0!

### What is Istio? Why should I care?

You should care because Istio adds a lot of features to your Kubernetes Cluster that you’re going to want. However if you don’t know what Istio is or why you should care I recommend going and reading the official documentation on Istio. They do an amazing job spelling it all out in real terms you can understand.
[**What is Istio?**
*Cloud platforms provide a wealth of benefits for the organizations that use them. There's no denying, however, that…*istio.io](https://istio.io/docs/concepts/what-is-istio/)

Assuming you read about Istio or already knew it is important, let’s focus on the purpose of this article, how to install Istio on Google Kubernetes Engine right now. I say right now because the Google Cloud folks are hard at work creating a managed instance of Istio. When this comes out it is likely that with just a command or two you’ll add Istio into your already running Kubernetes Cluster and gain some of the benefits.

For now, let’s look at the easiest way to integrate Istio into Google Kubernetes Engine (GKE).

*If you are unsure of what Kubernetes is, why it is important, or just getting started with Kubernetes, this isn’t the article for you. I recommend going and checking out my other posts on Google Cloud Platform (GCP) and GKE.*
[**Kubernetes: Day One**
*This is the obligatory step one Kubernetes post. If you’re interested in Kubernetes you’ve probably read 100 of these…*medium.com](https://medium.com/google-cloud/kubernetes-day-one-30a80b5dcb29)

## Setup Your Kubernetes Environment

First thing we will need to do is setup a Kubernetes Cluster in your Google Cloud Project. As with other articles I’ve written, I have created some scripts that you can run in your Google Shell environment to make this process super painless. The following command will [create a new Kubernetes Cluster](https://medium.com/google-cloud/kubernetes-day-one-30a80b5dcb29) and then [install Helm/Tiller into your Kubernetes Cluster with TLS](https://medium.com/google-cloud/installing-helm-in-google-kubernetes-engine-7f07f43c536e).

    $ git clone [https://github.com/jonbcampos/istio-series.git](https://github.com/jonbcampos/istio-series)
    $ cd [~/istio-series/getting-started/scripts](https://github.com/jonbcampos/istio-series/tree/master/getting-started/scripts)
    $ sh [startup.sh](https://github.com/jonbcampos/istio-series/blob/master/getting-started/scripts/startup.sh)
    $ sh [add-helm.sh](https://github.com/jonbcampos/istio-series/blob/master/getting-started/scripts/add-helm.sh)

If you have questions about how we setup these files I’ve included links to other articles that explain these steps along with links to the code in these files. Again, this isn’t the main purpose of this article so I’m trying to get to the new stuff quickly.

Now, for the new stuff…

## Download Istio

The first thing we need to do is download the Istio code into our environment and add Istio into our Command Line Environment. There are many ways to do this that I’ve seen across a variety of posts around the web. As always I’m going to show you the quickest, most direct way to make this happen. You can enter the following commands into Google Shell to download Istio.

    **$ export ISTIO_VERSION=1.0.0** # set the version we want
    **$ curl -L https://git.io/getLatestIstio | ISTIO_VERSION=${ISTIO_VERSION} sh -** # download Istio
    **$ cd istio-${ISTIO_VERSION}/** # change directory into Istio folder
    **$ export PATH=${PWD}/bin:$PATH** # add Istio to the path

With Istio downloaded and installed on your Path we can move to the next step and install Istio into our Kubernetes Cluster.

## Install Istio

There are [many ways that you can install Istio covered by the Istio documentation](https://istio.io/docs/setup/kubernetes/). I am focusing on using Helm for this installation because I feel it provides you the easiest way to install while also giving you the most features. On top of that, with Istio now hitting 1.0, Helm is the *recommended way* to install Istio in a Kubernetes Cluster by the Istio maintainers.

We will be able to do all of this with one line, however I am including some tweaks to the Helm install to include [Grafana](https://grafana.com/), [ServiceGraph](https://github.com/istio/istio/tree/release-1.0/addons/servicegraph), [Tracing](https://istio.io/docs/tasks/telemetry/distributed-tracing/), and [Kiali](https://github.com/kiali/kiali), along with [Mutual TLS Authentication](https://istio.io/docs/tasks/security/mutual-tls/). These values aren’t enabled by default but they will be required by most project’s you’ll come across. The following line will complete the install.

    **$ helm install install/kubernetes/helm/istio \** # use Helm Chart
        **--name istio \** # name install 'istio'
        **--tls \** # install using TLS
        **--namespace istio-system \** # set the namespace
        **--set global.mtls.enabled=true \** # enable MTLS
        **--set grafana.enabled=true \** # enable Grafana
        **--set servicegraph.enabled=true \** # enable ServiceGraph
        **--set tracing.enabled=true \** # enable Tracing
        **--set kiali.enabled=true** # enable Kiali

Once you run this the installation process will begin. There isn’t a log of logging that happens so you might think that nothing is happening. After about a minute or two you’ll see the process complete and then we can get to validating that everything is installed correctly.

There are [more settings you can play with as part of the Helm install](https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio). These are just some that I found were important to have. I recommend checking out the documentation to see all of the options you do have.
[**istio/istio**
*Connect, secure, control, and observe services. Contribute to istio/istio development by creating an account on GitHub.*github.com](https://github.com/istio/istio/tree/master/install/kubernetes/helm/istio)

## One Line Install

If you’re lazy and don’t like typing then I’ve created a bash script to handle the download and install for you.

    $ cd [~/istio-series/getting-started/scripts](https://github.com/jonbcampos/istio-series/tree/master/getting-started/scripts)
    $ sh [add-istio.sh](https://github.com/jonbcampos/istio-series/blob/master/getting-started/scripts/add-istio.sh)

With the download and install complete it is time to focus on validating everything works as intended.

## Confirm The Istio Installation Is Complete

It should be obvious that Istio is installed if you look at your Kubernetes Engine> Workloads view in your Google Cloud Project.

![All Of The Istio Parts In A GKE Cluster](https://cdn-images-1.medium.com/max/2000/1*tJzBjg7obDS_gTIJxeIvKg.png)*All Of The Istio Parts In A GKE Cluster*

However, to confirm that everything is complete we will install a sample project created by the Istio team to ensure everything is setup properly. To do this we can run a quick script to setup and install the sample project.

    $ sh [~/istio-series/getting-started/samples/launch-bookinfo.sh](https://github.com/jonbcampos/istio-series/blob/master/getting-started/samples/launch-bookinfo.sh)
    $ sh [~/istio-series/getting-started/samples/check-bookinfo-gateway.sh](https://github.com/jonbcampos/istio-series/blob/master/getting-started/samples/check-bookinfo-gateway.sh)

After you run the second script you should see output similar to the following.

    determine ingress host and port
    Gateway URL
    **http://[some IP Address]:80/productpage**
    confirm app is running
    200

If you hit the url that is provided you will see the running application, thus proving that everything was installed properly.

![The Book Info Sample Application Running](https://cdn-images-1.medium.com/max/2498/1*mbmq7qL5kz9omxKcE9MhlA.png)*The Book Info Sample Application Running*

## Conclusion

That’s it! A few minutes of work and you’ll have a Kubernetes Cluster with Istio running and ready for development. There is so so so much further to go and many other features to add, but for now we should be happy with this.

Shortly I’ll keep releasing posts that go through the various features built into Istio. Luckily there aren’t that many, but the ones that are there are powerful. Stay tuned!

## Teardown

Before you leave make sure to cleanup your project so you aren’t charged for the VMs that you’re using to run your cluster. Return to the Cloud Shell and run the teardown script to cleanup your project. This will delete your cluster and the containers that we’ve built.

    $ cd [~/istio-series/getting-started/scripts](https://github.com/jonbcampos/istio-series/tree/master/getting-started/scripts)
    $ sh [teardown.sh](https://github.com/jonbcampos/istio-series/blob/master/getting-started/scripts/teardown.sh)

## References To Other Posts In This Article
[**Installing Helm in Google Kubernetes Engine (GKE)**
*When I first started to really get into Kubernetes I would go find the docker image to various necessary programs and…*medium.com](https://medium.com/google-cloud/installing-helm-in-google-kubernetes-engine-7f07f43c536e)
[**Kubernetes: Day One**
*This is the obligatory step one Kubernetes post. If you’re interested in Kubernetes you’ve probably read 100 of these…*medium.com](https://medium.com/google-cloud/kubernetes-day-one-30a80b5dcb29)

Questions? Feedback? I’m very interested to hear what issues you might run across or if this helped you understand a bit better. If there is something I missed feel free to share that too. We are all in this together!

[Jonathan Campos](http://jonbcampos.com/) is an avid developer and fan of learning new things. I believe that we should always keep learning and growing and failing. I am always a supporter of the development community and always willing to help. So if you have questions or comments on this story please ad them below. Connect with me on [LinkedIn](https://www.linkedin.com/in/jonbcampos/) or [Twitter](https://twitter.com/jonbcampos) and mention this story.
