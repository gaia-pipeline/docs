---
title: "Gaia Worker" 
description: "Operations of Gaia Worker"
weight: 0
---

## Gaia Worker 

Scalability is one of the leading biggest issues around CI/CD and automation. Often, it is not trivial to scale pipelines horizontally whereas vertical scaling works until a specific point has been reached. Gaia Worker helps you to solve this problem by providing a remote worker functionality for your Gaia primary instance.

Gaia Worker allows you to dynamically add or remove remote worker to your primary Gaia instance and tag those differently. Once a worker has been registered at your primary Gaia instance, new pipeline runs will be automatically scheduled to your worker by matching pipeline tags and worker tags.

While your pipeline run is executed at the remote worker, all pipeline run information like current job status, log output and run status is transferred in real-time to the primary Gaia instance and can be observed via the UI.

The scheduler from the primary Gaia instance automatically monitors and detects the status from remote workers. If a worker goes down or loses the connection to the primary Gaia instance, it will automatically be flagged as inactive and no further pipeline runs will be scheduled on this node till the node is active again. Additionally, the primary Gaia instance also monitors the capacaity from all workers. If a worker is busy, the pipeline run will be scheduled to a
different worker or, if all workers are busy, waits in the queue until the next worker has free capacity again.

### Security information

On the initial startup of the Gaia primary instance, a random generated secret will be stored in the local Vault store. This secret can be obtained via the API and the UI (Settings -> Worker) by Gaia admins and should be used to register new worker at the primary instance. In case the secret has been leaked or intercepted, an Gaia admin can regenerate a new secret via the API and the UI.

Once a worker has been registered at a Gaia primary instance, the primary instance generates a new pair of client certificates for this specific worker. That means, all further communication between the worker and the Gaia primary instance uses from that point mutual TLS, gRPC and a unique worker identitifier which is validated on every request.

If a worker has been deregistered from the primary instance while the worker is still connected to the primary instance, the worker automatically removes its local pair of client certificates. The primary instance removes the worker from the registered worker table which implies that all further connection requests from this worker will be denied.

### Next steps

* [Installation]({{%relref "operations/worker/installation.md"%}})
