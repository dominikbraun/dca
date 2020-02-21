## Private registry

A registry is an instance of the `registry` image and runs within Docker.

### Running a local registry

In order to run a local registry, pull the `registry:2` image and start a container from it.

```shell script
$ docker container run -d -p 5000:5000 --rm --restart=always --name local-registry registry:2
```

This will start a registry instance that will be available under `localhost:5000`.

### Copy an image to a local registry

It is possible to pull an image from Docker Hub and push it to the local registry. First of all, pull the image:

```shell script
$ docker image pull ubuntu:18.04
```

Then tag the image, prefixing it with `localhost:5000`. A hostname and an optional port indicates an image registry.

```shell script
$ docker image tag ubuntu:18.04 localhost:5000/ubuntu:18.04
```

Since the image tag starts with a hostname and a port, Docker interprets this as a registry address when pushing the
image:

```shell script
$ docker image push localhost:5000/ubuntu
``` 

After that, we can remove the local images. This won't remove the image from the local registry.

```shell script
$ docker image remove ubuntu:18.04
$ docker image remove localhost:5000/ubuntu:18.04
```

The image is stored in the local registry, so it can be pulled down at any time.

```shell script
$ docker image pull localhost:5000/ubuntu:18.04
```

### Stopping the registry

The local registry can be stopped like any other container.

```shell script
$ docker container stop registry
```

### Customizing the registry storage

By default, all registry data is persisted as a docker volume on the host filesystem. However, it may also be useful to
create a bind mount on the local filesystem to store the data. In this example all data is persisted at `/mnt/registry`:

```shell script
$ docker container run -d \
  -p 5000:5000
  --rm
  --restart=always
  --name local-registry
  --volume /mnt/registry:/var/lib/registry
  registry:2
```

### Running a public available registry

Running a externally accessible registry requires you to secure the registry with TLS.

First, create a `certs` directory. Copy the `.crt` and `.key` files into it. Start the registry and mount the `cert`
directory in the container:

```shell script
$ docker container run -d \
  --restart=always \
  --name registry \
  --volume "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  -p 443:443
  registry:2
```

Note that this sets also the environment variables for the HTTP address, TLS certificate path and TLS key path. Instead
of mapping port 5000, port 443 for TLS will be used here.

### Adding an authentication

Registries should always implement access restrictions, maybe except for registries running on a secure local network.

The most basic and most simple authentication is through `htpasswd`. Just create the `htpasswd` file inside a directory:

```shell script
$ mkdir auth && echo "testuser test123" >> auth/htpasswd 
```

Start the registry container with basic authentication:

```shell script
$ docker container run -d \
  -p 5000:5000
  --restart=always \
  --name registry \
  --volume "$(pwd)"/auth:/auth
  -e "REGISTRY_AUTH=htpasswd"
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm"
  --volume "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key
  registry:2
```

Pushing or pulling an image will fail. Log in to the registry first:

```shell script
$ docker login localhost:5000
```

After providing a correct username and password, you can push and pull images from the registry. Is is also possible to
use Apache or nginx as authentication proxy in front of the registry.

### Running the registry as a swarm service

A service provides several advantages over standalone containers. The use a declarative model, which means that you
define the desired state and Docker works to keep your service in tha state. Also, services provide automatic load
balancing and scaling as well as the ability to control the distribution of service instances across the swarm nodes.

Whether you use a fully scaled service or a service with a single node or node constraints is determined by the storage
backend you use.
* A distributed storage driver like Amazon S3 allows you to create a fully scaled service. There won't be write
conflicts if multiple workers write to the storage backend.
* A local bind mount or volume implies that each worker node writes to its own storage location. This means that each
registry contains a different data set. To solve that problem, use a single-replica service and a node constraint to
ensure that only a single worker is writing to the bind mount.

First, save the TLS certificate and the TLS key as secrets.

```shell script
$ docker secret create domain.crt certs/domain.crt
$ docker secret create domain.key certs/domain.key
```

Next up, add a label to the node where you want to run the registry. You can retrieve the node name with
`docker node ls`.

```shell script
$ docker node update --label-add registry=true my-node
```

Create the service and grant it access to the two secrets. Also, constrain it to only run on worker nodes with the label
`registry=true`.

```shell script
$ docker service create \
  --name registry \
  --secret domain.crt \
  --secret domain.key \
  --constraint 'node.labels.registry==true' \
  --mount source=/mnt/registry,destination=/var/lib/registry,type=bind \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/run/secrets/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/run/secrets/key.crt \
  --publish pushlished=443,target=443 \
  --replicas 1 \
  registry:2
```

You can access the service on port 443 of any swarm node. Docker will forward the request on the node which is running
the service.

### 
