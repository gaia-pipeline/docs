---
title: "3 - Conclusion"
description: "Conclusion"
weight: 40
---

## Conclusion

Let's recapitulate what steps we've taken:

* We setup a local test environment with Kubernetes, HashiCorp Vault and Gaia.
* We saved sensitive data (Kube-Config) in our HashiCorp Vault instance.
* We build a powerful pipeline in Go and wrote tests for it.
* We executed the tests.
* We created a new pipeline in Gaia. Gaia automatically compiled our pipeline.
* We added secrets to Gaia's Vault and executed the pipeline.

With just a few steps we created a powerful pipeline which is secure, tested, includes no hidden magic (at the end it's just Go code), fast, and the most important part: It was fun!

