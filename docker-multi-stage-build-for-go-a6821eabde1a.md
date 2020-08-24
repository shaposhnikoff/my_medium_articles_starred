
# Docker multi-stage build for Go



In a modern software world, almost every application is containerized. In this short article, I am planning to give a quick glimpse about docker multi-stage build for Go applications.

As I have been learning these concepts for a while, I would like to share my experiments and the difficulties in which I have encountered with while learning to implement CI/CD flows and application deployments.

As the beginning of the article series about Docker deployments, CI/CD automation stage for application, I am going to give an example of deploying a simple web server, which is written with Go, with Docker. I am not going into details of developing a web server with Go though. The purpose is to give an example of how we can use Docker for our projects.

Building and deploying an application as a container drastically improves the performance of the microservice application. It is much easier to deploy and scalable services on production. These practical tools make our development&deployment processes less painful. We are coming closer to eradicate the common excuse of developers which is, “**it works on my machine” **:).

Alright then let's begin. First of all, we are going to create a simple web server with Go. Here we are simply starting the web server on 3030 port and defining the root handler for it.

It simply prints out the text as an output when the base URL is called.

<iframe src="https://medium.com/media/e2bc24885c2c565d76f76241a256dc66" frameborder=0></iframe>

Below lines show how to build and run the application’s executable file.
> # go build -o server.exe
> # run “server.exe”

Let us create the Dockerfile now. Here is where all magic happens.

1- First it pulls the latest golang image from docker hub.

2- Adding the files inside the working image. After that, we are changing our working directory inside this folder.

3- Creating a module file and downloading the dependencies.

4- Doing cross compile configurations and building the application. Then the application will be run inside the main folder.

5- This is where the second stage begins. The reason why I defined the second stage for this Dockerfile will be explained shortly.

6- Small alpine image is fetched from docker hub and the application’s executable file from the previous stage will be transferred inside the alpine image.

7- Giving permission for the executable file.

8- Defining an entrypoint for the container. When the container is started the entrypoint command will be called. Then port 3030 is being exposed.

<iframe src="https://medium.com/media/cb3c0931cf4985669089839072d1fb41" frameborder=0></iframe>

Here you go run the following command and you will have an up and running container.
> # docker build -t server.

The output will be like below.

![the output of docker build process.](https://cdn-images-1.medium.com/max/2946/1*r_fV_KbF4j9sl943x9OTqw.png)*the output of docker build process.*

If a single image is used for build stage, in the end, we will have a huge image file. This is not considered a practical approach. Our images must be as small as possible. Otherwise, it does not make sense of using Docker for application deployments. Because no one wants to deploy huge containers on the server.

However, the solution exists to solve this problem which is called **Multi-stage builds. **By defining an additional build stage in Dockerfile, it helps us to employ the feature of creating a small sized container.

As you can see from the below picture our image is 18 MB which is quite small. If you look at the size of golang image which is 775 MB, you can see the difference. A subtle change in the Dockerfile affects the total size of the image.

Mostly, a small-sized alpine image is preferred for the last stage. It is 4 MB — more or less. The alpine container has a sole purpose, that is to store the application’s executable file.

**Building application using golang image. Then copying and running the container inside a small-sized alpine image.**

![List of images. go_server is the image of our example. the golang image is used for first server. the alpine image is used for second stage.](https://cdn-images-1.medium.com/max/2724/1*F-ZAWOjl7YdMRL4OtZce6g.png)*List of images. go_server is the image of our example. the golang image is used for first server. the alpine image is used for second stage.*

**Running Docker container**

In order to run the container from the image file, the below command can be executed. Then call localhost:3030.
> # docker run -p 3030:3030 -t server

![Running the container](https://cdn-images-1.medium.com/max/2366/1*iBeOAOOPt0SglvJIxygRqA.png)*Running the container*

![running containers. the exposed port can be seen.](https://cdn-images-1.medium.com/max/4382/1*B7n0ptWAuuxssEJvcv2GtA.png)*running containers. the exposed port can be seen.*

Finally, we have a single up and running container. For the upcoming article, I am planning to write about creating a docker compose file for combining multiple services.

I have a couple of examples on my GitHub page.
[**Broke116/go_rest_api**
*Intention of this project is to create a Jenkins pipeline for GO rest API project. API is being deployed on…*github.com](https://github.com/Broke116/go_rest_api)

**Feedbacks are welcomed :)**

[02-Preparing docker compose file](https://medium.com/@ekiny018/preparing-docker-compose-file-for-applications-b7ee7d48e74c)

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and join our [Facebook Group](https://www.facebook.com/groups/364904580892967/) **💬**.**

**To join our community Slack **🗣️ **and read our weekly Faun topics **🗞️,** click here⬇**

![](https://cdn-images-1.medium.com/max/3200/0*oSdFkACJxs5iy1oR)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
