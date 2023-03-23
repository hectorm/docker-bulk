# docker bulk

Perform bulk actions on Docker resources.

## Installation

### Requirements
 * [`jq`](https://stedolan.github.io/jq/) `1.6+`

### Current user

```sh
[ -n "$DOCKER_CONFIG" ] || export DOCKER_CONFIG="$HOME"/.docker; mkdir -p "$DOCKER_CONFIG"/cli-plugins
curl --proto '=https' --tlsv1.3 -o "$DOCKER_CONFIG"/cli-plugins/docker-bulk 'https://raw.githubusercontent.com/hectorm/docker-bulk/v0.0.1/docker-bulk'
chmod +x "$DOCKER_CONFIG"/cli-plugins/docker-bulk
```

### System-wide

```sh
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl --proto '=https' --tlsv1.3 -o /usr/local/lib/docker/cli-plugins/docker-bulk 'https://raw.githubusercontent.com/hectorm/docker-bulk/v0.0.1/docker-bulk'
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-bulk
```

## Usage

```sh
docker bulk RESOURCE FILTER [COMMAND]
```

The command is executed with the resource ID as the last argument. If the command is omitted, the resource ID is printed.

> **Note**  
> `jq` is used to perform the filtering on the JSON specification of the resource, please refer to its [manual](https://stedolan.github.io/jq/manual/) to learn more about the filter syntax.

### Examples

#### Restart unhealthy containers:
```sh
docker bulk container '.State.Status == "running" and .State.Health.Status == "unhealthy"' docker container restart
```

#### Restart containers that have the Docker daemon socket mounted:
```sh
docker bulk container '.Mounts[].Source == "/var/run/docker.sock"' docker container restart
```

#### Remove volumes with the prefix `test_` in their name:
```sh
docker bulk volume '.Name | startswith("test_")' docker volume rm
```

#### List all images:
```sh
docker bulk image '.'
```

#### List images bigger than 500 MB:
```sh
docker bulk image '.Size > 500 * $MB'
```

#### List images older than 7 days:
```sh
docker bulk image '.Created | sub("[.][0-9]+Z$"; "Z") | fromdate < now - 7 * $DAY'
```

#### List IPv6 capable networks with the `bridge` driver:
```sh
docker bulk network '.EnableIPv6 == true and .Driver == "bridge"'
```
