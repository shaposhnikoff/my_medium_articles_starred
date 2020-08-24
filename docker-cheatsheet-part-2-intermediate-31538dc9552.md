Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m16[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m25[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m27[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m20[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m22[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m17[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m21[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m23[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m19[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m24[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m24[39m }

# Docker Cheatsheet : Part 2 (Intermediate)



Today, I am going to share with you all, part-2 of the Docker commands cheatsheet. You can refer to Part-1 (Beginners) [here](https://medium.com/@DevopsFollower/docker-cheatsheet-9b54a2ff8998).
> **Docker Cheatsheet (Intermediate):**

**Common docker commands**

1. **$ docker context create** to create a new docker context

1. **$ docker context** import a context from tar or zip file

1. **$ docker context **export an existing context to a tar or kubeconfig file

1. **$ docker events [options]** to get real-time events from the server

1. **$ docker load [options] **to load an image from a tar archive or STDIN

**Docker networking Commands:**

1. **$ docker network ls **shows all docker networks

1. **$ docker network create** to create a new network

1. **$ docker network inspect **displays detailed information on one or more networks

1. **$ docker network connect** to connect a container to a network

1. **$ docker network disconnect** disconnect a container from a network

1. **$ docker network prune** this removes all the unused networks

1. **$ docker network rm **to remove one or more networks

**Docker Trust commands: **these commands are used to manage the trust on Docker images

1. **$ docker trust inspect** it returns low-level information about keys and signatures

1. **$ docker trust key** to manage keys for signing Docker images

1. **$ docker trust revoke **this command revokes or removes trust for a docker image

1. **$ docker trust sign** used to sign am image with trust

1. **$ docker trust signer **to manage entities who can sign Docker images

**Docker Stack Command: **this command is used to manage docker stacks. The client and daemon API must both be at least 1.25 to use this command. A Docker stack enables you to define various functions of an application in a central file — the docker-compose.yml — and start it from there, run it together in an isolated runtime environment, and manage it centrally.

1. **$ docker stack ls** it lists all compose stacks

1. **$ docker stack ps **list the tasks in the stack

1. **$ docker stack deploy** deploy a new stack or update an existing stack

1. **$ docker stack services** list the services in the stack

**Docker Cluster Commands: **These commands are only available on Docker Enterprise Edition.

1. **$ docker cluster create** to create a new docker cluster

1. **$ docker cluster ls** shows a list of all available clusters

1. **$ docker cluster version** prints Version, Commit and Build type

1. **$ docker cluster inspect** display detailed information about a cluster

1. **$ docker cluster restore** restore a cluster from a backup

![](https://cdn-images-1.medium.com/max/2000/0*Piks8Tu6xUYpF4DU)

**Follow us on [Twitter](https://twitter.com/joinfaun) **🐦** and [Facebook](https://www.facebook.com/faun.dev/) **👥** and [Instagram](https://instagram.com/fauncommunity/) **📷 **and join our [Facebook](https://www.facebook.com/groups/364904580892967/) and [Linkedin](https://www.linkedin.com/company/faundev) Groups **💬**.**

**To join our community Slack team chat **🗣️ **read our weekly Faun topics **🗞️,** and connect with the community **📣** click here⬇**

![](https://cdn-images-1.medium.com/max/3000/1*6P3WpLjGv5v1ucm5dgkucg.png)

### If this post was helpful, please click the clap 👏 button below a few times to show your support for the author! ⬇
