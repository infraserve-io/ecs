# ECS CLI

The CLI is used to interact with the ECS server. A list of tasks and sample commands for each is listed below:

## Config Management (all users)

Config is stored in the Git repository with which the server is configured. Users work with either the whole or a subset of config. When a user pulls config, they will receive the latest version of any config that they are authorized to access. 

Commands to work with config are as follows:

### Retrieve latest config

`cto ecs config pull`

This will output the config ID (commit hash) giving you a fixed version you can use in builds

### Save config

`cto ecs config push`

If someone has edited any part of config you will not be able to push until you perform `cto ecs config pull` first. 

In the case where someone else has changed the same file as you, when you push you will be informed there is a merge conflict. Resolve this conflict as you would any Git merge conflict, examine the file(s) referenced, update to reflect correct changes and then do `git commit -am "any message"` and then `cto ecs config push` again. Note, the `git commit` is not ever sent to the server or the central repository, the merge resolution is handled in your local Git repo which is simply a repo with the config paths you are allowed to access.

### Show status 

`cto ecs config status`

This will tell you if you have made any changes that need to be pushed

### Save config and tag

`cto ecs config push --tag <your tag>`

### Decrypt secrets

Only works for users that have been created with the `--read-secrets` flag set

`cto ecs config decrypt --path <path to file`

### Use AI to generate a schema

`cto ecs config generate-schema --write --path config-examples/example-3 --strategy-name merge_common_ec2_to_teams_ec2`

The above command will create a schema file that will be used to validate the putput of the `cto ecs config build` command whenever the same path and strategy are built.

## Where Next?

Learn ECS CLI build commands for pipelines [here](./user-config-pipelines.md)