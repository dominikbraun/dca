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