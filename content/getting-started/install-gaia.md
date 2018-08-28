---
title: "0 - Installation"
description: "Installation"
weight: 10
---

### Installation

The installation of Gaia is simple and often takes a few minutes.

#### Using Docker

Gaia offers a various amount of docker images available at <a href="https://hub.docker.com/r/gaiapipeline/gaia/" target="_blank">https://hub.docker.com/r/gaiapipeline/gaia/</a>. If you use the `latest` image tag, you will get a docker image which contains everything to build all currently supporterd pipeline languages but also needs more disk-space.
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
        -capath string
    	    Folder path where the generated CA certificate files will be saved
        -dev
    	    If true, gaia will be started in development mode. Don't use this in production!
        -homepath string
    	    Path to the gaia home folder
        -hostname string
    	    The host's name under which gaia is deployed at e.g.: https://gaia-pipeline.com (default "https://localhost")
        -jwtPrivateKeyPath string
    	    A RSA private key used to sign JWT tokens
        -poll
    	    Instead of using a Webhook, keep polling git for changes on pipelines
        -port string
    	    Listen port for gaia (default "8080")
        -pval int
    	    The interval in minutes in which to poll vcs for changes (default 1)
        -vaultpath string
    	    Path to the gaia vault folder
        -version
    	    If true, will print the version and immediately exit
        -worker string
    	    Number of worker gaia will use to execute pipelines in parallel (default "2")

##### Run-time Arguments

It is possible to define run-time arguments in three ways.

###### As command-line arguments

For example:

    ./cmd/gaia/main -homepath=${PWD}/tmp -dev=true

###### Environment Properties

Environment variables can be used to overwrite any configuration settings / defaults that are already defined in Gaia.

All Environment variables must be pre-fixed with `GAIA_` in order to avoid collision with other variables.

For example:

    export GAIA_PORT=9999
    ./cmd/gaia/main -homepath=${PWD}/tmp -dev=true
    # will start gaia with port 9999
    ⇨ http server started on [::]:9999

###### Configuration file

Alternatively a configuration file can be used with all the properties that gaia defines, called .gaia_config.

If this file is present it will be used to set things up. For example:

    ❯ cat .gaia_config
    port=9994
    ./cmd/gaia/main -homepath=${PWD}/tmp -dev=true
    ⇨ http server started on [::]:9994
