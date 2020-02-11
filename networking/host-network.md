## Host network

It is possible to bind containers directly to the host network without any network isolation. From a networking point of view, the application inside the container is running directly on the Docker host, not containerized. However, in all other ways (storage, namespaces etc.) the application still is isolated from the host.

1. Create and start a container and specify the `--host` flag:

```shell script
$ docker container run -d --name my-nginx --network host nginx
```

2. Access nginx by browsing to localhost:80.
3. Examine the network stack using `ip addr show` and verify that the process is bound to port 80 using `sudo netstat -tulpn | grep :80`
4. Stop and remove the container:

```shell script
$ docker container rm $(docker container stop my-nginx)
```
