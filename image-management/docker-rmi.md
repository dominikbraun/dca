## docker rmi

The `docker rmi` command removes (and un-tags) one or more images from the host node. If an image has multiple tags,
using this command with the tag as parameter only removes the tag. If the tag is the only one for the image, both
the tag and the image are removed.

You can remove an image using its ID (short or long), its tag or its digest. If an image has one or more tags, you must
remove all these tags before the actual image is removed.

If you use the `-f` flag and specify the image's ID, then `rmi` un-tags and removes all images that match the ID.  