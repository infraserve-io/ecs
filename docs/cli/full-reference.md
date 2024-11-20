# CTO CLI

**Usage**:

```console
$ version [OPTIONS] COMMAND [ARGS]...
```

**Options**:

* `-v, --version`: Print the version and exit
* `--install-completion`: Install completion for the current shell.
* `--show-completion`: Show completion for the current shell, to copy it or customize the installation.
* `--help`: Show this message and exit.

**Commands**:

* `ecs`: ECS CLI

## `version ecs`

ECS CLI

**Usage**:

```console
$ cto ecs [OPTIONS] COMMAND [ARGS]...
```

**Options**:

* `--help`: Show this message and exit.

**Commands**:

* `config`
* `init`
* `users`

### `cto ecs config`

**Usage**:

```console
$ cto ecs config [OPTIONS] COMMAND [ARGS]...
```

**Options**:

* `--help`: Show this message and exit.

**Commands**:

* `build`
* `decrypt`
* `generate-schema`
* `pull`
* `push`
* `status`

#### `cto ecs config build`

**Usage**:

```console
$ cto ecs config build [OPTIONS]
```

**Options**:

* `--path TEXT`: [required]
* `--strategy-name TEXT`
* `--format TEXT`
* `--filter TEXT`: filter result using JMESPath
* `--config-id TEXT`
* `--detect-drift / --no-detect-drift`: [default: no-detect-drift]
* `--recursive / --no-recursive`: [default: no-recursive]
* `--show-secrets / --no-show-secrets`: [default: no-show-secrets]
* `--help`: Show this message and exit.

#### `cto ecs config decrypt`

**Usage**:

```console
$ cto ecs config decrypt [OPTIONS]
```

**Options**:

* `--path TEXT`: [required]
* `--help`: Show this message and exit.

#### `cto ecs config generate-schema`

**Usage**:

```console
$ cto ecs config generate-schema [OPTIONS]
```

**Options**:

* `--path TEXT`: [required]
* `--strategy-name TEXT`
* `--write / --no-write`: [default: no-write]
* `--help`: Show this message and exit.

#### `cto ecs config pull`

**Usage**:

```console
$ cto ecs config pull [OPTIONS]
```

**Options**:

* `--help`: Show this message and exit.

#### `cto ecs config push`

**Usage**:

```console
$ cto ecs config push [OPTIONS]
```

**Options**:

* `--tag TEXT`
* `--help`: Show this message and exit.

#### `cto ecs config status`

**Usage**:

```console
$ cto ecs config status [OPTIONS]
```

**Options**:

* `--help`: Show this message and exit.

### `cto ecs init`

**Usage**:

```console
$ cto ecs init [OPTIONS]
```

**Options**:

* `--help`: Show this message and exit.

### `cto ecs users`

**Usage**:

```console
$ cto ecs users [OPTIONS] COMMAND [ARGS]...
```

**Options**:

* `--help`: Show this message and exit.

**Commands**:

* `auth`
* `create`
* `delete`
* `edit`
* `list`
* `regenerate-token`

#### `cto ecs users auth`

**Usage**:

```console
$ cto ecs users auth [OPTIONS]
```

**Options**:

* `--username TEXT`: [required]
* `--action [add|delete|list]`: [required]
* `--help`: Show this message and exit.

#### `cto ecs users create`

**Usage**:

```console
$ cto ecs users create [OPTIONS]
```

**Options**:

* `--username TEXT`: [required]
* `--given-name TEXT`: [required]
* `--family-name TEXT`: [required]
* `--email TEXT`: [required]
* `--admin / --no-admin`: [default: no-admin]
* `--read-secrets / --no-read-secrets`: [default: no-read-secrets]
* `--edit-strategies / --no-edit-strategies`: [default: no-edit-strategies]
* `--edit-webhooks / --no-edit-webhooks`: [default: no-edit-webhooks]
* `--help`: Show this message and exit.

#### `cto ecs users delete`

**Usage**:

```console
$ cto ecs users delete [OPTIONS]
```

**Options**:

* `--username TEXT`: [required]
* `--help`: Show this message and exit.

#### `cto ecs users edit`

**Usage**:

```console
$ cto ecs users edit [OPTIONS]
```

**Options**:

* `--username TEXT`: [required]
* `--given-name TEXT`
* `--family-name TEXT`
* `--email TEXT`
* `--admin / --no-admin`
* `--read-secrets / --no-read-secrets`
* `--edit-strategies / --no-edit-strategies`
* `--edit-webhooks / --no-edit-webhooks`
* `--help`: Show this message and exit.

#### `cto ecs users list`

**Usage**:

```console
$ cto ecs users list [OPTIONS]
```

**Options**:

* `--help`: Show this message and exit.

#### `cto ecs users regenerate-token`

**Usage**:

```console
$ cto ecs users regenerate-token [OPTIONS]
```

**Options**:

* `--username TEXT`: [required]
* `--help`: Show this message and exit.