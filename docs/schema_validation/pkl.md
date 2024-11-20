# Configuration Build Schema Validation

## Overview

For Pkl outputs of type YAML or JSON it is possible to create a JSON schema validation file called `__schemas.yaml` alongside any `__strategies.yaml` file and the schema will be used to validate output from the Pkl build. When a schema is in place, any changes to the configuration need to create output of the same object structure, if they don't, the build will fail. This makes debugging pipeline failures a lot easier as ECS will gracefully fail the pipeline with a meaningful error informing you what caused the failure. 

You can either write this schema file yourself, or you can use the ECS AI to generate it for you automatically based on the build output for the particular strategy and path. 

Using the config examples in the getting started repo, the command `cto ecs config generate-schema --path config-examples/example-7/birds.pkl` will create the output below.

```yaml
bird:                                                                                          
  name: Pigeon                                                                                 
  diet: Seeds                                                                                  
  taxonomy:                                                                                    
    kingdom: Animalia                                                                          
    clade: Dinosauria                                                                          
    order: Columbiformes                                                                       
parrot:                                                                                        
  name: Parrot                                                                                 
  diet: ss                                                                                     
  taxonomy:                                                                                    
    kingdom: Animalia                                                                          
    clade: Dinosauria                                                                          
    order: Psittaciformes  
```

Use ECS AI to generate a schema: 

`cto ecs config generate-schema --path config-examples/example-7/birds.pkl --write`

This command generates the following file:

`__schemas.yaml`

```yaml
---
schemas:
  config-examples/example-7/birds.pkl:
    additionalProperties: false
    patternProperties:
      .*:
        additionalProperties: false
        properties:
          diet:
            type: string
          name:
            type: string
          taxonomy:
            additionalProperties: false
            properties:
              clade:
                type: string
              kingdom:
                type: string
              order:
                type: string
            required:
            - kingdom
            - clade
            - order
            type: object
        required:
        - name
        - diet
        - taxonomy
        type: object
    type: object
```

## AI Generated Schemas

ECS AI needs to be licensed and turned on for your ECS Server instance, you can use the following commands to generate and write schemas for any build output:

### Use AI to Generate Schema

`cto ecs config generate-schema --path <path to Pkl file>`

### Use AI to Generate and Write Schema

`cto ecs config generate-schema --write --path <path Pkl file>`

### See it in Action

![ECS Pkl Config Schema Validation](ecs-config-pkl-schema.gif)

## Manual Schema Creation and Editing

To write your own schemas or to understand the schema language, see <a target="_news" href="https://json-schema.org/learn/getting-started-step-by-step">json-schema.org</a>

## Where Next?

Check out Secret Management [here](../secret_management)