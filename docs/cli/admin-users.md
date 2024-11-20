# User Administration

All user admin functions are listed below for quick reference.

## List all users

`cto ecs users list`

## User Creation

Users are stored in a secret file in repository `/_users.yaml`.  By default users don't have access to any config, they need authorizing with `ecs users auth` command, examples are provided [here](#user-authorization).

When a new user is created, the CLI outputs a token that should be provided to the user along with their username. The user will be emailed to inform them their account has been created but we do not email their token. 

### Create a basic user

`cto ecs users create --username <username> --given-name <name> --family-name <name> --email <users email>`

### Create an admin user

`cto ecs users create --admin --username <username> --given-name <name> --family-name <name> --email <users email>`

### Modify an admin user to be non admin user

`cto ecs users edit --no-admin --username <username>`

### Create a basic user with ability to read secrets

This is typically needed only for a pipeline service account

`cto ecs users create --username <username> --given-name <name> --family-name <name> --email <users email> --read-secrets`

### Remove the ability to read secrets

`cto ecs users create --username <username> --no-read-secrets`

### Create a basic user with ability to edit merge strategies

`cto ecs users create --username <username> --given-name <name> --family-name <name> --email <users email> --edit-strategies`

### Remove ability to edit merge strategies

`cto ecs users edit --username <username> --no-edit-strategies`

### Create a basic user with ability to edit webhooks

`cto ecs users create --username <username> --given-name <name> --family-name <name> --email <users email> --edit-webhooks`

### Remove ability to edit webhooks

`cto ecs users edit --username <username> --no-edit-webhooks`

## User Authorization

Users are authorized to access specific areas of config. If a user has access to `/teams/team-1` they will have access to all config underneath that directory. 

Admin users can access entire config, therefore admin access should be granted with caution.

### List paths a user has access to

`cto ecs users auth --action list --username <username>`

### Authorize user to specific path

To add a user to a path, navigate to the path. `cto ecs users auth add` commands are executed in the context of that path.

`cto ecs users auth --action add --username <username>`

### Remove authorization for a specific path

To add a user to a path, navigate to the path. `cto ecs users auth delete` commands are executed in the context of that path.

`cto ecs users auth --action delete --username <username>`
