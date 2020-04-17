## docker port

The `docker port` command lists port mappings or a specific mapping for the container. It allows to identify which IP
and port a container is externally listening on.

### Usage

```shell script
docker port CONTAINER [RPIVATE PORT[/PROTOCOL]]
```

### Show all mapped ports

You can find out all the ports mapped by not specifying a `PRIVATE PORT`:

````shell script
$ docker port my-container
  7890/tcp -> 0.0.0.0:4321
  9876/tcp -> 0.0.0.0:1234

$ docker port my-container 7890/tcp
  0.0.0.0:4321
````
