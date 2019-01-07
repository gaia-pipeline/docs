---
title: "RFC: Distributed Execution Of Pipelines"
description: "An RFC describing the mechanics of distributed worker based execution of a pipeline instead of a single master."
weight: 20
---

# Abstract

This document discusses the problem of executing pipelines in a distributed
manner.

# Table of Contents

1. [Introduction](#introduction)
2. [Problem Statement](#problem-statement)
3. [Terminology](#terminology)
4. [Architecture Diagram](#architercutre-diagram)
5. [Proposed Worker Distribution Model](#proposed-worker-distribution-model)
6. [Managing Workers](#managing-workers)
7. [Worker Tags](#worker-tags)
8. [The Worker RPC API](#the-worker-rpc-api)
9. [Gaia Master - Agent](#gaia-master-agent)
10. [Scheduling Jobs](#scheduling-jobs)
11. [Implementation Approach](#implementation-approach)


# Introduction

# Problem Statement

The problem poses the following set of challenges for Gaia:

1. Manage workers
    - See what pipeline is running on which worker at any given point in time
    - Add / Delete / Suspend workers
    - Add specific environment variables to the worker
2. Either automatically, or manually choose which pipeline should run on which
worker.
3. Label the workers so the user knows it's a windows machine or a linux machine
or Go, Python, Java SDK is available on it... etc.

# Terminology

**Gaia Master**: The Gaia Master is a running instance of gaia launched via make or the
released Gaia binary.
**Worker**: A worker is a server which is connected to the Gaia Master and has
certain capabilities like, what kind of SDK it supports or what operating system
is installed on it.
**Pipeline**: A pipeline is a configured entity with a set of Jobs.
**Job**: A job is a single running task like, create a user. A pipeline can have multiple jobs.
**RPC**: Remote Procedure Call

# Architecture Diagram

![distributed workers](https://user-images.githubusercontent.com/182850/45118121-c890bd00-b157-11e8-9eec-04ed9e8e4a05.png)

# Proposed Worker Distribution Model

The proposed model which aims to solve this problem is laid out as follows.

# Managing Workers

The managing of the workers will happen through a set of API endpoints.
All workers are stored in the database with a designated set of labels
assigned name and IP address.

These endpoints will be Delete / Suspend. Since adding will be taken care of
by the Gaia agent, we don't support that operation here specifically.

**Delete**: Delete will simply remove the server from the rotation. It won't restart
the server, or shut it down, it will just simply delete it from the database which
holds the worker instances.
**Suspend**: Suspending a worker will take it out of rotation but will not delete it.
Suspended this worker will not be able to run any pipelines. This is a good option
if some kind of maintenance needs to be performed on the machine.

## Worker Tags

The workers will need to be tagged with what kind of resource they are providing. For example:

| name  | tags |
|-------|------|
| Worker 1 | Ubuntu Linux 64bit |
| Worker 2 | Windows 10 64bit |
| Worker 3 | Debian Linux 64bit |

When a pipeline is first created in needs to set on the pipeline creation window what
kind of resources it requires. These tags will need to be made accessible by a drop down
list for ease of usage. These tags can be created when a Worker is created and saved to Gaia.
Tagging them can be done manually on the Worker Manager screen.

## The Worker RPC API

The Workers will talk to the Gaia Master via a set of defined RPC interfaces.
These are as follows:

```go
// RegisterWorker will take a worker struct which contains the following information:
// Security: This will be protected by the TLS connection between master and worker.
// IP: The address of the worker
// Name: The name of the worker which typically can be `hostname`.
// Operating System: The OS of the worker to save as a label.
// SDK: The SDK the worker has.
rpc RegisterWorker(Worker) {}

// RunPipeline will take a pipeline, and execute it. This ia bi-directional endpoint.
// Pipeline struct:
// ID: Id of the pipeline
// Repo: The git repository for the pipeline. This is needed because the worker needs
// to build the pipeline.
rpc RunPipeline(Pipeline) returns (Success) {}

rpc GetAllPipelines(Worker) returns (Pipelines) {}
```

# Gaia Master - Agent

The current Gaia implementation will still hold and will be designated as Gaia Master.
The master will be a hub for the worker to connect to, get pipelines from, and report
back on the current state of the pipelines they are running.

As such, Gaia Master will no longer be solely responsible to build and distribute
binaries. Since the operation system of the worker decides in what format the binaries
will be in, the workers will build their own binaries.

Which means a worker will get a repository to pull code from and do the whole thing
that Gaia does currently. This will not involve duplicating code however, since the
whole thing will be in the `worker` package. Gaia Master will use this package by
setting worker to `localhost`.

The Workers will need to have the go-plugin extracted because HashiCorp's plugin
system does not support RPC calls over the network. Just strictly localhost communication
is allowed. Pipeline execution and communication between jobs' running and state
changes are all through RPC.

# Scheduling Jobs

Scheduling jobs will also have to be included into the workers. Workers will schedule
their own parallel jobs execution model and Gaia Master will have to schedule and manage
which worker to distribute pipelines to. This means that the workers will need an indicator
to define when they are too busy to accept more pipelines.

# Where jobs are built

Currently, once a user initiates a pipeline build, that pipeline is saved and built on Gaia Master.
This has to change in order for the worker to be able to run the pipeline. The binary
needs to be built on the worker. However, Gaia also needs to be aware of the jobs,
and does pre-validation which means it also needs to build the pipeline.

# Gaia Master remains

If Gaia does not have any workers, the Gaia master must still be able to take on jobs, build
and execute them. Thus the Gaia master must retain the abilities of a worker as well as a
server orchestrator.

## Scenario 1:
We build the pipeline on both, the Gaia master, and the Worker. Which means we get immediate
validation of the pipeline but have to duplicate the building process.

## Scenario 2:
We only build on the worker and just save the pipeline on the master to track it. The
validation will be deferred until it's actually built on one of the workers. This way,
validation is deferred but the building process isn't duplicated.

# Implementation Approach

1. Extract all functionality regarding running and building pipelines including
the SDK and the go-plugin facility into a worker package. This should not change
the current behavior of Gaia. All tests should still pass. Including the WebHook
capability which should be able to still just call build. The worker package should
take care of building and distributing the binary.

2. Create the API which handles most of the things worker related. But still don't
bother extracting it.

3. Create an Agent binary which calls back to master's RPC API and registers a
server as a worker.

4. Implement the managing of the servers below settings on the left of the admin
screen.
