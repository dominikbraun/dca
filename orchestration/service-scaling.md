## Service scaling

As soon as a service is deployed to the swarm (see service-create.md), the number of containers associated with that service can be scaled via CLI. Containers running in a service are referred to as _tasks_.

Scaling is done via the `docker service scale SERVICE=N` where `N` is the desired number of containers. The `service scale` command has to be executed on a manager node.

```shell script
$ docker service scale pinger=4

pinger scaled to 4
overall progress: 4 out of 4 tasks 
1/4: running   [==================================================>] 
2/4: running   [==================================================>] 
3/4: running   [==================================================>] 
4/4: running   [==================================================>] 
verify: Service converged
```

You can verify the individual replicas and their executing nodes with `docker service ps SERVICE` again.

```shell script
$ docker service ps pinger

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
kftvm0dvg1g8        hello-swarm.1       alpine:latest       linux               Running             Running 25 minutes ago                           
ih2ifbn8vol1        hello-swarm.2       alpine:latest       linux               Running             Running about a minute ago                       
ehhewacge329        hello-swarm.3       alpine:latest       linux               Running             Running about a minute ago                       
tqmz322bxrn2        hello-swarm.4       alpine:latest       linux               Running             Running about a minute ago
```

These are 3 new tasks, scaling the service up to 4 instances while being distributed among the swarm's nodes.

### See also
* [Scale the service in the swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/scale-service/)
* [Services, tasks, and containers](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/#services-tasks-and-containers)