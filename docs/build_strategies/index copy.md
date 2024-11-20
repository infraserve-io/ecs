# Configuration Build Strategies

Build strategies are used to configure how configuration is compiled for use in pipelines. 

## **Examples**
Let's get started with some examples. Note, in these configuration examples, the file names are not taken into consideration, the only thing the config compiler cares about is file content, the object structure contained is what is merged, not the file names.


### **Default Build Strategies**

ECS ships with 2 simple inbuilt build strategies and others that configuring as they are dependent on your config diretory structure. Let's dig into the default strategies first. 

#### **Emit Path**

Emit path strategy simply emits the YAML config for the path specified.

##### Config

`ec2.yaml`
```yaml
ec2:
    disk_size: 400
```
buckets.yaml
```yaml
s3:
    encrypted: true
```

##### Command

`cto ecs config build --path <path to file or directory> --format yaml`

##### Output

```yaml
ec2:
    disk_size: 400
s3:
    encrypted: true
```

#### **Merge Path and Subdirectories**

Emit path and subdirectories strategy emits the YAML config for the path specified.

##### Directory Tree
```
-teams  
  -ec2.yaml  
  -team-1  
    -ec2.yaml  
```
##### Config

`teams/ec2.yaml`
```yaml
ec2:
    instance_type: small
    disk_size: 400
```

`teams/team-1/ec2.yaml`
```yaml
ec2:
    disk_size: 600
```

##### Command

`cto ecs config build --path teams --recursive --format yaml`

##### Output

```yaml
ec2:
    instance_type: small
    disk_size: 600
```

### **Custom Build Strategies**

In some cases ECS can't know what you want to do as after all, you can define any structure you like for your configuration. For these more complex build strategies you need to provide metadata to the config compiler so that it knows which directories to merge with which and in what order. Let's look at some examples of these strategies.

#### **Merge Common Data into a Multi Directory Config Structure**

This is where ECS shines and facilitates your config to become DRY (Do Not Repeat Yourself).

Here we'll show you how to create a set of configuration files in a common directory and merge a separate multi directory structure over the top of the common thus allowing parameter overrides and additions according to pipeline needs.

In order to differentiate the distinct subdirectories into separate configurations we use a strategy type of `data_with_key_from_path`. This is a very useful and popular strategy.  

##### Directory Tree
```
-common  
  -ec2.yaml  
  -object_storage.yaml
-teams  
  -__strategies.yaml
  -teams  
    -team-1  
      -ec2.yaml  
      -object_storage.yaml
    -team-2  
      -ec2.yaml  
      -object_storage.yaml
```

##### Config

`common/ec2.yaml`
```yaml
ec2:
  instance_type    : t2.medium
  user_data        : null
  ami              : ami-1234567890
  instance_profile : instance-profile-1
  disk_size        : 200
  vpc_name         : default
  tags:
    terraform : true
```

`common/object_storage.yaml`
```yaml
s3:
    versioning: true
    encryption: true
    tags:
        terraform : true
```

`teams/team-1/ec2.yaml`
```yaml
ec2:
    instance_type: t3.large
```

`teams/team-1/object_storage.yaml`
```yaml
s3:
    bucket_name: team-1-bucket
```

`teams/teams/team-2/ec2.yaml`
```yaml
ec2:
    instance_type: c5.xlarge
```

`teams/team-1/object_storage.yaml`
```yaml
s3:
    bucket_name: team-2-bucket
```

##### Command

`cto ecs config build --path teams --strategy merge_common_to_teams --format yaml`

##### Output

```yaml
team-1:                                                                       
  ec2:                                                                        
    ami: ami-1234567890                                                       
    disk_size: 200                                                            
    instance_profile: instance-profile-1                                      
    instance_type: t3.large                                                   
    tags:                                                                     
      terraform: true                                                         
    user_data: null                                                           
    vpc_name: default                                                         
  s3:
    versioning: true
    encryption: true
    tags:
        terraform : true
    bucket_name: team-1-bucket
team-2:                                                                       
  ec2:                                                                        
    ami: ami-1234567890                                                       
    disk_size: 200                                                            
    instance_profile: instance-profile-1                                      
    instance_type: c5.xlarge                                                  
    tags:                                                                     
      terraform: true                                                         
    user_data: null                                                           
    vpc_name: default 
  s3:
    versioning: true
    encryption: true
    tags:
        terraform : true
    bucket_name: team-2-bucket
```

Note how the directory names have been added as keys in the emitted structure. This allows you to pick the config for any team, and the only configuration that was specified for the specific team was the `instance_type` and `bucket_name`.

To cut the config down to the specific part required for a pipeline, use the `--filter` parameter on the build command, see example below:


##### Strategy Definition

In order to accomplish this output it is necessary to tell the config compiler what directories to work with and in what order. This is done by creating a metadata file called `__strategies.yaml` in the locatiom you wish to use as the root of the build. As you will note in the directory tree depicted above, this file is defined in the `teams` directory, the same directory specified in the `--path` parameter that was passed to `cto ecs config build`. Also note, the relative path to the common directory. ECS build strategies can access any directory below where they are placed. 

`./teams/__strategies.yaml`
```yaml
strategies:
  merge_common_to_teams:
    operations:
      - for_each:
          iterator: $team_data
          action:
            inject:
              - $common
              - $each
    variable_definitions:
      common:
        path: ./common
        type: data
      team_data:
        path: ./teams/{team}
        type: data_with_key_from_path
```

This file can contain multiple strategies for specific operations on your underlaying structure. In this case we have only one strategy called `merge_common_with_teams`.

Let's see how to use this strategy. We'll use the `--filter` parameter to extract the area of config we are interested in. 

`cto ecs config build --path teams --strategy merge_common_to_teams --format yaml --filter '"team-1".s3`

##### Output

```yaml
team-1:                                                                       
  s3:
    versioning: true
    encryption: true
    tags:
        terraform : true
    bucket_name: team-1-bucket
```

##### Multiple Directory Common

It is possible to specify more directories whose contents will be merged in as necessary. Note the order below, `./common` is read in first, then `./override_common` is merged over the top, lastly the individual teams are merged over the top by using the `for_each` construct. 

`./teams/__strategies.yaml`
```yaml
strategies:
  merge_common_and_override_to_teams:
    operations:
      - for_each:
          iterator: $team_data
          action:
            inject:
              - $common
              - $override_common
              - $each
    variable_definitions:
      common:
        path: ./common
        type: data
      override_common:
        path: ./override_common
        type: data
      team_data:
        path: ./teams/{team}
        type: data_with_key_from_path
```

#### **Layer Common Data into a Multi Directory Config Structure**

The compiler allows us to pick certain objects and dynamically merge them into destination objects. Let's modify our `common/ec2.yaml` and add some different configuration types. We'll use the base config, layer over the particular ec2 type, and then override with the specific teams config.

`common/ec2.yaml`
```yaml
ec2:
  instance_type    : t2.medium
  user_data        : null
  ami              : ami-1234567890
  instance_profile : instance-profile-1
  disk_size        : 200
  vpc_name         : default
  tags:
    terraform : true

ec2-base:
  instance_type    : t2.medium
  user_data        : null
  ami              : ami-1234567890
  instance_profile : instance-profile-1
  disk_size        : 200
  vpc_name         : default
  tags:
    terraform : true

ec2-select-type:
    standard:
        instance_type: t3.large
        disk_size: 1000
    developer:
        instance_type: c5.2xlarge
        disk_size: 3000
```

`./teams/__strategies.yaml`
```yaml
strategies:
  standard-ec2:
    operations:
      - for_each:
          iterator: $team_data
          action:
            inject:
              - $base
              - $selected_type
              - $each
    variable_definitions:
      team_data:
        path: ./teams/{team}
        type: data_with_key_from_path
      base:
        path: ./common/ec2.yaml
        type: data_with_key_from_object_field
        source_object_path: ec2-base
        destination_object_path: ec2
      selected_type:
        path: ./common/ec2.yaml
        type: data_with_key_from_object_field
        source_object_path: ec2-select-type.standard
        destination_object_path: ec2

  developer-ec2:
    operations:
      - for_each:
          iterator: $team_data
          action:
            inject:
              - $base
              - $selected_type
              - $each
    variable_definitions:
      team_data:
        path: ./teams/{team}
        type: data_with_key_from_path
      base:
        path: ./common/ec2.yaml
        type: data_with_key_from_object_field
        source_object_path: ec2-base
        destination_object_path: ec2
      selected_type:
        path: ./common/ec2.yaml
        type: data_with_key_from_object_field
        source_object_path: ec2-select-type.developer
        destination_object_path: ec2
```

In the example strategies above, `standard-ec2` is going to first load `ec2-base` object from common, then it will overlay `ec2-select-type.standard`, then the team specific config.

`cto ecs config build --path "/teams" --strategy-name standard-ec2` config with `instance_type` of `t3-large` for `team-1` and `c5.xlarge` for `team-2` as for that team it's overriden.

##### Output
```yaml
team-1:                
  ec2:
    ami: ami-1234567890
    disk_size: 1000    
    instance_profile: instance-profile-1                                                                                                                
    instance_type: t3.large 
    tags:              
      terraform: true  
    user_data: null    
    vpc_name: default  
team-2:                
  ec2:                 
    ami: ami-1234567890
    disk_size: 1000    
    instance_profile: instance-profile-1                                                                                                                
    instance_type: c5.xlarge
    tags:              
      terraform: true  
    user_data: null    
    vpc_name: default
```

`cto ecs config build --path "/teams" --strategy-name developer-ec2` config with `instance_type` of `c5.2xlarge` for `team-1` and `c5.xlarge` for `team-2` as for that team it's overriden.

##### Output
```yaml
team-1:                
  ec2:
    ami: ami-1234567890
    disk_size: 3000    
    instance_profile: instance-profile-1                                                                                                                
    instance_type: c5.2xlarge 
    tags:              
      terraform: true  
    user_data: null    
    vpc_name: default  
team-2:                
  ec2:                 
    ami: ami-1234567890
    disk_size: 3000    
    instance_profile: instance-profile-1                                                                                                                
    instance_type: c5.xlarge
    tags:              
      terraform: true  
    user_data: null    
    vpc_name: default
```

#### **Dynamic Config Data Creation Using Data Injection**

The above config is great and gets us closer to DRY, but when we want to create multiple of some resource (imagine a Terraform for_each iterating over a list of instances to create), possibly with different configurations, we don't want to have to repeat the config for each resource. We need a way to specify a resource name/ ID and have config injected into it with overrides being possible.

##### Directory Tree
```
-common  
  -ec2.yaml  
-teams  
  -__strategies.yaml
  -teams  
    -team-1  
      -ec2s.yaml  
    -team-2  
      -ec2s.yaml  
```
##### Config

`common/ec2.yaml`
```yaml
ec2:
  instance_type    : t2.medium
  user_data        : null
  ami              : ami-1234567890
  instance_profile : instance-profile-1
  disk_size        : 200
  vpc_name         : default
  tags:
    terraform : true
```

`teams/team-1/ec2s.yaml`
```yaml
ec2:
    ec2-instance-1: {}
    ec2-instance-2:
        instance_type: t3.medium
    ec2-instance-3:
        instance_type: t3.large
    ec2-instance-4:
        instance_type: c5.2xlarge
```

`teams/team-2/ec2s.yaml`
```yaml
ec2:
    ec2-instance-1:
        disk_size: 10000
    ec2-instance-2:
        instance_type: c5.4xlarge
        disk_size: 20000
```

`./__strategies.yaml`
```yaml
strategies:
  resource_per_team:
    operations:
      - for_each:
          iterator: $team_data
          actions:
            - for_each:
                iterator: $team_data.$each_key.ec2
                actions:
                  - inject:
                      - $base
                      - $each_value
            - add_parent: ec2
    variable_definitions:
      base:
        path: ${config_path}/common/ec2.yaml
        type: data_with_key_from_object_field
        source_object_path: ec2-base
        destination_object_path: ''
      team_data:
        path: ${config_path}/teams/{team}
        type: data_with_key_from_path
```