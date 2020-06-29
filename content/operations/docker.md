---
title: "Gaia Docker Pipelines" 
description: "Operations of Gaia Docker Pipelines"
weight: 10
---

## Gaia Docker Pipelines

By default, all pipelines are built and executed on the host machine where Gaia is running.
When Gaia is running in a Docker container, pipelines are also built and executed in the same Docker container.
This behavior can be suboptimal in certain cases e.g. when the pipeline needs external libraries which are not available 
on the host system or when it comes to security considerations around pipelines.

To support these workflows, Gaia allows you to run (not build) your pipelines inside a Docker container.
When a new Pipeline run with Docker mode enabled has been triggered, Gaia automatically pulls the Gaia Docker image
and starts a new Gaia worker which will connect to the Gaia primary instance. The worker automatically pulls the pipeline
binary and runs it inside the Docker container. Once the pipeline run is finished, Gaia will deregister the worker and 
destroy the container. Pipeline run information like logs and the status are automatically pushed to the primary Gaia
instance and can be monitored, as always, in the UI.

### Configuration

If your Gaia instance is installed bare on the host machine (not in a Docker container), Gaia will automatically try
to look for the "/var/run/docker.sock" socket file. You can configure that by defining the "-docker-host-url" parameter.
It is very important that Gaia is able to access the Docker daemon API. Especially when your Gaia instance is running in
a Docker container.

Since Gaia starts a worker inside a Docker container to run Docker pipeline runs, the worker needs access to gRPC/API 
from the primary Gaia instance to register itself and push status information. Therefore, it is important to set
"-docker-worker-grpc-host-url" and "-docker-worker-host-url" to the correct address. Please note that the local loopback
address usually does not work since the worker is isolated in the container.  

Gaia automatically starts a Gaia worker inside a Docker container which then executes your pipeline. Sometimes it might
be helpful to change the used Docker image to run your pipeline. The parameter "-docker-run-image" can be used to define
a custom image which should be used for your Docker pipeline runs.

Please see an example configuration here:

    -docker-host-url="unix:///var/run/docker.sock"
    -docker-run-image="gaiapipeline/gaia:latest"
    -docker-worker-grpc-host-url="my-gaia-instance:8989"
    -docker-worker-host-url="http://my-gaia-instance:8080"

### Enable Docker Pipelines

Gaia allows you to enable Docker pipeline runs via two different options:
 
1. On the UI, set the Docker switch next to the "Start Pipeline" button to "Docker enabled" before running the pipeline.
   This will automatically enable Docker pipeline runs for this pipeline only.
2. Enable the auto docker mode via "-auto-docker-mode" which will automatically enable Docker pipeline runs
   for all pipelines globally when starting up Gaia from the command line.

### Docker Pipelines via Gaia worker

It is also possible that a remote Gaia worker executes your pipelines in a Docker container.
Please see the below example configuration for the remote worker:

    -worker-host-url=http://<ip/dns-name-of-primary-instance>:8080
    -mode=worker
    -worker-name=my-worker
    -worker-grpc-host-url=<ip/dns-name-of-primary-instance>:8989
    -worker-secret=<global-worker-register-secret>
    -docker-worker-host-url=http://<ip/dns-name-of-worker>:8282
    -docker-worker-grpc-host-url=<ip/dns-name-of-worker>:8888

