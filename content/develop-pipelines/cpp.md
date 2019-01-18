---
title: "C++"
description: "Develop pipelines in C++"
weight: 40
---

#### Requirements

* Gaia is up and running.
* Protobuf and gRPC needs to be installed: <a href="https://github.com/grpc/grpc/blob/master/src/cpp/README.md#make" target="_blank">Protobuf and gRPC Installation</a>
* C++ compiler. By default, the <a href="https://github.com/gaia-pipeline/cppsdk/blob/master/Makefile_pipeline#L19v" target="_blank">Makefile</a> looks for g++. Please change this if you use a different compiler.
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

Checkout the Gaia C++ SDK:

```
git clone https://github.com/gaia-pipeline/cppsdk
```

Remove the `.git` folder from the SDK:

```
rm -rf cppsdk/.git
```

Copy the `Makefile_pipeline` file into your top level cloned folder:

```
cp cppsdk/Makefile_pipeline Makefile
```

Create a new C++ source file in your top level cloned folder. It's important that the file ends with `.cc` as file type.

```
touch main.cc
```

Paste an example from below into `main.cc` and push the changes to remote. Afterwards you can use [create pipeline]({{%relref "getting-started/first-pipeline.md"%}}) as usual to compile and start your pipeline.
<br/><br/>


#### Simple pipeline with one job

This example shows the most simple way of developing a pipeline:

```cpp
#include "cppsdk/sdk.h"
#include <list>
#include <iostream>

void DoSomethingAwesome(std::list<gaia::argument> args) throw(std::string) {
    std::cerr << "This output will be streamed back to gaia and will be displayed in the pipeline logs." << std::endl;

    // An error occured? Return it back so gaia knows that this job failed.
    // throw "Uhh something badly happened!"
}

int main() {
    std::list<gaia::job> jobs;
    gaia::job awesomejob;
    awesomejob.handler = &DoSomethingAwesome;
    awesomejob.title = "DoSomethingAwesome";
    awesomejob.description = "This job does something awesome.";
    jobs.push_back(awesomejob);

    try {
        gaia::Serve(jobs);
    } catch (string e) {
        std::cerr << "Error: " << e << std::endl;
    }
}
```
<br/>


#### DependsOn example

You can fork this example <a href="https://github.com/gaia-pipeline/cpp-example" target="_blank">here.</a>

```cpp
#include "cppsdk/sdk.h"
#include <list>
#include <iostream>
#include <chrono>
#include <thread>

using std::list;
using std::cerr;
using std::endl;
using gaia::argument;
using gaia::job;

void CreateUser(list<argument> args) throw(string) {
    cerr << "CreateUser has been started!" << endl;

    // Print arguments
    for (auto const& arg : args) {
        cerr << "Key: " << arg.key << "; Value: " << arg.value << ";" << endl;
    }
    
    // lets sleep to simulate that we do something
    std::this_thread::sleep_for(std::chrono::milliseconds(2000));

    cerr << "CreateUser has been finished!" << endl;
}

void MigrateDB(list<argument> args) throw(string) {
    cerr << "MigrateDB has been started!" << endl;

    // Print arguments
    for (auto const& arg : args) {
        cerr << "Key: " << arg.key << "; Value: " << arg.value << ";" << endl;
    }

    // lets sleep to simulate that we do something
    std::this_thread::sleep_for(std::chrono::milliseconds(2000));

    cerr << "MigrateDB has been finished!" << endl;
}

void CreateNamespace(list<argument> args) throw(string) {
    cerr << "CreateNamespace has been started!" << endl;

    // lets sleep to simulate that we do something
    std::this_thread::sleep_for(std::chrono::milliseconds(2000));

    cerr << "CreateNamespace has been finished!" << endl;
}

void CreateDeployment(list<argument> args) throw(string) {
    cerr << "CreateDeployment has been started!" << endl;

    // lets sleep to simulate that we do something
    std::this_thread::sleep_for(std::chrono::milliseconds(2000));

    cerr << "CreateDeployment has been finished!" << endl;
}

void CreateService(list<argument> args) throw(string) {
    cerr << "CreateService has been started!" << endl;

    // lets sleep to simulate that we do something
    std::this_thread::sleep_for(std::chrono::milliseconds(2000));

    cerr << "CreateService has been finished!" << endl;
}

void CreateIngress(list<argument> args) throw(string) {
    cerr << "CreateIngress has been started!" << endl;

    // lets sleep to simulate that we do something
    std::this_thread::sleep_for(std::chrono::milliseconds(2000));

    cerr << "CreateIngress has been finished!" << endl;
}

void Cleanup(list<argument> args) throw(string) {
    cerr << "Cleanup has been started!" << endl;

    // lets sleep to simulate that we do something
    std::this_thread::sleep_for(std::chrono::milliseconds(2000));

    cerr << "Cleanup has been finished!" << endl;
}

int main() {
    std::list<gaia::job> jobs;
    gaia::job createuser;
    createuser.handler = &CreateUser;
    createuser.title = "Create DB User";
    createuser.description = "Create DB User with least privileged permissions.";

    gaia::job migratedb;
    migratedb.handler = &MigrateDB;
    migratedb.title = "DB Migration";
    migratedb.description = "Imports newest test data dump and migrates to newest version.";
    migratedb.depends_on.push_back("Create DB User");

    gaia::job createnamespace;
    createnamespace.handler = &CreateNamespace;
    createnamespace.title = "Create K8S Namespace";
    createnamespace.description = "Create a new Kubernetes namespace for the new test environment.";
    createnamespace.depends_on.push_back("DB Migration");

    gaia::job createdeployment;
    createdeployment.handler = &CreateDeployment;
    createdeployment.title = "Create K8S Deployment";
    createdeployment.description = "Create a new Kubernetes deployment for the new test environment.";
    createdeployment.depends_on.push_back("Create K8S Namespace");

    gaia::job createservice;
    createservice.handler = &CreateService;
    createservice.title = "Create K8S Service";
    createservice.description = "Create a new Kubernetes service for the new test environment.";
    createservice.depends_on.push_back("Create K8S Namespace");

    gaia::job createingress;
    createingress.handler = &CreateIngress;
    createingress.title = "Create K8S Ingress";
    createingress.description = "Create a new Kubernetes ingress for the new test environment.";
    createingress.depends_on.push_back("Create K8S namespace");

    gaia::job cleanup;
    cleanup.handler = &Cleanup;
    cleanup.title = "Clean up";
    cleanup.description = "Removes all temporary files.";
    cleanup.depends_on.push_back("Create K8S Deployment");
    cleanup.depends_on.push_back("Create K8S Service");
    cleanup.depends_on.push_back("Create K8S Ingress");

    jobs.push_back(createuser);
    jobs.push_back(migratedb);
    jobs.push_back(createnamespace);
    jobs.push_back(createdeployment);
    jobs.push_back(createservice);
    jobs.push_back(createingress);
    jobs.push_back(cleanup);

    // Serve
    try {
        gaia::Serve(jobs);
    } catch (string e) {
        std::cout << "Error: " << e << std::endl;
    }
}
```
<br/>


#### Pipeline parameters example

You can fork this example <a href="https://github.com/gaia-pipeline/cpp-example/tree/parameters_example" target="_blank">here.</a>

```cpp
#include "cppsdk/sdk.h"
#include <list>
#include <iostream>

using std::list;
using std::cerr;
using std::endl;
using gaia::argument;
using gaia::job;

void PrintParam(list<argument> args) throw(string) {
    for (auto const& arg : args) {
        cerr << "Key: " << arg.key << "; Value: " << arg.value << ";" << endl;
    }
}

int main() {
    std::list<job> jobs;
    job printparam;
    printparam.handler = &PrintParam;
    printparam.title = "Print Parameters";
    printparam.description = "This job prints out all given params.";
    printparam.args.push_back(argument{
        "Username for the database schema:",
        // textfield displays a text field in the UI.
        // You can also use textarea for text area, boolean
        // for boolean input.
        gaia::InputType::input_type::textfield,
        "username",
    });
    printparam.args.push_back(argument{
        "Description for username:",
        // textfield displays a text field in the UI.
        // You can also use textarea for text area, boolean
        // for boolean input.
        gaia::InputType::input_type::textarea,
        "usernamedesc",
    });

    jobs.push_back(printparam);

    // Serve
    try {
        gaia::Serve(jobs);
    } catch (string e) {
        std::cout << "Error: " << e << std::endl;
    }
}
```
<br/>


#### Vault parameter example

You can fork this example <a href="https://github.com/gaia-pipeline/cpp-example/tree/vault_example" target="_blank">here.</a>

{{% notice tip %}}
You must add a new vault entry with the key `dbpassword` before you can start the pipeline below. An error will be thrown if a pipeline requests a vault parameter which does not exist.
{{% /notice %}}

```cpp
#include "cppsdk/sdk.h"
#include <list>
#include <iostream>

using std::list;
using std::cerr;
using std::endl;
using gaia::argument;
using gaia::job;

void PrintParam(list<argument> args) throw(string) {
    for (auto const& arg : args) {
        cerr << "Key: " << arg.key << "; Value: " << arg.value << ";" << endl;
    }
}

int main() {
    list<job> jobs;
    job printparam;
    printparam.handler = &PrintParam;
    printparam.title = "Print Parameters";
    printparam.description = "This job prints out all given params which includes one from the vault.";
    printparam.args.push_back(argument{
        "",
        gaia::InputType::input_type::vault,
        "dbpassword",
    });

    jobs.push_back(printparam);

    // Serve
    try {
        gaia::Serve(jobs);
    } catch (string e) {
        std::cout << "Error: " << e << std::endl;
    }
}
```
