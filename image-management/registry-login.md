## Registry login

Logging in to a registry can be done using the `docker login` command.

### Login to a self-hosted registry

For logging in to a self-hosted registry, you can specify this by adding the server name of the registry.

```shell script
$ docker login localhost:8080
```

### Provide a password from STDIN

By default the `docker login` command runs interactively. However, you can set the `--password-stdin` flag to provide
a password through STDIN. This prevents the password from being logged to the shell history.

The following command reads the password from a file and passes it to the `docker login` command:

```shell script
$ cat $HOME/registry-password.txt | docker login --username myuser --password-stdin
```

### Privileged user requirement