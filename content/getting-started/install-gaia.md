---
title: "0 - Installation"
description: "Installation"
weight: 10
---

### Installation

The installation of Gaia is simple and often takes a few minutes.

#### Using Docker

The following command starts Gaia as a daemon process and mounts all data to the current folder. Afterwards, <br /> 
Gaia will be available on the host system on port 8080 (<a href="http://localhost:8080/" target="_blank">http://localhost:8080/</a>). <br /> 
Use the standard user "**admin**" and password "**admin**" as initial login. It is recommended to change the password afterwards.

    docker run -d -p 8080:8080 -v $PWD:/data gaiapipeline/gaia:latest

This Docker image also includes the Go-Compiler and Git which is needed to get the dependencies and compile pipelines written in go.
<br /><br />

#### Manually

It is possible to install gaia directly on the host system. This can be achieved by downloading the binary from the <br /> 
<a href="https://github.com/gaia-pipeline/gaia/releases" target="_blank">releases page</a>. <br />
Use the standard user "**admin**" and password "**admin**" as initial login. It is recommended to change the password afterwards.

gaia will automatically detect the folder of the binary and will place all data next to it. <br /> 
You can change the data directory with the startup parameter --homepath if you want. <br />
Other startup parameters are also available:

    Usage of gaia:
        -homepath string
    	    Path to the gaia home folder
        -port string
    	    Listen port for gaia (default "8080")
        -workers int
    	    Number of workers gaia will use to execute pipelines in parallel (default 2)
