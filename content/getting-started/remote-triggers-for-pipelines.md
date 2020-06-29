---
title: "4 - Remote triggering a pipeline"
description: "Trigger your pipeline remotely with a single api call."
weight: 50
---

#### Remote triggering a pipeline

Now that you have a pipeline, you would like to be able to trigger the pipeline run,
without actually having to log in, or authenticate with the main Gaia master instance. This is
useful for automated processes, cronjobs, remote calls of any kind which would like to do a
fire and forget.

You have the option to do so using a `remote trigger token`. This token, can be located under the
pipelines details view here:

![pipeline-detail](/images/remote-trigger-token.png?width=450px)

To use the token, simply initiate a curl call like this:

    curl -X POST https://gaia.example.org/api/v1/pipeline/11/6ec8e94f-d63a-5dc3-93f3-8d0d198a15c5/trigger

This will then return a 200 if triggering was successful.