## Raft consensus in swarm mode

When the Docker engine runs in swarm mode, manager nodes implement a consensus algorithm called _Raft_. A consensus algorithm helps to manage the global cluster state.

The reason why Docker swarm mode is using a consensus algorithm is to make sure that all the manager nodes are storing the same consistent state. Having the same consistent state across the cluster means that in the case of a failure, any Manager node can pick up the tasks and restore the services in order to match the desired state.

A system using consensus algorithms enures that the cluster stays consistent in the presence of failures by requiring a majority of nodes to agree on values. Raft tolerates (N-1)/2 failures and requires a majority or _quorum_ of (N/2)+1 members to agree on values proposed to the cluster.

This means that in a cluster of 5 managers running Raft, if 3 nodes are unavailable, the system cannot process any more requests to schedule additional tasks. Existing tasks keep running, but the scheduler can't rebalance tasks.

Implementing a consensus algorithm means to feature the properties inherent to distributed systems:
* agreement on values in a fault tolerant system
* mutual exclusion through the leader election process
* cluster membership management
* globally consistent object sequencing and campare-and-swap (CAS) primitives

### See also
[Raft consensus in swarm mode](https://docs.docker.com/engine/swarm/raft/)
[Raft: Understandable Distributed Consensus](http://thesecretlivesofdata.com/raft/)
