---
title: "Golang"
description: "Develop pipelines in Golang"
weight: 10
---

#### Requirements

* Gaia is up and running. Go compiler is available: [Install Gaia]({{%relref "getting-started/install-gaia.md"%}})
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

Create a new golang file. In this example we name it `main.go`:

```
touch main.go
```

Paste an example from below into `main.go` and push the changes to remote. Afterwards you can use [create pipeline]({{%relref "getting-started/first-pipeline.md"%}}) as usual to compile and start your pipeline.
<br/><br/>


#### Simple pipeline with one job

This example shows the most simple way of developing a pipeline:

{{% notice tip %}}
`package main` is required. This tells the go-compiler that our main entrypoint must be in this package.
{{% /notice %}}

```go
package main

import (
    "log"

    sdk "github.com/gaia-pipeline/gosdk"
)

// This is one job. Add more if you want.
func DoSomethingAwesome(args sdk.Arguments) error {
    log.Println("This output will be streamed back to gaia and will be displayed in the pipeline logs.")

    // An error occured? Return it back so gaia knows that this job failed.
    return nil
}

func main() {
    jobs := sdk.Jobs{
        sdk.Job{
            Handler:     DoSomethingAwesome,
            Title:       "DoSomethingAwesome",
            Description: "This job does something awesome.",
        },
    }

    // Serve
    if err := sdk.Serve(jobs); err != nil {
        panic(err)
    }
}
```
<br/>


#### DependsOn example

You can fork this example <a href="https://github.com/gaia-pipeline/go-example" target="_blank">here.</a>

`main.go`:
```go
package main

import (
	"log"
	"time"

	sdk "github.com/gaia-pipeline/gosdk"
)

func CreateUser(args sdk.Arguments) error {
	log.Println("CreateUser has been started!")

	// lets sleep to simulate that we do something
	time.Sleep(5 * time.Second)
	log.Println("CreateUser has been finished!")
	return nil
}

func MigrateDB(args sdk.Arguments) error {
	log.Println("MigrateDB has been started!")

	// lets sleep to simulate that we do something
	time.Sleep(5 * time.Second)
	log.Println("MigrateDB has been finished!")
	return nil
}

func CreateNamespace(args sdk.Arguments) error {
	log.Println("CreateNamespace has been started!")

	// lets sleep to simulate that we do something
	time.Sleep(5 * time.Second)
	log.Println("CreateNamespace has been finished!")
	return nil
}

func CreateDeployment(args sdk.Arguments) error {
	log.Println("CreateDeployment has been started!")

	// lets sleep to simulate that we do something
	time.Sleep(5 * time.Second)
	log.Println("CreateDeployment has been finished!")
	return nil
}

func CreateService(args sdk.Arguments) error {
	log.Println("CreateService has been started!")

	// lets sleep to simulate that we do something
	time.Sleep(5 * time.Second)
	log.Println("CreateService has been finished!")
	return nil
}

func CreateIngress(args sdk.Arguments) error {
	log.Println("CreateIngress has been started!")

	// lets sleep to simulate that we do something
	time.Sleep(5 * time.Second)
	log.Println("CreateIngress has been finished!")
	return nil
}

func Cleanup(args sdk.Arguments) error {
	log.Println("Cleanup has been started!")

	// lets sleep to simulate that we do something
	time.Sleep(5 * time.Second)
	log.Println("Cleanup has been finished!")
	return nil
}

func main() {
	// Serve
	if err := sdk.Serve(jobs); err != nil {
		panic(err)
	}
}
```

`jobs.go`:
```go
package main

import (
	sdk "github.com/gaia-pipeline/gosdk"
)

var jobs = sdk.Jobs{
	sdk.Job{
		Handler:     CreateUser,
		Title:       "Create DB User",
		Description: "Creates a database user with least privileged permissions.",
	},
	sdk.Job{
		Handler:     MigrateDB,
		Title:       "DB Migration",
		Description: "Imports newest test data dump and migrates to newest version.",
		DependsOn:   []string{"Create DB User"},
	},
	sdk.Job{
		Handler:     CreateNamespace,
		Title:       "Create K8S Namespace",
		Description: "Creates a new Kubernetes namespace for the new test environment.",
		DependsOn:   []string{"DB Migration"},
	},
	sdk.Job{
		Handler:     CreateDeployment,
		Title:       "Create K8S Deployment",
		Description: "Creates a new Kubernetes deployment for the new test environment.",
		DependsOn:   []string{"Create K8S Namespace"},
	},
	sdk.Job{
		Handler:     CreateService,
		Title:       "Create K8S Service",
		Description: "Creates a new Kubernetes service for the new test environment.",
		DependsOn:   []string{"Create K8S Namespace"},
	},
	sdk.Job{
		Handler:     CreateIngress,
		Title:       "Create K8S Ingress",
		Description: "Creates a new Kubernetes ingress for the new test environment.",
		DependsOn:   []string{"Create K8S Namespace"},
	},
	sdk.Job{
		Handler:     Cleanup,
		Title:       "Clean up",
		Description: "Removes all temporary files.",
		DependsOn:   []string{"Create K8S Deployment", "Create K8S Service", "Create K8S Ingress"},
	},
}
```
<br/>


#### Pipeline parameters example

```go
package main

import (
    "log"

    sdk "github.com/gaia-pipeline/gosdk"
)

// PrintParam prints out all given params.
func PrintParam(args sdk.Arguments) error {
    for _, arg := range args {
        log.Printf("Key: %s;Value: %s;\n", arg.Key, arg.Value)
    }
    return nil
}

func main() {
    jobs := sdk.Jobs{
        sdk.Job{
            Handler:     PrintParam,
            Title:       "Print Parameters",
            Description: "This job prints out all given params.",
            Args: sdk.Arguments{
			    sdk.Argument{
                    Description: "Username for the database schema:",
                    // TextFieldInp displays a text field in the UI.
                    // You can also use sdk.TextAreaInp for text area,
                    // sdk.BoolInp for boolean input.
				    Type:        sdk.TextFieldInp,
				    Key:         "username",
                },
                sdk.Argument{
                    Description: "Description for username:",
                    // TextFieldInp displays a text field in the UI.
                    // You can also use sdk.TextAreaInp for text area and
                    // sdk.BoolInp for boolean input.
				    Type:        sdk.TextAreaInp,
				    Key:         "usernamedesc",
			    },
		    },
        },
    }

    // Serve
    if err := sdk.Serve(jobs); err != nil {
        panic(err)
    }
}
```
<br/>


#### Vault parameter example

{{% notice tip %}}
You must add a new vault entry with the key `dbpassword` before you can start the pipeline below. An error will be thrown if a pipeline requests a vault parameter which does not exist.
{{% /notice %}}

```go
package main

import (
    "log"

    sdk "github.com/gaia-pipeline/gosdk"
)

// PrintVaultParam prints out all given params.
func PrintVaultParam(args sdk.Arguments) error {
    for _, arg := range args {
        log.Printf("Key: %s;Value: %s;\n", arg.Key, arg.Value)
    }
    return nil
}

func main() {
    jobs := sdk.Jobs{
        sdk.Job{
            Handler:     PrintVaultParam,
            Title:       "Print Vault Parameters",
            Description: "This job prints out all given params which includes one from the vault.",
            Args: sdk.Arguments{
			    sdk.Argument{
				    Type: sdk.VaultInp,
				    Key:  "dbpassword",
                },
		    },
        },
    }

    // Serve
    if err := sdk.Serve(jobs); err != nil {
        panic(err)
    }
}
```
