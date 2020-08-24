Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m186[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m115[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m997[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m615[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m35[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m45[39m }

# Run Multiple Services In Single Docker Container Using Supervisor



**Have you ever faced this scenario where you want to run two or more lightweight services within the same container?**

Although Docker provides a Docker-compose tool for building and running multi-services applications in multiple containers. Docker-compose requires a YAML file to configure your multiple services. But sometimes we want to run two or more lightweight services inside the same container.

In this article I will explain how we can start multiple services in the same docker container using the “Supervisor” tool.
> **So what is Supervisor?**

A supervisor is a tool that allows us to manage a number of different processes simultaneously in Linux like operating system. The supervisor tool requires a* .conf *file where we specify the processes and different options related to that process like the *output log location, auto start, auto restart, etc.*
> **Sample services that I am going to used**

I am going to run two different services “Gunicorn server running Django app” and “Redis server” inside the same container. So in the sample project that I have created contains a simple Django REST API for adding and fetching data from the Redis server. In a production setting these two services are used in separate containers with production-grade configuration. For the sake of demonstration, I will run both the services in one docker container.
> **Project Directory structure**

One of the good practices to start building your project is to have a neat and clear project structure. So considering this we will be using the below directory structure for our codes:

    multi_service
    ├── django_app
    │   ├── db.sqlite3
    │   ├── django_app
    │   ├── manage.py
    │   ├── redis_test
    │   └── requirements.txt
    ├── Dockerfile
    └── supervisor
        └── service_script.conf
> **Testing Our Services In Local(Django App and Redis)**

Before putting out services in docker, first, let’s test them locally. You can download the complete project from GitHub using the below link.
URL: [https://github.com/aakash-rathore/docker_multi_services.git](https://github.com/aakash-rathore/docker_multi_services.git)

For testing Redis server just download it using below command:

    Using apt package installer
    
    $ sudo apt-get install redis-server
    
    # Use apk add (For alpine linux)
    
    # apk add redis

Now run the server using below command:

    $ redis-serve

You will see the service running:

![Local redis server testing](https://cdn-images-1.medium.com/max/2424/1*WGJCe-NIag7lkOzdPzVlNw.png)*Local redis server testing*

For testing Django app in local we will use Gunicorn server. Use below command for running app in gunicorn server:

    $ gunicorn --bind 0.0.0.0:8000 django_app.wsgi

The output of this command is below:

![Running gunicorn in local](https://cdn-images-1.medium.com/max/2044/1*WMikyPraRrja68z6gzRkdQ.png)*Running gunicorn in local*
> **Supervisor config file**

Create a file ***“service_script.conf”*** as per the directory structure specified above. Add the django and redis services as per the configuration below:

    ## service_script.conf
    
    [supervisord]  ## This is the main process for the Supervisor    
    nodaemon=true  ## This setting is to specify that we are not running in daemon mode
    
    [program:redis_script] ## This is the part where we give the name and add config for our 1st service
    command=redis-server  ## This is the main command to run our 1st service
    autorestart=true ## This setting specifies that the supervisor will restart the service in case of failure
    stderr_logfile=/dev/stdout ## This setting specifies that the supervisor will log the errors in the standard output
    stderr_logfile_maxbytes = 0
    stdout_logfile=/dev/stdout ## This setting specifies that the supervisor will log the output in the standard output
    stdout_logfile_maxbytes = 0
    
    ## same setting for 2nd service
    [program:django_service] 
    command=gunicorn --bind 0.0.0.0:8000 django_app.wsgi
    autostart=true
    autorestart=true
    stderr_logfile=/dev/stdout
    stderr_logfile_maxbytes = 0
    stdout_logfile=/dev/stdout
    stdout_logfile_maxbytes = 0
> **Dockerfile Creation**

Now we will start making our docker file. The steps of adding layers are given below:
***Base Image -> Install Required Tools -> Add Source Code -> Add Config File -> Start Service***
Final Dockerfile is given below:

    # Base Image
    FROM alpine
    
    # Installing required tools
    RUN apk --update add nano supervisor python3 redis
    
    # Adding Django Source code to container 
    ADD /django_app /src/django_app
    
    # Adding supervisor configuration file to container
    ADD /supervisor /src/supervisor
    
    # Installing required python modules for app
    RUN pip3 install -r /src/django_app/requirements.txt
    
    # Exposing container port for binding with host
    EXPOSE 8000
    
    # Using Django app directory as home
    WORKDIR /src/django_app
    
    # Initializing Redis server and Gunicorn server from supervisord
    CMD ["supervisord","-c","/src/supervisor/service_script.conf"]
> **Building And Testing**

Finally, build the docker image using the below command:

    $ docker build -t multi_service:1 .

The output after a successful build operation:

![Build successful for docker](https://cdn-images-1.medium.com/max/2044/1*ViXmc5-fHrs1uW3MCNykuQ.png)*Build successful for docker*

Now run the docker container using the above built image:

    $ docker run -it -p 8000:8000 multi_service:1

Here the docker container is run in non-daemon mode if you want to run it in daemon mode use ***‘-d’*** option, the output of the above command:

![The output of running docker](https://cdn-images-1.medium.com/max/2000/1*ysOhsI3_-PSxUAATvVScSw.png)*The output of running docker*

Let's test the django app in API testing tool, I am using a lightweight “Advanced REST client too” available in Google Chrome extension store. You can use Postman, Curl, or any other tools for testing:

Testing add user Function:

![Testing add user function](https://cdn-images-1.medium.com/max/2024/1*9qz2LAsOCSKdO18wiWfaug.png)*Testing add user function*

Testing fetch user:

![Testing fetch user function](https://cdn-images-1.medium.com/max/2030/1*grQju01EGMGlVPiSam02mQ.png)*Testing fetch user function*
> **Conclusion**

In this article, I explained how we can run multiple services(Django app and Redis server) inside a single docker container. In the end, we also did some tests to check if both the services are running or not.

Please leave your comment about this article below and in case you are facing issues in any of the steps specified above you can reach out to me through [**Instagram](https://www.instagram.com/_aakash.rathore/)** and [**LinkedIn](https://www.linkedin.com/in/aakash-data-engineer).**
