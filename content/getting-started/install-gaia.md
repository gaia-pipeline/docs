---
title: "0 - Installation"
description: "Installation"
weight: 10
---

### Installation

The installation of Gaia is simple and often takes a few minutes.

#### Using Docker

Gaia offers a various amount of docker images available at <a href="https://hub.docker.com/r/gaiapipeline/gaia/" target="_blank">https://hub.docker.com/r/gaiapipeline/gaia/</a>. If you use the `latest` image tag, you will get a docker image which contains everything to build all currently supported pipeline languages but also needs more disk-space.
If you are confident that you just want to use one programming language for your pipelines you should consider taking the smaller images which are build just for this language. You can find a list of available images at the <a href="https://hub.docker.com/r/gaiapipeline/gaia/tags/" target="_blank">tags page</a>.

The following command starts Gaia as a daemon process and mounts all data to the current folder. Afterwards,
Gaia will be available on the host system on port 8080 (<a href="http://localhost:8080/" target="_blank">http://localhost:8080/</a>). 
Use the standard user "**admin**" and password "**admin**" as initial login. It is recommended to change the password afterwards.

    docker run -d -p 8080:8080 -v $PWD:/data gaiapipeline/gaia:latest

<br/>

#### Manually

It is possible to install gaia directly on the host system. This can be achieved by downloading the binary from the <a href="https://github.com/gaia-pipeline/gaia/releases" target="_blank">releases page</a>.
Please be aware that you, dependent on the programming language you want to use for your pipelines, might need other libraries to create pipelines via Gaia. In the <a href="https://github.com/gaia-pipeline/gaia/tree/master/docker" target="_blank">dockerfile</a> you should find all information for what is needed. Please open a issue if you have trouble to setup Gaia.

Use the standard user "**admin**" and password "**admin**" as initial login. It is recommended to change the password afterwards.

Gaia will automatically detect the folder of the binary and will place all data next to it.
You can change the data directory with the startup parameter --homepath if you want.
Other startup parameters are also available:

    Usage of gaia:
          -auto-docker-mode=false: If true, by default runs all pipelines in a docker container
          -ca-path="": Path where the generated CA certificate files will be saved
          -concurrent-worker=2: Number of concurrent worker the Gaia instance will use to execute pipelines in parallel
          -config=".gaia_config": this describes the name of the config file to use
          -dev=false: If true, Gaia will be started in development mode. Don't use this in production!
          -docker-host-url="unix:///var/run/docker.sock": Docker daemon host url which is used to build and run pipelines in a docker container
          -docker-run-image="gaiapipeline/gaia:latest": Docker image repository name with tag which will be used for running pipelines in a docker container
          -docker-worker-grpc-host-url="": The host url of the primary/worker gRPC endpoint used for docker worker communication
          -docker-worker-host-url="": The host url of the primary/worker API endpoint used for docker worker communication
          -home-path="": Path to the Gaia home folder where all data will be stored
          -hostname="https://localhost": The host's name under which Gaia is deployed at e.g.: https://gaia-pipeline.io
          -jwt-private-key-path="": A RSA private key used to sign JWT tokens used for Web UI authentication
          -mode="server": The mode which Gaia should be started in. Possible options are server and worker
          -pipeline-poll=false: If true, Gaia will periodically poll pipeline repositories, watch for changes and rebuild them accordingly
          -pipeline-poll-interval=1: The interval in minutes in which to poll source repositories for changes
          -port="8080": Listen port for Gaia
          -prevent-primary-work=false: If true, prevents the scheduler to schedule work on this Gaia primary instance. Only used in server mode
          -vault-path="": Path to the Gaia vault folder. By default, will be stored inside the home folder
          -version=false: If true, will print the version and immediately exit
          -worker-grpc-host-url="127.0.0.1:8989": The host url of an Gaia primary instance gRPC interface used for worker connection. Only used in worker mode or for docker runs
          -worker-host-url="http://127.0.0.1:8080": The host url of an Gaia primary instance to connect to. Only used in worker mode or for docker runs
          -worker-name="": The name of the worker which will be displayed at the primary instance. Only used in worker mode or for docker runs
          -worker-secret="": The secret which is used to register a worker at an Gaia primary instance. Only used in worker mode
          -worker-server-port="8989": Listen port for Gaia primary worker gRPC communication. Only used in server mode
          -worker-tags="": Comma separated list of custom tags for this worker. Only used in worker mode

##### Run-time Arguments

It is possible to define run-time arguments in three ways.

###### As command-line arguments

For example:

    ./cmd/gaia/main -home-path=${PWD}/tmp -dev=true

###### Environment Properties

Environment variables can be used to overwrite any configuration settings / defaults that are already defined in Gaia.

All Environment variables must be pre-fixed with `GAIA_` in order to avoid collision with other variables.

For example:

    export GAIA_PORT=9999
    ./cmd/gaia/main -home-path=${PWD}/tmp -dev=true
    # will start gaia with port 9999
    ⇨ http server started on [::]:9999

###### Configuration file

Alternatively a configuration file can be used with all the properties that gaia defines, called .gaia_config.

If this file is present it will be used to set things up. For example:

    ❯ cat .gaia_config
    port=9994
    ./cmd/gaia/main -home-path=${PWD}/tmp -dev=true
    ⇨ http server started on [::]:9994
