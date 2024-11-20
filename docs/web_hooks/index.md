# Webhooks

## Overview

When a config change is made, it is likely that a pipeline needs to be triggered to reflect the change. ECS webhooks make this possible in much the same way as Github webhooks. 

ECS currently has support for 5 webhook types:

- **Github actions**
    - Runs any workflow with configurable parameters
- **Bitbucket runners**
    - Runs any pipeline with configurable parameters
- **Gitlab**
    - Runs any workflow with configurable parameters
- **Jenkins**
    - Triggers any Jenkins job with configurable parameters
- **Custom**
    - Trigger any pipeline via API call. Specify your own endpoint and payload


## Configuring Webhooks

Webhooks are configured for a config path by adding a __webhooks.yaml file to that path and specifying glob patters of files that if changed, will trigger the webhook. When the webhook runs, it should call `cto ecs config build` with the appropriate parameters to output the configuration to the pipeline.

Any number and conbination of webhook types can be configured per path.

On failure of a webhook, all listed email addresses under `notifications` are notified.

Examples of the 5 types of webhooks are provided below:

### Github

An example Github webhook configuration is shown below:

__webhooks.yaml
```yaml
webhooks:
  test_webhook_github:
    notification_emails: 
      - <notification email address>
    file_pattern: "*.yaml"
    webhook_type: github
    repo_name: webservices
    org_name: Cloud-Technology-Office
    workflow_file: development.yaml
    ref_name: main
    token: $token
    inputs:
      env: development
    variables:
      token:
        file_path:  webhook_tokens.yaml
        key: github_token
```

In this example, the `development.yaml` workflow is triggered in repo `webservices` branch `main`. The workflow receives all input variables under `inputs` with the webhook being triggered using an encrypted authentication token provided in `webhook_tokens.yaml`. An example of the token file is shown below:

webhook_tokens.yaml
```yaml
github_token_ecs_secret: <your Github token>
```

The tokens file can have any name you require, simple reference it in the `__webhooks.yaml` file. When this file is pushed to ECS using `cto ecs config push` it is encrypted. See [secret management](../secret_management) for more details on encrypting secrets in ECS.


### Gitlab

An example Gitlab webhook configuration is shown below:

__webhooks.yaml
```yaml
webhooks:
  test_webhook_gitlab:
    notification_emails: 
      - <notification email address>
    webhook_type: gitlab
    file_pattern: "*.yaml"
    project_id: '1234567'
    token: $token
    ref_name: main
    gitlab_vars:
      my_var: test
      my_var2: test
      my_var3: test
    token: $token
    variables:
      token:
        file_path:  token.yaml
        key: gitlab_token
```

In this example, the job is triggered in project `1234567` branch `main`. The workflow receives all input variables under `gitlab_vars` with the webhook being triggered using an encrypted authentication token provided in `webhook_tokens.yaml`. See example webhook_tokens.yaml in the Github example.

### Bitbucket

An example Bitbucket webhook configuration is shown below:

__webhooks.yaml
```yaml
webhooks:
  test_webhook_bitbucket:
    notification_emails: 
        - <notification email address>
    webhook_type: bitbucket
    file_pattern: "*.yaml"
    token: $token
    custom_pipeline_name: default
    workspace: test-workspace
    repo_slug: webservices
    ref_name: master
    bitbucket_vars:
      - key: "var1key"
        value: "var1value"
        secured: true    
    variables:
      token:
        file_path:  token.yaml
        key: gitlab_token
```

In this example, the job is triggered in workspace `test-workspace`, repo `webservices` branch `master`. The workflow receives all input variables under `bitbucket_vars` with the webhook being triggered using an encrypted authentication token provided in `webhook_tokens.yaml`. See example webhook_tokens.yaml in the Github example.

### Jenkins

To use Jenkins webhooks, install the Jenkins plugin <a href="https://plugins.jenkins.io/generic-webhook-trigger/" target="__new" >Generic webhook trigger</a>

An example Jenkins webhook configuration is shown below:

__webhooks.yaml
```yaml
webhooks:
  test_webhook_jenkins:
    token: $token
    notification_emails: 
        - <notification email address>
    file_pattern: "*.yaml"
    url: <Jenkins server URL>
    webhook_type: jenkins
    token: $jenkins_token
    params:
      env: development
      other_param: anything
    variables:
      token:
        file_path:  webhook_tokens.yaml
        key: jenkins_token
```

In this example, the Jenkins job referenced by the token is triggered. The job receives all parameters under `params` with the webhook being triggered using an encrypted job token provided in `webhook_tokens.yaml`. See example webhook_tokens.yaml in the Github example.


### Custom

Using custom webhooks, any API can be called:

__webhooks.yaml
```yaml
webhooks:
  test_webhook_custom:
    token: $token
    notification_emails: 
      - <notification email address>
    webhook_type: custom
    file_pattern: "*.yaml"
    url: <your endpoint URL>
    method: POST
    headers:
      Authorization: Bearer $token
      Content-Type: "application/json"
    body:
      var1: value1
      var2: value2
      var3: value3
    variables:
      token:
        file_path:  webhook_tokens.yaml
        key: custom_webhook_token
```

In this example, the API referenced in `url` is called using the method specified in `method`. The API receives payload specified under `payload` with the API token being provided by an encrypted password token provided in `webhook_tokens.yaml`. See example webhook_tokens.yaml in the Github example.
