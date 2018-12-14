---
title: "Kubernetes" 
description: "Kubernetes deployment with HashiCorp Vault integration"
weight: 0
---

## Kubernetes deployment with HashiCorp Vault integration

<a href="https://kubernetes.io/" target="_blank">Kubernetes</a>, the well known open source container management system, which has been widely adopted in the last years and has proven to be battle-hardened, is the de facto standard for deploying containerized applications.
Thousands of companies are already using it in production and this is only the beginning.
Besides solving a lot of problems, it also brings new ones on the board which are sometimes tedious. 
Nowadays, when complex microservice architectures with hundreds of services are deployed with Kubernetes, the configuration management of deployment files becomes a nightmare. 
The amount of different YAML-Files which need to be maintained are tremendous and often has it's own dangers. 

Gaia is the perfect tool to automate Kubernetes deployments in a reproducible way, coordinate maintenance tasks for your cluster, cleanup of dangling resources in your cluster and much more.
In this guide we will setup a small local test environment including Kubernetes and <a href="https://www.vaultproject.io/" target="_blank">HashiCorp Vault</a> which we will use to store safely our secrets. We will create a powerful pipeline in Go which deploys an application to this Kubernetes cluster (with namespace and service).

We will also write tests for our pipeline which we can execute locally to test our jobs and make sure they are actually working. Let us start with the requirements and the setup of the test environment!
<br /><br />

### Requirements

* Docker for Mac or Docker for Windows for a local test Kubernetes cluster. 
* (Optional) Golang 1.10.3 or later (to run tests locally).
<br /><br />

#### Setup test environment: Docker with Kubernetes support

This is probably the easiest way to get a local Kubernetes cluster running. Open the Docker preferences window to activate Kubernetes:

![activate-kubernetes-docker](/images/activate-kubernetes-docker.png?width=450px)

After some setup time, your local Kubernetes cluster should be available and your `kubectl` should be already configured. You can verify this by typing `kubectl cluster-info` which should print something like this:

```
Kubernetes master is running at https://localhost:6443
KubeDNS is running at https://localhost:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

That's it for now. Continue with the next chapter: [{{%icon circle-arrow-right%}}0 - Setup]({{%relref "tutorials/kube-vault-deploy/setup.md"%}})
<br /><br />

#### (Optional): External Kubernetes Cluster

If you don't want to start a local Kubernetes cluster you can also use an external cluster. The API-Server must be reachable from your computer and you need to have a valid Kube-Config. Additionally, we assume that all required certificates are contained in the Kube-Config file. If this is not the case, please copy the certificates plain into the Kube-Conf file. Example:

```
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://localhost:6443
  name: docker-for-desktop-cluster
contexts:
- context:
    cluster: ""
    namespace: default
    user: ""
  name: default-context
- context:
    cluster: docker-for-desktop-cluster
    user: docker-for-desktop
  name: docker-for-desktop
current-context: docker-for-desktop
kind: Config
preferences: {}
users:
- name: docker-for-desktop
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM5RENDQWR5Z0F3SUJBZ0lJSXEwTThJL1Y1cEV3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB4T0RBME1UZ3hNVFEwTlRWYUZ3MHhPVEEzTVRNeU1ERTNNRGxhTURZeApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sc3dHUVlEVlFRREV4SmtiMk5yWlhJdFptOXlMV1JsCmMydDBiM0F3Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRREduaHFTandBK05nbU0KemI0WlJRemRyUEpUdTRHbWkwalIxenVXQjRlSDZJN1FXb3VmalJrTGNXdm54NXNuSisva0E0SVRrclNjZG5hNApVeTJOTDEzVlJWamViTU5CNUl2TTA2VjBXUDVzMXFiT1pyaUhhT1gxeERTMDZuTXA2dmJIZlBsdDV3L0NKVnFTCnVLdGNoTG41ZWFWS1dFdWdXU3VlS2o1UE9UdmJZSDVKUU9RUzFpUUttcmxKQndKb0Z3NmlaRHJYS1lZK2pneUwKamlrdTdGMW1sNkRLM2tYNXB0MFErbm0xUkt4a21UTkppS29FUDZCUDRLajBILzQwUU9hRGVpdTJaUW5UaUxqawpjN25jQmovQ3k5L0VTRFV2RDhyMWJmSUtCUW12SjM0TllWZXdnNzEybjk3ZW9qMVUrVmRQWWVzVlNJdno4MkNaCittekkyeXdMQWdNQkFBR2pKekFsTUE0R0ExVWREd0VCL3dRRUF3SUZvREFUQmdOVkhTVUVEREFLQmdnckJnRUYKQlFjREFqQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFWdHAybFlibFIyekg5VVhOWWVhZ0p5ZTdwWm0rRWhaaApzckYxNlhySVRlSE9ubFQ0cjB2dXh1eVVaY3cyaFFRRnJpT0lZSTZqcHhCWitJa1BKMXUvZTJCY1pWMHR2RUhhCmZSQjVjbkhlY3ZqNnFVd1c3M29FVENwRG4xdEJPa3pRNTBmNVJBdUo3NTk3WVVLZGlSc0lkWGVTS293Zkd3akYKRGQxVEFsZ1luejVBYk5YNUNtaC9yWklzakdROTBvNVVlRDFRMng1bmdDMU9wZTQ5eWhYWVR5NTZiQ3pZVmZETwp5eWFFQnRPTUpONTBsU1pCWlBWRGtvdnRqSmE5clMvUW81Y2ZKeG1iUTJneEVXaWR3RmwrNXZYKzJVeFNTWXJ5CkdJeHZ4ZFFoWEVyMWpPNzFIZFZNeEdmdXNIcEFMQVIrSnpPZm14bFZoQmkyeUlEOFh0TTZoUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBeHA0YWtvOEFQallKak0yK0dVVU0zYXp5VTd1QnBvdEkwZGM3bGdlSGgraU8wRnFMCm40MFpDM0ZyNThlYkp5ZnY1QU9DRTVLMG5IWjJ1Rk10alM5ZDFVVlkzbXpEUWVTTHpOT2xkRmorYk5hbXptYTQKaDJqbDljUTB0T3B6S2VyMngzejViZWNQd2lWYWtyaXJYSVM1K1htbFNsaExvRmtybmlvK1R6azcyMkIrU1VEawpFdFlrQ3BxNVNRY0NhQmNPb21RNjF5bUdQbzRNaTQ0cEx1eGRacGVneXQ1RithYmRFUHA1dFVTc1pKa3pTWWlxCkJEK2dUK0NvOUIvK05FRG1nM29ydG1VSjA0aTQ1SE81M0FZL3dzdmZ4RWcxTHcvSzlXM3lDZ1VKcnlkK0RXRlgKc0lPOWRwL2UzcUk5VlBsWFQySHJGVWlMOC9OZ21mcHN5TnNzQ3dJREFRQUJBb0lCQUU5ZHBpaWlVK3FJRlZEYQpkZmdMQzVVWklzd3F4U2dUeUVseHhER3pXSWtLZU9ieEI1SCtBOS82dHErcnAyZ0NJVzN2cU83QlZNS2c0OWZNCkJRdkJ2YkVYUU9mQWRsWENTY3JUVis0aUVhalVMVnVVMkcvamp1Q2lRcDE0Z2dSaUM3S3pVY2lFNkZzZ0tnMHYKRmVxbWJ0b3RyY3NEZFZUaHpQZ3EwVE0vSDVnTlJsVW1NdU0zVldjUmtSMlRjWTNDN2pvdmdXZmlqMWRNdURrOQpVUWNjczBnNEZPY25UVSt3bE42d2FGR284dDNXUW41bnRyYVI3T21nZVJFT3NaUDd4Vmk2dENsK0RqY3U4RVo1CnAxSGFhZFQ0S1lLOVZZdVZXQ1BiMjZFZjdPMVpCS3A1UWlwRmwydGNlNjhSRnl0bGhHUEltWWVsNnBYenJpeUQKTjhPM2xDRUNnWUVBOEY4OXNTNWxGVWUzUVJwbWN1SGJLekhKQ2RHNVZ6SzRKc25zemc5STJvUjlTMzZGVzF4NQpmdG1JbEtKaFlSVDNlaytoWGpvNTdmYUpUak1zb0pxSkp1bkQ1N0VabFZEOWE2aDh6bm1xYk04aENYMHgwQWw3ClQrRWhTa29jeVpOZ2FsSlhaV2ZQZXNOcUVNOUp2TkRUTlQvQlBmSzNYazdhbkdVZ3hac0RWZFVDZ1lFQTA0Zm0KazJ2S1pqOXdKLzRZcHFrdnNGS2pJaGNjMDMyVkVqd3pHd2dMRzVzdG8zaHB3OHV3b0tobDBuRHFYZEZ3WDIwbgpmWXNqNDIwYy9Qanlac05qSG5UcXAweG1QaVZ1YVNPZjBuRXBUcmNlUGRTSHY2aWZDbjVEK3NyY0w5NW9aRWtUCndUYzZYUXVlMTRicFpFZW0wY1d0SXhCWjNjelhXUWE4U09vMENsOENnWUIyNW9XN3VUbGpOMkJjb2RSL2kxMUEKbHBYZGQ1SjRvYXdaODlSaGNZb1dIV2RsQ3Fhb3RLdWNwYm83Mjc3VHFPMXA0UzN2VUZvTGJlSXBmb0xheHRhRgpHeWsrMkluUkpJalcwamM2WTFCOEZsRS9RbUI3aWRVbmhETlZiaWVqUm5WdzRsNDgyUWIyc09jc2ZYejZHMG4rCmt4VGhzY2dtckZiUytlc21GREdvS1FLQmdFaGNYZ2tpUDR1NHVkSkVmd1JNTGc4Z1JjUDhxaFREQ2dMQjZ5MmQKRThldXp1N3oyeUpxaEpLQTZNd1RhbWtMbzJoUmU4ZmJtRHhOY0RRdHFTWjBRbTBCeTkvTko5Q3NsMWVLSXpzbgpFTjFua1FYUHRWeGYvMy9rYjdiVVBIeDNsYmh3c3p4T2V6Mm5Jd0JSbTNkOWQxaWRTYndMOU9JR0Y4alJvQWxGCmJPWDdBb0dCQUk2a3hCa2hSaEVRWDBBd0ljWFlNdnR0Y1Q1UFAwdHpES1l6Mzc5WVpEbWJkS0RmT0NtdHcyTTcKVlByZ0hFTXlHY3NPZVFxOG9uaEhnTVd6dWhOd05UZnVmTGZ4a24zS1hzdHFXekFWS3JJeVNMUmx4YzRNMHVXRgowTWwzdDc2WVU0VldXSXp1UGZFSTVlbnRRZ1p4a2hkQi8rdlpYVytVajhOcnB5NWZYTUdQCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```

Continue with the next chapter: [{{%icon circle-arrow-right%}}0 - Setup]({{%relref "tutorials/kube-vault-deploy/setup.md"%}})
