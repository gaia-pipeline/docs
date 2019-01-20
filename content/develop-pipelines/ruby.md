---
title: "Ruby"
description: "Develop pipelines in Ruby"
weight: 50
---

#### Requirements

* Gaia is up and running.
* Ruby is installed and `bin` folder is available via PATH variable. Tested and supported version is `2.5.3` and later. 
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

Create a new gemspec file which is required for Gaia:

```
touch gaia.gemspec
```

Add usual gemspec options:
{{% notice tip %}}
Gem name should always include the variable key `${NAME}` which will be automatically replaced during pipeline creation process.
{{% /notice %}}

```
Gem::Specification.new do |s|
  s.name        = '${NAME}'
  s.version     = '0.0.1'
  s.authors     = ["Michel Vocks"]
  s.files       = Dir["{lib}/**/*.rb", "bin/*"]
  s.add_runtime_dependency "rubysdk", ["~> 0.0.1"]
end
```

Create a lib folder:

```
mkdir lib
```

Add your entry ruby file in the lib folder. This file must be named `gaia.rb`. Other files can be named arbitrary: 

```
touch lib/gaia.rb
```

Paste an example from below into `lib/gaia.rb` and push the changes to remote. Afterwards you can use [create pipeline]({{%relref "getting-started/first-pipeline.md"%}}) as usual to compile and start your pipeline.
<br/><br/>


#### Simple pipeline with one job

This example shows the most simple way of developing a pipeline:
{{% notice tip %}}
Your gem entry file should always have the `Main` class and the `main` function defined.
{{% /notice %}}

```ruby
require 'rubysdk'

class Main
    AwesomeJob = lambda do |args|
        STDERR.puts "This output will be streamed back to gaia and will be displayed in the pipeline logs."

        # An error occurred? Raise an exception and gaia will fail the pipeline.
        # raise "Oh gosh! Something went wrong!"
    end

    def self.main
        awesomejob = Interface::Job.new(title: "Awesome Job",
                                        handler: AwesomeJob,
                                        desc: "This job does something awesome.")

        begin
            RubySDK.Serve([awesomejob])
        rescue => e
            puts "Error occured: #{e}"
            exit(false)
        end
    end
end
```
<br/>


#### DependsOn example

You can fork this example <a href="https://github.com/gaia-pipeline/ruby-example" target="_blank">here.</a>

```ruby
require 'rubysdk'

class Main

    CreateUser = lambda do |args|
        STDERR.puts "CreateUser has been started!"
        # lets sleep to simulate that we do something.
        sleep(2.0)
        STDERR.puts "CreateUser has been finished!"
    end

    MigrateDB = lambda do |args|
        STDERR.puts "MigrateDB has been started!"
        # lets sleep to simulate that we do something.
        sleep(2.0)
        STDERR.puts "MigrateDB has been finished!"
    end

    CreateNamespace = lambda do |args|
        STDERR.puts "CreateNamespace has been started!"
        # lets sleep to simulate that we do something.
        sleep(2.0)
        STDERR.puts "CreateNamespace has been finished!"
    end

    CreateDeployment = lambda do |args|
        STDERR.puts "CreateDeployment has been started!"
        # lets sleep to simulate that we do something.
        sleep(2.0)
        STDERR.puts "CreateDeployment has been finished!"
    end

    CreateService = lambda do |args|
        STDERR.puts "CreateService has been started!"
        # lets sleep to simulate that we do something.
        sleep(2.0)
        STDERR.puts "CreateService has been finished!"
    end

    CreateIngress = lambda do |args|
        STDERR.puts "CreateIngress has been started!"
        # lets sleep to simulate that we do something.
        sleep(2.0)
        STDERR.puts "CreateIngress has been finished!"
    end

    Cleanup = lambda do |args|
        STDERR.puts "Cleanup has been started!"
        # lets sleep to simulate that we do something.
        sleep(2.0)
        STDERR.puts "Cleanup has been finished!"
    end

    def self.main
        createuser = Interface::Job.new(title: "Create DB User", 
                                        desc: "Create DB User with least privileged permissions.",
                                        handler: CreateUser)

        migratedb = Interface::Job.new(title: "DB Migration", 
                                       handler: MigrateDB,
                                       desc: "Imports newest test data dump and migrates to newest version.",
                                       dependson: ["Create DB User"])

        createnamespace = Interface::Job.new(title: "Create K8S Namespace",
                                             handler: CreateNamespace,
                                             desc: "Create a new Kubernetes namespace for the new test environment.",
                                             dependson: ["DB Migration"])

        createdeployment = Interface::Job.new(title: "Create K8S Deployment",
                                              handler: CreateDeployment,
                                              desc: "Create a new Kubernetes deployment for the new test environment.",
                                              dependson: ["Create K8S Namespace"])

        createservice = Interface::Job.new(title: "Create K8S Service",
                                           handler: CreateService,
                                           desc: "Create a new Kubernetes service for the new test environment.",
                                           dependson: ["Create K8S Namespace"])

        createingress = Interface::Job.new(title: "Create K8S Ingress",
                                           handler: CreateIngress,
                                           desc: "Create a new Kubernetes ingress for the new test environment.",
                                           dependson: ["Create K8S Namespace"])

        cleanup = Interface::Job.new(title: "Clean up",
                                     handler: Cleanup,
                                     desc: "Removes all temporary files.",
                                     dependson: ["Create K8S Deployment", "Create K8S Service", "Create K8S Ingress"])

        begin
            RubySDK.Serve([createuser, migratedb, createnamespace, createdeployment, createservice, createingress, cleanup])
        rescue => e
            puts "Error occured: #{e}"
            exit(false)
        end
    end
end
```
<br/>


#### Pipeline parameters example

You can fork this example <a href="https://github.com/gaia-pipeline/ruby-example/tree/parameters_example" target="_blank">here.</a>

```ruby
require 'rubysdk'

class Main

    PrintParam = lambda do |args|
        args.each do |arg|
            STDERR.puts "Key: #{arg.key}; Value: #{arg.value}"
        end
    end

    def self.main
        args = []
        args.push Interface::Argument.new(desc: "Username for the database schema:",
                                          # textfield displays a text field in the UI.
                                          # You can also use "TextAreaInput", "BooleanInput",
                                          # or "VaultInput".
                                          type: Interface::TextFieldInput,
                                          key: "username")
        args.push Interface::Argument.new(type: Interface::TextAreaInput,
                                          key: "usernamedesc")
        printparam = Interface::Job.new(title: "Print Parameters",
                                        handler: PrintParam,
                                        desc: "This job prints out all given params.",
                                        args: args)

        begin
            RubySDK.Serve([printparam])
        rescue => e
            puts "Error occured: #{e}"
            exit(false)
        end
    end
end
```
<br/>


#### Vault parameter example

You can fork this example <a href="https://github.com/gaia-pipeline/ruby-example/tree/vault_example" target="_blank">here.</a>

{{% notice tip %}}
You must add a new vault entry with the key `dbpassword` before you can start the pipeline below. An error will be thrown if a pipeline requests a vault parameter which does not exist.
{{% /notice %}}

```ruby
require 'rubysdk'

class Main

    PrintParam = lambda do |args|
        args.each do |arg|
            STDERR.puts "Key: #{arg.key}; Value: #{arg.value}"
        end
    end

    def self.main
        args = []
        args.push Interface::Argument.new(key: "dbpassword",
                                          # VaultInput automatically gets a secret from Gaia's vault.
                                          # You can also use "TextAreaInput", "BooleanInput",
                                          # or "VaultInput".
                                          type: Interface::VaultInput)
        printparam = Interface::Job.new(title: "Print Parameters",
                                        handler: PrintParam,
                                        desc: "This job prints out all given params.",
                                        args: args)

        begin
            RubySDK.Serve([printparam])
        rescue => e
            puts "Error occured: #{e}"
            exit(false)
        end
    end
end
```
