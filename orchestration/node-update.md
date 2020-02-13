## Node update

`docker node update` updates metadata for a given node.

### Labels

The `docker node update` command allows the addition of labels using `docker node update --label-add KEY=VALUE NODE` as well as the removal of labels using `docker node update --label-rm KEY NODE`.

Labels may be used as deployment constraints when creating a service with `docker service create`: For example, a `type` label can be used to identify nodes where the scheduler should only deploy message queue services tasks:

````shell script
$ docker node update --label-add type=queue worker1
````

Any labels set for nodes apply only to the node entity in the swarm. Don't confuse them with the docker daemon labels. Note that it is also possible to specify a label as a key with an empty value: `docker node update --label-add KEY NODE`.

### Availability

The `--availability` option indicates the availability of the node. Valid values are `active`, `pause` or `drain`.

### Role

The `--role` option defines the role of the node. Valid values are `manager` or `worker`.
