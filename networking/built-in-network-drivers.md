## Built-in network drivers

Which network driver should I use? Each driver offers trade-offs and has different advantages depending on the use case.
There are built-in network drivers that come included with Docker Engine and there are also plugin network drivers
offered by networking vendors and the community.

The most commonly used built-in network drivers are `bridge`, `overlay` and `macvlan`. Together they cover a very broad
list of networking use cases and environments.

### Bridge Network Driver

The `bridge` network driver is a simple to use, simple to troubleshoot and simple to understand network driver, which
makes it a good choice for developers new to Docker. The `bridge` driver creates a private network internal to the host
so containers on this network can communicate. External access is granted by exposing ports to containers.

Behind the scenes, Docker creates the necessary Linux bridges, internal interfaces, iptables rules and host routes to
make this connectivity possible.

Creating a new bridge network and attaching two containers to it is simple:

```shell script
$ docker network create --driver bridge my-net
$ docker container run -d --net my-net redis
$ docker container run -d --net my-net -p 8000:5000 php
```

With no extra configuration, the Docker Engine does the necessary wiring, provides service discovery for the containers
and configures security rules to prevent communication to other networks. A built-in IPAM driver provides the container
interfaces with private IP addresses from the subnet of the bridge network.

The bridge driver als does the service discovery for us automatically if the containers are on the same network. All of
the port mappings, security rules and pipework between Linux bridges is handled for us by the networking driver as
containers are scheduled and rescheduled across a cluster.

The bridge driver is a local scope driver, which means it only provides service discovery, IPAM and connectivity on a
single host. Multi-host service discovery requires an external solution that can map containers to their host location.
This is exactly what makes the `overlay` driver so great.

### Overlay network driver

The built-in `overlay` network driver radically simplifies many of the complexities in multi-host networking. It is a
swarm scope driver, which means that it operates across an entire Swarm or UCP cluster rather than individual hosts.
IPAM, service discovery, multi-host connectivity, encryption and load balancing are built right in. For control, the
`overlay` drier uses the encrypted Swarm control plane to manage large scale clusters at low convergence times.

Creating an overlay network and connecting services to it works like so:

```shell script
$ docker network create --driver overlay my-overlay
$ docker service create --network my-overlay --name mysql mysql
$ docker service create --network my-overlay --name php php:apache
```

The `overlay` network-driver is a feature-rich driver that handles much of the complexity and integration that
organizations struggle with when crafting piecemeal solutions.

### MACVLAN driver

The `macvlan` driver is a very lightweight driver, because rather than using any Linux bridging or port mapping, it
connects container interfaces directly to host interfaces. Containers are addressed with routable IP addresses that are
on the subnet of the external network.

As a result of routable IP addresses, containers communicate directly with resources that exist outside a Swarm cluster
without the use of NAT and port mapping. This can aid in network visibility and troubleshooting. Additionally, the
direct traffic path between containers and the host interface helps reduce latency. `macvlan` is a local scope network
driver which is configured per-host. As a result, there are stricter dependencies between MACVLAN and external networks,
which is both a contraint and an advantage that is different from `overlay` or `bridge`.

The `macvlan` driver uses the concept of a parent interface. This interface can be a host interface such `eth0`, a
sub-interface or even a bonded host adaptor which bundles Ethernet interfaces into a single logical interface. A gateway
address from the external network is required during MACVLAN network configuration, as a MACVLAN network is a L2 segment
from the container to the network gateway. Like all Docker networks, MACVLAN networks are segmented from each other -
providing access within a network, but not between networks.