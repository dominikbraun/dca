## Mount volumes

> Note: This summary focuses on 'real' volumes, not bind mounts.

### Volumes vs bind mounts

Advantages of volumes over bind mounts:
* Volumes are easier to back up or migrate
* Volumes are manageable via Docker CLI commands/the Docker API
* Volumes work on both Linux and Windows containers
* Volumes can be more safely shared among multiple containers
* Volume drivers let you store volumes on remote hosts (including cloud providers)
* New volumes can have their content pre-populated by a container

Also, volumes often are the better choice than persisting data in a container's writeable layer: A volume does not increase the size of the container using it and the contents of a volume are independent of the container's lifecycle.

### --volume vs. --mount

Originally, the `--volume` flag (or short: `-v`) was used for standalone containers and the `--mount` flag was used for swarm services. Since Docker 17.06 `--mount` is also possible for standalone containers.

In general, `--mount` is more explicit and verbose so it is somewhat self-documenting. The biggest difference is that the `-v` syntax combines all the options together in one field while `--mount` separates them. If you need to specify driver options, `--mount` is required anyway.

* `--volume` consists of 3 fields separated by colon characters (`:`):
    1. In the case of named volumes, the first field is the volume name. For anonymous volumes, this field is omitted.
    2. The second field is the path to the mounted file or directory inside the container.
    3. The third field is an optional comma-separated list of options, such as `ro`.
* `--mount` consists of multiple key-value pairs separated by commas. This is more verbose than `--volume`, but also more explicit and the order of keys doesn't matter:
    * `type` indicates the mount type. Possible values are `bind`, `volume` or `tmpfs`.
    * `source` (or `src`) is the mount source. For named volumes, this is simply the name of the volume. For anonymous volumes, this field is omitted.
    * `destination` (or `dst` or `target`) takes the path where the file or directory is mounted in the container.
    * `readonly` is an option that, if present, causes the bind mount to be mounted into the container as ready-only.
    * `volume-opt` takes a key-value par consisting of the option name and its value.
    
### Start a container with volumes

Unlike bind mounts, volumes can be created and managed outside the scope of a container.

```shell script
$ docker volume create my-vol
```

Starting a container with a volume that does not yet exist causes Docker to create that volume. This mounts the `my-vol` volume into `/app` in the container:

```shell script
$ docker container run -d \
  --name test \
  --mount source=my-vol,target=/app
  nginx:latest
```

Using `docker inspect test`, you can verify that the volume has been mounted correctly.

### Start a service with volumes

When you create a service and define a volume, each container uses its own local volume. Non of these containers can share the volume data when using the `local` volume driver, but there are drivers that support shared storage.

The following command creates a new service with four replicas - each of them uses a local volume called `service-vol`.

```shell script
$ docker service create --replicas 4 --name nginx --mount source=service-vol,destination=/app nginx:latest

47fxsxro9aohfoh48hzcfchsh
overall progress: 4 out of 4 tasks 
1/4: running   [==================================================>] 
2/4: running   [==================================================>] 
3/4: running   [==================================================>] 
4/4: running   [==================================================>] 
verify: Service converged
```

Use `docker service ps nginx` to verify that the service tasks are up and running.

### Use a read-only volume

...


### Use a volume driver

...

### See also

[Use volumes](https://docs.docker.com/storage/volumes/#start-a-container-with-a-volume)
