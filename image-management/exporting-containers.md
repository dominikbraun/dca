## Exporting containers

A container's filesystem can be exported as a TAR using `docker container export`. The command streams the filesystem
to STDIN, which allows the following syntax to export a container:

```shell script
$ docker container export a8b14091b4e7 > app-container.tar
``` 

This is useful if the build of the app is expensive or generates many compute-intensive artifacts. Cloning or removing
it would require a rebuild from scratch. It is much faster to `export` a snapshot of the container.

 ### Importing images
 
There's no way to import a container exported with `docker container export` (which wouldn't make sense since a
container is a running environment). `docker container export` exports the filesystem, and `docker image import`
takes this filesystem and imports it as an image. We can execute a container from that image or use the image as a
layer for another image. 

This may look like so:

 ```shell script
$ docker image import app-container.tar app:latest
```

### See also
image-load.md
[Docker import/export vs. load/save](https://pspdfkit.com/blog/2019/docker-import-export-vs-load-save/)
