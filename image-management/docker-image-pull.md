## Pull an image

Most images are created on top of a base image which gets pulled from the Docker Hub registry.

### Pull an image from Docker Hub

To download an image or a set of images (i. e. a repository), use the `docker image pull` command. Just as always, if no
tag is provided, the `:latest` tag will be used as a default. This command pulls the `debian:latest` image:

```shell script
docker image pull debian

Using default tag: latest
latest: Pulling from library/debian
fdd5d7827f33: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:e7d38b3517548a1c71e41bffe9c8ae6d6d29546ce46bf62159837aad072c90aa
Status: Downloaded newer image for debian:latest
```

As images consist of multiple layers and these layers are re-used among images, if a layer is already present locally,
it won't be pulled.

```shell script
$ docker image pull debian:jessie
  
  jessie: Pulling from library/debian
  fdd5d7827f33: Already exists
  a3ed95caeb02: Already exists
  Digest: sha256:a9c958be96d7d40df920e7041608f2f017af81800ca5ad23e327bc402626b58e
  Status: Downloaded newer image for debian:jessie
```

Docker uses a content-addressable image store, and the image ID is a SHA256 digest covering the image's configuration
and layers. In the example above, both `debian:jessie` and `debian:latest` have the same image ID since they refer to
the exact same layers (in fact, it is the same image tagged differently). Because they're the same image, their layers
are only stored once.

### Pull an image by digest

Docker allows users to pull an image by its digest, which can be useful when you don't want images to be updated to
newer versions and use a fixed version instead.

When pulling an image by digest, you specify the exact version of the image to pull. _Pinning_ the versions guarantees
reproducible builds.

To find out the digest of an image, pull it.

```shell script
$ docker image pull ubuntu:14.04
  
  14.04: Pulling from library/ubuntu
  5a132a7e7af1: Pull complete
  fd2731e4c50c: Pull complete
  28a2f68d1120: Pull complete
  a3ed95caeb02: Pull complete
  Digest: sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2
  Status: Downloaded newer image for ubuntu:14.04
```

This will print the digest of the image when the pull has finished:

```shell script
sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2
```

The image digest will also be printed when _pushing_ the image to a registry. This digest takes the place of the image
tag when pulling an image. To pull the same image from the example above again, but using the digest, run the following
command:

```shell script
$ docker image pull ubuntu@sha256:45b23dee08af5e43a7fea6c4cf9c25ccf269ee113168c19722f87876677c5cb2

  ...
```

This digest notation can also be used in a Dockerfile `FROM` instruction.

> Pinning prevents Docker from pulling newer versions, which may contain security updates. To receive those updates, you
> have to update the image digest to the newer version accordingly.

### Pull from a different registry

By default, `docker image pull` pulls from the Docker Hub registry. It is also possible to specify another registry to
pull from. A registry path is similar to an URL, but without a protocol specifier like `https://`.

The following command pulls the `testing/test-image` from a local registry available under `localhost:5000`.

```shell script
$ docker image pull localhost:5000/testing/test-image
```

Docker uses `https://` to connect to a registry, unless insecure connections are allowed - see [insecure registries](https://docs.docker.com/engine/reference/commandline/dockerd/#insecure-registries).

### Pull a repository with multiple images

`docker image pull` pulls a single image by default. However, a repository can  contain multiple images and Docker is
able to pull all of these images using the `--all-tags` flag.

This command will pull all images from the `fedora` repository:

```shell script
$ docker image pull --all-tags fedora
```

To list the images that have been pulled and are now available locally, run `docker images fedora`.
