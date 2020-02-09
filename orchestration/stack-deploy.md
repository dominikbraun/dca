## Docker stack deploy

When running Docker in swarm mode, `docker stack deploy` allows the deployment of a complete application stack to the swarm. The `stack` command supports any Compose file of version 3.0 or higher as a stack description.

### Set up a registry

Since there are multiple Docker Engines involved in a swarm, a common registry is required.

```shell script
$ docker service create --name registry --publish published=5000,target=5000 registry:2
``` 

This creates an own registry service that exposes port 5000.


### Test a python app with Compose

The python application in /examples/python-app is described by `docker-compose.yml`. Running `docker-compose up -d` pulls the images from the specified registries or builds them via Dockerfile and launches the containers.

```shell script
$ docker-compose up -d

WARNING: The Docker Engine you're using is running in swarm mode.

Compose does not use swarm mode to deploy services to multiple nodes in
a swarm. All containers are scheduled on the current node.

To deploy your application across the swarm, use `docker stack deploy`.

Creating network "stackdemo_default" with the default driver
Building web
...(build output)...
Creating stackdemo_redis_1
Creating stackdemo_web_1
```

After verifying that the app runs using `curl localhost:8000`, we can bring the python app down again.

```shell script
$ docker-compose down

Stopping python-app_web_1   ... done
Stopping python-app_redis_1 ... done
Removing python-app_web_1   ... done
Removing python-app_redis_1 ... done
Removing network python-app_default
```

### Distribute the image across the swarm

To make the generated python app's image available to all nodes in the swarm, it has to be uploaded to the registry service. Using Compose, this is quite easy:

```shell script
$ docker-compose push

Pushing web (localhost:5000/python-app:latest)...
The push refers to repository [localhost:5000/python-app]
e11aab1fce5a: Pushed
8ccf582aa48d: Pushed
62de8bcc470a: Pushed
58026b9b6bf1: Pushed
fbe16fc07f0d: Pushed
aabe8fddede5: Pushed
bcf2f368fe23: Pushed
latest: digest: sha256:ccc4f2b9c8fb3f7f9f41303d1fc82032cab225df9308c99437e8ed18c2d757ac size: 1786
```

Now that the images (the referred registry localhost:5000 comes from `docker-compose.yml`), the whole stack can be deployed to the swarm.

### Deploy the stack

Deploying the whole stack to the swarm is fairly simple:

```shell script
$ docker stack deploy --compose-file examples/python-app/docker-compose.yml python-app

Ignoring unsupported options: build

Creating network python-app_default
Creating service python-app_redis
Creating service python-app_web
```

Each volume, network and service name will be prefixed with the stack name.

To list all services running in a stack, use `stack services STACK`:

```shell script
$ docker stack services python-app

ID                  NAME                MODE                REPLICAS            IMAGE                              PORTS
bkae4i3e554u        python-app_web      replicated          1/1                 localhost:5000/python-app:latest   *:8000->8000/tcp
lyr8xsniezt2        python-app_redis    replicated          1/1                 redis:alpine
```

Of course, these services also appear in `docker service ls`. The stack merely groups those services.

To bring the stack down, use `stack rm STACK`.

```shell script
$ docker stack rm python-app

Removing service python-app_redis
Removing service python-app_web
Removing network python-app_default
```