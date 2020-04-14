## Bridge network

### Use user-defined bridge networks

Creating an own bridge network is fairly simple:

```shell script
$ docker network create --driver bridge alpine-net
```

Running `docker network inspect alpine-net` will show you its IP address and the containers connected to it.

### See also
[Networking with standalone containers](https://docs.docker.com/network/network-tutorial-standalone)