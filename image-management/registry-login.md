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

`docker login` requires the user to use `sudo` (or be root), except when connecting to a remote daemon (docker-machine)
or when the user is added to the `docker` group.

> Being a member of the `docker` group impacts the system security since it is root equivalent. See [Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface)

### Credentials store

The Docker Engine can keep user credentials in an external credentials store such as the OS' native keychain. Using such
an external store is more secure than storing credential in the Docker configuration file.

To use a credentials store, you need an external helper program to interact with a specific keychain/external store.
Docker requires that helper program to be in `$PATH`.

[List of the available credential helpers](https://docs.docker.com/engine/reference/commandline/login/#parent-command#credentials-store)

### Configure the credentials store

The credentials store needs to be specified in `$HOME/.docker/config.json` to tell Docker to use it. If the program's
name is `docker-credential-osxkeychain`, the JSON value for `credsStore` is `osxkeychain`.

### Default behaviour

By default, Docker looks for the native binary on each platform, i. e. _osxkeychain_ on macOS, _wincred_ on Windows and
_pass_ (or _secretservice_ if that can't be found) on Linux.

If none of these default binaries are found, Docker stores the credentials in base64 encoding in the config files
described above.
