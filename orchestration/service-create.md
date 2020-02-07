## docker service create

### Deploy a service to the swarm

After creating a swarm (and optionally after adding worker nodes - see join-nodes.md), a new service can be created and deployed to the swarm. This is done by running `docker service create` on a manager node.

The syntax is `docker service create [OPTIONS] IMAGE COMMAND`, for example:

```shell script
docker service create --replicas 1 --name pinger alpine:latest ping docker.com
```

`--replicas` indicates the desired state of 1 running service.

### Get service information

All services can be listed like this:

```shell script
$ docker service ls

D                  NAME                MODE                REPLICAS            IMAGE               PORTS
m64bu7fybz85        hello-swarm         replicated          1/1                 alpine:latest       

```

To get the nodes that are currently running one or more service instances, execute `docker service ps SERVICE`.

```shell script
$ docker service ps pinger

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
kftvm0dvg1g8        hello-swarm.1       alpine:latest       linux               Running             Running about a minute ago
```

Running `docker container ps` on a node that runs the task will print details about the container for the task.

`docker service inspect SERVICE` prints even more detail about a service.

```shell script
$ docker service inspect --pretty pinger

ID:             m64bu7fybz85ce6o7ds09xe0n
Name:           pinger
Service Mode:   Replicated
 Replicas:      1
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
...
```

### See also
* join-nodes.md
* (Deploy a service to the swarm)[https://docs.docker.com/engine/swarm/swarm-tutorial/deploy-service/]
* (Inspect a service on the swarm)[https://docs.docker.com/engine/swarm/swarm-tutorial/inspect-service/]