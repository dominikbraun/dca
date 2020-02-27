## Tagging

Tagging an existing image can be accomplished using the `docker image tag` command. An image name is made of one or more
components separated by a slash `/`, optionally prefixed with a registry hostname and optionally a port.

If no image registry hostname is present, the command uses Docker's public registry `registry-1.docker.io` by default.

The command takes a source image followed by the desired tagged image name. For example, the image `ubuntu` from the
public Docker registry can be tagged to `localhost:5000/ubuntu` like so:

```shell script
$ docker image tag ubuntu localhost:5000/ubuntu
``` 

This also assigns a registry to the image: If the push it to its registry using `docker image push`, it will be pushed
to `localhost:5000` instead of the Docker registry where the image originally came from.

If no source image tag is specified, it defaults to `:latest` as always.

### Further examples

Tag an image referenced by its ID:

```shell script
$ docker image tag 0e5574283393 fedora/httpd:version1.0
```

Tag an image referenced by its name:

```shell script
$ docker image tag httpd fedora/httpd:version1.0
```

Tag an image referenced by its name and tag:

```shell script
$ docker image tag httpd:test fedora/httpd:version1.0.test
```

Tag an image for a private registry:

```shell script
$ docker image tag 0e5574283393 localhost:5000/fedora/httpd:version1.0
```
