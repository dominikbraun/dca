## Troubleshooting container networking

### Issue

Typically, container-container networking issues manifest themselves as intermittent
connection issues. They could also manifest as a complete failure of container-container
networking.

### Prerequisites

First, determine the names of the affected Docker swarm services, networks and hosts.

Identify a Docker host from which network requests are not answered correctly. If the requests are submitted through the
ingress network, then to a frontend service, then via another network to a backend service, then start troubleshooting
by splitting the problem in half and using `netshoot` to connect from the frontend directly to the backend service.

To troubleshoot container-container networking issues, start by examining the user-defined network. If the problem isn't
found, move to troubleshooting the ingress network.

### Troubleshooting an user-defined network

1. On the host where the frontend container is running, start a netshoot container reusing the network namespace
affected container:

```shell script
$ docker run -it --rm --network container:<CONTAINER> nicolaka/netshoot
```

2. Look up all backend IP addresses by DNS name: DNS names are created for containers and services and are scoped to
each overlay network the container or service attaches to. Standalone containers use the container name for the
hostnames. Looking up the name of a service returns the IP of the service's load balancing VIP. To look up the IP of
each task created by a service, use `task.service_name` as the domain name. For example, to look up the IP addresses for
the backend service, use:

```shell script
$ nslookup tasks.backend_service_name
```

3. Issue a netcat TCP test to each backend task IP address on the port where it should be listening:

`nc -zvw2 $backend_ip <listening_port>`

4. If no connections fail but requests submitted via the ingress framework continue to have problems, move to
troubleshooting the ingress network. If no connections fail, and the issue has only been seen container-container, then
check another set of services or hosts until one task fails to reach another.

5. For any backend IP addresses reported as failed, do a reverse name lookup of the affected IP to determine its service
name and task ID:

```shell script
$ nslookup <ip_address>
```

Exit the netshoot container and collect `docker network inspect -v` against the network between the containers.

7. On a manager, for the set of all failed service names and tasks, collect the following:

```shell script
$ docker service ps <service>
$ docker inspect --type task <task>
```

8. For tasks that are still present, note their `Created` and `Updated` times. Collect daemon logs from this time frame
from the netshoot host, all managers, and hosts of unresponsive tasks as identified by their HostIP.

9. Collect the output of `docker run -v /var/run:/var/run  --network host --privileged dockereng/network-diagnostic:support.sh`
from all hosts in the network path.

### Troubleshooting the ingress network

Service discovery is disabled on the ingress network for security reasons, so you can't use `nslookup tasks.service` to
establish which backend IPs to test. Instead use the ipvs load balancer programming of the kernel.

1. On a manager, use `docker service inspect` to identify the VIP for the service on the ingress network:

```shell script
$ ingress_id=$(docker network ls -qf name=ingress --no-trunc); docker service inspect <service_name> --format '{{range .Endpoint.VirtualIPs}}{{if eq "'${ingress_id}'" .NetworkID}}{{.Addr}}{{end}}{{end}}'
```

2. Using `curl`, identify a Docker host from which network requests to the ingress published port of the service are not
answered correctly.

...

### See also
[Troubleshooting container networking](https://success.docker.com/article/troubleshooting-container-networking)