---
title: "3 - Conclusion"
description: "Conclusion"
weight: 40
---

## Conclusion

We developed in a few minutes a powerful pipeline which can be easily used for continuous deployment. You can change the version in vault and simply restart your pipeline. This will update the deployment object and Kubernetes automatically rotates all running pods to the new version. You can even add additional secrets to vault and integrate them into your pipeline. This makes it really powerful.

After all, we should also talk about aspects which are currently (Gaia is alpha!) not perfect/awful and how we can fix them. First of all, it's a security issue to store the Vault-Token hard-coded in your pipeline source code. This problem will be solved soon via Pipeline Run Parameters. This allows you to pass in parameters before you start your pipeline. Additionally, it is planned to have a local secret store in Gaia where you can store such information safely and access them in your pipelines.

Due to a bug in Gaia, it is currently not possible to share data between two pipeline jobs in one run. We worked around this issue by saving the data to local space. This bug will be fixed soon too.
