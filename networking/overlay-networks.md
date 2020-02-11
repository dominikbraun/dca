## Overlay networks

When a new swarm is initialized or a new host is joined to an existing swarm, two new networks will be created on the respective host:
* the `ingress` overlay network: It handles control and data traffic related to swarm services. If you don't connect a new service to a user-defined network, it will connect to the `ingress` network
* the `docker_gwbridge` bridge network: It connects the host's Docker daemon to other daemons participating in the swarm

In general, the `overlay` driver is a network driver that creates a network among multiple Docker daemon hosts. Sitting on top of the host-specific networks, it provides a secure communication way for containers.

`docker network create` allows the definition of a custom overlay networks as well as custom bridge networks. It is important to distinguish between operations/configuration that apply to all overlay networks (1), those that apply to service networks (2) and those that apply to standalone containers (3).

### 1. Operations for all overlay networks

This set of operations applies to swarm services and standalone containers.

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

By default, all swarm management traffic is AES-encrypted. Managers in the swarm rotate the encryption key every 12 hours.

> Overlay networks can't be encrypted on Windows. It won't fail to connect, but cannot communicate in the network.

In order to achieve the same encryption for standalone-container networks, use the `--opt encrypted` flag additionally.

```shell script
$ docker network create -d overlay --attachable --opt encrypted custom-encrypted-attachable-overlay
```

### Customizing the default ingress network

Customizing the default `ingress` network can be useful if the automatically chosen subnet conflicts with an existing subnet or if you need to customize low-level settings. This should be done before creating any services, since customizing the network usually means removing and recreating it. However, if there are services connected to the network already, all services with published ports need to be removed in order to make the `ingress` network removable.

In the time period where no `ingress` network exists, services with unpublished ports continue to function but won't be load balanced by swarm.

1. Inspect the `ingress` network using `docker network inspect ingress` and remove/stop all services whose containers are connected to it.
2. Remove the `ingress` network using `docker network rm ingress`.
3. Create a new overlay network using the `--ingress` flag:

```shell script
$ docker network create \
  --driver overlay \
  --ingress \
  --subnet=10.11.0.0/16 \
  --gateway=10.11.0.2 \
  --opt com.docker.network.driver.mtu=1200 \
  custom-ingress
```

4. Restart the affected services.

### Customizing the docker_gwbridge interface

`docker_gwbridge` is a virtual bridge that connects the overlay networks - including `ingress` - to a Docker daemon's physical network. Since this bridge is not a Docker device but exists in the kernel of the Docker host instead, any customizations must be accomplished before joining the host to the swarm, or after temporarily removing it from the swarm.

1. Stop docker.
2. Delete the existing `docker_gwbridge` interface:

```shell script
$ sudo ip link set docker_gwbridge down
$ sudo ip link del dev docker_gwbridge
```

3. Start Docker without initializing or joining a swarm.
4. Create the `docker_gwbridge` interface manually using `docker network create`.

```shell script
$ docker network create \
  --subnet 10.11.0.0/16 \
  --opt com.docker.network.bridge.name=docker_gwbridge \
  --opt com.docker.network.bridge.enable_icc=false \
  --opt com.docker.network.bridge.enable_ip_masquerade=true \
  docker_gwbridge
```

5. Initialize or join a swarm. Docker won't recreate the `gwbridge` network since it already exists.

### 2. Operations for swarm services.

This set of operations only applies to swarm services and their containers.

### Publish ports on an overlay network

Swarm services connected to the same overlay network expose all ports to each other. If you want to make a port accessible outside of the service, the port needs to be published using the `--publish` flag on `docker service create` or `docker service update`.

Map TCP port 80 on the service to port 8080 on the routing mesh:

```shell script
-p 8080:80
-p published=8080,target=80
```

Map UDP port 80 on the service to port 8080 on the routing mesh:

```shell script
-p 8080:80/udp
-p published=8080,target=80,protocol=udp
```

Map TCP port 80 on the service to TCP port 8080 on the routing mesh, and map UDP port 80 on the service to UDP port 8080 on the routing mesh.

```shell script
-p 8080:80/tcp -p 8080:80/udp
-p published=8080,target=80,protocol=tcp -p published=8080,target=80,protocol=udp
```

In general, the longer form is preferred since it is self-documenting.

### Bypass the routing mesh for a service

If a service publishes ports, it does so using the routing mesh. When you connect to a published port on any swarm node, you are redirected to a worker that runs the corresponding service. Effectively, Docker acts as a load balancer for swarm services and there's no guarantee about which node will serve the request when using the routing mesh. Services using the routing mesh are running in virtual IP (VIP) mode.

By setting the `--endpoint-mode` flag to `dnsrr`, you can start a service using DNS Round Robin (DNSRR) mode and bypass the routing mesh. This also means that you have to run your own load balancer in front of the service. For a given service name, the DNSRR mode will return a list of IP addresses for nodes running the service.

### Separate control and data traffic 

By default, control traffic (for swarm management) and data traffic (traffic to and from applications) run over the same network, even though swarm control traffic is AES-encrypted. However, Docker can be configured to use different networks for the different types of traffic:

```shell script
$ docker swarm init --advertise-addr ADDRESS --datapath-addr ADDRESS
```

Defining these two flags separately will separate the traffic types. This has also to be done when joining the swarm.

### 3. Operations for standalone containers on overlay networks

This set of operations only applies to standalone containers.

### Attach a standalone container to a overlay network

The `ingress` network is created without the `attachable` flag so that only swarm services can use it. You can define a custom overlay network and specify the `--attachable` flag, giving standalone containers the ability to communicate across distributed Docker hosts.

### Publish ports

Map TCP port 80 in the container to port 8080 on the overlay network:

```shell script
-p 8080:80
```

Map UDP port 80 in the container to port 8080 on the overlay network:

```shell script
-p 8080:80/udp
```

Map TCP port 80 in the container to port 8080 on the overlay network, and map UDP port 80 in the container to port 8080 on the overlay network:

```shell script
-p 8080:80/tcp -p 8080:80/udp
```

### Container discovery

To get a list of all containers (tasks) backing a service, do a DNS lookup for `tasks.<service-name>`.

### See also

[Use overlay networks](https://docs.docker.com/network/overlay/)
[Networking with overlay networks](https://docs.docker.com/network/network-tutorial-overlay/)