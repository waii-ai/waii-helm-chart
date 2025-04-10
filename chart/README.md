# Waii Helm Chart

This Helm chart deploys the Waii application in a Kubernetes cluster.

Ensure you have access to the Waii container registry. Reach out to Waii support for access

## Installation

To install the chart with the release name `waii`:

```bash
helm install waii .
```

## Configuration

The following table lists the configurable parameters of the Waii chart and their default values.

| Parameter | Description | Default | Type |
|-----------|-------------|---------|------|
| `namespace` | Kubernetes namespace | `waii` | string |
| `image.repository` | Container image repository | `waiilabs/sandbox` | string |
| `image.tag` | Container image tag | `latest` | string |
| `image.digest` | Container image digest | | string |
| `image.pullPolicy` | Container image pull policy | `Always` | string |
| `service.type` | Kubernetes Service type | `ClusterIP` | string |
| `service.port` | Service port | `3456` | integer |
| `env.enableLogStreamingDocker` | Enable Docker log streaming | `true` | boolean |
| `env.enableLogStreamingApplication` | Enable application log streaming | `true` | boolean |
| `env.redirectLogsDocker` | Redirect Docker logs | `false` | boolean |
| `env.rds.override` | Override RDS configuration | `false` | boolean |

### Waii Image tag and digest
Set only one of (`image.tag`, `image.digest`) i.e run with either of the following:
- `--set image.tag="image-tag" --set image.digest=""`
- `--set image.tag="" --set image.digest="image-digest"`

If both are set, preference is given to `image.digest`.

### RDS Configuration

When `env.rds.override` is set to `true`, the following RDS parameters can be configured:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `env.rds.host` | RDS host | `localhost` |
| `env.rds.port` | RDS port | `5432` |
| `env.rds.user` | RDS username | `postgres` |
| `env.rds.password.secret` | RDS password secret name | `rds-secret` |
| `env.rds.database` | RDS database name | `database` |

## Upgrading

To upgrade the chart with the release name `waii`:

```bash
helm upgrade waii .
```

## Uninstalling

To uninstall/delete the deployment:

```bash
helm uninstall waii
``` 