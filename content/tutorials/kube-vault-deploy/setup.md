---
title: "1 - Setup"
description: "Setup of the environment"
weight: 10
---

## Setup

The first step is to download and start HashiCorp Vault and Gaia. For this tutorial we will use Docker but you can also install them manually if you prefer that. Run the following commands in your terminal:

```
docker network create gaia-vault
```

```
docker run --cap-add=IPC_LOCK -d \
    -e 'VAULT_DEV_ROOT_TOKEN_ID=root-token' \
    -e 'VAULT_ADDR=http://localhost:8200' \
    -e 'VAULT_TOKEN=root-token' \
    -p 8200:8200 --name=vault --net=gaia-vault vault:latest
```

```
docker run -d -p 8080:8080 --net=gaia-vault --name=gaia gaiapipeline/gaia:latest
```

This will create a network called `gaia-vault` which is used to allow communication between Gaia and HashiCorp Vault. Then we start HashiCorp Vault with a development token (Don't do this in production!). We expose the service on port 8200 which is optional and can be omitted if prefered. The last command starts Gaia and exposes it on port 8080. For this tutorial we don't mount the data directory to the host system. That means if you restart the Gaia Docker container all your data is lost. If you want to persist your data you can mount the data folder via the following parameter: `-v $PWD:/data`.
<br /><br />

## Store Kube-Config into Vault

The Kube-Config is particularly important for the connection to the Kubernetes API. It tells the Kube-Client (kubectl in general) where the API is located and a certificate for authentication and authorization purpose.
If you use the local Kubernetes cluster from Docker for Mac or Docker for Windows the Kube-Config should be already generated and placed on your file system (usually ~/.kube/config).
Our Gaia pipeline should have later access to the Kubernetes API and therefore needs access to this Kube-Config file.
The perfect place for this sensitive file is HashiCorp Vault where we will save it now.

Let's assume that HashiCorp Vault is not locally installed. If you have HashiCorp Vault locally installed and did set VAULT_ADDR and VAULT_TOKEN correctly you can skip the next two steps. 

Copy your Kube-Config into the Vault container:

```
docker cp ~/.kube/config vault:/tmp/config
```

Now we have to get into the container to have access to the vault client:

```
docker exec -it vault sh
```

We are now in the container and have access to the vault client. We already set VAULT_ADDR and VAULT_TOKEN during the startup so we can now directly use the vault client to store our Kube-Config:

```
vault kv put secret/kube-conf conf="$(cat /tmp/config | base64)"
```

We encoded our config in Base64 so there are no problems with special characters. 
Let's continue with the next chapter: [{{%icon circle-arrow-right%}}2 - Create Pipeline]({{%relref "tutorials/kube-vault-deploy/create-pipeline.md"%}})
