---
title: "Get started with Gaia"
description: "Get started with Gaia"
weight: 10
alwaysopen: true
---

### Introduction

Welcome to the intro guide to Gaia! In the next steps you will learn what Gaia is and how to use it effectively. <br />
Please keep in mind that Gaia is currently as alpha version available and many things will change in the future. <br />
Essential features might be missing and you will probably face critical errors.
<br /><br />

### What is Gaia?

Gaia is an open source automation platform which makes it easy and fun to build powerful pipelines in any programming language. <br /> 
Based on <a href="https://github.com/hashicorp/go-plugin" target="_blank">HashiCorp's go-plugin</a> and <a href="https://grpc.io/" target="_blank">gRPC</a>, gaia is efficient, fast, lightweight and developer friendly. <br />

Develop pipelines with the help of SDKs (currently only Go) and simply check-in your code into a git repository. <br /> 
Gaia automatically clones your code repository, compiles your code to a binary and executes it on-demand. <br />
All results are streamed back and formatted to a user-friendly graphical output.
<br /><br />

### Motivation

*Automation Engineer, DevOps, SRE, Cloud Engineer, Platform Engineer* - they all have one in common: <br />
The majority of tech people are not motivated to take up this work and they are hard to recruit. <br />

One of the main reasons for this is the abstraction and poor execution of many automation tools. <br />
They come with their own configuration (<a href="https://en.wikipedia.org/wiki/YAML" target="_blank">YAML syntax</a>) specification or limit the user to one specific programming language. <br /> 
Testing is nearly impossible because most automation tools lack the ability to mock services and subsystems. <br />
Even tiny things, for example parsing a JSON file, are sometimes really painful because external, <br />
outdated libraries were used and not included in the standard framework.

We believe it's time to remove all these abstractions and come back to our roots. <br />
Are you tired of writing endless lines of YAML-code? Are you sick of spending days forced to write in a language <br />
that does not suit you and is not fun at all? Do you enjoy programming in a language you like? Then gaia is for you.
<br /><br />

### How does it work?

Gaia is based on <a href="https://github.com/hashicorp/go-plugin" target="_blank">HashiCorp's go-plugin</a>. It's a plugin system that uses gRPC to communicate over HTTP2. <br /> 
HashiCorp developed this tool initially for Packer but it's now heavily used by Terraform, Nomad, and Vault too.

Pipelines can be written in any programming language (gRPC support is a prerequisite) and can be compiled locally <br />
or simply over the build system. Gaia clones the git repository and automatically builds the included pipeline.

After a pipeline has been started, all log output from the included jobs are returned back to gaia and displayed <br />
in a detailed overview with their final result status.

Gaia uses for storage boltDB. This makes the installation step super easy. No external database is currently required.