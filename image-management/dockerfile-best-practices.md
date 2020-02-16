## Dockerfile Best Practices

A Docker image consists of multiple read-only layers. Each layer represents a Dockerfile instruction. The layers are stacked, each one being a delta of the changes from the previous layer.

When you run an image and generate the container, a new _writeable layer_ - the container layer - will be added on top of the existing underlying layers. All changes made to the container, such as new files, modified files or deleted files are written to this writable container layer.

### Create ephemeral layers

The generated containers should be ephemeral as possible: They should be stop and destroyed, rebuilt and replaced with an absolute minimum setup and configuration.

### Understand build context

When the `docker build` command is executed, the current working directory is called the _build context_. All recursive contents in this directory are sent to the Docker daemon as build context.
