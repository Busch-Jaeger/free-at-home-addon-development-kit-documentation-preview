---
title: "Concepts"
draft: false
weight: 500
ShowTOC: true
---

Addons provide the ability to add extensions to the existing free@home system.

## What is an addon

The addons are executed in an encapsulated environment on the Access Point 2.0 system.
This allows, among other things, third-party devices to be integrated into the free@home system.
The implementation of own logics for the control of actuators within free@home is also conceivable.

## NodeJs Bibliothek

To simplify the development of addons, a NodeJS library is provided. This simplifies the creation and use of virtual devices. In addition, functions for the use of parameters are provided.

## Environment Limitations for Addons

Limitations of the environment in which an addon runs:

- ram 64MB max
- cpu 40% max
- tasks (threads) max: 20
