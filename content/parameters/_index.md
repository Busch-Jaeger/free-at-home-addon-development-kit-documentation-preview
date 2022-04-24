---
title: "Using parameters in free@home Addons"
draft: false
weight: 300
ShowTOC: true
---

## Using parameters in free@home Addons

In many situations the Addon may want to allow the end-user to configure one or more aspects of the
Addon, such as providing the address and/or username/password of an external device to the Addon, or
to configure runtime behavior such as a polling interval.

To do so, you can define parameters in the `free-at-home-metadata.json` file, see the
section about parameters in the [metadata]({{< relref metadata>}}) documentation for details.

Once specified in the metadata file, the parameters can be configured by the end-user and the
configured values can be accessed by the Addon scripts.
