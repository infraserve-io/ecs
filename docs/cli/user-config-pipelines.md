# ECS CLI



## Output Config in Pipelines

Config for a specific path is compiled on the server according to a specified merge stratagy and returned in a suitable format with filtering applied. Some key things to remember:

- Pipelines config is typically best authorized with a service user

- Grant the user access to the paths for which it will need access

- Decide if the pipeline user needs access to decrpyt secrets in the paths it has access to and configure it appropriately

### Output specific path only

`cto ecs config build --path <path>`

### Output specific path only with a specific config ID

Config IDs are versions of config, either tags that are added via the CLI or commit hashes that are output on each build. 

`cto ecs config build --path <path> --config_id 1.0.1`

### Output specific path only with a specific config ID and check if config has changed

If someone has edited common config, or some other config that is being used by your build strategy, `--drift-detect` will build both configs and if they are different, it will fail your pipeline so you can either run it with upgraded the config after you have tested the new version.

`cto ecs config build --path <path> --config_id 1.0.1 --detect-drift`

### Output specific path only with YAML output instead of JSON

`cto ecs config build --path <path> --format yaml`

### Output specific path with secret decryption

`cto ecs config build --path <path> --show-secrets`

### Output specific path and all config underneith

`cto ecs config build --path <path> --recursive`

### Output specific path with a specific strategy

`cto ecs config build --path <path> --strategy-name <strategy name> --show-secrets`

### Output specific path with a specific strategy and filtering

`cto ecs config build --path <path> --strategy-name <strategy name> --filter '"some-field"."some-other-deeper-nested-field"'`

Note, filters follow JMSPath syntax. Read about JMSPath and try out filtering in the playground [here](https://jmespath.org/tutorial.html).


## Where Next?

Learn all ECS CLI commands [here](./full-reference.md)