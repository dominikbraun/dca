## Container vs. Service

### Difference between running a Service and running a Container

`docker run` creates a writeable container layer over the specified image and starts the container. Internally, the API endpoints `/containers/create` and `/containers/{id}/start` are used for this.

`docker service create` creates a microservice within the context of a larger application, based on the specified image. When creating a service, you have to specify which image to use for the service and which commands to execute inside its containers.
Next to that, there are some options for the service itself:
* The public port of the service exposed by the swarm
* The overlay network for the service for connecting to other services
* The policy for rolling updates
* The number of image replicas to run
* CPU and memory limits

### See also
* service-create.md
* [Deploy a service to the swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/deploy-service/)
* [docker service is the new docker run](https://events.static.linuxfound.org/sites/events/files/slides/ContainerCon%20Berlin%20%28Goelzer%29%20-%20Upload%209-18-2016.pdf)
