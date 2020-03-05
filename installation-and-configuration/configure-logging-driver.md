## Configure the logging driver

Docker includes multiple logging mechanisms. These mechanisms are called _logging drivers_. Each Docker daemon has a
default logging driver which is used by each container - unless you configure it to use a different logging driver.

### Configure the default logging driver

To configure the Docker daemon to default to a specific logging driver, set the value of `log-driver` to the name of the
driver in the `/etc/docker/daemon.json` file. The default value is `json-file`.

This example sets the default logging driver to `syslog`:

```json
{
  "log-driver": "syslog"
}
```

If the logging driver has configurable options, you can provide these options as a JSON object with the `log-opts` key.
For example:

```json
{
  "log-driver": "syslog",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "label": "production_status",
    "env": "os,customer"
  }
}
```

Note that all values must be provided as strings.

Since the default logging driver is `json-file`, the default output of commands like `docker container inspect` is JSON.

To find the current logging driver, use the `docker info` command like this:

```shell script
$ docker info --format "{{.LoggingDriver}}"

  json-file
```

### Configure the logging driver for a container

When you start a container, you can configure it to use a different logging driver than the daemon's default driver.
This can be achieved by specified the `--log-driver` flag for `docker container run`.

If the logging driver has configurable options, you can set them using one or more `--log-opt KEY=VALUE` flags. Even if
the container uses the default logging driver (without the `--log-driver` flag specified), it can use different
configurable options.

This will start an Alpine container with the `none` logging driver.

```shell script
$ docker container run -it --log-driver none alpine ash
```

To find the current logging driver for a container, if the daemon uses the `json-file` driver, run the following
`docker container inspect` command:

```shell script
$ docker container inspect --format "{{.HostConfig.LogConfig.Type}}" my-container
  
  json-file
```

### Configure the delivery mode of log messages

Docker provides two modes for delivering messages from the container to the log driver:
* direct, blocking delivery from container to driver (default)
* non-blocking delivery that stores log messages in an intermediate per-container ring buffer for consumption by driver

The `non-blocking` message delivery mode prevents application from blocking due to logging back pressure. Applications
are likely to fail in unexpected ways when STDERR or STDOUT block.

Using the `--log-opt` flag, you can specify these and other logging options. The `mode` log option controls whether to
use `blocking` or `non-blocking` message delivery. The `max-buffer-size` log option controls the size of the ring buffer
used when `mode` is set to `non-blocking` and defaults to 1 MB.

This command will start an Alpine container with log output in non-blocking mode and a 4 MB buffer:

```shell script
$ docker run -it --log-opt mode=non-blocking --log-opt max-buffer-size=4m alpine ping 127.0.0.1
```

### Use environment variables or labels with logging drivers

Some logging drivers add the value of a containers `--env` or `--label` flags to the container's logs. This command runs
a container using the daemon's default logging driver but sets the environment variable `os=ubuntu`:

```shell script
$ docker container run -dit --label production_status=testing -e os=ubuntu alpine sh
```

If the logging driver supports it, this adds additional fields to the logging output. The following output is generated
by the `json-file` logging driver:

```json
"attrs": {
  "production_status": "testing",
  "os": "ubuntu",
}
```
