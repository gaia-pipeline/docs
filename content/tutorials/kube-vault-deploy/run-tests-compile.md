---
title: "2 - Run tests and compile pipeline"
description: "Run local tests and compile pipeline in Gaia"
weight: 30
---

## Run tests

Gaia really shines when it comes to test your developed pipelines. You should now be able to run all tests locally in the cloned pipline source folder:

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
nginx-6c5f974bd7-mlhgz   1/1       Running   0          13s
```

It works! For testing purpose, let us now remove the namespace (and therefore all included objects):

```
kubectl delete ns nginx
```
<br />

## Compile pipeline and start it via Gaia

The last steps should be quite straightforward. We will log into Gaia, create a new pipeline and Gaia will automatically build the pipeline for us. We will store our credentials (HashiCorp Vault token) in Gaia's Vault and then start the pipeline.

Open Gaia and log in via the default credentials admin/admin: <a href="http://localhost:8080" target="_blank">http://localhost:8080</a>

Create now a new pipeline and fill all required information. Please have a look at [1 - First pipeline]({{%relref "getting-started/first-pipeline.md"%}}) if you have problems to create a new pipeline.

In the left Menubar you should be able to find the entry point `Vault`. Click on it to open Gaia's internal Vault which should be used to store credentials and any sensitive data required by your pipelines during execution. 
Click on `Add Secrets` to add a new secret to vault. Please create now two new secrets:

* Key: vault-token; Value: "root-token"
* Key: vault-address; Value: "http://vault:8200"

Now we can start our pipeline by clicking on the entry `Overview` and on `Start Pipeline`. 
The execution should be quite fast (usually a 1-5 seconds). 
You can now verify the results via `kubectl get po -n nginx` after successful execution.

Continue with the last chapter: [{{%icon circle-arrow-right%}}3 - Conclusion]({{%relref "tutorials/kube-vault-deploy/conclusion.md"%}})

