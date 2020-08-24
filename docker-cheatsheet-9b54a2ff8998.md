Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m47[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m13[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m15[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m12[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m11[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }

# Docker Cheatsheet



Today ,I am going to share with you all, some of the commonly used Docker commands. This would be part-1 of 3 part series.
> **Docker Cheatsheet:**

1. $ docker version to know the current docker version

1. $ docker version — format ‘{{.Server.Version}}’. to get the server version

### **Docker Container Lifecycle:**

1. $ docker create creates a container but does not start it

1. $ docker rename allows the container to be renamed

1. $ docker run creates and starts a container in one operation

1. $ docker rm deletes a container

1. $ docker rm -v delete container and also remove the volumes associated with the container

**Starting & Stopping a Docker container**

1. $ docker starts starts a container so it is running

1. $ docker stop stops a running container

1. $ docker pause pauses or suspend a running container

1. $ docker restart stops and starts a container.

1. $ docker wait blocks until running container stops

1. $ docker kill sends a SIGKILL to a running container

1. $ docker attach will connect to a running container.

**Other Docker commands:**

1. $ docker cp copies files or folders between a container and the local filesystem

1. $ docker export turns container filesystem into tarball archive stream to STDOUT

1. $ docker history shows history of image

1. $ docker tag tags an image to a name (local or registry)

1. $ docker ps lists all running containers

1. $ docker inspect looks at all the info on a container (including IP address)
