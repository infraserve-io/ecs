# ECS CLI Overview

The ECS CLI is used to interact with the ECS server. The CLI can only be used by authorized users. There are two user types, regular and admin.

Actions CLI users can take are:

| **Action**                                                                          | **Admin** | **Regular** |
|-------------------------------------------------------------------------------------|-----------|----------|
| Create/ remove users                                                                |     X     |          |
| Grant access to areas of config hierarchy                                           |     X     |          |
| Use build strategies                                                             |     X     |          |
| Edit build strategies (regular user needs --edit-strategies)                                                             |     X     |     X    |
| Create / edit/ remove config at level of hierarchy they have been granted access to |     X     |     X    |
| Consume pipeline config at level of hierarchy they have been granted access to      |     X     |     X    |
| Encrpyt secrets                                                                     |     X     |     X    |
| Decrpyt secrets (regular user needs --show-secrets)      |     X     |     X    |

When a config repo is empty, i.e, when initializing your ECS setup, the first user to interact with the CLI via `ecs init` automatically gets created as an admin user and that users access token is automatically stored in `~/.cto/ecs_settings.json`. 

Subsequent users need to provide a token for the user that has been created for them by an admin. Thr token is emitted on user creation.

User config is saved in a file `~/.cto/ecs_settings.json` including the users server access token.

## CLI Commands

A list of commands by function is provided in the following sections. 

### Admin Commands

[User](./admin-users)

### User Commands

[Config Pull, Edit and Save](./user-config-manage)

[Pipeline Config](./user-config-pipelines)