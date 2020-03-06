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

(Table)

### See also
* [Recovering from losing the quorum](https://docs.docker.com/engine/swarm/admin_guide/#recover-from-losing-the-quorum)