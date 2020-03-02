## Engine upgrade

### Docker CE

Upgrading the Docker CE Engine is quite simple. First, run:

```shell script
$ sudo apt-get update
```

Then choose the new version to install. To get the latest version, run:

```shell script
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

However, this may not be appropriate for you stability needs. To list the available versions in the repository, run:

```shell script
$ apt-cache madison docker-ce

  docker-ce | 5:18.09.1~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 5:18.09.0~3-0~ubuntu-xenial | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 18.06.1~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
  docker-ce | 18.06.0~ce~3-0~ubuntu       | https://download.docker.com/linux/ubuntu  xenial/stable amd64 Packages
```

This allows you to select and install a specific engine version using a command in the following form:

```shell script
$ sudo apt-get install docker-ce=VERSION docker-ce-cli=VERSION containerd.io
```