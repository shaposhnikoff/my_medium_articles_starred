
# Introduction to DockerFile

Used to create an image

*Basic Dockerfile look like this*

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

* *Here step refer to the layer*

* *To check the image(Name of the image, is the result of tag we passed during docker build command)*

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

* *When I run the docker build again, as you can see as the instruction inside Dockerfile changed, its trying to build that layer again and all the subsequent layers*

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

* *If we don’t want to use cache as well as pull from dockerhub*

    *$ docker build --pull --no-cache -t mynewapache:v3 .*

    *Sending build context to Docker daemon  2.048kB*
