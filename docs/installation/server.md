# ECS On-Premise

You have a choice of using ECS Cloud or ECS on-premise. If you choose to run ECS on-premise you're in the right place. Follow the steps below to get started. If you want to try ECS Cloud, click [here](./cloud.md)

By running ECS on your personal machine it is possible to get up to speed on how it works and to start small by creating configuration for a new pipeline you are creating, retrofit into an existing pipeline or you can build the beginnings of your enterprise configuration.

When you're ready to go into production, it's easy to deploy to Kubernates, AWS Fargate etc. Just follow the docker-compose settings.

## Let's get started! 

1) Clone the <a href="https://github.com/Cloud-Technology-Office/ecs-getting-started" target="_new"> getting started repo</a>

2) Create a repository in your Git server of choice, leave it empty

3) Create a Git token that has write access to the repository

4) Install the ECS CLI by doing ```pip install cto-cli```

5) Create a key for SOPS to use for secret encryption:

- **AWS**

    - Create an AWS KMS key and note it's ARN.
    - Get AWS CLI creds that have access to the key typically via an AWS profile or via SSO CLI credentials. 

- **Azure**

    - Create an Microsoft Vault key
    - Create a Service Principle that has role "Key Vault Crypto User" granted for the key
    - Take a note of the following Service Principle variables
        - appId
        - password
        - tenant

- **GCP**

    - Create a GCP Vault and key or add a key to an existing vault
    - Create a service account for ECS that has decrypt access to the key
    - Note the service account credentials file location for your environment variables as below

6) Obtain an ECS API key, <a target="_new" href="https://account.cloudtechnologyoffice.com/pricing/ecs">here</a>. Note, only paid plans support on-premise. If you are unable to trial using ECS Cloud, contact us and we'll set you up a dedicated license.

7) Edit docker-compose.yaml environment variable values as below:

Note, ECS supports multiple configuration repositories. Each repository needs a name, and a user of that repository configures their CLI client to communicate with that repository. In the example below, the repository has been named REPO1, choose any name (upper case alphanumeric only).

It is possible to have encryption keys in different clouds for each repository, or all repositories can use the same key.

```yaml
version: '3.7'

services:
  api:
    image: cloudtechnologyoffice/ecs
    ports:
      - '8000:8000'
    environment:
      ACCEPT_EULA: Y

      ECS_API_KEY: "<The API key you have been issued when you signed up for ECS>"

      REPOSITORY_REPO1_URL: "<Empty configuration repository URL>"
      REPOSITORY_REPO1_PROVIDER: "<github|bitbucket|gitlab|azure>"
      REPOSITORY_REPO1_ACCESS_TOKEN: "<Token with read/write access an empty Git repo that will be used for config storage>"

      # ENCRYPTION KEY SPECIFICATION AND CREDENTIALS

      # If using AWS KMS, set the following variables

      # REPOSITORY_REPO1_SOPS_KMS_ARN: "<AWS KMS key ARN>"

      # AWS_ACCESS_KEY_ID: "<AWS_ACCESS_KEY_ID>"
      # AWS_SECRET_ACCESS_KEY: "<AWS_SECRET_ACCESS_KEY>"
      # AWS_SESSION_TOKEN: "<AWS_SESSION_TOKEN>"
      # or 
      # if container running in AWS with role that can access KMS key, no vars required


      # If using Azure Vault, set the follow variables

      # REPOSITORY_REPO1_SOPS_AZURE_KV: <Key URL. i.e. https://<Vault name>.vault.azure.net/keys/<key name>/<string>

      # AZURE_TENANT_ID: <tenant ID for the Service Principle>
      # AZURE_CLIENT_ID: <appId for the Service Principle>
      # AZURE_CLIENT_SECRET: <password for the Service Principle>


      # If you are using GCP Vault, set the following variable

      # REPOSITORY_REPO1_SOPS_GCP_KMS: projects/<project>/locations/global/keyRings/<keyring>/cryptoKeys/<key name>

      # GOOGLE_APPLICATION_CREDENTIALS: "${VOLUME_MAPPED_DIR}/<MyGoogleServiceAccountJsonFile.json>"
      # or 
      # GOOGLE_CREDENTIALS: "<String encoded JSON Google credentials file"
```

AWS Note: For local use with docker compose, if you have an AWS profile configured, you do not need to set the AWS_ values above in the compose file.

Azure Note: For local use, if you are logged in via `az login`, you do not need to set the AZURE_ values above in compose file.

GCP Note: For local use, if you are logged in via `gloud auth login`, you do not need to set the GOOGLE_ value above in compose file.

8) Uncommment your cloud providers SOPS settings in `docker-compose.yaml`.

9) Use ``docker-compose up`` to start ECS server.

That's ECS setup and ready to use. It already supports secret encryption as you specified the KMS key to encrypt so any secrets you want to store, go right ahead. See Secret storage for more information on how to designate a key as a secret.

## Multiple Configuration Repository Support

ECS supports an unlimited number of indpendent configuration repositories. Due to ECS RBAC, even large organizations can organize the stucture of a single repository to fulfull the entire organizations needs, but where there is a need for greater segregation between org units or teams, ECS fully supports this.

An end user uses the ECS CLI to interact with a single repository. The repository name is configured when the CLI is initialized.

To configure ECS server with multiple repositories, multiple sets of environment variables are defined in docker compose environment section. In the example below, we have 3 repos, the first is called REPO1, second is REPO2 and third is REPO3. REPO1 uses an encrption key in AWS, REPO2 uses Azure and REPO3 uses GCP. 

Typically your repos would have meaningful names such as TEAM-DEVOPS or in the case of geographic separation, perhaps UK or USA.

```
REPOSITORY_REPO1_URL="<Empty configuration repository URL>"
REPOSITORY_REPO1_PROVIDER="<github|bitbucket|gitlab|azure>"
REPOSITORY_REPO1_ACCESS_TOKEN="<Token with read/write access an empty Git repo that will be used for config storage>"
REPOSITORY_REPO1_SOPS_KMS_ARN="<AWS KMS key ARN>"

REPOSITORY_REPO2_URL="<Empty configuration repository URL>"
REPOSITORY_REPO2_PROVIDER="<github|bitbucket|gitlab|azure>"
REPOSITORY_REPO2_ACCESS_TOKEN="<Token with read/write access an empty Git repo that will be used for config storage>"
REPOSITORY_REPO2_SOPS_AZURE_KV="<Key URL. i.e. https://<Vault name>.vault.azure.net/keys/<key name>/<string>"

REPOSITORY_REPO3_URL="<Empty configuration repository URL>"
REPOSITORY_REPO3_PROVIDER="<github|bitbucket|gitlab|azure>"
REPOSITORY_REPO3_ACCESS_TOKEN="<Token with read/write access an empty Git repo that will be used for config storage>"
REPOSITORY_REPO3_SOPS_GCP_KMS=projects/<project>/locations/global/keyRings/<keyring>/cryptoKeys/<key name>
```

## Where Next?

Install and setup the CTO CLI to interact with the ECS server by following instructions [here](./cli.md).