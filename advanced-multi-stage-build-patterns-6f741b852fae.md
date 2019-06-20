
# Advanced multi-stage build patterns



Multi-stage builds feature in Dockerfiles enables you to create smaller container images with better caching and smaller security footprint. In this blog post, I’ll show some more advanced patterns that go beyond copying files between a build and a runtime stage, allowing to get most out of the feature. If you are new to multi-stage builds you probably want to start by reading the [usage guide](https://docs.docker.com/develop/develop-images/multistage-build/) first.

**Version compatibility**

Multi-stage builds support was added to Docker in version v17.05. All the patterns here work with any version after that, but some of them run much more efficiently with the builders that use the [BuildKit](https://github.com/moby/buildkit) backend. For example, BuildKit efficiently skips unused stages and builds stages concurrently when possible. I’ve marked these cases under the individual examples. BuildKit is [currently being added to Moby](https://github.com/moby/moby/pull/37151) as an experimental builder backend and is scheduled to be available in Docker CE v18.06 . It can also be used [standalone](https://github.com/moby/buildkit#quick-start) or as part of the [img](https://github.com/genuinetools/img) project.

### Inheriting from a stage

Multi-stage builds added a couple of new syntax concepts. First of all, you can name a stage that starts with a FROM command with AS stagename and use --from=stagename option in a COPY command to copy files from that stage. In fact, FROM command and --from flag have much more in common and it is not accidental that they are named the same. They both take the same argument, resolve it and then either start a new stage from that point or use it as a source for file copy. That means that same way as you can use --from=stagename you can also use FROM stagename to use a previous stage as a source image for your current stage. This is useful when multiple commands in the Dockerfile share the same common parts. It makes the shared code smaller and easier to maintain while keeping the child stages separate so that when one is rebuilt it doesn’t invalidate the build cache for the others. Each stage can also be built individually using the--target flag while invoking docker build.

    FROM ubuntu AS base
    RUN apt-get update && apt-get install git

    FROM base AS src1
    RUN git clone …

    FROM base as src2
    RUN git clone …

In BuildKit, the second and third stage in this example would be built concurrently.

### Using images directly

Similarly to using build stage names in FROM commands that previously only supported image references, we can turn this around and directly use images with --from* *flag. This allows copying files directly from other images. For example, in the following code, we can use linuxkit/ca-certificates image to directly copy the TLS CA roots into our current stage.

    FROM alpine
    COPY --from=linuxkit/ca-certificates / /

### Alias for a common image

A build stage doesn’t need to contain any commands — it may just be a single FROM line. When you are using an image in multiple places this can be useful to improve readability and making sure that when a shared image needs to be updated, only a single line needs to be changed.

    FROM alpine:3.6 AS alpine

    FROM alpine
    RUN …

    FROM alpine
    RUN …

In this example, any place that uses image alpine is actually fixed to alpine:3.6 not alpine:latest. When it comes time to update to alpine:3.7, only a single line needs to be changed and we can be sure that all parts of the build are now using the updated version.

This is even more powerful when a build argument is used in the alias. The following example is equal to the previous one but lets the user override all the instances the alpine image is being used in this build with setting the --build-arg ALPINE_VERSION=value option. Remember that any arguments used in FROM commands need to be defined [before the first build stage](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact).

    ARG ALPINE_VERSION=3.6

    FROM alpine:${ALPINE_VERSION} AS alpine

    FROM alpine
    RUN …

### Using build arguments in ` — from`

The value specified in --from flag of the COPY command may not contain build arguments. For example, the following example is not valid.

    // THIS EXAMPLE IS INTENTIONALLY INVALID

    FROM alpine AS build-stage0
    RUN …

    FROM alpine
    ARG src=stage0
    COPY --from=build-${src} . .

This is because the dependencies between the stages need to be determined before the build can start, so that we don’t need to evaluate all commands every time. For example, an environment variable defined in alpine image could have an effect on the evaluation of the --from value. The reason we can evaluate the arguments for the FROM command is that these arguments are defined globally before any stage begins. Luckily, as we learned before, we can just define an alias stage with a single FROM command and refer that instead.

    ARG src=stage0

    FROM alpine AS build-stage0
    RUN …

    FROM build-${src} AS copy-src

    FROM alpine
    COPY --from=copy-src . .

Overriding a build argument src would now cause the source stage for the final COPY element to switch. Note that if this causes some stages to become unused, only BuildKit based builders have the capability to efficiently skip these stages so they never run.

### Conditions using build arguments

There have been requests to add IF/ELSE style conditions support in the Dockerfile. It is unclear yet if something like this will be added in the future — with the help of custom frontends support in BuildKit we may try that in the future. Meanwhile, with some planning, there is a possibility to use current multi-stage concepts to get a similar behavior.

    // THIS EXAMPLE IS INTENTIONALLY INVALID

    FROM alpine
    RUN …
    ARG BUILD_VERSION=1

    IF $BUILD_VERSION==1
    RUN touch version1
    ELSE IF $BUILD_VERSION==2
    RUN touch version2
    DONE

    RUN …

The previous example shows pseudocode how conditions could be written with IF/ELSE. To have the same behavior with current multi-stage builds you would need to define different branch conditions as separate stages and use an argument to pick the correct dependency path.

    ARG BUILD_VERSION=1

    FROM alpine AS base
    RUN …

    FROM base AS branch-version-1
    RUN touch version1

    FROM base AS branch-version-2
    RUN touch version2

    FROM branch-version-${BUILD_VERSION} AS after-condition

    FROM after-condition 
    RUN …

The last stage in this Dockerfile is based on after-condition stage that is an alias to an image that is resolved by BUILD_VERSION build argument. Depending on the value of BUILD_VERSION, a different middle section stage is picked.

Note that only BuildKit based builders can skip the unused branches. In previous builders all stages would be still built, but their results would be discarded before creating the final image.

### Development/test helper for minimal production stage

Let’s finish up with an example of combining the previous patterns to show how to create a Dockerfile that creates a minimal production image and then can use the contents of it for running tests or for creating a development image. Start with a basic example Dockerfile:

    FROM golang:alpine AS stage0
    …
    FROM golang:alpine AS stage1
    …

    FROM scratch
    COPY --from=stage0 /binary0 /bin
    COPY --from=stage1 /binary1 /bin

This is quite a common when creating a minimal production image. But what if you wanted to also get an alternative developer image or run tests with these binaries in the final stage? An obvious way would be just to copy the same binaries to the test and developer stages as well. A problem with that is that there isn’t a guarantee that you will test all the production binaries in the same combination. Something may change in the final stage and you may forget to make identical changes to the other stages or make a mistake to the path where the binaries are copied. After all, we want to test the final image not an individual binary.

An alternative pattern would be to define a developer and test stage after production stage and copy the whole production stage contents. A single FROM command with the production stage can be then used to make the production stage default again as the last step.

    FROM golang:alpine AS stage0
    …

    FROM scratch AS release
    COPY --from=stage0 /binary0 /bin
    COPY --from=stage1 /binary1 /bin

    FROM golang:alpine AS dev-env
    COPY --from=release / /
    ENTRYPOINT ["ash"]

    FROM golang:alpine AS test
    COPY --from=release / /
    RUN go test …

    FROM release

By default, this Dockerfile will continue building the default minimal image, while building for example with --target=dev-env option will now build an image with a shell that always contains the full release binaries.

I hope this was helpful and gave you some ideas for creating more efficient multi-stage Dockerfiles. If you are participating at [DockerCon2018](https://2018.dockercon.com/) and want to learn more about multi-stage builds, Dockerfiles, BuildKit or anything related, feel free to sign up for a builder [Hallway track](https://hallwaytrack.dockercon.com/) or catch the Docker Platform Internals talks in the [Contribute and Collaborate](https://dockercon18.smarteventscloud.com/connect/sessionDetail.ww?SESSION_ID=187206) or [Black Belt](https://dockercon18.smarteventscloud.com/connect/sessionDetail.ww?SESSION_ID=193373) tracks.
