---
title: "2 - Store secrets in the vault"
description: "Storing secrets in the vault"
weight: 30
---

#### Use the vault to store secret data

Gaia provides a Vault which can be used to store secret values in a secure way. Details on how the vault works
are located here: [Vault Details](https://github.com/gaia-pipeline/gaia/blob/master/security/README.md).

The vault can be found on the left side menu here:

![accessing-the-vault](/images/vault-button.png?width=450px)

If you click on it, you should be redirected to the vault manager view.

![vault](/images/vault.png?width=450px)

Here is where you can manage your secrets. Create, Delete or Edit them as you like. You will, however, never
be able to view them. That is not supported for security reasons. Once a vault secret is created it can only
be updated to something else.

Secrets are always overwritten if one already exists.

To create a secret follow these steps:

![create-secret](/images/create-secret.png?width=450px)

If the operation was successful a green pop-up will indicate it in the top right corner.

![success-popup](/images/create-secret-success.png?width=450px)
![listing-secrets](/images/list-secret.png?width=450px)

Failure is indicated by a red pop-up.

![failed-popup](/images/create-secret-failed.png?width=450px)

To Delete a secret click on the trashcan icon:

![delete-secret](/images/delete-secret.png?width=450px)
![confirm-delete-secret](/images/confirm-delete-secret.png?width=450px)
![delete-secret-success](/images/delete-secret-success.png?width=450px)

To Edit a secret click on the notepad icon:

![edit-secret](/images/edit-secret.png?width=450px)
