---
title: "Distributed Execution Of Pipelines"
description: "An RFC describing the mechanics of distributed worker based execution of a pipeline instead of a single master."
weight: 10
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
9. [Gaia Primary - Agent](#gaia-primary-agent)
10. [Scheduler](#scheduler)
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

**Gaia Primary instance**: The Gaia Primary is a running instance of Gaia launched via docker or the
released Gaia binary.
**Worker**: A worker is a server which is connected to the Gaia Primary instance and has
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
assigned name and a unique identifier.

| API ENDPOINT | Method | Description |
|-------|-------|-------|
| worker/register | POST | Registers a new worker with a name and a global registration secret. Returns a unique identifier, a client certificate, a client key and the required CA cert. This endpoint is mainly used by the remote worker to register itself. |
| worker/secret | GET | Returns the global worker registration secret. |
| worker/secret | POST | Resets the global worker registration secret. |
| worker/:workerid | DELETE | Deregisters a worker from this Gaia primary instance. |
| worker | GET | Returns a list of all registered workers. |
| worker/suspend/:workerid | POST | Suspending a worker will take it out of rotation but will not delete it. Suspended this worker will not be able to run any pipelines. This is a good option if some kind of maintenance needs to be performed on the machine. |

## Worker Tags

The workers will need to be tagged with what kind of resource they are providing. For example:

| name  | tags |
|-------|------|
| Worker 1 | Ubuntu Linux 64bit |
| Worker 2 | Windows 10 64bit |
| Worker 3 | Debian Linux 64bit |

Workers automatically detect on startup which pipeline languages (e.g. Go, Java, Python etc.) are supported
and update their tags accordingly. Negative tags can be used to manually declare a specific language is not 
support on this worker. During pipeline creation, custom tags can be set per pipeline. Pipelines with custom
tags will be only executed on workers with at least the same set of tags.

## The Worker RPC API

The Workers will talk to the Gaia Primary instance via a set of defined gRPC interfaces.
These are as follows:

```go
// WorkerInstance represents the identity of
// a worker instance.
message WorkerInstance {
    string   unique_id    = 1;
    int32    worker_slots = 2;
    repeated string tags  = 3;
}

// PipelineRun represents one pipeline run.
message PipelineRun {
    string   unique_id     = 1;
    int64    id            = 2;
    string   status        = 3;
    int64    start_date    = 4;
	int64    finish_date   = 5;
	int64    schedule_date = 6;
    int64    pipeline_id   = 7;
    string   pipeline_name = 8;
    string   pipeline_type = 9;
    bytes    sha_sum       = 10;
    repeated Job jobs      = 11;
}

// Job represents one job from a pipeline run.
message Job {
    uint32   unique_id      = 1;
	string   title          = 2;
	string   description    = 3;
	repeated Job depends_on = 4;
	string   status         = 5;
	repeated Argument args  = 6;
}

// Argument represents one argument from a job.
message Argument {
    string description = 1;
	string type        = 2;
	string key         = 3;
	string value       = 4;
}

// LogChunk represents one chunk of a log file.
message LogChunk {
    int64 run_id      = 1;
    int64 pipeline_id = 2;
    bytes chunk       = 3;
}

// FileChunk represents one chunk of a file.
message FileChunk {
    bytes chunk = 1;
}

service Worker {
    // GetWork pulls work from the primary instance.
    rpc GetWork (WorkerInstance) returns (stream PipelineRun);

    // UpdateWork updates work information at the primary instance.
    rpc UpdateWork (PipelineRun) returns (google.protobuf.Empty);

    // StreamBinary streams a pipeline binary back to a worker instance.
    rpc StreamBinary (PipelineRun) returns (stream FileChunk);

    // StreamLogs streams pipeline run logs to the primary instance.
    rpc StreamLogs (stream LogChunk) returns (google.protobuf.Empty);

    // Deregister deregister a registered worker from the primary instance.
    rpc Deregister (WorkerInstance) returns (google.protobuf.Empty);
}
```

### Gaia Primary - Agent

The current Gaia implementation will still hold and will be designated as Gaia Primary.
The primary will be a hub for the worker to connect to, get pipelines from, and report
back on the current state of the pipelines they are running.

The Agent acts as a client, it is responsible for continuously pulling work from the
primary instance, sending updates about the status of running or finished pipeline runs
and scheduling work locally. 

The Workers will need to have the go-plugin extracted because HashiCorp's plugin
system does not support RPC calls over the network. Just strictly localhost communication
is allowed. Pipeline execution and communication between jobs' running and state
changes are all through RPC.

### Scheduler

The scheduler will be also included into the workers package. Workers will schedule
their own parallel jobs execution model and Gaia Primary will have to schedule and manage
which worker to distribute pipelines to. This means that the workers will need an indicator
to define when they are too busy to accept more pipelines.

### Pipeline build execution

Currently, once a user initiates a pipeline build, that pipeline is built and stored on Gaia Primary.
For the first implementation, this will not change. Pipelines will be still built on the primary
and once a pipeline run is scheduled on a worker, the pipeline binary will be automatically
streamed to the worker.

In the future, this has to change in order to support different operating systems.
The binary will be built and validated on the worker. All information about the pipeline
(e.g. included jobs, type, binary SHASum etc.) will be streamed back to the primary instance.

### Gaia Primary remains

If Gaia does not have any workers, the Gaia primary must still be able to take on jobs, build
and execute them. Thus, the Gaia primary must retain the abilities of a worker as well as a
server orchestrator.

### Implementation Approach

1. Extract all functionality regarding running and building pipelines including
the SDK and the go-plugin facility into a worker package. This should not change
the current behavior of Gaia. All tests should still pass. Including the WebHook
capability which should be able to still just call build. The worker package should
take care of building and distributing the binary.

2. Create the API which handles most of the things worker related. But still don't
bother extracting it.

3. Create an Agent package which calls back to master's API and registers a
server as a worker.

4. Add a new `mode` option to Gaia which indicates the mode - `server` or `worker` -
Gaia should start in.

5. Implement the managing of the servers below settings on the left of the admin
screen.
