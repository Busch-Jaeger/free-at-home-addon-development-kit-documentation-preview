---
title: "Samples"
draft: false
weight: 600
ShowTOC: true
---

To get started with the development of an add on the free@home example add on can be used as a base.


## Configuration of target SysAP for development

The library can be configured to connect to a remote SysAP. This can be used to debug an add on that runs on the developers environment.

Windows
```
$env:FREEATHOME_BASE_URL = 'http://[ip of sysap]'
$env:FREEATHOME_API_BASE_URL = 'http://[ip of sysap]/fhapi/v1'
$env:FREEATHOME_API_USERNAME = '[username]'
$env:FREEATHOME_API_PASSWORD = '[password of user]'
```

Linux / Unix
```bash
export FREEATHOME_BASE_URL=http://[ip of sysap]
export FREEATHOME_API_BASE_URL=http://[ip of sysap]/fhapi/v1
export FREEATHOME_API_USERNAME=[username]
export FREEATHOME_API_PASSWORD=[password of user]
```

## Debug a script in a NodeJS IDE

Visual Studio Code (launch.json)

```Javascript
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "pwa-node",
            "request": "launch",
            "name": "Launch Program",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "program": "${workspaceFolder}\\build\\main.js",
            "args": [
            ],
            "env": {
                "FREEATHOME_API_BASE_URL":"http://192.168.42.90/fhapi/v1",
                "FREEATHOME_BASE_URL":"http://192.168.42.90",
                "FREEATHOME_API_USERNAME": "installer",
                "FREEATHOME_API_PASSWORD": "12345"
            },
            "outFiles": [
                "${workspaceFolder}/**/*.js"
            ],
            "preLaunchTask": "tsc: build - tsconfig.json",
        }
    ]
}
```


## Attachment of virtual devices via the local API

The free@home library can be used to simplify the creation of a virtual device. The library also helps to handle events on the device and to concentrate on the implementation of the business logic.

Quality of life features are:
- automatic handling of keep alive calls for virtual devices
- automatic reflection of input datapoints to matching output datapoints. This can be used to simplify the implementation.

```javascript
import { FreeAtHome } from 'free-at-home';

const freeAtHome = new FreeAtHome();

const virtualSwitch = await freeAtHome.createSwitchingActuatorDevice("123switch", "Virtual Switch");
virtualSwitch.setAutoKeepAlive(true);
virtualSwitch.isAutoConfirm = true; // reflects inputs to output Datapoints
virtualSwitch.on('isOnChanged', (value: boolean) => {
  console.log("switch state is:", (value) ? "on" : "off");
});
```

## Use of the parameters for the configuration of scripts

Excerpt from free-at-home-metadata.json:

```
    "parameters": {
        "default": {
            "name": "Settings",
            "items": {
                "username": {
                    "name": "Username",
                    "type": "string"
                },
                "ignoreReachability": {
                    "name": "Ignore reachability",
                    "type": "boolean"
                },
                "port": {
                    "name": "Port",
                    "type": "number",
                    "min": 1024,
                    "max": 65536
                }
            }
        }
    }
```
Simple version:

```javascript
import {ScriptingHost} from 'free-at-home';

const metaData = ScriptingHost.readMetaData();

const scriptingHost = new ScriptingHost.ScriptingHost(metaData.id);

scriptingHost.on("configurationChanged", (configuration: API.Configuration) => {
  console.log(configuration);
});

scriptingHost.connectToConfiguration();
```

Extended variant with typed configuration:

```javascript
import {ScriptingHost} from 'free-at-home';
import {ScriptingAPI as API} from 'free-at-home'

export interface ConfigurationProperties extends API.Configuration {
    default: {
        items: {
            username: string,
            ignoreReachability: boolean,
            port: number,
        }
    },
};

const metaData = ScriptingHost.readMetaData();

const scriptingHost = new ScriptingHost.ScriptingHost<ConfigurationProperties>(metaData.id);

scriptingHost.on("configurationChanged", (configuration: ConfigurationProperties) => {
  console.log(configuration);
});

scriptingHost.connectToConfiguration();
```

## Automatic logout of virtual devices on completion of a script

When a script terminates, all virtual devices created by that script should be unregistered as unresponsive from the system access point.
This can be achieved by calling the method markAllDevicesAsUnresponsive() in the class FreeAtHome.

```javascript
import { FreeAtHome } from 'free-at-home';

const freeAtHome = new FreeAtHome();

if (process.platform === "win32") {
  var rl = require("readline").createInterface({
    input: process.stdin,
    output: process.stdout
  });

  rl.on("SIGINT", function () {
    process.emit("SIGINT" as any);
  });
}

process.on('SIGINT', async () => {
  console.log("SIGINT received, cleaning up...")
  await freeAtHome.markAllDevicesAsUnresponsive();
  console.log("clean up finished, exiting procces")
  process.exit();
});
process.on('SIGTERM', async () => {
  console.log("SIGTERM received, cleaning up...")
  await freeAtHome.markAllDevicesAsUnresponsive();
  console.log("clean up finished, exiting procces")
  process.exit();
});
```
