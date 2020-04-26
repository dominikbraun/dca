## External access for containers

By default, all containers on the same Docker network (multi-host swarm scope or local scope) have connectivity with
each other on all ports. Communications between different Docker networks and container ingress traffic that originates
from outside Docker is firewalled.

For most Docker networks (`bridge` and `overlay` included), external ingress access for applications must be explicitly
granted. This is done through internal port mapping. Docker publishes ports exposed on host interfaces to internal
container interfaces.

Outbound (egress) container traffic is allowed by default. Egress connections initiated by containers are masqueraded/SNATed
to an ephemeral port. Return traffic on this connection is allowed, and thus the container uses the best routable IP
address of the host on the ephemeral port.

Ingress access is provided through explicit port publishing. Port publishing is done by Docker Engine and can be
controlled though UCP or the Engine CLI. A specific or randomly chosen port can be configured to expose a service or
container. All traffic is mapped from this port to a port and interface inside the container. The port can be set to
listen on a specific or on all host interfaces.

External access is configured using `--publish`/`-p` in the Docker CLI or UCP.

````shell script
$ docker run -d --name c2 --net my-bridge -p 5000:80 nginx
````

After running this command, `c2` is connected to the `my-bridge` network. It exposes its service to the outside world
on port `5000` of the host interface (for example `192.168.0.2:5000`). All traffic going through this interface:port is
published to port 80 of the container interface.

Outbound traffic initiated by `c2` is masqueraded so that it is sourced from the ephemeral port on the host interface.
Return traffic uses the same IP address and port for its destination and is masqueraded internally back to the container
address:port. When using port publishing, external traffic on the network always uses the host IP and exposed port and
never the container IP and internal port.
