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