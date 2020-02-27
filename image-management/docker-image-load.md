## Image load

`docker image load` loads an image from a tar archive or from STDIN. It is common practice to save and load an image.

### Saving an image

For example, let's assume the following Dockerfile:

```Dockerfile
FROM alpine
CMD ["echo", "Hi!"]
```

After building the image with `docker image build -t app .`, we can `save` it into a TAR:

```shell script
$ docker image save app > app.tar
```

`docker image save` will stream the image to STDOUT which allows us to write it into a TAR using the `>` operator. The
resulting archive will contain all parent layers and all tags and versions of the image.

Extracting the archive and displaying its structure will print an output such as:

```shell script
$ tree .
.
├── 42a55273af416f888782dd4ec0946454519c88da64cd82d6861d5a4f786130c0.json
├── f05a67e1903fd8fd83a0cd12ed82d2d062ed9b56434e9d68e156eb925329f7b4
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── manifest.json
└── repositories
```

The output indicates that the image consists of two layers, as expected. The less complicated layer (which contains the
CMD instruction) is a single JSON configuration file.

### Loading an image

To load an existing image, use `docker image load`. This command accepts an input stream from STDIN, which enables the
following syntax to load an image from a TAR:

```shell script
$ docker image load < app.tar
``` 

You can verify that the image has been extracted using `docker image ls`.
