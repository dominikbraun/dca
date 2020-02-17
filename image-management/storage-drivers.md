## Storage drivers

To use storage drivers efficiently, it is important to know how Docker builds and stored images - and how these images
are used by containers. You can use this information to make informed choices about the best way to persist data from
your applications and avoid performance problems along the way.

**Storage drivers allow you to create data in the writable container layer.** This data won't be persisted after the
container is deleted.

### Images and layers

A Docker image is built up from a series of layers. Each layer represents an instruction in the image's Dockerfile. Each
layer - except the last one - is read-only. For example, consider the following Dockerfile:

```Dockerfile
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

This Dockerfile contains four commands, each of which writes a layer. The `FROM` statement starts out by creating a
layer from the `ubuntu:18.04` image. The `COPY` instruction adds some files from the current directory. The `RUN`
instruction builds the application using `make`. Finally, the last layer specifies which command to run within the
container.

Each layer is only a set of differences from the preceding layer. The layers are stacked on top of each other. When you
create a new container, a new writeable layer ("container layer") on top of the other layers will be created. All
changes made to the container are written to this writable container layer.

### Containers and layers

The major difference between a container and an image is the top writable layer. All writes to the container that add or
modify existing data are stored in this writeable layer. The container layer is deleted when the container is deleted.

Since each container has its own container layer, multiple containers can share access to the same underlying image and
yet have their own data state.

### Container size on disk

Using `docker container ps -s` you can approximate the size of a running container. There are two different size-related
columns:
* size: the amount of data on disk used for the respective container layer.
* virtual_size: the amount of data used for the read-only image data used by the container plus the writable layer size.

The total disk space used by all of the running containers is some combination of each container's size and the
virtual_size values. If multiple containers started from the same image, the total size on disk for these containers
is size_1 + size_2 + 1 image size (which is virtual_size - size).

### The copy-on-write strategy


