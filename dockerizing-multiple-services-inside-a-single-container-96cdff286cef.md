Unknown markup type 10 { type: [33m10[39m, start: [33m27[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m92[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m17[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m82[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m147[39m, end: [33m152[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m73[39m, end: [33m91[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m29[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m44[39m, end: [33m51[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m61[39m, end: [33m79[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m48[39m, end: [33m55[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m57[39m, end: [33m64[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m10[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m35[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m20[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m43[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m11[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m25[39m, end: [33m32[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m38[39m, end: [33m45[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m84[39m, end: [33m87[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m97[39m, end: [33m106[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m111[39m, end: [33m118[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m121[39m, end: [33m125[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m146[39m, end: [33m154[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m163[39m, end: [33m170[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m66[39m, end: [33m74[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m109[39m, end: [33m127[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m137[39m, end: [33m146[39m }

# Dockerizing Multiple Services Inside a Single Container

Tutorial on defining and creating multiple Docker containers, including database setup

![](https://cdn-images-1.medium.com/max/2000/1*WJXpV75HFx6og-U8NBR7lw.png)

Recently, I got a chance to work on Docker containers for a project. It was a great learning experience as I had no idea about Docker's usage before.

The objective of this blog is to create multi-container Docker Applications using Docker Compose (app server, client-server and PostgreSQL database).

## **What Is Docker? (Basic Definition)**
> [“Docker](https://github.com/docker/docker) is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and ship it all out as one package.
> By doing so, thanks to the container, the developer can rest assured that the application will run on any other Linux machine, regardless of any customized settings that machine might have that could differ from the machine used for writing and testing the code.” — Source [https://opensource.com/resources/what-docker](https://opensource.com/resources/what-docker)

I like to think of Docker as LEGO pieces for development tools, which can be joined together to ease the development process and remove the setup overhead.

## **Current Project Structure and Its Solution**

Code for this project can be found on [GitHub](https://github.com/mukul13/docker-example). For this project, we are going to set up containers for two Node.js servers and a [Postgres](https://www.postgresql.org/) database.

We can have both servers written in different languages or frameworks, as long as we initialize the Docker file correctly for it.

For simplicity’s sake, we are going to host two Node.js servers with two basic endpoints to test this project.

    ### Sample requests
    curl [http://localhost:4000/](http://localhost:4000/)

    # Expected response
    #
    #  {"message":"Hello from Server 1"}
    #

    curl [http://localhost:3001/](http://localhost:4000/)

    # Expected response
    #
    #  {"message":"Hello from Server 2"}
    #

The current project structure is as follows:

    docker-example
    - server1 (will run on port 4000)
    - server2 (will run on port 3001)
    - docker-compose.yml (will initialize all Docker containers: postgres, server1, and server2)

First, we will have to add Dockerfile to each Node.js server project.

Depending on the languages and frameworks we use, these Dockerfiles may differ. A Dockerfile is a text document that contains all the commands a user could call on the command-line to assemble an image.

For server1, our Dockerfile looks something like this:

    FROM node:12.4.0
    EXPOSE 4000

    WORKDIR /home/server1

    COPY package.json /home/server1/
    COPY package-lock.json /home/server1/

    RUN npm ci

    COPY . /home/server1

    RUN npm install

    ADD [https://github.com/ufoscout/docker-compose-wait/releases/download/2.2.1/wait](https://github.com/ufoscout/docker-compose-wait/releases/download/2.2.1/wait) /wait
    RUN chmod +x /wait

    ## Launch the wait tool and then your application
    CMD /wait && PORT=4000 yarn start

Basically, what we are trying to do is:

* Expose port 4000 and download Node version 12.4.0 Docker image to build this project.

* Create a working directory and copy the required folders in it.

* When we start the build process for the project, all Docker services mentioned in docker-compose.yml are initialized and run at the same time. The /wait command is to wait for an image to load before running the next command. For example, we would want the PostgreSQL database to initialize first and then all servers so that there is no problem in the database connection when the servers are started. Another method is to add retries in the Node.js server code to wait for the database connection but I find this method pretty clean to solve the above problem.

* Finally, we have created a Docker container which can be initialized via docker-compose.yml. Basically, we have joined multiple LEGO pieces in a way to make things works (Node is a LEGO piece and the Node.js codebase is another LEGO piece).

Now, there will be a similar Dockerfile for server2 as well. docker-compose.yml is as follows:

    # docker-compose.yml
    version: "3.3"
    services:
      postgres:
        image: postgres
        hostname: postgres
        environment:
          POSTGRES_DB: postgres
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
        ports:
          - 5432:5432

    app:
        build: ./server1
        hostname: app
        env_file: ./server1/.env
        depends_on:
          - postgres
        links:
          - postgres
        ports:
          - 4000:4000
        environment:
          WAIT_HOSTS: postgres:5432

    client:
        build: ./server2
        hostname: client
        env_file: ./server2/.env
        ports:
          - 3001:3001
        depends_on:
          - postgres
        links:
          - postgres
        environment:
          WAIT_HOSTS: postgres:5432

docker-compose.yml is the starting point of our application.

It is used to initialize all Docker containers (server1, server2, and Postgres), to expose ports and to map with respective environment variables.

What we are trying to do is:

* Create a separate configuration for the different services we have.

* WAIT_HOSTS is used to wait for the postgres service to finish execution.

* Expose the required ports and add the build path.

To run the current project:

    cd to-the-parent-directory
    docker-compose up

To stop and remove containers for the current project:

    cd to-the-parent-directory
    docker-compose down

To get more info about running containers:

    docker ps

We can try curl requests to verify if servers are running correctly or not.

If you are trying to hit server1 from server2, you will have to set the host URL as app (and not localhost) in server2’s .env file as that is the hostname for the server1.

Similarly, while connecting to the database, you will have to use postgres (which is the service name in our docker-compose.yml) and not localhost.

Thanks for reading. I hope this helps. Don’t hesitate to correct any mistakes in the comments or provide suggestions for future posts!
