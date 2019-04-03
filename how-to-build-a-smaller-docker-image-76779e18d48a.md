
# How to build a smaller Docker image

How to build a smaller Docker image

### When you’re building a Docker image it’s important to keep the size under control. Having small images means ensuring faster deployment and transfers.

The first thing to consider when working with containers is to have a clear understanding of what the application needs. In fact, for each service, you should know what data needs to be persistent. It’s also very important to know how each service will communicate with each other through the network.

You have to consider the language, framework, tools and third-party packages you need. Everything that is added to the image increases its size.

This way of thinking helps you develop more secure and solid microservices.
> # Proper planning prevents poor performance

### Find the right balance with the cache layers

Docker containers are building blocks for applications. A docker image describes the instructions for running a container. Each of these instructions, contained in the Dockerfile, create a layer. The set of these layers is useful to speed up the build operations and the image transfer. This saves a lot of disk space when many images share the same layers.

<iframe src="https://medium.com/media/21e358b00c9190aed0d174ff001d2789" frameborder=0></iframe>

Each statement, like RUN, creates a new layer. In the previous example, at the top of the layers from the Debian image, Docker creates nine extra layers.

Once the image is built, you can view all the layers created with the command docker history <image>.

![](https://cdn-images-1.medium.com/max/2212/1*QOgpDI6tXGZoGH3hMHMj_g.png)

This command shows the different layers and their sizes. 
With our Dockerfile we created the first nine levels, the other two come from the Debian image.

Layers are like Git commits, each level stores the difference between the previous and current versions of the image. And like Git commits are useful if you share them with other branches or, in the Docker world, with other images.

When you pull an image from the registry you download only the layers that you don’t already have in cache. This is a very powerful feature that allows you to speed up the download/upload of images. 
In addition, the build system will only build the modified layers while it’ll take the others directly from the cache.
> Be careful to extend the images using the *ONBUILD* statement.** **When this statement is present and its layer is invalidated the system will force any derived image to rebuild itself. This is because any *ONBUILD* declaration will be a layer in the derived image above the others.

**But layers aren’t free.**

Layers use space and as the levels increase, the final image size also increases. This is because the system keeps all the changes between the various statements.

It may be a good practice to combine the various instructions in a single line. But beware, it’s important to design the Dockerfile to use the layers cache system as much as possible.

<iframe src="https://medium.com/media/5336041e5d09ee2c54423df1f4831d77" frameborder=0></iframe>

The example above creates just two layers on top of the Debian image instead of nine.

As you can see the last commands used are apt-get autoremove and apt-get clean. It’s very important to delete temporary files that do not serve the purpose of the image, such as the package manager’s cache. These files increase the size of the final image without giving any advantage. 
In Alpine OS, you can use apk --no-cache add <package> when installing a package to bypass the use of the cache.

![](https://cdn-images-1.medium.com/max/2212/1*dDaVb16dG8uJ0nsmY0zS8g.png)

Note that although the image is different, it uses the same three base layers as the previous one.

This way, the image size is reduced but, as the complexity of the service grows, the file risks becoming unreadable.

### Use --squash flag on build

The squash flag is an experimental feature. It allows you to merge the new layers into one layer during the build time. To use it just add the flag to the build command: docker build --squash -t <image> .

You can use it by activating the experimental features in the Docker settings. You can find more details about it in [the documentation](https://docs.docker.com/engine/reference/commandline/dockerd/#description).

Let’s try rebuilding the Dockerfile of the first example with the --squash flag.

![](https://cdn-images-1.medium.com/max/2212/1*_x6JEozpDQaOqqZy3TGU_Q.png)

Docker merges the various levels in a single one, keeping a more maintainable Dockerfile. This method also allows you to take full advantage of the cache in your development environment. Then you can still optimize the build before pushing it.

### Use .dockerignore files

When you run docker build command the first output line is:

    Sending build context to Docker daemon 2.048kB
    Step 1/9 : FROM ...

To build a new image the Docker daemon needs to access the files needed to create the image. So, every time you run the docker build command, the Docker CLI packs all build context files into a tar archive and sends it to the daemon.
> The docker build command can accept an already created tar archive or the URL of a Git repository.

This causes an expensive waste of time, as well as an increase in the size of the final image. As we know, **the size and the build time of an image in a microservices continuous delivery flow are critical.**

You can define the Docker build context you need using the .dockerignore file. This has a syntax like the .gitignore file used by the Git tool. Check [the documentation](https://docs.docker.com/engine/reference/builder/#dockerignore-file) for more details.

<iframe src="https://medium.com/media/82157666b4059ed863e7b86f5fa69763" frameborder=0></iframe>

It can also be useful to avoid exposing secrets ​​such as a .env file, all the Git history or any other sensitive data.

A common pattern is to inject all the service’s codebase into an image with an instruction like this:

    COPY . /var/www

This way, we’re copying the **entire** **build context** into the image and, of course, we’re creating a new cache layer. So, if any of those files changes, the layer will invalidate.

Files that are frequently updated in the build context (logs, cache files, Git history, etc.) will regenerate the layer for every docker build. You must avoid adding these files to the context. If you still need these files, put them on a layer at the end of the file, this way you’ll avoid invalidating other subsequent layers.

### Use the multi-stage builds feature

Another aspect to consider is the size of build dependencies. Many of these, installed in our Dockerfile, are needed during build time but will be useless during runtime. Of course, these will greatly affect the final size of the image

A simple test can be performed on a Dockerfile that contains a small service with only an API that will return “*Hello world!*” after being called.

Let’s see a small implementation of this type in Golang.

<iframe src="https://medium.com/media/78be5632ff2a2669d674355d7258a75a" frameborder=0></iframe>

And here’s the Dockerfile.

<iframe src="https://medium.com/media/311109ffad7c240a1155983ab46e8c42" frameborder=0></iframe>

After building the image we can check its size with the following command:

    docker images <image> --format "{{.Repository}}:{{.Tag}} {{.Size}}"

**The result is: 823MB. It’s a huge number for a simple API service with a single endpoint.**

This is caused by the operating system and the Go tools present in the base image. All these are useful tools in the build stage but not during runtime. Although the built image works well, we can certainly resize it down.

The solution would be to run go build in the Golang image and then move the binary to a plain Debian* *image that does not contain all the Go tools. The second image is the image which we will use to run our service.

Let’s try the multi-stage Docker build.

<iframe src="https://medium.com/media/77d36f7e7cec0aeda8a5b146ee59d445" frameborder=0></iframe>

With multi-stage builds, you use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base image, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image.
> *When using multi-stage builds, you are not limited to copying from stages you created earlier in your Dockerfile. You can use the COPY --from instruction to copy from a separate image, either using the local image name, a tag available locally or on a Docker registry, or a tag ID. The Docker client pulls the image if necessary and copies the artifact from there.*
Reference: [https://docs.docker.com/develop/develop-images/multistage-build/#use-an-external-image-as-a-stage](https://docs.docker.com/develop/develop-images/multistage-build/#use-an-external-image-as-a-stage)

With this method, the final image size has become: 107MB. The dimensions are almost reduced by 8 times but the result is not yet enough. We can still improve it.

After removing the build tools from runtime, we can now focus on optimising the operating system. This is the other aspect that increases the size of the final image.

### Choose the right base image

The current image is based on Debian and includes many binaries. Docker containers should include a single process and contain the minimum required to run it. You do not need an entire operating system.

You can replace the Golang and the Debian base image with an Alpine-based image.

<iframe src="https://medium.com/media/74d64c4ef3ccf10cf8785821100020c9" frameborder=0></iframe>
> Alpine Linux is built around musl, libc and busybox. This makes it smaller and more resource efficient than traditional GNU/Linux distributions.

In other words, a smaller and more secure Linux distribution.

Alpine images are based on musl an alternative standard library for C. However, most Linux distributions such as Debian and CentOS are based on glibc. The two libraries should implement the same interface with the kernel but they have different goals:

* glibc is the most common and faster;

* musl uses less space and is written with a focus on security.

When an application is compiled, it’s compiled mostly on a specific C library. If you wish to use them with another C library, you must recompile them. So, building images with Alpine can lead to unexpected behaviour because the standard C library is different. Fortunately, most of the commonly used software stacks have their libraries already compiled for Alpine, thus it is easy to find Alpine based images on Docker hub.

If you’re having these problems, I suggest you take a look at the [Google Distroless project](https://github.com/GoogleContainerTools/distroless).

With Alpine, our image has reached the size of: 10.8MB. This is a great result, the image has been reduced about 76 times compared to the initial one.

### Let’s create the smallest image possible

Each binary file that is added to a Docker image adds a certain amount of risk to the overall application. It’s possible to reduce the risk by having only one binary installed in the container.

The goal is to isolate our process removing anything we don’t need.

The scratch image is the most minimal image in Docker. This is the base ancestor for all other images and is mostly used for building other base images. The scratch image is actually empty, it doesn't contain any folders or files. Using this base image, the first cache layer will match the first statement in the Dockerfile. It isn’t always possible to use it. It depends on what are the minimum requirements necessary to execute the software.

Golang is special because, disabling cgo, it can create a statically linked binary that fully contains the application and its dependencies. Other languages can do it, but not all.

Let’s see how our Dockerfile evolved.

<iframe src="https://medium.com/media/3f37fcdf9d67c2906b3ae75a07a0319a" frameborder=0></iframe>

Our final image is: 6.5MB. It’s reduced about 126 times and contains only our executable, good job!

### In conclusion

There are many things to consider, below are the key points:

* Think about your application’s needs;

* Develop the Dockerfile in logically separated blocks but compact it in the final version;

* Don’t install a package if you can avoid it;

* Don’t add files to the build context if you don’t need them;

* Don’t create files if you don’t have to, use streams and pipes as much as possible;

* Remove unnecessary files, such as packages cache files;

* If you have to add a file, install a dependency or extend an image, use the smallest one possible.

Consider that, in the various examples above, we haven’t taken into account various security best practices like* “never run a [process as root](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) in a container”*. This wasn’t the purpose of this article.

Despite this I’ll give you a quick tip; Tools like *“[Docker bench security](https://github.com/docker/docker-bench-security)”* can help you highlight any security issues in your images. In the enterprise environment, you can use *“[Docker security scanning](https://docs.docker.com/datacenter/dtr/2.5/guides/admin/configure/set-up-vulnerability-scans/)”* feature.

You can find more details about the benchmarks and the Dockerfiles tested in the following repository:
[**Docker images size benchmark**
*Comparison between various images containing the same Golang application.*github.com](https://github.com/gadiener/docker-images-size-benchmark)

### Check out the documentation for more details

* [Docker overview](https://docs.docker.com/engine/docker-overview/)

* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

* [Use multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)

* [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

***Thanks for reading and add any other tips or tricks that you use below in the comments!***
