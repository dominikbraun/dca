## Daemon troubleshooting

You can enable debugging on the daemon to learn about the runtime activity of the daemon and to aid in troubleshooting.
If the daemon is completely non-responsive, you can also force a full stack trace of all threads to be added to the log
by sending the `SIGUSR` signal to the Docker daemon.

## Troubleshooting conflicts between the `daemon.json` and startup scripts

If you use a `daemon.json` file and also pass options to the `dockerd` command manually or using start-up scripts, and
these options conflict, Docker fails to start with an error such as:

```shell script
unable to configure the Docker daemon with file /etc/docker/daemon.json:
the following directives are specified both as a flag and in the configuration
file: hosts: (from flag: [unix:///var/run/docker.sock], from file: [tcp://127.0.0.1:2376])
```

If you see an error similar to this one and you are starting the daemon manually with flags, you may need to adjust your
flags or the `daemon.json` file to remove the conflict.

**Use the hosts key in `daemon.json` with systemd**

One notable example of a configuration conflict that is difficult to troubleshoot is when you want to specify a
different daemon address from the default. Docker listens to a socket by default. On Debian and Ubuntu systems using
systemd, this means that a host flag `-H` is always used when starting `dockerd`. If you specify a `hosts` entry in the
`daemon.json`, this causes a configuration conflict and Docker fails to start.

To work around this problem, create a new file `/etc/systemd/docker.service.d/docker.conf` with the following contents,
to remove the `-H` argument that is used when starting the daemon by default.

```shell script
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```

Run `sudo systemctl daemon-reload` before attempting to start Docker. If Docker starts successfully, it is now listening
on the IP address specified in the `hosts` key of the `daemon.json` instead of a socket.