---
title: "NodeJS"
description: "Develop pipelines in NodeJS"
weight: 60
---

#### Requirements

* Gaia is up and running.
* Node (min. v10.15.0) and Tar (usually always preinstalled on Unix systems) is installed. 
* Empty git repository which can be accessed from Gaia. For this example we use <a href="https://github.com" target="_blank">https://github.com</a>.
<br/><br/>


#### Setup

Checkout your empty git repository locally:

```
git clone https://github.com/username/gitreponame
```

Switch into your cloned folder:

```
cd gitreponame
```

Create a new Node project by using NPM:

```
npm init
```

This should create a new `package.json` file. Now you need to add the Gaia nodejs SDK 
to the dependency list:

```
npm i @gaia-pipeline/nodesdk --save
```

Create the main index file which will be invoked on pipeline startup:
{{% notice tip %}}
The main index file must always have the name `index.js` and must be in the main folder
to allow Gaia to find the entry file.
{{% /notice %}}

```
touch index.js
```
 
It is recommended (but optional) to create a `.gitignore` file to prevent that certain files
(node_modules) are tracked by Git. Gaia automatically installs dependencies during installation:

```
echo "node_modules/" >> .gitignore
```

Paste an example from below into `index.js` and push the changes to remote. Afterwards you can use [create pipeline]({{%relref "getting-started/first-pipeline.md"%}}) as usual to compile and start your pipeline.
<br/><br/>


#### Simple pipeline with one job

This example shows the most simple way of developing a pipeline:

```javscript
const nodesdk = require("@gaia-pipeline/nodesdk");

function DoSomethingAwesome(args) {
    console.error("This output will be streamed back to gaia and will be displayed in the pipeline logs.");

    // An error occurred? Throw it back so gaia knows that this job failed.
    // throw new Error('My error message');
}

// Serve
try {
    nodesdk.Serve([{
        handler: DoSomethingAwesome,
        title: "DoSomethingAwesome",
        description: "This job does something awesome."
    }]);
} catch (err) {
    console.error(err);
}
```
<br/>


#### DependsOn example

You can fork this example <a href="https://github.com/gaia-pipeline/nodejs-example/tree/depends_on" target="_blank">here.</a>

```javascript
const nodesdk = require('@gaia-pipeline/nodesdk');

function CreateUser(args) {
    console.error('CreateUser has been started!');
    // Lets sleep a few seconds to simulate that we do something.
    wait(2000);
    console.error('CreateUser has been finished!');
}

function MigrateDB(args) {
    console.error('MigrateDB has been started!');
    // Lets sleep a few seconds to simulate that we do something.
    wait(2000);
    console.error('MigrateDB has been finished!');
}

function CreateNamespace(args) {
    console.error('CreateNamespace has been started!');
    // Lets sleep a few seconds to simulate that we do something.
    wait(2000);
    console.error('CreateNamespace has been finished!');
}

function CreateDeployment(args) {
    console.error('CreateDeployment has been started!');
    // Lets sleep a few seconds to simulate that we do something.
    wait(2000);
    console.error('CreateDeployment has been finished!');
}

function CreateService(args) {
    console.error('CreateService has been started!');
    // Lets sleep a few seconds to simulate that we do something.
    wait(2000);
    console.error('CreateService has been finished!');
}

function CreateIngress(args) {
    console.error('CreateIngress has been started!');
    // Lets sleep a few seconds to simulate that we do something.
    wait(2000);
    console.error('CreateIngress has been finished!');
}

function Cleanup(args) {
    console.error('Cleanup has been started!');
    // Lets sleep a few seconds to simulate that we do something.
    wait(2000);
    console.error('Cleanup has been finished!');
}

// This is not the right way to do it but it works for this example.
// You should not use this for real pipelines/applications.
function wait(ms) {
    let start = new Date().getTime();
    let end = start;
    while (end < start + ms) {
        end = new Date().getTime();
    }
}

// Serve
try {
    nodesdk.Serve([
        {
            handler: CreateUser,
            title: 'Create DB User',
            description: 'Create DB User with least privileged permissions.'
        },
        {
            handler: MigrateDB,
            title: 'DB Migration',
            description: 'Imports newest test data dump and migrates to newest version.',
            dependson: ['Create DB User']
        },
        {
            handler: CreateNamespace,
            title: 'Create K8S Namespace',
            description: 'Create a new Kubernetes namespace for the new test environment.',
            dependson: ['DB Migration']
        },
        {
            handler: CreateDeployment,
            title: 'Create K8S Deployment',
            description: 'Create a new Kubernetes deployment for the new test environment.',
            dependson: ['Create K8S Namespace']
        },
        {
            handler: CreateService,
            title: 'Create K8S Service',
            description: 'Create a new Kubernetes service for the new test environment.',
            dependson: ['Create K8S Namespace']
        },
        {
            handler: CreateIngress,
            title: 'Create K8S Ingress',
            description: 'Create a new Kubernetes ingress for the new test environment.',
            dependson: ['Create K8S Namespace']
        },
        {
            handler: Cleanup,
            title: 'Clean up',
            description: 'Removes all temporary files.',
            dependson: ['Create K8S Deployment', 'Create K8S Service', 'Create K8S Ingress']
        }
    ]);
} catch (err) {
    console.error(err);
}
```
<br/>


#### Pipeline parameters example

You can fork this example <a href="https://github.com/gaia-pipeline/nodejs-example/tree/parameters_example" target="_blank">here.</a>

```javascript
const nodesdk = require('@gaia-pipeline/nodesdk');

function PrintArgs(args) {
    for (let i = 0; i < args.length; i++) {
        console.error('Key: ' + args[i].key + '; Value: ' + args[i].value);
    }
}

// Serve
try {
    nodesdk.Serve([
        {
            handler: PrintArgs,
            title: 'Print Arguments',
            description: 'This job prints out all given arguments.',
            args: [
                {
                    description: 'Please provide the username:',
                    // This will use a textfield in the UI. You can also use
                    // "textarea", "boolean" and "vault".
                    type: 'textfield',
                    key: 'username'
                }
            ]
        },
    ]);
} catch (err) {
    console.error(err);
}
```
<br/>


#### Vault parameter example

You can fork this example <a href="https://github.com/gaia-pipeline/nodejs-example/tree/vault_example" target="_blank">here.</a>

{{% notice tip %}}
You must add a new vault entry with the key `dbpassword` before you can start the pipeline below. An error will be thrown if a pipeline requests a vault parameter which does not exist.
{{% /notice %}}

```javascript
const nodesdk = require('@gaia-pipeline/nodesdk');

function PrintArgs(args) {
    for (let i = 0; i < args.length; i++) {
        console.error('Key: ' + args[i].key + '; Value: ' + args[i].value);
    }
}

// Serve
try {
    nodesdk.Serve([
        {
            handler: PrintArgs,
            title: 'Print Arguments',
            description: 'This job prints out all given arguments.',
            args: [
                {
                    // This will use a textfield in the UI. You can also use
                    // "textarea", "boolean" and "vault".
                    type: 'vault',
                    key: 'dbpassword'
                }
            ]
        },
    ]);
} catch (err) {
    console.error(err);
}
```
