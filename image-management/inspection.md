## Inspection

To view detailed information about a Docker image, use the `docker image inspect IMAGE` command. To view detailed
information about a container, use the `docker container inspect CONTAINER` command. By default, this will render the
results as a JSON array.

### Formatting the output

The `--format` flag allows to specify a template that the inspected image data will be applied to.

Get a container's IP address:

```shell script
$ docker container inspect --format="{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" my-container
```

Get a container's MAC address:

```shell script
$ docker container inspect --format="{{range .NetworkSettings.Networks}}{{.MacAddress}}{{end}}" my-container
```

Get a container's log path:

```
$ docker container inspect --format="{{.LogPath}}" my-container
```

Get a container's image name:

```
$ docker container inspect --format="{{.Config.Image}}" my-container
```

Get a container's port bindings:

```
$ docker container inspect --format="{{range $p, $conf := .NetworkSettings.Ports}}{{$p}} -> {{(index $conf 0).HostPort}}{{end}}" my-container
```
