## Dockerfile Best Practices

A Docker image consists of multiple read-only layers. Each layer represents a Dockerfile instruction. The layers are stacked, each one being a delta of the changes from the previous layer.

When you run an image and generate the container, a new _writeable layer_ - the container layer - will be added on top of the existing underlying layers. All changes made to the container, such as new files, modified files or deleted files are written to this writable container layer.

### Create ephemeral layers

The generated containers should be ephemeral as possible: They should be stop and destroyed, rebuilt and replaced with an absolute minimum setup and configuration.

### Understand build context

When the `docker build` command is executed, the current working directory is called the _build context_. All recursive contents in this directory are sent to the Docker daemon as build context.

> Inadvertently including files that aren't required for building an image will result in a larger build context and larger image size. This can also increase the time to build, pull and push the image and increase the container size as well.

### Pipe Dockerfile through `stdin`

Docker has the ability to build images by piping the Dockerfile through `stdin` with a local or remote build context. Example:

```shell script
$ echo -e 'FROM busybox\nRUN echo "hello world"' | docker build -
```

Use this syntax to build an image using a Dockerfile from `stdin` without sending additional files as build context. The hypen `-` instructs Docker to read the build context from `stdin` instead of a directory:

```shell script
$ docker build [OPTIONS] -
```

The following example builds an image using a Dockerfile that is passed through `stdin`. No files are sent as build context.

```shell script
$ docker build -t myimage -<<EOF
FROM busybox
RUN echo "hello world"
EOF
```

This is useful for Dockerfiles that don't require files to be copied into the image, improving the build speed since no files are sent to the daemon.

### Exclude with .dockerignore

To exclude files that are not relevant to the build, use a `.dockerignore` file.

### Use multi-stage builds

Multi-stage builds allow you to drastically reduce the size of your final image. Because an image is built during the final stage of the build, you can minimize image layers by leveraging build cache.

For example, if your build contains several layers, you can order them from the less frequently changed to the more frequently changed, ensuring that the build cache is reusable.
* Install tools you need to build your application 
* Install or update library dependencies
* Generate your application

### Don't install unnecessary packages

To reduce complexity, dependencies, file sizes and build times, avoid installing extra or unnecessary packages.

### Decouple applications

Each container should have only one concern. Decoupling applications into multiple containers makes it easier to scale horizontally and reuse containers.

Limiting each container to one process is a good rule of thumb. However, containers can be spawned with an init process and some programs might spawn additional processes of their own accord.

### Minimize the number of layers

In older versions of Docker, it was important to minimize the number of layers to ensure the images were performant. There were some features added to reduce this limitation:
* Only `RUN`, `COPY` and `ADD` creates a new layer. Other instructions create temporary intermediate layers and don't increase the size of the build.
* Multi-stage builds allow to only copy artifacts that are really needed in the final image

### Sort multiline arguments

Whenever possible, sort multiline arguments alphanumerically. This helps to avoid duplication of packages and make the list easier to update.

### Leverage build cache

The instructions in the Dockerfile are executed each in the order specified when building an image. As each instruction is examined, Docker looks for an existing image in its cache that it can reuse - rather than creating a new image.

Docker follows some rules to examine if it can use its cache:
* Starting with a parent image that is already in the cache, the next instruction is compared against all child images to see if one of them was built using the same instruction. If not, the cache is invalidated.
* In most cases, simply comparing the instruction in the Dockerfile with one of the child images is sufficient, but certain instructions require more examination.
* For the `ADD` and `COPY` instructions, the contents of the files in the image are examined and a checksum is calculated for each file. During the cache lookup, these checksums are compared against the checksum in the existing images. If they don't match, the cache is invalidated.
* Aside from `ADD` and `COPY`, cache checking does not involve looking at the files in the container.

If you don't want to use the cache, perform a build using the `--no-cache=true` flag.

### See also
[About storage drivers](https://docs.docker.com/storage/storagedriver/)
[Use multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)
[Leverage build cache](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache)
