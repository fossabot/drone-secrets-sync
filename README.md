[![Build Status](https://nobadkitty.tplinkdns.com:8900/api/badges/colin-nolan/drone-secrets-sync/status.svg)](https://nobadkitty.tplinkdns.com:8900/colin-nolan/drone-secrets-sync)

## About

`drone-secrets-sync` is able to idempotently synchronise [Drone CI](https://www.drone.io) secrets (currently, only repository (not organisation) secrets are supported).

```shell
# Synchronise multiple repository secrets from JSON map on stdin
echo '{"SOME_SECRET": "example", "OTHER_SECRET": "value"}' \
    | drone-secrets-sync repository octocat/hello-world
```

```shell
# Synchronise repository secrets from JSON file
drone-secrets-sync repository octocat/hello-world secrets.json
```

The tool will output what secrets have changed, e.g.

```json
["SOME_SECRET","OTHER_SECRET"]
```

The Drone CI API does not provide access to secret values. Therefore, to allow the determination as to whether a secret already contains the required value, two secrets are created:

1. The requested secret with the name, and value supplied.
1. A corresponding secret with a name that contains a salted hash of the secret value, and a dummy value.

```shell
drone secret ls octocat/hello-world
```

```text
SECRET 
Pull Request Read:  false
Pull Request Write: false

SECRET___e861b26001c00803bb492889c1cf3faaf5a093ebc59f2c6838c7e10edfae4d0a 
Pull Request Read:  false
Pull Request Write: false
```

Be aware that exposing hashes makes it possible for an attacker that has gained access to the Drone API to brute force secret values offline. Hashes are generated using [Argon2](https://github.com/P-H-C/phc-winner-argon2/blob/master/argon2-specs.pdf) to make attacks more difficult.

## Installation

```shell
make install
```

## Usage

The tool uses the [Drone API](https://docs.drone.io/api/overview) via the official [drone-go](https://github.com/drone/drone-go) library. It requires `DRONE_TOKEN` and `DRONE_SERVER` environment variables to be setup, e.g.

```shell
# Configure environment - see: https://docs.drone.io/cli/configure
export DRONE_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
export DRONE_SERVER=http://drone.mycompany.com
```

```text
Usage: drone-secrets-sync [--verbose] <command> [<args>]

Options:
  --verbose, -v          enable verbose logging
  --help, -h             display this help and exit
  --version              display version and exit

Commands:
  repository             sync secrets for a repository
```

```text
Usage: drone-secrets-sync repository REPOSITORY [SECRETSFILE]

Positional arguments:
  REPOSITORY             repository to sync secrets for, e.g. octocat/hello-world
  SECRETSFILE            location to read secrets from (default: - (stdin))

Global options:
  --verbose, -v          enable verbose logging
  --help, -h             display this help and exit
  --version              display version and exit
```

## Development

### Build and Run

#### Executable

```shell
make build
```

To run after building:

```shell
./bin/drone-secrets-sync --help
```

#### Docker Image

```shell
image="$(make build-docker)"
```

To run after building:

```shell
docker run --rm --pull never -e DRONE_SERVER -e DRONE_TOKEN "${image}" --help
```

### Test

```shell
make test
```

### Linting

```shell
make lint
```

Requires:

- [golangci-lint](https://github.com/golangci/golangci-lint)
- [mdformat-gfm](https://github.com/executablebooks/mdformat)

#### Apply Format

```shell
make format
```

#### CI

To run a Drone CI step manually:

```shell
drone exec -pipeline=lint
```

### Clear Secrets

When testing against a Drone CI installation, to clear all secrets on a repository:

```shell
repository=colin-nolan/drone-testing
drone secret ls --format '{{ .Name }}' "${repository}" \
    | xargs -I {} drone secret rm --name {} "${repository}"
```

Requires:

- [drone-cli](https://docs.drone.io/quickstart/cli/)

## Alternatives

- [drone-secret-sync](https://github.com/appleboy/drone-secret-sync) can synchronise secrets across multiple orgs/repositories. Not idempotent - will update all secrets every time.

## Legal

GPL v3 (contact for other licencing). Copyright 2023 Colin Nolan.

This work is in no way related to any company that I may work for.
