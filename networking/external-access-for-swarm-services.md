## External access for swarm services

You can expose services externally by using the `--publish` flag when creating or updating the service. Publishing ports
in Docker Swarm mode means that every node in your cluster is listening on that port. But what happens if the service's
task isn't on the node that is listening on that port?

This is where routing mesh comes into play. Routing mesh leverages operating system primitives (IPVS+iptables on Linux
and VFP on Windows) to create a powerful cluster-wide transport layer (L4) load balancer. It allows the Swarm nodes to
accept connections on the services' published ports. When any Swarm node receives traffic destined to the published TCP/UDP
port of a running service, it forwards it to the service's VIP using a pre-defined overlay network called `ingress`.

The `ingress` network behaves similarly to other overlay networks but its sole purpose is to provide inter-host
transport for mesh routing traffic from external clients to cluster services.

Once you launch services, you can create an external DNS record for your applications and map it to any or all Docker
Swarm nodes. You do not need to know where the container is running as all nodes in your cluster look as one with the
routing mesh routing feature.

This is how routing mesh works:
* A service is created with two replicas. It is port-mapped externally to port `8000`.
* The routing mesh exposes port `8000` on each host in the cluster.
* Traffic destined for the service can enter on any host. In this case, the external load balancer sends the traffic
to a host without a service replica.
* The kernel's IPVS load balancer redirects traffic on the `ingress` overlay network to a healthy service replica.

### See also
[Swarm External L4 Load Balancing (Docker Routing Mesh)](https://success.docker.com/article/networking#swarmexternall4loadbalancingdockerroutingmesh)
