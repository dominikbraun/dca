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

> Fault Tolerance = Swarm Size - Quorum
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

An _unreachable_ health status means that this particular manager is unreachable from other manager nodes. In this case
you need to take action to restore the unreachable manager:
* Restart the daemon and see if the manager comes back as reachable
* Reboot the machine
* Add another manager node or promote a worker to be a manager node. If you do this, you need to cleanly remove the
failed node entry from the manager set using `docker node demote NODE` and `docker node rm NODE`.

Alternatively, you can also get an overview of the swarm health from a manager node with `docker node ls`.

### Troubleshoot a manager node

You should _never_ restart a manager node by copying the `raft` directory from another node. The data directory is
unique to a node ID. A node can only use a node ID once to join the swarm. The node ID space should be globally unique.

To cleanly re-join a manager node to a cluster:
* To demote the node to a worker, run `docker node demote NODE`.
* To remove the node from the swarm, run `docker node rm NODE`.
* Re-join the node to the swarm with a fresh state using `docker swarm join`.

### Forcibly remove a node

In most cases, you should shut down a node before removing it from a swarm. If a node becomes unreachable, unresponsive
or compromised you can forcefully remove the node without without it down by passing the `--force` flag. For instance,
if `node9` becomes compromised:

```shell script
$ docker node rm node9

Error response from daemon: rpc error: code = 9 desc = node node9 is not down and can't be removed

$ docker node rm --force node9

  Node node9 removed from swarm
```

```shell script
$ docker node rm --force node9

  Node node9 removed from swarm
```

### Back up the swarm

Docker manager nodes store the swarm state and manager logs in the `/var/lib/docker/swarm` directory. In 1.13 and
higher, this data includes the keys used to encrypt the Raft logs. Without these keys, you cannot restore the swarm.

You can back up the swarm using any manager. Use the following procedure:
1. If the swarm has auto-lock enabled, you need the unlock key to restore the swarm from backup. Retrieve the unlock key
if necessary and store it in a safe location.
2. Stop Docker on the manager before backing up the data, so that no data is being changed during the backup. It is
possible to take a backup while the manager is running (a _hot backup_), but this is not recommended and your results
are less predictably when restoring. While the manager is down, other nodes continue generating swarm data that is not
part of the backup.

> Be sure to maintain the quorum of swarm managers. During the time that a manager is down, your swarm is more
> vulnerable to losing the quorum if further nodes are lost. The number of managers you run is a trade-off. If you
> regularly take down managers to do backups, consider running a five manager swarm so that you can lose an additional
> manager while the backup is running.

3. Back up the entire `/var/lib/docker/swarm` directory.
4. Restart the manager.

### Restore from a backup

After backing up the swarm, use the following procedure to restore the data to a new swarm:
1. Shut down Docker on the target host machine for the restored swarm.
2. Remove the contents of the `/var/lib/docker/swarm` directory.
3. Restore the `/var/lib/docker/swarm` directory with the contents of the backup.
4. Start Docker on the new node. Unlock the swarm if necessary and re-initialize the swarm using the following command:

```shell script
$ docker swarm init --force-new-cluster
```

This prevents the node from attempting to connect to nodes that were part of the old swarm.

5. Verify that the state of the swarm is as expected (for example by checking if all services are present).
6. If you use auto-lock: Rotate the unlock key.
7. Add manager and worker nodes to your swarm and reinstate your previous backup regimen on the new swarm.

### Recover from losing the quorum

Swarm is resilient to failures and the swarm can recover from any number of temporary node failures or other transient
errors. However, a swarm cannot automatically recover if it loses quorum. Tasks on existing worker nodes continue to
run, but administrative tasks are not possible, including scaling or updating services and joining or removed nodes form
the swarm. **The best way to recover is to bring the missing manager nodes back online.** If that is not possible,
consider the following:

In a swarm of `n` managers, a quorum of manager nodes must always be available. For example, in a swarm with five
managers, a minimum of three must be operational and in communication with each other. In other words, the swarm can
tolerate up to `(n-1)/2` permanent failures beyond which requests involving swarm management cannot be processed.

If you lose the quorum of managers, you cannot administer the swarm. If you have lost the quorum and you attempt to
perform any administrative management operation on the swarm, an error occurs:

```
Error response from daemon: rpc error: code = 4 desc = context deadline exceeded
```

The best way to recover from quorum is to bring the failed nodes back online. If you can't do that, the only way to
recover from this state is to use the `--force-new-cluster` action from a manager node. THis removes all managers except
the manager the command was run from. The quorum is achieved because there is now only one manager. Promote nodes to be
managers until you have the desired number of managers.

### Force the swarm to re-balance

Generally, you do not need to force the swarm to re-balance tasks. When you add a new node to a swarm, or a node
reconnects to the swarm, the swarm does not automatically give a workload to the idle node. This is a design decision.
If the swarm periodically shifted tasks to different nodes for the sake of balance, the clients using those tasks would
be disrupted. The goal is to avoid disrupting running services. When new tasks start, or when a node with running tasks
becomes unavailable, those tasks are given to less busy nodes. The goal is eventual balance, with minimum disruption to
the end user.

With Docker 1.13 and higher, you can use the `--force` or `-f` flag with the `docker service update` command to force
the service to redistribute its tasks across the available worker nodes. THis causes the service tasks to restart.
Client applications may be disrupted. If you have configured it, your service uses a rolling update.

If you use an earlier version and you want to achieve an even balance of load across workers and don't mind disrupting
running tasks, you can force your swarm to re-balance by temporarily scaling the service upward. Use
`docker service inspect --pretty SERVICE` to see the configured scale of a service. When you use `docker service scale`,
the nodes with the lowest number of tasks are targeted to receive new workloads. There may be multiple under-loaded
nodes in you swarm. You may need to scale the service up by modest increments a few times to achieve the balance you
want across all the nodes.

When the load is balanced to your satisfaction, you can scale the service back down to the original scale. You can use
`docker service ps SERVICE` to assess the current balance of your service across nodes. 

### See also
* [Recovering from losing the quorum](https://docs.docker.com/engine/swarm/admin_guide/#recover-from-losing-the-quorum)
* [Recover from disaster](https://docs.docker.com/engine/swarm/admin_guide/#recover-from-disaster)
* [Add nodes to a swarm](https://docs.docker.com/engine/swarm/join-nodes/)
