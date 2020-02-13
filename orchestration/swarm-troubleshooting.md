## Swarm troubleshooting

### Viewing Software as a Stack

Issues are usually discovered at the top of the stack so errors can be tracked down from there. Tracing through to the other end will reveal the underlying cause or resulting effects. Some examples:
* Physical node failures
* Local storage volume filling to capacity
* OS kernel panics
* Exhaustion of file descriptors

### Viewing Software as an Onion

Inspecting software components is like peeling an onion, starting from outside and working your way in. In Docker, there are many layers of encapsulation: the OS kernel, containers, tasks that encapsulate containers as units of work, services or Pods that represent application components, and stacks that represent full applications. All of these are first class objects in Swarm und Kubernetes and can be inspected independently.

### Viewing Software as a Flow

Many issues in distributed computing can be traced through multiple disparate components (like hosts or containers connected via an application). These processes can be viewed as flows. When troubleshooting, the full end-to-end path should be considered.

Examples of issues that can be viewed as flows:
* Network partitions
* DNS resolution
* Distributed multi-container application issues
* Loss of quorum for consensus-based systems like Swarm or Kubernetes

### Docker troubleshooting Best Practices

### Using a Client Bundle

The UCP client bundle sets the context for Docker engine and kubectl. When interacting with a UCP cluster, you should use the Client Bundle:
* Client Bundles apply your account permissions so that you're prevented from causing harm to infrastructure containers.
* Client Bundles allow you to run commands that apply to the entire cluster.

### Troubleshooting from the Service Inward

The Software as an Onion principle allows troubleshooting the first class swarm objects from the outside in. The general flow from the outside working in is:
* `docker service ls`
* `docker service ps SERVICE`
* `docker service inspect SERVICE`
* `docker inspect TASK`
* `docker container inspect CONTAINER`
* `docker container logs CONTAINER`

**docker service ls** prints a full list of services running in the swarm. The command is helpful for showing if the desired amount of replicas are running.

**docker service inspect** prints the specified configuration values for a service. `CreatedAt` and `UpdatedAt` indicate whether the service was deployed right around when the issue started. Also take a look at the `Labels` since they may contain scheduling constraints.

**docker service ps SERVICE** displays a list of tasks for a given service. This is helpful in determining if there were failures in the past or if tasks are currently running or restarting. A large list of failed tasks and tasks that are being recreated and exiting frequently indicate a restart loop: The container is coming up, exiting, and being rescheduled again in quick succession.

**docker inspect TASK**: Using the Task IDs from `docker service ps` we can pull out the container IDs to identify which container belongs to a given service task. Using that ID, we can retrieve the container logs or view information about the container information.

**docker container inspect CONTAINER** prints all container configuration values. The most important configuration keys provide useful information:
* `ContainerState`: `StartedAt` and `FinishedAt` are useful for understanding the timeline of issues. `ExitCode` and `Error` provide information about why a container has exited. `OOMKilled` indicates memory pressure issues.
* `Node`: `IP` displays the IP addresses if any host interfaces, `Cpu` and `Memory` indicate host resources.
* `NetworkSettings`: `SandboxKey` indicates the name and location of the network namespace used by the container. Also take a look at `IPAddress`, `Ports` and `Networks`.