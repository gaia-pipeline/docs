---
title: "Java"
description: "Develop pipelines in Java"
weight: 20
---

#### Requirements

* Gaia is up and running. Java SDK and maven is available: [Install Gaia]({{%relref "getting-started/install-gaia.md"%}})
* <a href="https://github.com/gaia-pipeline/java-example" target="_blank">Java example</a> forked git repository. We fork it to get the `pom.xml` file and the required folder structure.
<br/><br/>


#### Setup

Checkout your forked git repository locally:

```
git clone https://github.com/username/java-example
```

Open `Pipeline.java` file:

```
vi gitreponame/src/main/java/io/gaiapipeline/Pipeline.java
```

Replace an example from below with the content of this file and push the changes to remote. Afterwards you can use [create pipeline]({{%relref "getting-started/first-pipeline.md"%}}) as usual to compile and start your pipeline.
<br/><br/>


#### Simple pipeline with one job

This example shows the most simple way of developing a pipeline:

```java
package io.gaiapipeline;

import io.gaiapipeline.javasdk.*;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.logging.Logger;

public class Pipeline
{
    private static final Logger LOGGER = Logger.getLogger(Pipeline.class.getName());

    private static Handler MyAwesomeJob = (gaiaArgs) -> {
        LOGGER.info("This output will be streamed back to gaia and will be displayed in the pipeline logs.");
        // Job failed? Throw an error to inform gaia
        // throw new java.lang.Error("something bad happened!");
    };

    public static void main( String[] args )
    {
        PipelineJob myjob = new PipelineJob();
        myjob.setTitle("MyAwesomeJob");
        myjob.setDescription("Do something awesome.");
        myjob.setHandler(MyAwesomeJob);

        Javasdk sdk = new Javasdk();
        try {
            sdk.Serve(new ArrayList<>(Arrays.asList(myjob)));
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```
<br/>


#### DependsOn example

You can fork this example <a href="https://github.com/gaia-pipeline/java-example" target="_blank">here.</a>

```java
package io.gaiapipeline;

import io.gaiapipeline.javasdk.*;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.logging.Logger;

public class Pipeline
{
    private static final Logger LOGGER = Logger.getLogger(Pipeline.class.getName());

    private static Handler CreateUser = (gaiaArgs) -> {
        LOGGER.info("CreateUser has been started!");
        Thread.sleep(5000);
        LOGGER.info("CreateUser has been finished!");
    };

    private static Handler MigrateDB = (gaiaArgs) -> {
        LOGGER.info("MigrateDB has been started!");
        Thread.sleep(5000);
        LOGGER.info("MigrateDB has been finished!");
    };

    private static Handler CreateNamespace = (gaiaArgs) -> {
        LOGGER.info("CreateNamespace has been started!");
        Thread.sleep(5000);
        LOGGER.info("CreateNamespace has been finished!");
    };

    private static Handler CreateDeployment = (gaiaArgs) -> {
        LOGGER.info("CreateDeployment has been started!");
        Thread.sleep(5000);
        LOGGER.info("CreateDeployment has been finished!");
    };

    private static Handler CreateService = (gaiaArgs) -> {
        LOGGER.info("CreateService has been started!");
        Thread.sleep(5000);
        LOGGER.info("CreateService has been finished!");
    };

    private static Handler CreateIngress = (gaiaArgs) -> {
        LOGGER.info("CreateIngress has been started!");
        Thread.sleep(5000);
        LOGGER.info("CreateIngress has been finished!");
    };

    private static Handler Cleanup = (gaiaArgs) -> {
        LOGGER.info("Cleanup has been started!");
        Thread.sleep(5000);
        LOGGER.info("Cleanup has been finished!");
    };

    public static void main( String[] args )
    {
        PipelineJob createuser = new PipelineJob();
        createuser.setTitle("Create DB User");
        createuser.setDescription("Creates a database user with least privileged permissions.");
        createuser.setHandler(CreateUser);

        PipelineJob migratedb = new PipelineJob();
        migratedb.setTitle("DB Migration");
        migratedb.setDescription("Imports newest test data dump and migrates to newest version.");
        migratedb.setHandler(MigrateDB);
        migratedb.setDependsOn(new ArrayList<>(Arrays.asList("Create DB User")));

        PipelineJob createnamespace = new PipelineJob();
        createnamespace.setTitle("Create K8S Namespace");
        createnamespace.setDescription("Creates a new Kubernetes namespace for the new test environment.");
        createnamespace.setHandler(CreateNamespace);
        createnamespace.setDependsOn(new ArrayList<>(Arrays.asList("DB Migration")));

        PipelineJob createdeployment = new PipelineJob();
        createdeployment.setTitle("Create K8S Deployment");
        createdeployment.setDescription("Creates a new Kubernetes deployment for the new test environment.");
        createdeployment.setHandler(CreateDeployment);
        createdeployment.setDependsOn(new ArrayList<>(Arrays.asList("Create K8S Namespace")));

        PipelineJob createservice = new PipelineJob();
        createservice.setTitle("Create K8S Service");
        createservice.setDescription("Creates a new Kubernetes service for the new test environment.");
        createservice.setHandler(CreateService);
        createservice.setDependsOn(new ArrayList<>(Arrays.asList("Create K8S Namespace")));

        PipelineJob createingress = new PipelineJob();
        createingress.setTitle("Create K8S Ingress");
        createingress.setDescription("Creates a new Kubernetes ingress for the new test environment.");
        createingress.setHandler(CreateIngress);
        createingress.setDependsOn(new ArrayList<>(Arrays.asList("Create K8S Namespace")));

        PipelineJob cleanup = new PipelineJob();
        cleanup.setTitle("Clean up");
        cleanup.setDescription("Removes all temporary files.");
        cleanup.setHandler(Cleanup);
        cleanup.setDependsOn(new ArrayList<>(Arrays.asList("Create K8S Deployment", "Create K8S Service", "Create K8S Ingress")));

        Javasdk sdk = new Javasdk();
        try {
            sdk.Serve(new ArrayList<>(Arrays.asList(createuser, migratedb, createnamespace, createdeployment, createservice, createingress, cleanup)));
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```
<br/>


#### Pipeline parameters example

```java
package io.gaiapipeline;

import io.gaiapipeline.javasdk.*;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.logging.Logger;

public class Pipeline
{
    private static final Logger LOGGER = Logger.getLogger(Pipeline.class.getName());

    private static Handler MyAwesomeJob = (gaiaArgs) -> {
        for (PipelineArgument arg: gaiaArgs) {
            LOGGER.info("Key: " + arg.getKey() + "; Value: " + arg.getValue());
        }
    };

    public static void main( String[] args )
    {
        PipelineJob myjob = new PipelineJob();
        myjob.setTitle("MyAwesomeJob");
        myjob.setDescription("Do something awesome.");
        myjob.setHandler(MyAwesomeJob);
        PipelineArgument argUsername = new PipelineArgument();
        // Instead of InputType.TextFieldInp you can also use InputType.TextAreaInp
        // for a text area or InputType.BoolInp for boolean input.
        argUsername.setType(InputType.TextFieldInp);
        argUsername.setKey("username");
        argUsername.setDescription("Please provide the username:");
        myjob.setArgs(new ArrayList<>(Arrays.asList(argUsername)));

        Javasdk sdk = new Javasdk();
        try {
            sdk.Serve(new ArrayList<>(Arrays.asList(myjob)));
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```
<br/>


#### Vault parameter example

{{% notice tip %}}
You must add a new vault entry with the key `dbpassword` before you can start the pipeline below. An error will be thrown if a pipeline requests a vault parameter which does not exist.
{{% /notice %}}

```java
package io.gaiapipeline;

import io.gaiapipeline.javasdk.*;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.logging.Logger;

public class Pipeline
{
    private static final Logger LOGGER = Logger.getLogger(Pipeline.class.getName());

    private static Handler MyAwesomeJob = (gaiaArgs) -> {
        for (PipelineArgument arg: gaiaArgs) {
            LOGGER.info("Key: " + arg.getKey() + "; Value: " + arg.getValue());
        }
    };

    public static void main( String[] args )
    {
        PipelineJob myjob = new PipelineJob();
        myjob.setTitle("MyAwesomeJob");
        myjob.setDescription("Do something awesome.");
        myjob.setHandler(MyAwesomeJob);
        PipelineArgument vaultParam = new PipelineArgument();
        vaultParam.setType(InputType.VaultInp);
        vaultParam.setKey("dbpassword");
        myjob.setArgs(new ArrayList<>(Arrays.asList(vaultParam)));

        Javasdk sdk = new Javasdk();
        try {
            sdk.Serve(new ArrayList<>(Arrays.asList(myjob)));
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```
