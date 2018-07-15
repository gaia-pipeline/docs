---
title: "2 - Run tests and compile pipeline"
description: "Run locally tests and compile pipeline in Gaia"
weight: 30
---

## Run tests

Gaia really shines when it comes to test your developed pipelines. You should now be able to run all tests locally in the cloned folder:

```
go test -v ./...
```

This should print out something like this:

```
2018/07/13 23:28:40 All data has been retrieved from vault!
2018/07/13 23:28:40 Service 'nginx' has been created!
=== RUN   TestCreateService
2018/07/13 23:28:40 Service 'nginx' has been created!
--- PASS: TestCreateService (0.02s)
=== RUN   TestCreateDeployment
2018/07/13 23:28:40 Deployment 'nginx' has been created!
--- PASS: TestCreateDeployment (0.08s)
PASS
ok  	github.com/gaia-pipeline/tutorial-k8s-deployment-go	0.156s
```

Beautiful! You can now verify if everything worked out correctly:

```
kubectl get po -n nginx
```

This should print out something like this:

```
NAME                     READY     STATUS    RESTARTS   AGE
nginx-65847d99c6-lcngz   1/1       Running   0          1m
nginx-65847d99c6-mnmsb   1/1       Running   0          1m
```

It works! For testing purpose, let us now remove the namespace (and therefore all included objects):

```
kubectl delete ns nginx
```
<br />

## Compile pipeline in Gaia

Now it's time to open Gaia: <a href="http://localhost:8080" target="_blank">http://localhost:8080</a>

Create now a new Pipeline and fill all required information. Please have a look at [1 - First pipeline]({{%relref "getting-started/first-pipeline.md"%}}) if you have problems to create a new pipeline.

Don't forget to start your pipeline after compilation. You can also verify the results via `kubectl get po -n nginx` afterwards.

Let's continue with the last chapter: [{{%icon circle-arrow-right%}}3 - Conclusion]({{%relref "tutorials/kube-vault-deploy/conclusion.md"%}})

