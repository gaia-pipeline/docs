---
title: "0 - Setup"
description: "Setup of the environment"
weight: 10
---

## Setup

Let's start with the first step which is the setup of Vault and Gaia. Run the following commands in your terminal:

```
docker network create gaia-rocks
```

```
docker run --cap-add=IPC_LOCK -d \
    -e 'VAULT_DEV_ROOT_TOKEN_ID=root-token' \
    -e 'VAULT_ADDR=http://localhost:8200' \
    -e 'VAULT_TOKEN=root-token' \
    -p 8200:8200 --name=vault --net=gaia-rocks vault:latest
```

```
docker run -d -p 8080:8080 --net=gaia-rocks --name=gaia gaiapipeline/gaia:latest
```

This will first create a network. Then we start vault which joins this network with a development token (keep in mind this is a test environment. Don't do this in production!) and expose it on port 8200. The last command starts Gaia on port 8080 and also joins the network. Gaia is now able to send requests to Vault.
<br /><br />

## Store Kube-Config and more information into Vault

We will use Vault to safely store all our deployment information. For now, it will be only the Kube-Config and the version information of the Docker Image but you could actually store everything you want into Vault! 

I assume that vault is not locally installed. If you have vault locally installed and did set VAULT_ADDR and VAULT_TOKEN correctly you can skip the next two steps. 

Copy your Kube-Config into the Vault container:

```
docker cp ~/.kube/config vault:/tmp/config
```

Now we have to get into the container to have access to the vault client:

```
docker exec -it vault sh
```

We are now in the container and have access to the vault client. We already set VAULT_ADDR and VAULT_TOKEN during the startup so we can now directly use the vault client to store our kube-config:

```
vault kv put secret/kube-conf conf="$(cat /tmp/config | base64)"
```

We encoded our config in Base64 so there are no problems with special characters. Let us now add an additional secret for the docker image version information:

```
vault kv put secret/nginx version="1.14.0"
```

That's it!
Let's continue with the next chapter: [{{%icon circle-arrow-right%}}1 - Create Pipeline]({{%relref "tutorials/kube-vault-deploy/create-pipeline.md"%}})