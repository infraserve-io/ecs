# Configuration Build Schema Verification

## Overview

It is possible to create a JSON schema validation file called `__schemas.yaml` alongside any `__strategies.yaml` file and the schema will be used to validate output from the strategy. 

You can either write this schema file yourself, or you can use the ECS AI to generate it for you automatically based on the build output for the particular strategy and path. 

As an example, the following build output that uses the strategy `generate_ec2s`can be validated by using the `__schemas.yaml` file that follows.

```yaml
team-1:                                                                                
  ec2:                                                                                 
    server-1:                                                                          
      ami: ami-123456789                                                               
      disk_size: 300                                                                   
      instance_profile: instance-profile-1                                             
      instance_type: t2.medium                                                         
      tags:                                                                            
        terraform: true                                                                
      user_data: null                                                                  
      vpc_name: default                                                                
    server-2:                                                                          
      ami: ami-123456789                                                               
      disk_size: 800                                                                   
      instance_profile: instance-profile-1                                             
      instance_type: t2.medium                                                         
      tags:                                                                            
        terraform: true                                                                
      user_data: null                                                                  
      vpc_name: default                                                                
team-2:                                                                                
  ec2:                                                                                 
    server-1:                                                                          
      ami: ami-123456789                                                               
      disk_size: 1400                                                                  
      instance_profile: instance-profile-1                                             
      instance_type: t2.medium                                                         
      tags:                                                                            
        terraform: true                                                                
      user_data: null                                                                  
      vpc_name: default                                                                
    server-2:                                                                          
      ami: ami-123456789                                                               
      disk_size: 1800                                                                  
      instance_profile: instance-profile-1                                             
      instance_type: t2.medium                                                         
      tags:                                                                            
        terraform: true                                                                
      user_data: null                                                                  
      vpc_name: default  
```

`__schemas.yaml`

```yaml
---
schemas:
  generate_ec2s:
    $schema: "http://json-schema.org/draft-07/schema#"
    title: "Team EC2 Instances Configuration"
    type: "object"
    patternProperties:
      "^team-\\d+$":
        type: "object"
        properties:
          ec2:
            type: "object"
            patternProperties:
              "^server-\\d+$":
                type: "object"
                properties:
                  ami:
                    type: "string"
                    description: "AMI ID"
                  disk_size:
                    type: "integer"
                    description: "Size of the disk in GB"
                  instance_profile:
                    type: "string"
                    description: "Instance profile name"
                  instance_type:
                    type: "string"
                    description: "Type of the instance"
                  tags:
                    type: "object"
                    properties:
                      terraform:
                        type: "boolean"
                        description: "Terraform tag"
                    required:
                      - "terraform"
                    additionalProperties: false
                  user_data:
                    type:
                      - "string"
                      - "null"
                    description: "User data script"
                  vpc_name:
                    type: "string"
                    description: "Name of the VPC"
                required:
                  - "ami"
                  - "disk_size"
                  - "instance_profile"
                  - "instance_type"
                  - "tags"
                  - "user_data"
                  - "vpc_name"
                additionalProperties: false
        required:
          - "ec2"
        additionalProperties: false
    additionalProperties: false
```

## AI Generated Schemas

If ECS AI is licensed and turned on for your ECS Server instance, you can use the following commands to generate and write schemas for any build output:

### Use AI to Generate Schema

`cto ecs config generate-schema --path <path to your config> --strategy_name <strategy name>`

### Use AI to Generate and Write Schema

`cto ecs config generate-schema --write --path <path to your config> --strategy_name <strategy name>`

## Manual Schema Creation and Editing

To write your own schemas or to understand the schema language, see <a target="_news" href="https://json-schema.org/learn/getting-started-step-by-step">json-schema.org</a>

## Where Next?

Check out Secret Management [here](../secret_management)