## Services

To deploy an application image when the Docker engine is in swarm mode, you create a service. Frequently, a service is the image for a microservice within the context of some larger application.

When you create a service, you specify the container image to be used and which commands to execute inside the containers. There are also some options when creating a service:
* the port where the swarm makes the service available outside the swarm
* an overlay network for the service to connect other services in the swarm
* the number of replicas of the image to run in the swarm
* a rolling update policy
* CPU and memory limits

### Services, tasks and containers

When a new service gets deployed to the swarm, the swarm manager considers the service definition to be the desired state of the service. After that, the manager schedules the services on nodes in the swarm: For the nodes' point of view, this are replica tasks. They run independently of each other.

 In the swarm mode model, each task invokes exactly one container.  A task is analogous to a slot where the scheduler places a container. Once the container is up and running, the scheduler recognizes that the task is in a running state. The task will be terminated if the container fails health checks or terminates.
 
### Tasks and scheduling

*A task is the smallest unit of scheduling in a swarm*. When you declare a desired service state by creating or updating a service, the orchestrator realizes the desired state by scheduling tasks.

For instance: You define a service that instructs the orchestrator to keep 3 instances of an HTTP listener running all the times. The orchestrator responds by creating 3 tasks. Each task is a slot that the scheduler fills by spawning a container. You can think of the container as an instantiation of the task. If the HTTP listener fails its health checks or terminates, the orchestrator creates a new replica task and the scheduler spawns a new container.

Note that a task is a one-way mechanism. It progresses monotonically through a fixed series of states: assigned, prepared, running etc. - and if it fails at any point, the orchestrator removes the task and its container and creates a new task to replace it. This all happens in order to achieve the desired state specified by the service.

### Pending services

...
