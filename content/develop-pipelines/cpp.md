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

Checkout the Gaia C++ SDK:

```
git clone https://github.com/gaia-pipeline/cppsdk
```

Checkout your empty git repository locally:

```
git clone https://github.com/username/gitreponame
```

Switch into your cloned folder:

```
cd gitreponame
```

Copy the Gaia C++ SDK into your repository:

```
cp -R ../cppsdk gitreponame/
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
    // throw "Uhh something badly happend!"
}

int main() {
    std::list<gaia::job> jobs;
    gaia::job awesomejob;
    awesomejob.handler = &DoSomethingAwesome;
    awesomejob.title = "DoSomethingAwesome";
    awesomejob.description = "This job does something awesome.";

    try {
        gaia::Serve(jobs);
    } catch (string e) {
        std:cerr << "Error: " << e << std::endl;
    }
}
```
<br/>


#### DependsOn example

You can fork this example <a href="https://github.com/gaia-pipeline/cpp-example" target="_blank">here.</a>

```cpp
TODO
```
<br/>


#### Pipeline parameters example

```cpp
TODO
```
<br/>


#### Vault parameter example

{{% notice tip %}}
You must add a new vault entry with the key `dbpassword` before you can start the below pipeline. An error will be thrown if a pipeline requests a vault parameter which does not exist.
{{% /notice %}}

```cpp
TODO
```
