### Docker Content Trust

When transferring data among networked systems, trust is a central concern. In particular, when communicating over an
untrusted medium such as the internet, it is critical to ensure the integrity and the publisher of all the data a system
operates on. You use the Docker Engine to push and pull images (data) to a public or private registry. Content Trust
gives you the ability to verify both the integrity and the publisher of all the data received from a registry over any
channel.

## About Docker Content Trust (DCT)

Docker Content Trust (DCT) provides the ability to use digital signatures for data sent to and received from remote
Docker registries. These signatures allow client-side or runtime verification of the integrity and publisher of specific
image tags.

Through DCT, image publishers can sign their images and image consumers can ensure that the images they pull are signed.
Publishers could be individuals or organizations manually signing their content or automated software supply chains
signing content as part of their release process.

## Image tags and DCT

An individual image record has the following identifier:

```shell script
[REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]
```

A particular image `REPOSITORY` can have multiple tags. For example, `latest` and `3.1.2` are both tags on the `mongo`
image. An image publisher can build an image and tag combination many times changing the image with each build.

DCT is associated with the `TAG` portion of an image. Each image repository has a set of keys that image publishers use
to sign an image tag. Image publishers have discretion on with tags they sign.

An image repository can contain an image with one tag that is signed and another tag that is not. Publishers can choose
to sign a specific tag or not. As a result, the content of an unsigned image tag and that of a signed image tag with the
same name may not match. For example, a publisher can push a tagged image `someimage:latest` and sign it. Later, the
same publisher can push an unsigned `someimage:latest` image. This second push replaces the last unsigned tag `latest`
but does not affect the signed `latest` version. The ability to choose which tags they can sign, allows publishers to
iterate over the unsigned version of an image before officially signing it.

Image consumers can enable DCT to ensure that images they use were signed. If a consumer enables DCT, they can only
pull, run or build with trusted images. Enabling DCT is a bit like applying a "filter" to your registry. Consumers "see"
only signed image tags and the less desirable, unsigned image tags are "invisible" to them.

To the consumer who has not enabled DCT, nothing about how they work with Docker images changes. Every images is visible
regardless of whether it is signed or not.