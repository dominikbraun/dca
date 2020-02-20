## Private registry

A registry is an instance of the `registry` image and runs within Docker.

### Running a local registry

In order to run a local registry, pull the `registry:2` image and start a container from it.

```shell script
$ docker container run -d -p 5000:5000 --rm --restart=always --name local-registry registry:2
```

This will start a registry instance that will be available under `localhost:5000`.

### Copy an image to a local registry

It is possible to pull an image from Docker Hub and push it to the local registry. First of all, pull the image:

```shell script
$ docker image pull ubuntu:18.04
```

Then tag the image, prefixing it with `localhost:5000`. A hostname and an optional port indicates an image registry.

```shell script
$ docker image tag ubuntu:18.04 localhost:5000/ubuntu:18.04
```

Since the image tag starts with a hostname and a port, Docker interprets this as a registry address when pushing the
image:

```shell script
$ docker image push localhost:5000/ubuntu
``` 

After that, we can remove the local images. This won't remove the image from the local registry.

```shell script
$ docker image remove ubuntu:18.04
$ docker image remove localhost:5000/ubuntu:18.04
```

The image is stored in the local registry, so it can be pulled down at any time.

```shell script
$ docker image pull localhost:5000/ubuntu:18.04
```

### Stopping the registry

The local registry can be stopped like any other container.

```shell script
$ docker container stop registry
```

### Customizing the registry storage

By default, all registry data is persisted as a docker volume on the host filesystem. However, it may also be useful to
create a bind mount on the local filesystem to store the data. In this example all data is persisted at `/mnt/registry`:

```shell script
$ docker container run -d \
  -p 5000:5000
  --rm
  --restart=always
  --name local-registry
  --volume /mnt/registry:/var/lib/registry
  registry:2
```

### Running a public available registry

Running a externally accessible registry requires you to secure the registry with TLS.
