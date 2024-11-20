# Overview of ECS Pkl Support

As a configuration platform, we decided to build in Pkl support so that users have an alternative to our YAML strategies language for building DRY configurations. Pkl also provides this functionality, and is a valid choice for a user of ECS that is already using Pkl for their configurations. 

It is perfectly OK to mix Pkl and YAML and JSON in the same ECS repository, even in the same path, but typically, a Pkl configuration would be stored in its own config path.

The great thing about using Pkl with the ECS platform is that no matter what config language you want to use for a particular project, you always use the same build commands. No need to install Pkl anywhere, the ECS server handles all the compilation and the CLI returns results to std out or file if using the `--out` option. 

Pkl output format options are controlled via the `--format` command line flag. All options Pkl supports are supported through the ECS CLI. Note, only JSON and YAML output formats work with ECS schema validation. 

At the time of writing the current Pkl binary version is `0.25.3`.

## Pkl vs YAML/ JSON Feature Comparisons

There are a few differences in capabilities and techniques to use when using the 2 language options. Those are explored below.

### ECS Build File Paths

When using Pkl, on the `cto config build --path` command you always specify the path to target one or more .pkl files. The most basic example is a single file but ECS also supports passing Pkl glob patterns just as you would if you were calling the Pkl binary directly. Examples are provided below:

#### Single file:

`cto config build --path "species/birds.pkl"`

#### Multiple files in one directory

The following command would produce a multi file YAML output that is piped directly into kubectl.

`cto config build --path "/team-1/guestbook/production/us-west/*.pkl" | kubectl apply -f -`

#### All Pkl files in path

`cto config build --path "/team-1/**/*.pkl"`


By contrast, using ECS own YAML/ JSON merge strategies, it is possible to build a single file or an entire directory structure with no specification of a file or glob by virtue of the ECS strategy language. An example follows:

`cto ecs config build --path /config-examples/example-4 --strategy-name generate_ec2s`

This command has only specified the directory name and all the compilation logic to merge config from multiple directories and files is stored in `__strategies.yaml` strategy `generate_ec2s`. See [build strategies](../../build_strategies) for more information.

In summary, in Pkl the compilation logic is embedded in the config, with ECS processing YAML / JSON, it is external to the config and is stored in strategies files. 

### ECS Feature Comparison Table

A comparison of ECS feature support as of today is provided below. Our roadmap is moving towards feature parity for both config languages:

| **Feature**       	| <center>**YAML/ JSON**</center> 	| <center>**Pkl**</center> 	|
|-------------------	|----------------	|---------	|
| **Webhooks** (trigger pipelines on changes)          	|        <center>X</center>       	|    <center>X</center>    	|
| **AI Based Schema Generation/ Validation** 	|        <center>X</center>       	|     <center>X</center>      	|
| **Config Versioning** (allows pipeline pinning and rollback)	|        <center>X</center>       	|    <center>X</center>    	|
| **Config Drift Detection**   	|        <center>X</center>       	|         	|
| **Secret Management** (store encrypted secrets in config)	|        <center>X</center>       	|         	|


## Where Next

See Pkl in action in ECS [here](../#pkl-examples).
