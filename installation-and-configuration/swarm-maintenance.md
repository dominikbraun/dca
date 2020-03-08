## Swarm maintenance

When you run a swarm of Docker Engines, _manager nodes_ are the key components for managing the swarm and storing the
swarm state. It is important to understand some key features of manager nodes to properly deploy and maintain a swarm.

### Operating manager nodes

Swarm manager nodes use the Raft Consensus Algorithm to manage the swarm state. You only need to understand some general
concepts of Raft in order to manage a swarm.

There is no limit on the number of manager nodes. The decision about how many manager nodes to implement is a trade-off
between performance and fault-tolerance: Adding a manager node makes the swarm more fault-tolerant. However, additional
manager nodes reduce write performance because more nodes must acknowledge proposals to update the swarm state. This
means more network round-trip traffic.

Raft requires a majority of managers, also called the _quorum_, to agree on proposed updates on the swarm, such as node
additions or node removals. Membership operations are subject to the same constraints as state replication.

**Maintain the quorum of managers**

* If the swarm loses the quorum of managers, the swarm cannot perform management tasks.
* If the swarm has multiple managers, always have more than two.
* To maintain quorum, a majority of managers must be available.
* An odd number of managers is recommended, because the next even number does not make the quorum easier to keep.
* If you have 3 or 4 managers, you can loose 1 manager. If you have 5 or 6 managers, you can loose 2 managers.

Even if a swarm loses the quorum of managers, swarm tasks on existing worker nodes continue to run. However, swarm nodes
cannot be added, updated or removed, and new or existing tasks cannot be started, stopped, moved or updated.

### Configure the manager to advertise on a static IP address

When initializing a swarm you must specify the `--advertise-addr` flag to advertise your address to other manager nodes
in the swarm. Because manager nodes are meant to be a stable component of the infrastructure, you should use a fixed IP
address for the advertise address to prevent the swarm from becoming unstable on machine reboot.

If the whole swarm restarts and every manager node gets a new IP address, there is no way for any node to contact an
existing manager. The swarm is hung while nodes try to contact one another at their old IP addresses.

Dynamic IP addresses are OK for worker nodes.

### Add manager nodes for fault tolerance

You should maintain an odd number of managers in the swarm to support manager node failures. Having an odd number of
managers ensures that during a network partition, there is a higher chance that the quorum remains available to process
requests if the network is partitioned into two sets. Keeping the quorum is not guaranteed if you encounter more than
two network partitions.

* Swarm Size = 1, Majority = 1 -> Fault Tolerance = 0
* Swarm Size = 2, Majority = 2 -> Fault Tolerance = 0
* Swarm Size = 3, Majority = 2 -> Fault Tolerance = 1
* Swarm Size = 4, Majority = 3 -> Fault Tolerance = 1
* Swarm Size = 5, Majority = 3 -> Fault Tolerance = 2
* Swarm Size = 6, Majority = 4 -> Fault Tolerance = 2
* Swarm Size = 7, Majority = 4 -> Fault Tolerance = 2
* Swarm Size = 8, Majority = 5 -> Fault Tolerance = 3
* Swarm Size = 9, Majority = 5 -> Fault Tolerance = 4

> Fault Tolerance = Swarm Size - Majority
> => Fault Tolerance = Swarm Size - (floor(Swarm Size / 2) + 1)

For example, in a swarm with 5 nodes, if you lose 3 nodes, you don't have a quorum. Therefore you can't add or remove
nodes until you recover one of the unavailable manager nodes or recover the swarm with disaster recovery commands.

While it is possible to scale a swarm down to a single manager node, it is impossible to demote the last manager node.
This ensures that you maintain access to the swarm and that the swarm can still process requests. Scaling down to a
single manager node is an unsafe operation and is not recommended. If the last node leaves the swarm unexpectedly during
the demote operation, the swarm becomes unavailable until you reboot the node or restart with `--force-new-cluster`.

You manage a swarm membership with the `docker swarm` and `docker node` subsystems.

### Distribute manager nodes

In addition to maintaining an odd number of manager nodes, pay attention to datacenter topology when placing managers.
For optimal fault-tolerance, distribute manager nodes across a minimum of 3 availability-zones to support failures of an
entire set of machines or common maintenance scenarios.

If you suffer a failure in one of these zones, the swarm should maintain the quorum of manager nodes available to
process requests and re-balance workloads.

* Manager nodes = 3 -> Repartition (on 3 availability-zones) = 1-1-1
* Manager nodes = 5 -> Repartition (on 3 availability-zones) = 2-2-1
* Manager nodes = 7 -> Repartition (on 3 availability-zones) = 3-2-2
* Manager nodes = 9 -> Repartition (on 3 availability-zones) = 3-3-3

### Run manager-only nodes

By default, manager nodes also act as worker nodes. This means the scheduler can assign tasks to a manager node. For
small and non-critical swarms, assigning tasks to managers is relatively low-risk as long as you schedule services using
resource constraints for CPU and memory (otherwise, these services may consume to much CPU/memory and the manager nodes
don't have any resources left for actual swarm management).

However, because manager nodes use the Raft consensus algorithm to replicate data in a consistent way, they re sensitive
to resource starvation. **You should isolate managers in your swarm from processes that might block swarm operations
like swarm heartbeat or leader elections.**

To avoid interference with manager node operations, you can drain manager nodes to make them unavailable as worker
nodes:

```shell script
$ docker node update --availability drain NODE
```

When you drain a node, the scheduler re-assigns any tasks running on the node to other available worker nodes in the
swarm. It also prevents the scheduler from assigning tasks to the node.

### Add manager nodes for load balancing

Add nodes to the swarm to balance your swarm's load. Replicated service tasks are distributed accross the swarm as evenly as possible over time, as long as the worker nodes are matches to the requirements of the services.

When limiting a service to run only specific types of nodes, remember that worker nodes that do not meet that requirements cannot run these tasks.

### Monitor swarm health

You can monitor the health of manager nodes by querying the docker nodes API in JSON format through the `/nodes` HTTP endpoint. From the command line, run `docker node inspect NODE` to query the nodes. For instance, to query the reachability of the node as a manager:

```shell script
$ docker node inspect manager1 --format "{{.ManagerStatus.Reachability}}"

  reachable
```

To query the status of the node as a worker that accepts tasks:

```shell script
$ docker node inspect manager1 --format "{{.Status.State}}"

ready
```

For those commands, we can see that `manager1` is both at the status `reachable` as a manager and `ready` as a worker.

An _unreachable_ health status means that this particular node ...

### See also
* [Recovering from losing the quorum](https://docs.docker.com/engine/swarm/admin_guide/#recover-from-losing-the-quorum)
* [Recover from disaster](https://docs.docker.com/engine/swarm/admin_guide/#recover-from-disaster)
* [Add nodes to a swarm](https://docs.docker.com/engine/swarm/join-nodes/)
