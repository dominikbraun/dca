## Registry configuration

`registry:2` comes with useful default configuration values for its YAML configuration by default. However, these
values should be reviewed exhaustively before moving the registry into production.

### Override specific configuration values

In a typical setup where you run the official registry image, any configuration variable from the environment can
be specified by passing `-e` to the `docker container run` command.

To override a specific configuration value, create an environment variable in the form `REGISTRY_<VARIABLE>`, where
`<VARIABLE>` is the name of the configuration option and a `_` represents indention levels.

For example, the YAML configuration for the filesystem storage's root directory looks like this:

```YAML
storage:
  filesystem:
    rootdirectory: /var/lib/registry
```

In order to override the `rootdirectory` value manually, specify the following environment variable whether using `-e`
or in an `ENV` instruction:

```shell script
REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/path/to/root/directory
```

### Override the entire configuration

An alternative is to override the entire configuration file by mounting a different YAML configuration file `config.yml`
to the container. The mounting point should be `/etc/docker/registry`.

```shell script
$ docker container run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v "$(pwd)"/config.yml:/etc/docker/registry/config.yml \
  registry:2
```

### See also
[List of configuration options](https://docs.docker.com/registry/configuration/#list-of-configuration-options)