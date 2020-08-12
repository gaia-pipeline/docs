---
title: "API"
description: "Description of the API and how to use Gaia without a frontend"
weight: 25
alwaysopen: false
---

## API

The frontend of Gaia is actually just a user of Gaia's API. The API can be used with anything
that can talk JSON and HTTP. A terraform client, a CLI tool or simply, curl. This looks like the following:

First, you have to obtain a JWT token to use most of the API endpoints. To do this, log-in with like this:

```bash
# Curl call
curl -H "content-type: application/json" -X POST -d '{"username":"admin", "password":"admin"}' http://localhost:8080/api/v1/login

# Response from Gaia
{"username":"admin","display_name":"admin","tokenstring":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZXMiOlsiUGlwZWxpbmVDcmVhdGUiLCJQaXBlbGluZUxpc3QiLCJQaXBlbGluZUdldCIsIlBpcGVsaW5lVXBkYXRlIiwiUGlwZWxpbmVEZWxldGUiLCJQaXBlbGluZVN0YXJ0IiwiUGlwZWxpbmVSdW5TdG9wIiwiUGlwZWxpbmVSdW5HZXQiLCJQaXBlbGluZVJ1bkxpc3QiLCJQaXBlbGluZVJ1bkxvZ3MiLCJTZWNyZXRMaXN0IiwiU2VjcmV0RGVsZXRlIiwiU2VjcmV0Q3JlYXRlIiwiU2VjcmV0VXBkYXRlIiwiVXNlckNyZWF0ZSIsIlVzZXJMaXN0IiwiVXNlckNoYW5nZVBhc3N3b3JkIiwiVXNlckRlbGV0ZSIsIlVzZXJQZXJtaXNzaW9uR2V0IiwiVXNlclBlcm1pc3Npb25VcGRhdGUiLCJXb3JrZXJHZXRSZWdpc3RyYXRpb25TZWNyZXQiLCJXb3JrZXJHZXRPdmVydmlldyIsIldvcmtlckdldFdvcmtlciIsIldvcmtlckRlcmVnaXN0ZXJXb3JrZXIiLCJXb3JrZXJSZXNldFdvcmtlclJlZ2lzdGVyU2VjcmV0Il0sImV4cCI6MTU5NzMwMjY4OCwiaWF0IjoxNTk3MjU5NDg4LCJzdWIiOiJHYWlhIFNlc3Npb24gVG9rZW4ifQ.D0r3cUfp6hIw3fgDqvk52tkQSzOal84aUqO5gQauRHs","jwtexpiry":1597302688,"lastlogin":"2020-08-12T21:11:28.243895+02:00"}
```

For any further calls include this token as an Auth header:

```bash
# Curl call
curl -H "content-type: application/json" -H 'Authorization: bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwicm9sZXMiOlsiUGlwZWxpbmVDcmVhdGUiLCJQaXBlbGluZUxpc3QiLCJQaXBlbGluZUdldCIsIlBpcGVsaW5lVXBkYXRlIiwiUGlwZWxpbmVEZWxldGUiLCJQaXBlbGluZVN0YXJ0IiwiUGlwZWxpbmVSdW5TdG9wIiwiUGlwZWxpbmVSdW5HZXQiLCJQaXBlbGluZVJ1bkxpc3QiLCJQaXBlbGluZVJ1bkxvZ3MiLCJTZWNyZXRMaXN0IiwiU2VjcmV0RGVsZXRlIiwiU2VjcmV0Q3JlYXRlIiwiU2VjcmV0VXBkYXRlIiwiVXNlckNyZWF0ZSIsIlVzZXJMaXN0IiwiVXNlckNoYW5nZVBhc3N3b3JkIiwiVXNlckRlbGV0ZSIsIlVzZXJQZXJtaXNzaW9uR2V0IiwiVXNlclBlcm1pc3Npb25VcGRhdGUiLCJXb3JrZXJHZXRSZWdpc3RyYXRpb25TZWNyZXQiLCJXb3JrZXJHZXRPdmVydmlldyIsIldvcmtlckdldFdvcmtlciIsIldvcmtlckRlcmVnaXN0ZXJXb3JrZXIiLCJXb3JrZXJSZXNldFdvcmtlclJlZ2lzdGVyU2VjcmV0Il0sImV4cCI6MTU5NzMwMjY4OCwiaWF0IjoxNTk3MjU5NDg4LCJzdWIiOiJHYWlhIFNlc3Npb24gVG9rZW4ifQ.D0r3cUfp6hIw3fgDqvk52tkQSzOal84aUqO5gQauRHs' -X GET http://localhost:8080/api/v1/pipeline

# Response from Gaia
[{"id":16,"name":"NodeTest","repo":{"url":"https://github.com/Skarlso/nodejs-example","privatekey":{},"selectedbranch":"refs/heads/master"},"type":"nodejs","sha256sum":"ho7cWXEZR1M40fmau4B8bvVRMcfuTJwLhB858M+I7Ic=","jobs":[{"id":3969984711,"title":"DoSomethingAwesome","desc":"This job does something awesome.","status":"waiting for execution"}],"created":"2020-08-07T13:28:54.962771+02:00","uuid":"5b496893-e94b-4fa4-84ee-4ddba739ee2b","trigger_token":"aff2cbe4-6a25-5a51-a0ce-ba3a62fdec80","tags":["nodejs"],"docker":false},{"id":17,"name":"NodeWebhook","repo":{"url":"https://github.com/Skarlso/nodejs-example","privatekey":{},"selectedbranch":"refs/heads/master"},....
```

To use it in a script, you can do something like this:

```bash
AUTH=$(curl -s -H "content-type: application/json" -X POST -d '{"username":"admin", "password":"admin"}' http://localhost:8080/api/v1/login | jq -r '.tokenstring')
curl -H "content-type: application/json" -H "Authorization: bearer ${AUTH}" -X GET http://localhost:8080/api/v1/pipeline
```

## Swagger

In order to load the [Swagger](https://swagger.io/) docs of the API backend, start your Gaia
instance, or navigate to the deployed Gaia service of your choice and access the swagger url.
On a local instance that's `http://localhost:8080/api/v1/swagger/index.html`.

### Coming soon

There will also be a Gaia hosted version of the swagger api docs here.
