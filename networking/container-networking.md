## Container networking

The type of network a container uses is transparent from within the container (be it a bridge, overlay or macvlan network). From the container's point of view, it has an IP address, a gateway, a routing table, DNS services and other networking details, assuming it does not use the `none` network driver.

### Published ports

By default, a container does not publish any of its ports to the outside world. To make a port available to services outside of Docker or to containers that are not in the network, use the `--publish` or `-p` flag.

**Publishing a port using the `--publish` flag will create a firewall rule that maps a container port to a port on the Docker host. The port will be available to services outside of Docker or to Docker containers that are not connected to the container's network.

Map TCP port 80 in the container to port 8080 on the Docker host:

```shell script
--publish 8080:80
```

Map TCP port 80 in the container to port 8080 on the Docker host for connections to host IP 192.168.1.100:

```shell script
--publish 192.168.1.100:8080:80
```

Map UDP port 80 in the container to port 8080 on the Docker host:

```shell script
--publish 8080:80/udp
```

Map TCP port 80 in the container to TCP port 8080 on the Docker host, and map UDP port 80 in the container to UDP port 8080 on the Docker host:

```shell script
--publish 8080:80/tcp --publish 8080:80/udp
```

### IP address and hostname

By default, the container is assigned an IP address for every Docker network it connects to. The IP address is assigned from the pool assigned to the network, so the Docker daemon acts as a DHCP server.

When the container starts, it can only be connected to a single network using `--network`. However, a running container can be connected to multiple networks using `docker network connect`. The `--ip` or `--ip6` flag can be used to assign an IP from the corresponding network to the container for both `container create` and `network connect`.

The container's default hostname is its ID in Docker. The hostname can be overridden using the `--hostname` flag. When connecting a container to a network with `docker network connect`, the `--alias` flag allows the definition of an alias in that network.

### DNS services

By default, a container inherits the DNS settings of the Docker daemon, including `/etc/hosts` and `/etc/resolv.conf`. These setting can be overriden:
* `--dns` specifies the IP of the DNS server - may be used multiple times. If no specified DNS server can be reached, Google's public DNS server `8.8.8.8` will be added.
* `--dns-search` specifies a DNS search domain to search non-fully-qualified hostnames - may be used multiple times.
* `--dns-opt` specifies a DNS option and its value. Valid options can be found in the operating system's documentation for `/etc/resolv.conf`.
* `--hostname` specifies the container's hostname and defaults to the container ID. 

### See also
[Container networking](https://docs.docker.com/config/containers/container-networking/)