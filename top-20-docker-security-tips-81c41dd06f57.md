Unknown markup type 10 { type: [33m10[39m, start: [33m154[39m, end: [33m164[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m169[39m, end: [33m178[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m15[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m71[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m50[39m, end: [33m60[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m65[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m135[39m, end: [33m149[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m184[39m, end: [33m193[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m94[39m, end: [33m98[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m99[39m, end: [33m141[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m90[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m99[39m, end: [33m101[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m29[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m144[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m92[39m, end: [33m106[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m110[39m, end: [33m114[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m187[39m, end: [33m201[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m296[39m, end: [33m300[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m39[39m, end: [33m63[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m54[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m91[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m67[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m190[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m33[39m, end: [33m69[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m30[39m, end: [33m43[39m }

# Top 20 Docker Security Tips

AIMing for safety!

This article is full of tips to help you use Docker safely. If you’re new to Docker I suggest you first check out my previous articles on Docker [concepts](https://towardsdatascience.com/learn-enough-docker-to-be-useful-b7ba70caeb4b), the Docker [ecosystem](https://towardsdatascience.com/learn-enough-docker-to-be-useful-1c40ea269fa8), [Dockerfiles](https://towardsdatascience.com/learn-enough-docker-to-be-useful-b0b44222eef5), [slimming down images](https://towardsdatascience.com/slimming-down-your-docker-images-275f0ca9337e), [popular commands](https://towardsdatascience.com/learn-enough-docker-to-be-useful-b0b44222eef5), and [data in Docker](https://towardsdatascience.com/pump-up-the-volumes-data-in-docker-a21950a8cd8?source=friends_link&sk=87c74aa7fefeaacd8509e24d7c432b91).

![](https://cdn-images-1.medium.com/max/2560/1*Ti9kLDnemNDjwfADTBlzig.jpeg)

How concerned do you need to be about security in Docker? It depends. Docker comes with sensible security features baked in. If you are using official Docker images and not communicating with other machines, you don’t have much to worry about.

However, if you’re using unofficial images, serving files, or running apps in production, then the story is different. In those cases you need to be considerably more knowledgeable about Docker security.

![Looks safe](https://cdn-images-1.medium.com/max/NaN/1*kOT_4Wj5lx6UR0UO3oO3pw.png)*Looks safe*

Your primary security goal is to prevent a malicious user from gaining valuable information or wreaking havoc. Toward that end, I’ll share Docker security best practices in several key areas. By the end of this article you’ll have seen over 20 Docker security tips! 😃

We’ll focus on three areas in the first section:

* **A**ccess management

* **I**mage safety

* **M**anagement of secrets

Think of the acronym **AIM** to help you remember them.

First, let’s look at limiting a container’s access.

## Access Management — Limit Privileges

When you start a container, Docker creates a group of namespaces. Namespaces prevent processes in a container from seeing or affecting processes in the host, including other containers. Namespaces are a primary way Docker cordons off one container from another.

Docker provides private container networking, too. This prevents a container from gaining privileged access to the network interfaces of other containers on the same host.

So a Docker environment comes somewhat isolated, but it might not be isolated enough for your use case.

![Does not look safe](https://cdn-images-1.medium.com/max/NaN/1*sdxIs0ExwFil2G2XaVMsdw.jpeg)*Does not look safe*

Good security means following the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege). Your container should have the abilities to do what it needs, but no more abilities beyond those. The tricky thing is that once you start limiting what processes can be run in a container, the container might not be able to do something it legitimately needs to do.

There are several ways to adjust a container’s privileges. First, avoid running as root (or re-map if must run as root). Second, adjust capabilities with --cap-drop and --cap-add.

Avoiding root and adjusting capabilities should be all most folks need to do to restrict privileges. More advanced users might want to adjust the default AppArmor and seccomp profiles. I discuss these in my forthcoming book about Docker, but have excluded them here to keep this article from ballooning. 🎈

### Avoid running as root

Docker’s default setting is for the user in an image to run as root. Many people don’t realize how dangerous this is. It means it’s far easier for an attacker to gain access to sensitive information and your kernel.

As a general best practice, don’t let a container run as root.

![Roots](https://cdn-images-1.medium.com/max/NaN/1*5088gV23KBQqYNYS8ttDtA.jpeg)*Roots*

“The best way to prevent privilege-escalation attacks from within a container is to configure your container’s applications to run as unprivileged users.” — the Docker [Docs](https://docs.docker.com/engine/security/userns-remap/).

You can specify a userid other than root at build time like this:

docker run -u 1000 my_image

The -- user or -u flag, can specify either a username or a userid. It's fine if the userid doesn't exist.

In the example above *1000* is is an arbitrary, unprivileged userid. In Linux, userids between 0 and 499 are generally reserved. Choose a userid over 500 to avoid running as a default system user.

Rather than set the user from the command line, it’s best to change the user from root in your image. Then folks don’t have to remember to change it at build time. Just include the USER Dockerfile instruction in your image after Dockerfile instructions that require the capabilities that come with root.

In other words, first install the packages you need and then switch the user. For example:

    FROM alpine:latest
    RUN apk update && apk add --no-cache git
    USER 1000
    …

If you must run a processes in the container as a root user, re-map the root to a less-privileged user on the Docker host. See the [Docker docs](https://docs.docker.com/engine/security/userns-remap/).

You can grant the privileges the user needs by altering the capabilities.

### Capabilities

Capabilities are bundles of allowed processes.

Adjust capabilities through the command line with --cap-drop and --cap-add. A best policy is to drop all a container's privileges with --cap-drop all and add back the ones needed with --cap-add.

![Stop or go](https://cdn-images-1.medium.com/max/NaN/1*BJYdZujv_cmCMuRpwyuKDw.jpeg)*Stop or go*

You can adjust a container’s capabilities at runtime. For example, to drop the ability to use kill to stop a container, you can remove that default capability like this:

docker run --cap-drop=Kill my_image

Avoid giving SYS_ADMIN and SETUID privileges to processes, as they are give broad swaths of power. Adding this capabilities to a user is similar to giving root permissions (and avoiding that outcome is kind of the whole point of not using root).

It’s safer to not allow a container to use a port number between 1 and 1023 because most network services run in this range. An unauthorized user could listen in on things like logins and run unauthorized server applications. These lower numbered ports require running as root or being explicitly given the CAP_NET_BIND_SERVICE capability.

To find out things like whether a container has privileged port access, you can use *inspect*. Using docker container inspect my_container_name will show you lots of details about the allocated resources and security profile of your container.

[Here’s the Docker reference](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) for more on privileges.

As with most things in Docker, it’s better to configure containers in an automatic, self-documenting file. With Docker Compose you can specify capabilities in a service configuration like this:

cap_drop: ALL

Or you can adjust them in Kubernetes files as discussed [here](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).

The full list of Linux capabilities is [here](http://man7.org/linux/man-pages/man7/capabilities.7.html).

If you want more fine grained control over container privileges, check out my discussion of AppArmor and seccomp in my forthcoming book. Subscribe to [my email newsletter](http://eepurl.com/gjfLAz) to be notified when it’s available.

![Closed road](https://cdn-images-1.medium.com/max/NaN/1*GkbiVqPQQgMmmvoTXxC6Lw.jpeg)*Closed road*

## Access Management — Restrict Resources

It’s a good idea to restrict a container’s access to system resources such as memory and CPU. Without a resource limit, a container can use up all available memory. If that happens the Linux host kernel will throw an Out of Memory Exception and kill kernel processes. This can lead the whole system to crash. You can imagine how attackers could use this knowledge to try to bring down apps.

If you have multiple containers running on the same machine it’s smart to limit the memory and CPU any one container can use. If your container runs out of memory, then it shut downs. Shutting down your container can cause your app to crash, which isn’t fun. However, this isolation protects the host from running out of memory and all the containers on it from crashing. And that’s a good thing.

![Wind resource](https://cdn-images-1.medium.com/max/NaN/1*Odgf25AAaLqi-KXUyfHMSw.jpeg)*Wind resource*

Docker Desktop CE for Mac v2.1.0 has default resource restrictions. You can access them under the Docker icon -> Preferences. Then click on the *Resources* tab. You can use the sliders to adjust the resource constraints.

![Resource settings on Mac](https://cdn-images-1.medium.com/max/NaN/1*fNdy8Sx6bSRhyEM5NteTDg.png)*Resource settings on Mac*

Alternatively, you can restrict resources from the command line by specifying the --memory flag or -m for short, followed by a number and a unit of measure.

4m means 4 mebibytes, and is the minimum container memory allocation. A mebibyte (MiB) is slightly more than a megabyte (1 MiB = 1.048576 MB). The docs are currently incorrect, but hopefully the maintainers will have accepted my PR to change it by the time you read this.

To see what resources your containers are using, enter the command [docker stats](https://deploy-preview-9237--docsdocker.netlify.com/config/containers/runmetrics/) in a new terminal window. You'll see running container statistics regularly refreshed.

![Stats](https://cdn-images-1.medium.com/max/NaN/1*y2rxG1CvFG1PsYIfzIHXSQ.png)*Stats*

Behind the scenes, Docker is using Linux Control Groups (cgroups) to implement resource limits. This technology is battle tested.

Learn more about resource constraints on Docker [here](https://docs.docker.com/config/containers/resource_constraints/).

## Image safety

Grabbing an image from Docker Hub is like inviting someone into your home. You might want to be intentional about it.

![Someone’s home](https://cdn-images-1.medium.com/max/NaN/1*hYp7NMZkmbL72eur9MBLwA.jpeg)*Someone’s home*

### Use trustworthy images

Rule one of image safety is to only use images you trust. How do you know which images are trustworthy?

It’s a good bet that popular official images are relatively safe. Such images include alpine, ubuntu, python, golang, redis, busybox, and node. Each has over [10M downloads](https://hub.docker.com/search?q=&type=image&page=1) and lots of eyes on them. 🔒

[Docker explains](https://blog.docker.com/2019/02/docker-security-update-cve-2018-5736-and-container-security-best-practices/):
> Docker sponsors a dedicated team that is responsible for reviewing and publishing all content in the Official Images. This team works in collaboration with upstream software maintainers, security experts, and the broader Docker community to ensure the security of these images.

### Reduce your attack surface

Related to using official base images, you can use a minimal base image.

With less code inside, there’s a lower chance for security vulnerabilities. A smaller, less complicated base image is more transparent.

It’s a lot easier to see what’s going on in an Alpine image than your friend’s image that relies on her friend’s image that relies on another base image. A short thread is easier to untangle.

![Tangled](https://cdn-images-1.medium.com/max/NaN/1*ixifAq0FGHI3ZkaTuufFLw.jpeg)*Tangled*

Similar, only install packages you actually need. This reduces your attack surface and speeds up your image downloads and image builds.

### Require signed images

You can ensure that images are signed by using Docker content trust. 🔏

Docker content trust prevents users from working with tagged images unless they contain a signature. Trusted sources include Official Docker Images from Docker Hub and signed images from user trusted sources.

![Signed](https://cdn-images-1.medium.com/max/NaN/1*s3dMsYD9OjSPQMQmw9Ndfw.jpeg)*Signed*

Content trust is disabled by default. To enable it, set the DOCKER_CONTENT_TRUST environment variable to 1. From the command line, run the following:

export DOCKER_CONTENT_TRUST=1

Now when I try to pull down my own unsigned image from Docker Hub it is blocked.

Error: remote trust data does not exist for docker.io/discdiver/frames: notary.docker.io does not have trust data for docker.io/discdiver/frames

Content trust is a way to keep the riffraff out. Learn more about content trust [here](https://docs.docker.com/engine/security/trust/content_trust/).

Docker stores and accesses images by the cryptographic checksum of their contents. This prevents attackers from creating image collisions. That’s a cool built-in safety feature.

## Managing Secrets

Your access is restricted, your images are secure, now it’s time to manage your secrets.”

Rule 1 of managing sensitive information: do not bake it into your image. It’s not too tricky to find your unencrypted sensitive info in code repositories, logs, and elsewhere.

Rule 2: don’t use environment variables for your sensitive info, either. Anyone who can run docker inspect or exec into the container can find your secret. So can anyone running as root. Hopefully we've configured things so that users won't be running as root, but redundancy is part of good security. Often logs will dump the environment variable values, too. You don't want your sensitive info spilling out to just anyone.

Docker volumes are better. They are the recommended way to access your sensitive info in the Docker docs. You can use a volume as temporary file system held in memory. Volumes remove the docker inspect and the logging risk. However, root users could still see the secret, as could anyone who can exec into the container. Overall, volumes are a pretty good solution.

Even better than volumes, use Docker secrets. Secrets are encrypted.

![Secrets](https://cdn-images-1.medium.com/max/NaN/1*-OAd39UZ-4e7DuGSBNEGpw.jpeg)*Secrets*

Some [Docker docs](https://docs.docker.com/engine/swarm/secrets/) state that you can use secrets with Docker Swarm only. Nevertheless, you can use secrets in Docker without Swarm.

If you just need the secret in your image, you can use BuildKit. BuildKit is a better backend than the current build tool for building Docker images. It cuts build time significantly and has other nice features, including build-time secrets support.

BuildKit is relatively new — Docker Engine 18.09 was the first version shipped with BuildKit support. There are three ways to specify the BuildKit backend so you can use its features now. In the future, it will be the default backend.

1. Set it as an environment variable with export DOCKER_BUILDKIT=1.

1. Start your build or run command with DOCKER_BUILDKIT=1.

1. Enable BuildKit by default. Set the configuration in /*etc/docker/daemon.json* to *true* with: { "features": { "buildkit": true } }. Then restart Docker.

1. Then you can use secrets at build time with the --secret flag like this:

docker build --secret my_key=my_value ,src=path/to/my_secret_file .

Where your file specifies your secrets as key-value pair.

These secrets are not stored in the final image. They are also excluded from the image build cache. Safety first!

If you need your secret in your running container, and not just when building your image, use Docker Compose or Kubernetes.

With Docker Compose, add the secrets key-value pair to a service and specify the secret file. Hat tip to [Stack Exchange answer](https://serverfault.com/a/936262/535325) for the Docker Compose secrets tip that the example below is adapted from.

Example docker-compose.yml with secrets:

    version: "3.7"
    
    services:
    
      my_service:
        image: centos:7
        entrypoint: "cat /run/secrets/my_secret"
        secrets:
          - my_secret
    
    secrets:
      my_secret:
        file: ./my_secret_file.txt

Then start Compose as usual with docker-compose up --build my_service.

If you’re using [Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/), it has support for secrets. [Helm-Secrets](https://github.com/futuresimple/helm-secrets) can help make secrets management in K8s easier. Additionally, K8s has Role Based Access Controls (RBAC) — as does Docker Enterprise. RBAC makes access Secrets management more manageable and more secure for teams.

A best practice with secrets is to use a secrets management service such as Vault. [Vault](https://www.vaultproject.io/) is a service by HashiCorp for managing access to secrets. It also time-limits secrets. More info on Vault’s Docker image can be found [here](https://hub.docker.com//vault).

[AWS Secrets Manager](https://aws.amazon.com/secrets-manager/) and similar products from other cloud providers can also help you manage your secrets on the cloud.

![Keys](https://cdn-images-1.medium.com/max/NaN/1*-L9oPjWgRBGBBoBJsvZQtA.jpeg)*Keys*

Just remember, the key to managing your secrets is to keep them secret. Definitely don’t bake them into your image or turn them into environment variables.

## Update Things

As with any code, keep your the languages and libraries in your images up to date to benefit from the latest security fixes.

![Hopefully your security is more up to date than this lock](https://cdn-images-1.medium.com/max/NaN/1*SvzouB47pMF_0Hu6u0IO9A.jpeg)*Hopefully your security is more up to date than this lock*

If you refer to a specific version of a base image in your image, make sure you keep it up to date, too.

Relatedly, you should keep your version of Docker up to date for bug fixes and enhancements that will allow you to implement new security features.

Finally, keep your host server software up to date. If you’re running on a managed service, this should be done for you.

Better security means keeping things updated.

## Consider Docker Enterprise

If you have an organization with a bunch of people and a bunch of Docker containers, it’s a good bet you’d benefit from Docker Enterprise. Administrators can set policy restrictions for all users. The provided RBAC, monitoring, and logging capabilities are likely to make security management easier for your team.

With Enterprise you can also host your own images privately in a [Docker Trusted Registry](https://docs.docker.com/ee/dtr/). Docker provides built-in security scanning to make sure you don’t have known vulnerabilities in your images.

Kubernetes provides some of this functionality for free, but Docker Enterprise has additional security capabilities for containers and images. Best of all, [Docker Enterprise 3.0](https://blog.docker.com/2019/07/announcing-docker-enterprise-3-0-ga/) was released in July 2019. It includes Docker Kubernetes Service with “sensible security defaults”.

## Additional Tips

* Don’t ever run a container as -- privileged unless you need to for a special circumstance like needing to run Docker inside a Docker container — and you know what you're doing.

* In your Dockerfile, favor COPY instead of ADD. ADD automatically extracts zipped files and can copy files from URLs. COPY doesn’t have these capabilities. Whenever possible, avoid using ADD so you aren’t susceptible to attacks through remote URLs and Zip files.

* If you run any other processes on the same server, run them in Docker containers.

* If you use a web server and API to create containers, check parameters carefully so new containers you don’t want can’t be created.

* If you expose a REST API, secure API endpoints with HTTPS or SSH.

* Consider a checkup with [Docker Bench for Security](https://github.com/docker/docker-bench-security) to see how well your containers follow their security guidelines.

* Store sensitive data only in volumes, never in a container.

* If using a single-host app with networking, don’t use the default bridge network. It has technical shortcomings and is not recommended for production use. If you publish a port, all containers on the bridge network become accessible.

* Use Lets Encrypt for HTTPS certificates for serving. See an example with NGINX [here](https://medium.com/@pentacent/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71).

* Mount volumes as read-only when you only need to read from them. See several ways to do this [here](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Docker_Security_Cheat_Sheet.md).

## Summary

You’ve seen many of ways to make your Docker containers safer. Security is not set-it and forget it. It requires vigilance to keep your images and containers secure.

![Keys](https://cdn-images-1.medium.com/max/NaN/1*AGYrfC2NpGXLYe6xtiiLng.png)*Keys*

### When thinking about security, remember AIM

1. **A**ccess management

* Avoid running as root. Remap if must use root.

* Drop all capabilities and add back those that are needed.

* Dig into AppArmor if you need fine-grained privilege tuning.

* Restrict resources.

2. **I**mage safety

* Use official, popular, minimal base images.

* Don’t install things you don’t need.

* Require images to be signed.

* Keep Docker, Docker images, and other software that touches Docker updated.

3. **M**anagement of secrets

* Use secrets or volumes.

* Consider a secrets manager such as Vault.

![Bullseye!](https://cdn-images-1.medium.com/max/2000/1*HoEJRUtVcjExCCS7WDBSRA.jpeg)*Bullseye!*

Keeping Docker containers secure means AIMing for safety.

Don’t forget to keep Docker, your languages and libraries, your images, and your host software updated. Finally, consider using Docker Enterprise if you’re running Docker as part of a team.

I hope you found this Docker security article helpful. If you did, please share it with others on your favorite forums or social media channels so your friends can find it, too! 👏

Do you have other Docker security suggestions? If so, please share them in the comments or on[ Twitter @discdiver](https://twitter.com/discdiver). 👍

If you’re interested in being notified when my *Memorable Docker* book with even more Docker goodness is released, sign up for my [mailing list](https://dataawesome.com).

![](https://cdn-images-1.medium.com/max/NaN/1*oPkqiu1rrt-hC_lDMK-jQg.png)

I write about articles about Python, data science, AI, and other tech topics. Check them out follow [me](https://medium.com/@jeffhale) on Medium if you’re into that stuff.

Happy securing! 😃
