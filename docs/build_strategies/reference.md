# Strategy syntax reference

## Operations 

### inject
`inject` reads data specified as a string variable and adds its content to the main object.

```yaml
inject:
  - $common
  - team_data
```

Let's assume the variables have the following values:

```yaml
common:
  content:
    common_var: common_test
team_data:
  content:
    var: team1
```

The result:

```yaml
content:
    common_var: common_test
    var: team1
```

The inject operation supports object key unpacking using the dot notation.  

To unpack the `content` key from example above:

```yaml
inject:
  - team_data.content
```

The result:

```yaml
var: team1
```

### add_parent
`add_parent` modifies the current object by adding a parent object.

```yaml
inject:
  - $common
  - team_data
add_parent: parent_key
```

```yaml
common:
  content:
    common_var: common_test
team_data:
  content:
    var: team1
```

The result:

```yaml
prent_key:
    content:
        common_var: common_test
        var: team1
```

### for_each
`for_each` loops are used to build new objects based on data read from variables.

#### Standard for_each loop:
By default this loops adds the key it iterates over to the main result. 

```yaml
directory_data:
  team1:
    data:
      path: team1
      organization: organization1
  team2:
    data:
      path: team2
      organization: organization2
```

```yaml
for_each:
  iterator: $directory_data
  alias: directory_iterator
  actions:
    - inject:
          - $each_value.data
    
```

The result
```yaml
team1: # <- this key was added automatically by the loop 
  path: team1
  organization: organization1
team2: # <- this key was added automatically by the loop 
  path: team2
  organization: organization2
```

`iterator` - an object the loops iterates over

`alias` - it provides a name for the loop, the alias can be used to access the loop's data from an inner for loop. Without aliases it would not be possible to access data in the loop above the immediate parent as the immediate parent is accessed with `$each_key` and `$each_value`. An alias is accessed by using $ in front of alias name, and each_key and each_value are available by appending to the alias name with # in between. Examples are `$<alias_name>#each_key` and `$<alias_name>#each_value`.

`actions` - a list of actions that will be applied within the loops. Actions: `inject`, `add_parent`, `for_each`

`$each_value` - an object in the current iteration. In the example above this is how `each_value` looks for the first iteration:
```yaml
    path: team1
    organization: organization1
```

`$each_key` - a string in the current iteration. In the example above the first iteration `each_key=path`


#### List iterator for_each loop:
`list_iterator` loop iterates over string literals.

```yaml
common_resources:
  resources:
    type_x:
      common_resource_x_1:
        type: resource
      common_resource_x_2:
        type: resource
    type_y:
      common_resource_y_1:
        type: resource
      common_resource_y_2:
        type: resource
team_data:
  resources:
    type_x:
        team_resource_x_1:
          type: team_resource
    type_y:
        team_resource_y_1:
          type: team_resource
    
  
```

```yaml
for_each:
  list_iterator: [type_x, type_y]
  alias: list_iterator
  actions:
    inject:
      - $common.resources.$each_value
      - team_data.resources.$each_value
```

The result:

```yaml
common_resource_x_1:
  type: resource
common_resource_x_2:
  type: resource
team_resource_x_1:
  type: team_resource
common_resource_y_1:
  type: resource
common_resource_y_2:
  type: resource
team_resource_y_1:
  type: team_resource
     
```

**IMPORTANT**: this type of for loop returns only `each_value`

## Variables

### data
`data` loads data from the specified path. If the path is a directory, it loads all the files in that directory and stores them as one big object. Alternatively, an individual file can be specified as below.

```yaml
variable_definitions:
  common:
    path: /common/ec2.yaml
    type: data
```

### data_with_key_from_path
`data_with_key_from_path` extends the data type functionality. The loaded data is organized based on a variable in curly braces. This data type is useful for for_each loops. 

```yaml
variable_definitions:
    common:
      path: /teams/{team}
      type: data_with_key_from_path
```

Let's assume that we have the following paths:
```
.
├── team1
│  ├── nested
│  │  └── second.yaml
│  └── one.yaml
└── team2
    ├── nested
    │  └── second.yaml
    └── one.yaml
```

The loaded data for that example is:

```yaml
team1:
  <merged_files_for_team1>
team2:
  <merged_files_for_team2>
```


### data_with_key_from_object_field
`data_with_key_from_object_field` extends the base data type functionality. Apart from loading data it allows to manipulate the loaded object, like choosing only a part of the object and adding parents to it.

```yaml
variable_definitions:
    common:
      path: /common/ec2.yaml
      type: data_with_key_from_object_field
      source_object_path: ec2
      destination_object_path: my_resource
```

ec2.yaml
```yaml

ec2:
  instance_type: t3.medium
  owner: Mike
```

The loaded that would be:

```yaml

my_resource:
  instance_type: t3.medium
  owner: Mike
```

## Where Next?

Learn all the CLI commands [here](../../cli).
