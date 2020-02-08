## Swarm manager locking

* With Docker >= 1.13, the Raft logs used by the swarm managers are encrypted on disk
* Encryption protects the service configuration from attackers
* On restart, the TLS key used to encrypt communication between the nodes and the key used to encrypt the Raft logs are loaded into each managers's memory
* Docker >= 1.13 allows taking ownership of these keys and to require manual unlocking of the managers
* This protects the TLS keys at rest and is referred to as _autolock_

When Docker restarts, the swarm has to be unlocked first with yet another key generated when the swarm was locked.

* When a new node joins the swarm, this key will be propagated to it -> the swarm does not have to be unlocked manually

### Initialize an autolocked swarm

Using the `--autolock` flag of `swarm init` will enable autolocking of each manager node.

```shell script
$ docker swarm init --autolock --advertise-addr 192.168.188.28
```

Next to the usual output of `swarm init`, the instructions for unlocking the current manager are shown:

```shell script
To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-tgUW8ZKnbYy6xQ4CGSDacuA5Gn8AfLGI/jwGQ2/M7UA

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
```

Let's restart Docker ...

```shell script
$ sudo systemctl restart docker
```

... and list the swarm's services:

```shell script
$ docker service ls

Error response from daemon: Swarm is encrypted and needs to be unlocked before it can be used. Please use "docker swarm unlock" to unlock it.
```

The error occurring indicates that the manager has to get unlocked first.

```shell script
$ docker swarm unlock
```

### Enabling autolock for an existing swarm

Using `docker swarm update`, autolock can be enabled for an existing swarm:

```shell script
$ docker swarm update --autolock=true
```

Autolock can disabled with the `--autolock=false` flag as well. The mutual TLS key and the key for encrypting the Raft logs will be stored unencrypted on the disk after that.

### View the current unlock key

In a situation where a manager node suddenly becomes unavailable and you have to unlock the manager to read the encrypted Raft logs, you need the autolock key.

If that key has _not_ been rotated since the manager node left the swarm, you can view the current unlock key using `docker swarm unlock-key`. The only constraint is that there is a quorum of other functional manager nodes in the swarm.

```shell script
$ docker swarm unlock-key

To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-tgUW8ZKnbYy6xQ4CGSDacuA5Gn8AfLGI/jwGQ2/M7UA

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
``` 

If the key has been rotated after the manager node became unavailable and you don't know the previous key, you have to force the manager to leave the swarm (using `docker swarm leave --force`) and join it back as a new manager (using `docker swarm join`).

### Rotate the unlock key

The `docker swarm unlock-key` command is not only useful for printing the current unlock key, but also for rotating it:

```shell script
$ docker swarm unlock-key --rotate

Successfully rotated manager unlock key.

To unlock a swarm manager after it restarts, run the `docker swarm unlock`
command and provide the following key:

    SWMKEY-1-rQAb/I2naQr5ENcQaHf82TmuiLU3SQkfHC1oP/6Xz34

Please remember to store this key in a password manager, since without it you
will not be able to restart the manager.
```
