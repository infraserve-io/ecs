# Secret Management

## Overview

It is possible to store secrets in ECS config and it is perfectly safe to do so. Secrets are encrypted using Mozilla SOPS and use an encryption key that resides in either Azure, AWS or GCP, or if you're using ECS Cloud, you have the option of using a key that is generated specifically for your account.

A user has to have `read-secrets` privilege in order to view secrets, and if they have `read-secrets` privilege, they need access to the config path where your secrets reside. Using this mechanism it's possible to divide up your config so only a specific user or team can access secrets in that path.

Typically only pipeline service accounts need access to be able to decrypt secrets.

## Add a Secret to Config

To add a secret to config, all you need to do is specify the field name of the secret with `_ecs_secret` as a suffix. As an example:

```yaml
my-config:
    secret: my-secret
```

will not encrypt your secret

```yaml
my-config:
    secret_ecs_secret: my-secret
```

When you push the config to the server, the fields secret value will have been encrpyted:

```yaml
my-config:
    secret_ecs_secret: ENC[AES256_GCM,data:Ev22+cln/Bn1,iv:QZejNi/1Nb64mtXkU4r4ptoFO1CweB0rhfMP+1EjDRk=,tag:JHaI+jp02adFryYixonBiQ==,type:str]
sops:
    kms:
        - arn: arn:aws:kms:eu-west-1:123456789012:key/1234-1234-1234-1234
          created_at: "2024-02-30T12:51:51Z"
          enc: <encrypted value>
          aws_profile: ""
    gcp_kms: []
    azure_kv: []
    hc_vault: []
    age: []
    lastmodified: "2024-02-30T12:51:51Z"
    mac: <encrypted value>
    pgp: []
    encrypted_suffix: _ecs_secret
    version: 3.8.1
```

## Control Who Can Decrpyt Secrets

Only users with `read-secrets` can decrypt secrets. To create a user with this permission:

`cto ecs users create --username <username> --given-name <name> --family-name <name> --read-secrets`

## Output With Secret Decryption

In your pipelines, you will want to decrypt any secrets using the command below:

`cto ecs config build --path <path> --show-secrets`

## Where Next?

Check out  Drift Detection [here](../drift_detection)