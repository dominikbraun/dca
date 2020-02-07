## Swarm Setup

### Create a swarm

Create a new swarm using `swarm init`. The `--advertise-addr` tells the manager node to use the specified IP as public address for other nodes. 

```shell script
$ docker swarm init --advertise-addr 192.168.188.28
```

This makes the current machine a manager node and outputs the command for joining a new node to the swarm:

```shell script
Swarm initialized: current node (4us39x1tsmpfh6266hqbe3i2w) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-461bx8vdgas19hp9v0m74u7wpprnfk55wwuirf5evhy9wix8yk-ccg5aettadmsm7otgbzzqpg6y 192.168.188.28:2377

```

Depending on the `--token` flag, a joining node will become a worker or manager node.

### Retrieve information about a swarm

On the manager node (maybe on worker nodes as well?), `docker info` prints relevant information for the current swarm.

```shell script
$ docker info

Containers: 2
Running: 0
Paused: 0
Stopped: 2
  ...
Swarm: active
  NodeID: dxn1zf6l61qsb1josjja83ngz
  Is Manager: true
  Managers: 1
  Nodes: 1
  ...
```

Information about nodes can be printed like this:

```shell script
$ docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
4us39x1tsmpfh6266hqbe3i2w *   linux               Ready               Active              Leader              19.03.5

```

A `*` after an ID indicates that this is the node we're connected to. 