## Service templates

You can use templates for some flags of `docker service create`, using the syntax provided by Go's text/template package. Supported flags are:
* --hostname
* --mount
* --env

Valid placeholders for the Go template are:
* .Service.ID
* .Service.Name
* .Service.Labels
* .Node.ID
* .Node.Hostname
* .Task.ID
* .Task.Name
* .Task.Slot

In the following example, we'll set the template for all created containers' hostnames based on the service name, the node's ID and the default hostname:

````shell script
$ docker service create --name hosttemplate \
  --hostname="{{ .Node.Hostname }}-{{ .Node.ID }}-{ .Service.Name }}" \
  busybox top
````

Verify the created hostname via `docker container inspect`:

```shell script
$ docker container inspect --format="{{ .Config.Hostname }}" 2e7a8a9c4da2-wo41w8hg8qanxwjwsg4kxpprj-hosttempl

x3ti0erg11rjpg64m75kej2mz-hosttempl
```