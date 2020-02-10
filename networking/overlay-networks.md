## Overlay networks

When a new swarm is initialized or a new host is joined to an existing swarm, two new networks will be created on the respective host:
* the `ingress` overlay network: It handles control and data traffic related to swarm services. If you don't connect a new service to a user-defined network, it will connect to the `ingress` network
* the `docker_gwbridge` bridge network: It connects the host's Docker daemon to other daemons participating in the swarm

In general, the `overlay` driver is a network driver that creates a network among multiple Docker daemon hosts. Sitting on top of the host-specific networks, it provides a secure communication way for containers.

`docker network create` allows the definition of a custom overlay networks as well as custom bridge networks. It is important to distinguish between operations/configuration that apply to all overlay networks, those that apply to service networks and those that apply to standalone containers.

### Operations for all overlay networks

> Before creating an overlay network, a new swarm has to be created (or the current node has to join an existing swarm). This will create the `ingress` network, the default network for swarm services.

Creating an overlay network for swarm services is achieved through the `network create` command.

```shell script
$ docker network create -d overlay custom-overlay
```

The `-d overlay` flag indicates the network driver which defaults to `bridge`.

To create an overlay network for swarm services or for standalone containers, it has to be attachable:

```shell script
$ docker network create -d overlay --attachable custom-attachable-overlay
```

Over options enable the definition of IP ranges, subnets and gateways.

### Encrypted traffic for overlay networks

...