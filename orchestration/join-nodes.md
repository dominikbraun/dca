## Join nodes to the swarm

After creating the swarm and its manager node (see swarm-setup.md), we're able to add worker nodes.

### Join a worker node

To add the current node to a swarm, use the output produced by `docker swarm init`:

```shell script
docker swarm join --token SWMTKN-1-461bx8vdgas19hp9v0m74u7wpprnfk55wwuirf5evhy9wix8yk-ccg5aettadmsm7otgbzzqpg6y 192.168.188.28:2377
```

A new token for joining the swarm can be retrieved from any manager node:

```shell script
docker swarm join-token worker1
```

### Check the swarm's nodes

Just like before, `docker node ls` will display relevant information for all joined nodes. Just like `docker swarm join-token`, this command has to be executed on a manager node.

```shell script
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
03g1y59jwfg7cf99w4lt0f662    worker2   Ready   Active
9j68exjopxe7wfl6yuxml7a7j    worker1   Ready   Active
dxn1zf6l61qsb1josjja83ngz *  manager1  Ready   Active        Leader
```

### See also
(Add nodes to the swarm)[https://docs.docker.com/engine/swarm/swarm-tutorial/add-nodes/]