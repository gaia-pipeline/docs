---
title: "Python"
description: "Develop pipelines in Python"
weight: 30
---

#### Requirements

* Gaia is up and running. Python2.7, pip and virtualenv is available: [Install Gaia]({{%relref "getting-started/install-gaia.md"%}})
* <a href="https://github.com/gaia-pipeline/python-example" target="_blank">Python example</a> forked git repository. We fork it to get the `setup.py` file and the required folder structure.
<br/><br/>


#### Setup

Checkout your forked git repository locally:

```
git clone https://github.com/username/python-example
```

Open `__init__.py` file:

```
vi gitreponame/pipeline/__init__.py
```

Replace an example from below with the content of this file and push the changes to remote. Afterwards you can use [create pipeline]({{%relref "getting-started/first-pipeline.md"%}}) as usual to compile and start your pipeline.
<br/><br/>


#### Simple pipeline with one job

This example shows the most simple way of developing a pipeline:

```python
from gaiasdk import sdk
import logging

def MyAwesomeJob(args):
    logging.info("This output will be streamed back to gaia and will be displayed in the pipeline logs.")
    # Just raise an exception to tell Gaia if a job failed.
    # raise Exception("Oh no, this job failed!")

def main():
    logging.basicConfig(level=logging.INFO)
    myjob = sdk.Job("MyAwesomeJob", "Do something awesome", MyAwesomeJob)
    sdk.serve([myjob])
```
<br/>


#### DependsOn example

You can fork this example <a href="https://github.com/gaia-pipeline/python-example" target="_blank">here.</a>

```python
from gaiasdk import sdk
import logging
import time

def CreateUser(args):
    logging.info("CreateUser has been started!")
    time.sleep(5)
    logging.info("CreateUser has been finished!")

def MigrateDB(args):
    logging.info("MigrateDB has been started!")
    time.sleep(5)
    logging.info("MigrateDB has been finished!")

def CreateNamespace(args):
    logging.info("CreateNamespace has been started!")
    time.sleep(5)
    logging.info("CreateNamespace has been finished!")

def CreateDeployment(args):
    logging.info("CreateDeployment has been started!")
    time.sleep(5)
    logging.info("CreateDeployment has been finished!")

def CreateService(args):
    logging.info("CreateService has been started!")
    time.sleep(5)
    logging.info("CreateService has been finished!")

def CreateIngress(args):
    logging.info("CreateIngress has been started!")
    time.sleep(5)
    logging.info("CreateIngress has been finished!")

def Cleanup(args):
    logging.info("Cleanup has been started!")
    time.sleep(5)
    logging.info("Cleanup has been finished!")

def main():
    logging.basicConfig(level=logging.INFO)
    createuser = sdk.Job("Create DB User", "Creates a database user with least privileged permissions.", CreateUser)
    migratedb = sdk.Job("DB Migration", "Imports newest test data dump and migrates to newest version.", MigrateDB, ["Create DB User"])
    createnamespace = sdk.Job("Create K8S Namespace", "Creates a new Kubernetes namespace for the new test environment.", CreateNamespace, ["DB Migration"])
    createdeployment = sdk.Job("Create K8S Deployment", "Creates a new Kubernetes deployment for the new test environment.", CreateDeployment, ["Create K8S Namespace"])
    createservice = sdk.Job("Create K8S Service", "Creates a new Kubernetes service for the new test environment.", CreateService, ["Create K8S Namespace"])
    createingress = sdk.Job("Create K8S Ingress", "Creates a new Kubernetes ingress for the new test environment.", CreateIngress, ["Create K8S Namespace"])
    cleanup = sdk.Job("Clean up", "Removes all temporary files.", Cleanup, ["Create K8S Deployment", "Create K8S Service", "Create K8S Ingress"])
    sdk.serve([createuser, migratedb, createnamespace, createdeployment, createservice, createingress, cleanup])

```
<br/>


#### Pipeline parameters example

```python
from gaiasdk import sdk
import logging

def MyAwesomeJob(args):
    for arg in args:
        logging.info("Key: " + str(arg.key) + "; Value: " + str(arg.value))

def main():
    logging.basicConfig(level=logging.INFO)
    # Instead of sdk.InputType.TextFieldInp you can also use sdk.InputType.TextAreaInp
    # for a text area or sdk.InputType.BoolInp for boolean input.
    argParam = sdk.Argument("Type in your username:", sdk.InputType.TextFieldInp, "username")
    myjob = sdk.Job("MyAwesomeJob", "Do something awesome", MyAwesomeJob, None, [argParam])
    sdk.serve([myjob])
```
<br/>


#### Vault parameter example

{{% notice tip %}}
You must add a new vault entry with the key `dbpassword` before you can start the pipeline below. An error will be thrown if a pipeline requests a vault parameter which does not exist.
{{% /notice %}}

```python
from gaiasdk import sdk
import logging

def MyAwesomeJob(args):
    for arg in args:
        logging.info("Key: " + str(arg.key) + "; Value: " + str(arg.value))

def main():
    logging.basicConfig(level=logging.INFO)
    # Instead of sdk.InputType.TextFieldInp you can also use sdk.InputType.TextAreaInp
    # for a text area or sdk.InputType.BoolInp for boolean input.
    vaultParam = sdk.Argument("", sdk.InputType.VaultInp, "dbpassword")
    myjob = sdk.Job("MyAwesomeJob", "Do something awesome", MyAwesomeJob, None, [vaultParam])
    sdk.serve([myjob])
```
