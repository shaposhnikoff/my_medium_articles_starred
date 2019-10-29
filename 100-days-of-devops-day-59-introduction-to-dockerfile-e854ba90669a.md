Unknown markup type 10 { type: [33m10[39m, start: [33m2[39m, end: [33m12[39m }

# 100 Days of DevOps — Day 59- Introduction to DockerFile

Welcome to Day 59 of 100 Days of DevOps, Focus for today is DockerFile
> *What is DockerFile?*

* *A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image*

* *Basic Dockerfile look like this*

    *#My first test docker file*

    *FROM centos:7*

    *LABEL maintainer="plakhera@example.com"*

    *RUN yum -y install httpd*

* *When we build a dockerfile it should name as Dockerfile(case-sensitive)*

* *FROM specify from where we building this image*

* *LABEL refer to the person who is maintaining/creating this image(earlier maintainer option is deprecated and it’s label now)*

* *RUN refer to the command we are running*

* *Now we are going to build in the context(. indicate the current directory)*

    *# docker build -t myfirstimage:v1 .*

    *Sending build context to Docker daemon  2.048kB*

    *Step 1/3 : FROM centos:7*

    *---> e934aafc2206*

    *Step 2/3 : LABEL maintainer="plakhera@example.com"*

    *---> a4f7e2fbcc92*

    *Step 3/3 : RUN yum -y install httpd*

    *---> Running in 57199dabb5ec*

* *Here step refers to the layer*

* *To check the image(Name of the image, is the result of the tag we passed during docker build command)*

    *# docker images*

    *REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE*

    *myfirstimage        v1                  2b72d79436b1        2 minutes ago       334MB*

* *If we run this command again, it’s going to use the cache(if the instruction inside the dockerfile is not changed)*

    *# docker build -t myfirstimage:v1 .*

    *Sending build context to Docker daemon  2.048kB*

    *Step 1/3 : FROM centos:7*

    *---> e934aafc2206*

    *Step 2/3 : LABEL maintainer="plakhera@example.com"*

    *---> Using cache*

    *---> a4f7e2fbcc92*

    *Step 3/3 : RUN yum -y install httpd*

    *---> Using cache*

    *---> 2b72d79436b1*

    *Successfully built 2b72d79436b1*

    *Successfully tagged myfirstimage:v1*

* *Now what would be the case if I change any instruction inside Dockerfile(here I changed maintainer from plakhera to plakhera1*

    *FROM centos:7*

    *label maintainer="plakhera1@example.com"*

    *RUN yum -y install httpd*

* *When I run the docker build again, as you can see as the instruction inside Dockerfile changed, it's trying to build that layer again and all the subsequent layers*

    *$ docker build -t mynewapache:v2 .*

    *Sending build context to Docker daemon  2.048kB*

    *Step 1/3 : FROM centos:7*

    *---> e934aafc2206*

    *Step 2/3 : label maintainer="plakhera1@example.com"*

    ***---> Running in 622b311b5f37***

    ***Removing intermediate container 622b311b5f37***

    ***---> c61c698e355a***

    *Step 3/3 : RUN yum -y install httpd*

    *---> Running in f8f239688e77*

    *Loaded plugins: fastestmirror, ovl*

* *We can check the image again*

    *$ docker images*

    *REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE*

    *mynewapache         v2                  8dcd8ccceb87        11 minutes ago      334MB*

* *If we don’t want to use the cache as well as pull from dockerhub*

    *$ docker build --pull --no-cache -t mynewapache:v3 .*

    *Sending build context to Docker daemon  2.048kB*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
