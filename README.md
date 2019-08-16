# Portainer Stack Utils

[![CircleCI](https://circleci.com/gh/greenled/portainer-stack-utils.svg?style=svg)](https://circleci.com/gh/greenled/portainer-stack-utils)
[![Docker Automated build](https://img.shields.io/docker/automated/greenled/portainer-stack-utils.svg)](https://hub.docker.com/r/greenled/portainer-stack-utils/)
[![Docker Pulls](https://img.shields.io/docker/pulls/greenled/portainer-stack-utils.svg)](https://hub.docker.com/r/greenled/portainer-stack-utils/)
[![Microbadger](https://images.microbadger.com/badges/image/greenled/portainer-stack-utils.svg)](http://microbadger.com/images/greenled/portainer-stack-utils "Image size")
[![Go Report Card](https://goreportcard.com/badge/github.com/greenled/portainer-stack-utils)](https://goreportcard.com/report/github.com/greenled/portainer-stack-utils)

## Overview

Portainer Stack Utils is a CLI client for [Portainer](https://portainer.io/) written in Go.

## Supported Portainer API

This application was created for the latest Portainer API, which at the time of writing is [1.22.0](https://app.swaggerhub.com/apis/deviantony/Portainer/1.22.0).

## How to install

Download the binaries for your platform from [the releases page](https://github.com/greenled/portainer-stack-utils/releases). The binaries have no external dependencies.

You can also install the source code with `go` and build the binaries yourself.

## How to use

The application is built on a structure of commands, arguments and flags.
                   
**Commands** represent actions, **Args** are things and **Flags** are modifiers for those actions:

```text
APPNAME COMMAND ARG --FLAG
```

Here are some examples:

```bash
psu help
psu status --help
psu stack ls --endpoint primary --format "{{ .Name }}"
psu stack deploy mystack --stack-file docker-compose.yml -e .env --log-level debug
psu stack rm mystack
```

Commands can have subcommands, like `stack ls` and `stack deploy` in the previous example. They can also have aliases (i.e. `create` and `up` are aliases of `deploy`).

Some flags are global, which means they affect every command (i.e. `--log-level`), while others are local, which mean they only affect the command they belong to (i.e. `--stack-file` flag from `deploy` command). Also, some flags have a short version (i.e `--insecure`, `-i`).

### Configuration

The program can be configured through [inline flags](#inline-flags) (i.e. `--user`), [environment variables](#environment-variables) (i.e. `PSU_USER=admin`) and/or [configuration files](#configuration-files), which translate into multi-level configuration keys in the form `x[.y[.z[...]]]`. Run `psu config ls` to see all available configuration options.

All three methods can be combined. If a configuration key is set in several places the order of precedence is:

1. Inline flags
2. Environment variables
3. Configuration file
4. Default values

#### Inline flags

Configuration can be set through inline flags. Valid combinations of commands and flags directly map to configuration keys:

| Configuration key | Command | Flag |
| :---------------- | :------ | :--- |
| user | psu | --user |
| stack.list.format | psu stack list | --format |
| stack.deploy.env-file | stack deploy | --env-file |

Run `psu help COMMAND` to see all available flags for a given command.

#### Environment variables

Configuration can be set through environment variables. Supported environment variables follow the `PSU_[COMMAND_[SUBCOMMAND_]]FLAG` naming pattern:

| Configuration key | Environment variable |
| :---------------- | :------------------- |
| user | PSU_USER |
| stack.list.format | PSU_STACK_LIST_FORMAT |
| stack.deploy.env-file | PSU_STACK_DEPLOY_ENV_FILE |

*Note that all supported environment variables are prefixed with "PSU_" to avoid name clashing. Characters "-" and "." in configuration key names are replaced with "_" in environment variable names.*

#### Configuration files

Configuration can be set through a configuration file. Supported file formats are [JSON](#json-configuration-file), TOML, [YAML](#yaml-configuration-file), HCL, envfile and Java properties config files. Use the `--config` global flag to specify a configuration file. File `$HOME/.psu.yaml` is used by default if present.

##### YAML configuration file

A Yaml configuration file should look like this:

```yaml
log-level: debug
user: admin
insecure: true
stack.list.format: table
stack:
  deploy.env-file: .env
  deploy:
    stack-file: docker-compose.yml
```

*Note that flat and nested keys are both valid.*

##### JSON configuration file

A JSON configuration file should look like this:

```json
{
  "log-level": "debug",
  "user": "admin",
  "insecure": true,
  "stack.list.format": "table",
  "stack": {
    "deploy.env-file": ".env",
    "deploy": {
      "stack-file": "docker-compose.yml"
    }
  }
}
```

*Note that flat and nested keys are both valid.*

### Environment variables for deployed stacks

You will often want to set environment variables in your deployed stacks. You can do so through the `stack.deploy.env-file` [configuration key](#configuration). :

```bash
touch .env
echo "MYSQL_ROOT_PASSWORD=agoodpassword" >> .env
echo "ALLOWED_HOSTS=*" >> .env

# Using --env-file flag
psu stack deploy django-stack -c /path/to/docker-compose.yml -e .env

# Using PSU_STACK_DEPLOY_ENV_FILE environment variable
PSU_STACK_DEPLOY_ENV_FILE=.env
psu stack deploy django-stack -c /path/to/docker-compose.yml

# Using a config file
echo "stack.deploy.env-file: .env" > .config.yml
psu stack deploy django-stack -c /path/to/docker-compose.yml --config .config.yml
```

### Log level

You can control how much noise you want the program to do by setting the log level. There are seven log levels:

- *panic*: Unexpected errors that stop program execution.
- *fatal*: Expected errors that stop program execution.
- *error*: Errors that should definitely be noted but don't stop the program execution.
- *warning*: Non-critical events that deserve eyes.
- *info*: General events about what's going on inside the program. This is the default level.
- *debug*: Very verbose logging. Usually only enabled when debugging.
- *trace*: Finer-grained logging than the *debug* level.

**WARNING**: **trace** level will print sensitive information, like Portainer API requests and responses (with authentication tokens, stacks environment variables, and so on). Avoid using **trace** level in CI/CD environments, as those logs are usually recorded.

This is an example with *debug* level:

```bash
psu stack deploy asd --endpoint primary --log-level debug
```

The output would look like:

```text
DEBU[0000] Getting endpoint's Docker info     endpoint=5
DEBU[0000] Getting stack                      endpoint=primary stack=asd
DEBU[0000] Stack not found                    stack=asd
INFO[0000] Creating stack                     endpoint=primary stack=asd
INFO[0000] Stack created                      endpoint=primary id=89 stack=asd
```

## Contributing

So, you want to contribute to the project:

- Fork it
- Download your fork to your PC (git clone https://github.com/your_username/portainer-stack-utils && cd portainer-stack-utils)
- Create your feature branch (git checkout -b my-new-feature)
- Make changes and add them (git add .)
- Commit your changes (git commit -m 'Add some feature')
- Push to the branch (git push origin my-new-feature)
- Create a new pull request

If you are submitting a complex feature, create a small design proposal on the [issue tracker](https://github.com/greenled/portainer-stack-utils/issues) before you start.

## License

Source code contained by this project is licensed under the [GNU General Public License version 3](https://www.gnu.org/licenses/gpl-3.0.en.html). See [LICENSE](LICENSE) file for reference.
