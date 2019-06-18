---
title: "0 - Installation"
description: "Installation of a new Gaia Worker"
weight: 10
---

## Download the binary

Gaia Worker is embedded into the normal Gaia binary file. Therefore, the first step is to copy the Gaia binary file to
your remote worker host or pull the Gaia docker image from docker hub.

## Configure the worker

Before we can start the worker, we need to configure it. Since Gaia Worker uses the configuration loading mechanism
like Gaia, you have a few options to provide the needed configuration.


### Configuration file

Provide configuration via a configuration file. Create a new file called `.gaia_config` in the same folder where the
Gaia binary is located with the following example configuration:

```
hosturl=http://gaia-url.com
mode=worker
workername=my-worker
grpchosturl=gaia-url.com:8989
workersecret=55d79ff1-37b9-595d-a973-65d50e95db85
```

### Environment variables

You have the possibility to provide configuration via environment variables. See the example configuration below:

```
export GAIA_HOSTURL=http://localhost:8080
export GAIA_MODE=worker
export GAIA_WORKERNAME=my-worker
export GAIA_GRPCHOSTURL=localhost:8989
export GAIA_WORKERSECRET=55d79ff1-37b9-595d-a973-65d50e95db85
```

* HOSTURL (required) - The URL to the Gaia primary instance where this worker should be connected to.
* MODE (required) - Tells Gaia on start which mode it should start in. Since we want to start a worker it should be always "worker".
* WORKERNAME (optional) - Gives the worker a name which is displayed in the primary Gaia UI. If empty, a random generated name will be set.
* GRPCHOSTURL (required) - The URL including the port to the Gaia primary instance. Should be mostly the like `HOSTURL` but the port should be different.
* WORKERSECRET (required) - The global registration secret which is used to register at the Gaia primary instance. Can be obtained in the primary Gaia instance UI (Settings -> Worker).

## Starting the worker

As soon as the worker configuration is finished, we are able to start the worker. It should only take a few seconds
until the worker is visible in the primary instance's UI. If not, please have a look at the worker output logs.
