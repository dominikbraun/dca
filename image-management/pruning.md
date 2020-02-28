## Pruning

Docker takes a conservative approach to cleaning up unused objects, such as images, containers, volumes and networks:
These objects are generally _not_ removed unless you explicitly ask Docker to to so.

For each type of object, Docker provides a `prune` command.

### Prune images

The `docker image prune` command allows you to clean up unused images. By default, it only cleans up _danging images_.
A dangling image is one that is not tagged and is not referenced by any container. To remove dangling images:

```shell script
$ docker image prune

  WARNING! This will remove all dangling images.
  Are you sure you want to continue? [y/N] y
```

To remove all images which are not used by existing containers, use the `-a` flag. This will prompt you to continue, but
you can bypass that prompt with the `--force` or `-f` flag.

It is also possible to filter the images that will be pruned. For example, to prune all dangling images created more
than 24 hours ago, run `docker image prune -a --filter="until=24h"`.

### Prune containers

When you stop a container, it is not automatically removed unless you specified the `--rm` flag before. A stopped
container's writeable layer will still take up disk space. You can prune those containers like so:

```shell script
$ docker container prune
``` 

This will remove all stopped containers. You can bypass the confirmation prompt using the `--force` flag again. Filters
are applicable just like with `docker image prune`.

### Prune networks

Docker networks don't take up much disk space, but they do create `iptable` rules, bridge network devices and routing
table entries. The `docker network prune` command cleans up networks that aren't used by any containers.

This command allows the same `--force` and `--filter` flags as the other `prune` commands.

### Prune everything

There's a shortcut for pruning all these objects at once: `docker system prune`. For Docker 17.06.1 and higher, you have
to provide the `--volumes` flag to prune volumes as well.

Just as the other `prune` commands, you'll be prompted to continue. You can bypass that prompt using the `--force` or
`f` flag.
