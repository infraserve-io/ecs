# Configuration Build Strategies

Build strategies are used to configure how configuration is compiled for use in pipelines. 


## Background

When ECS config is written in YAML or JSON, the strategy language can be used to merge config objects in meaningful ways. It is a powerful feature that is the basis of how ECS promotes DRY configuration. 

### How Does it Work

When the config is built/ compiled using the `cto ecs config build` command, a base directory is specified using `--path` parameter. All the files under this path are recursively loaded into a set of dictionaries. Using a set of declarative constructs that are written in a `__strategies.yaml` file, the strategy language gives you control over how these dictionaries are merged to produce the final output. 

#### Merging

An example dictionary is shown below, treat this as configuration that is common to all of a certain area of config and it might live in a file in a directory somewhere in your config called `common`:

```json
{
    "vpc": "vpc-common",
    "s3-encryption": true
}
```

A second dictionary lives in a file in an account specific area of the config hierarchy.
```json
{
    "vpc": "vpc-account-specific-override",
}
```

When these 2 dictionaries are merged using common as the base, the result is the following dictionary:

```json
{
    "vpc": "vpc-account-specific-override",
    "s3-encryption": true
}
```

It can be seen that the common values with no matching keys in the account specific file are preserved, whilst the keys that match are overwritten. 

#### Iterators

Iterators allow the loaded dictionaries to be traversed and merged as above, with certain manipulations being possible such as injecting additional parent keys. We'll use the following dictionary to explain how iterators can be used:

```json
{
    "key-1": "value-1",
    "key-2": "value-2",
    "key-3": "value-3"
}
```

To traverse the dictionary above, we define an iterator using `for_each` and use `each_key` to access the keys and `each_value` to access the values. The values would typically be object structures as defined in your config hierarchy such as the following:


## Permissions

Build strategies can be managed by admin users and users that were created or edited with the `--edit-stratgies` flag. To remove the ability for a user to change strategies, use `cto ecs users edit --no-edit-strategies`.


## __strategy.yaml files

These files contain the logic for config compilation. You can learn how to use the strategy language by following <a target="_new" href="https://github.com/Cloud-Technology-Office/ecs-getting-started/tree/main/config-examples">config examples</a> and by reading the [strategy language reference](reference.md)

Strategy files can exist in 2 distinct areas of config, either in the path of the config they are intended to operate with, or in the root of the config where they can be written once by an ECS admin and consumed by any user within the context of paths to which they have access.

### Path Specific Strategies

Strategies can be created and managed by users to suit their needs. Of course, the user has to have access to the particular path, meaning they can't change strategies they are not authorized to access. Users also need to have the `--edit-strategies` flag set to true in order to change strategies files. 

Path based strategies need to exist in the path specified in the `cto ecs config build --path <path>` command.

Paths inside the `__strategies.yaml` file are specified relative to the path of the file and can be relative upwards in the directory structure (../../), providing the user has access to the specified path. 


## Where Next?

Check out the strategy reference [here](reference.md)