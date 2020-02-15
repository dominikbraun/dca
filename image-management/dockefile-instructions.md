## Dockerfile instructions

### FROM

The `FROM` instruction initializes a new build stage and sets the base image for subsequent instructions. A new build stage can be named by adding `AS name` to the `FROM` instruction.

### RUN

`RUN` has two forms:
* The shell form `RUN command` runs in a shell, which by default is `/bin/bash -c` on Linux and can be changed using the `SHELL` command
* The exec form `RUN ["executable", "param1", "param2"]`

The `RUN` instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting commited image will be used for the next step in the Dockerfile.

Layering `RUN` instructions and generating commits conforms to the core concepts of Docker: Commits are cheap and containers can be created from any point in an image's history.

### CMD

The `CMD` instruction has three forms:
* The exec form `CMD ["executable, "param1", "param2"]`
* Default parameters to `ENTRYPOINT` `CMD ["param1", "param2"]`
* The shell form `CMD command param1 param2`

There can only be _one_ `CMD` instruction in a Dockerfile (if you list more than one, only the last one will take effect).

**The main purpose of a `CMD` instruction is to provide defaults for an executing container.** These defaults can include an executable or omit the executable (in which case an `ENTRYPOINT` has to be specified).

> If `CMD` is used to provide default arguments for the `ENTRYPOINT` instruction, both instructions should be specified in the JSON array format.

> The exec form doesn't invoke a command shell like the shell form, meaning that a shell processing does not happen and `CMD ["echo", "$HOME"]` won't work. Use `CMD ["sh", "-c", "echo $HOME"]` instead.

The command specified in `CMD` can be overridden when running `docker container run IMAGE COMMAND` with a command argument.

### ADD

`ADD` has two forms:
* `ADD src dest`
* `ADD ["src", ..., "dest"]` which is required for paths containing whitespaces

This instruction copies new files, directories or remote URLs from `src` and adds them to the filesystem of the image at the path `dest`.

Multiple `src` resources my be specified, but if they're files or directories, their paths are interpreted as relative to the build context. Each `src` may contain wildcards. Examples:

Add all files starting with "hom":

```Dockerfile
ADD hom* /mydir/
``` 

Add all .txt files starting with "hom" and a succeeding single character:

```Dockerfile
ADD home?.txt /mydir/
```

`ADD`-specific rules:
* The `src` path must be _inside_ the build context - `../mydirectory` is not possible because `docker build` sends the context directory to the Docker daemon.
* If `src` is an URL and `dest` _does not_ end with a trailing slash, then a file is downloaded from the URL and copied into `dest`
* If `src` is an URL and `dest` _does_ end with a trailing slash, then the filename is inferred from the URL and the file is downloaded into `dest/filename`.
* An URL must have a nontrivial path so that an appropriate filename can be discovered (example.com is not valid, but example.com/file.txt is)
* If `src` is a _local_ tar archive, it is unpacked as a directory. A _remote_ tar archive is _not_ decompressed.
* If `src` is any other kind of file, it is copied along with its metadata. In this case, if `dest` ends with a trailing slash, it will be considered a directory and the contents of `src` will be written at `dest/base(src)`.
* If multiple `src` resources are specified, `dest` has to be a directory ending with a trailing slash.
* If `dest` does not end with a trailing slash, it will be considered a regular file and the contents of `src` will be written at `dest`.
* If `dest` does not exist, it will be created.

### COPY

`COPY` has two forms:
* `COPY src dest`
* `COPY ["src", ..., "dest"]` which is required for paths containing whitespaces

This instructions copies new files or directories from `src` and adds them to the filesystem of the container at the path `dest`.

Multiple `src` resources may be specified and it follows the same rules as `ADD` regarding path names.

`COPY`-specific rules:
* The `src` path must be _inside_ the build context - `../mydirectory` is not possible because `docker build` sends the context directory to the Docker daemon.
* If `src` is a directory, the entire contents of the directory are copied.
* If `src` is any other kind of file, it is copied along with its metadata. If `dest` ends with a trailing slash, it will be considered a directory and the contents of `src` will be written at `dest/base(src)`.
* If multiple `src` resources are specified, `dest` has to be a directory ending with a trailing slash.
* If `dest` does not end with a trailing slash, it will be considered a regular file and the contents of `src` will be written at `dest`.
* If `dest` does not exist, it will be created.

### ENTRYPOINT

`ENTRYPOINT` has two forms:
* The exec form `ENTRYPOINT ["executable", "param1", "param2"]`
* The shell form `ENTRYPOINT command param1 param2`

An entrypoint allows you to configure a container that will run as an executable. For example, the following will start nginx with its default content:

```shell script
$ docker run --rm -it -p 80:80 nginx
```

Command line arguments to `docker run IMAGE` will be appended after all elements in an exec form `ENTRYPOINT` and will override all elements specified using `CMD`. This allows arguments to be passed to the entrypoint, e. g. `docker run IMAGE -d` will pass `-d` to the entrypoint.

The shell form prevents any `CMD` or `run` command line arguments from being used, but has the disadvantage that the `ENTRYPOINT` will be started as a subcommand of `/bin/sh -c`, which does not pass signals -> a SIGTERM won't be received from `docker stop CONTAINER`.

Just with `CMD`, only the last `ENTRYPOINT` instruction in a Dockerfile will have an effect.

### EXPOSE

The `EXPOSE` instruction informs Docker that the container listens on the specified network ports at runtime.

Note that `EXPOSE` does not actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container about which ports are intended to be published. To actually publish the ports, use the `-p` flag when running the container.