---
title: "Writing free@home Addons"
draft: false
weight: 150
ShowTOC: true
---

## Writing free@home Addons

------------------------------------------------------------------------

When the System Access Point and the local development computer are prepared as described in the
[prerequisites]({{<relref prerequisites>}}) section, the actual Addon development can start.

Addons are written in TypeScript/JavaScript using NodeJS. They communicate with the System Access
Point (SysAP) using the "local API" provided by the SysAP. In principle, the Addon developer can
make all calls to the local API manually, see the
[local API documentation](https://developer.eu.mybuildings.abb.com/fah_local/) for details.
However we strongly recommend to use the Node.js based `free-at-home` convenience library that is
provided for Addon development and that implements many common use-cases, such as the creation and
use of virtual devices and the use of configuration parameters. Examples in this
documentation will always use this library.

### Initialization

To initialize a new free@home Addon, simply import the `FreeAtHome` class from the `free-at-home`
library and create a new object:

```typescript
import { FreeAtHome } from 'free-at-home';

const freeAtHome = new FreeAtHome();

async function main() {
    // Addon code here
}

main()
```

This will set up the Addon for communication with the SysAP. When running on a local development
machine, this involves reading the `FREEATHOME_BASE_URL`, `FREEATHOME_API_USERNAME` and
`FREEATHOME_API_PASSWORD` environment variables, as described in the
[walkthrough]({{<relref gettingstarted>}}), when running on the SysAP itself, the addon can use a
local unix socket instead.

The `FreeAtHome` class is the main class to interact with when developing a free@home Addon.

### Creating virtual devices

When writing an Addon, a common use-case is to integrate some external device into the free@home
system. This can be done by "virtual" devices: The SysAP does not have (direct) access to the actual
hardware device, so instead sends all commands and updates to a virtual device instead (and
similarly can receive commands from this virtual device). The Addon can then perform it's own
actions whenever necessary.

To create a device, you first have to know what *type* of device you need: A switch obviously
behaves differently to a media player. The `FreeAtHome` class knows many different types of devices
(usually "actuators"), see the individual `create*` functions of that class, such as
`createSwitchingActuatorDevice`, `createDimActuatorDevice`, `createCeilingFan`, `createMediaPlayer`
 and so on.

A new virtual device needs at least two parameters:

- A native ID, with characters a-z, A-Z and 0-9
- A name that is displayed to the user in the App and the web interface

The native ID should uniquely identify your device. If a device with that ID already exists, this
function will simply use this device - the `create*` call should therefore be done on every startup
of the script:

```typescript
import { FreeAtHome } from 'free-at-home';

const freeAtHome = new FreeAtHome();

async function main() {
    const mySwitch = await freeAtHome.createSwitchingActuatorDevice("mySwitch", "Switch from Addon");
    mySwitch.setAutoKeepAlive(true);
    mySwitch.isAutoConfirm = true;
}

main()
```

Notice that this example also calls `setAutoKeepAlive(true)` and sets `isAutoConfirm=true`. To
detect devices that are no longer available (e.g. defective), the SysAP wants an occasional
keepalive message from every device. If this is not practical for a given device, simply set
`setAutoKeepAlive(true)`, then the `FreeAtHome` class of the Addon will send such a message
automatically, effectively disabling the unresponsive detection for this device.

This is all to create a new free@home device in an Addon. When the Addon is started, the device will
show up in the device list of the SysAP. Next steps are to interact with the device in the Addon:

### Receiving updates from the SysAP about devices

After creating a device, the Addon normally needs to be informed about updates to the device state.
For example when creating a switch, the Addon normally wants to know when the user turns the switch
 "on" or "off" in the App or the web interface. This can be done by adding the corresponding event
listener after creating the device:

```typescript
import { FreeAtHome } from 'free-at-home';

const freeAtHome = new FreeAtHome();

async function main() {
    const mySwitch = await freeAtHome.createSwitchingActuatorDevice("mySwitch", "Switch from Addon");
    mySwitch.setAutoKeepAlive(true);
    mySwitch.isAutoConfirm = true;

    mySwitch.on('isOnChanged', (value: boolean) => {
        console.log("Switch state is: ", (value) ? "on" : "off");
    });
}

main()
```

The list of events that make sense depend on the given device type. In case of a switch actuator,
only `isOnChanged` makes sense - other devices have different "channels" (each channel describes a
certain functionality or action that the device can perform), so for example a media player device
provides the `MediaPlayerChannel` instead of the `SwitchingActuatorChannel` and therefore provides
`playModeChanged`, `playCommandChanged`, `muteChanged` and `playVolumeChanged` instead.

For details, see the class of the object that is returned (`mySwitch` in the example above)
by the individual `create*` functions.

### Sending updates to the SysAP for devices

Similar to *receiving* updates from a device when the user changes something, an Addon usually also
wants to *send* updates - for example when integrating an external switch into the free@home system,
the "on"/"off" state of the switch should be reported to the SysAP when the (hardware) switch is
pressed.

For this, call the corresponding function of the object returned by the `create*()` functions. What
function exactly depends on the desired action and on the type of the device that was created (i.e.
the channel class that is used), similar to receiving data:

```typescript
    // ...
    mySwitch.setOn(true);
    // ...
    mySwitch.setOn(false);
    // ...
```

The tricky part is *when* to update the state of the virtual device. Obviously, this depends on the
use-case your Addon aims to implement:

- Whenever the external device sends a REST command in the local network: Consider using the
  [local API](https://developer.eu.mybuildings.abb.com/fah_local/) directly using the device serial.
  This requires configuring the external device individually for the device serial - if this is
  infeasible, the Addon might open a dedicated HTTP server instead.
- Whenever a connected USB stick sends an update. If your Addon implements functionality that is
  provided by an USB stick/dongle attached to the SysAP (e.g. providing a Modbus interface), you may
  want to react to events from that device.
- By polling the external device or external state. Here you could start a timer in your Node.js
  based Addon and query the state of the external device - for example every few seconds. If you
  notice that the state changed, set the corresponding state in the virtual device of the Addon.


One simple way is to provide a dedicated HTTP server. ***WARNING: This may have potential security
implications, as unauthenticated users that have access to the local network also have access to
this server. Consider if that is acceptable before doing this.***

```typescript
import { FreeAtHome } from 'free-at-home';
import http  from 'http';

const freeAtHome = new FreeAtHome();
let port: number = 8099; // Choose a port for your Addon. Consider using a parameter to make it configurable.

async function main() {
    const mySwitch = await freeAtHome.createSwitchingActuatorDevice("mySwitch", "Switch from Addon");
    mySwitch.setAutoKeepAlive(true);
    mySwitch.isAutoConfirm = true;

    function startServer() {
        return http.createServer(function (request, response) {
            if (request.method == 'POST' && request.url == '/rest/switch/on') {
                mySwitch.setOn(true);
                response.writeHead(200, {'Content-Type': 'text/plain'});
                response.write("OK\r\n");
            } else if (request.method == 'POST' && request.url == '/rest/switch/off') {
                mySwitch.setOn(false);
                response.writeHead(200, {'Content-Type': 'text/plain'});
                response.write("OK\r\n");
            } else {
                response.writeHead(404, {'Content-Type': 'text/plain'});
                response.write("Not Found\r\n");
            }
            response.end();
        }).listen(port);
    }
    let server = startServer();
}

main()
```

If you now send a HTTP POST request to this address, the switch will turn on/off in the free@home
next App and in the web interface.

```shell
# Turn switch on
curl -X POST http://<IP>:8099/rest/switch/on
# Turn switch off
curl -X POST http://<IP>:8099/rest/switch/off
```

NOTE: The `<IP>` of this HTTP server is the SysAP IP if the Addon is
[deployed]({{< relref deployment>}}) to the SysAP, but when testing on a local development machine,
then you must use the IP of the local development machine:

```shell
# Turn switch on
curl -X POST http://localhost:8099/rest/switch/on
# Turn switch off
curl -X POST http://localhost:8099/rest/switch/off
```

### Configuration Parameters

In many situations the Addon may want to allow the end-user to configure one or more aspects of the
Addon, such as providing the address/port and/or username/password of an external device to the
Addon, or to configure runtime behavior such as a polling interval.

To do so, you can define parameters in the `free-at-home-metadata.json` file, see the
section about parameters in the [metadata]({{< relref metadata>}}) documentation for details.

Once specified in the metadata file, the parameters can be configured by the end-user and the
configured values can be accessed by the Addon scripts.

For example to configure the `port` of the http server used above, you can use the following
metadata snippet:

```json
    "parameters": {
        "default": {
            "name": "Settings",
            "items": {
                "port": {
                    "name": "Port",
                    "type": "number",
                    "min": 1024,
                    "max": 65535,
                    "value": 8099
                }
            }
        }
    }
```

Then, modify the Addon to use the value configured by the user:

```typescript
import {ScriptingHost} from 'free-at-home';
import {ScriptingAPI} from 'free-at-home';

let port: number = 8099;

// Register for configuration changes in the metadata
const metaData = ScriptingHost.readMetaData();
const scriptingHost = new ScriptingHost.ScriptingHost(metaData.id);
scriptingHost.on('configurationChanged', (configuration: ScriptingAPI.Configuration) => {
    console.log(configuration);
    const origPort = port;
    port = configuration.default.items?.port ?? origPort;
    if (port != origPort) {
        // restart the server
        server.close();
        server = startServer();
    }
})
scriptingHost.connectToConfiguration();
```

This example will get notified about changes to the configuration and will restart the http server
(that was used above) if the `port` has changed.

