---
title: "free@home Addons"
topTitle: "free@home Addons"
draft: false
weight: 40
---

## free@home Addons

The free@home smart house automation system can be extended with the use of addons, if desired. An addon can be installed to the free@home System Access Point (SysAP) and can provide additional functionality, such as integrating new devices from external vendors into the free@home system, so that they can be controlled just like normal free@home devices using the free@home apps, used in in timers or scenes, and so on.

From a technical point of view, free@home addons are Node.js archives that are deployed to the System Access Point of the end-user and run there in a container. The addon as access to the [local-api](https://developer.eu.mybuildings.abb.com/fah_local/) on the System Access Point and can therefore create and control (virtual) devices, communicate with external devices (e.g. using a REST API provided by that device) and so on.

### Target audience

This documentation describes how to develop new addons for the free@home system. As such, this documentation is aimed primarily at developers - device vendors or simply enthusiasts with basic development experience.
No detailed knowledge of the free@home system is required, other than being able to use it. Some basic knowledge of TypeScript and Node.js are expected though, see the [prerequisites]({{< relref prerequisites >}}) for details.

### Overview

To get started immediately, please refer to the [walkthrough]({{<relref gettingstarted >}}).

The remainder of this documentation gives further details on addons:

- [Prerequisites for developing and using addons]({{< relref prerequisites >}})

  Note in particular that both, end-users and developer need to activate the local-API on their System Access Point prior to using addons.

- [Getting started with development: A walkthrough]({{< relref gettingstarted >}}).

  This section describes how to setup the development environment and the SysAP for Addon development. Refer to the following sections for further details, if required.

