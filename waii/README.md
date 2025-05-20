# Waii Helm Chart

This Helm chart deploys the Waii application in a Kubernetes cluster. This chart is located and maintained at: https://github.com/waii-ai/waii-helm-chart

# Prerequisites

- Ensure the `waii` namespace is already created and the required K8s secrets are present in it.
- If the required K8s secrets are missing, they will be created as pre-install hooks. You can override the secret values from values.yaml file.
- Ensure that you have access to the Waii container registry. Please contact Waii support for details and credentials.
- Ensure that you have access to the Waii chart registry. Please contact Waii support for details and credentials.
- You should be connected to the K8s cluster by exporting the `KUBECONFIG` env variable to the kube config file path.

# Notes

- You can install the chart in any namespace (by overriding it in the values.yaml file) or install it in the release namespace itself (by overriding it to null/empty in values.yaml file).

# Installation

To install the chart with the release name `waii-release`:

## Local chart

```Bash
# Assuming you are in the chart's directory.
helm install waii-release . --create-namespace -n release-ns
```

## Local packaged chart

```Bash
# Assuming you are in the chart's parent directory.
helm package ./waii
helm install waii-release ./waii-1.0.0.tgz --create-namespace -n release-ns
```

## From registry

```Bash
helm registry login registry-1.docker.io
# Enter username and password when prompted.
helm install waii-release oci://registry-1.docker.io/waiilabs/waii --version 1.0.0 \
--create-namespace -n release-ns
```

# Configuration

The following table lists the configurable parameters of the Waii chart and their default values.

| Parameter                                    | Description                                                | Default                      | Type    |
|----------------------------------------------|------------------------------------------------------------|------------------------------|---------|
| `image.repository`                           | Container image repository                                 | `waiilabs/sandbox`           | String  |
| `image.tag`                                  | Container image tag, ignored if digest is specified        | `1.30.10-x86`                | String  |
| `image.digest`                               | Container image digest, has higher priority over image tag | `""`                         | String  |
| `dockerRegistry.username`                    | Username for Docker registry authentication                | `getitfromwaii`              | String  |
| `dockerRegistry.password`                    | Password for Docker registry authentication                | `getitfromwaii`              | String  |
| `containerArgs`                              | Extra arguments to pass to the container entrypoint        | `["--api_key_auth_enabled"]` | List    |
| `env.enableLogStreamingDocker`               | Stream Docker container logs to stdout in real-time        | `true`                       | Boolean |
| `env.enableLogStreamingApplication`          | Stream application logs to stdout in real-time             | `true`                       | Boolean |
| `env.redirectLogsDocker`                     | Dump Docker logs to stdout after container starts          | `false`                      | Boolean |
| `env.loadSampleDb`                           | Start container with toy database                          | `true`                       | Boolean |
| `env.rds.override`                           | Whether to override RDS values                             | `false`                      | Boolean |
| `env.rds.host`                               | RDS host                                                   | `localhost`                  | String  |
| `env.rds.port`                               | RDS port                                                   | `5432`                       | Integer |
| `env.rds.user`                               | RDS username                                               | `postgres`                   | String  |
| `env.rds.password.secret`                    | Secret name for RDS password                               | `rds-secret`                 | String  |
| `env.rds.password.hardcoded`                 | Hardcoded RDS password (fallback)                          | `mypassword`                 | String  |
| `env.rds.database`                           | RDS database name                                          | `database`                   | String  |
| `files.llmEndpointConfig.rawContent`         | Raw content for LLM endpoint config file                   | `""`                         | String  |
| `files.llmEndpointConfig.apiKey`             | API key for OpenAI in the example config                   | `myapikeytooverride`         | String  |
| `secrets.llmEndpointConfig.secret`           | Name of the secret for LLM endpoint config                 | `waii-llm-config`            | String  |
| `secrets.rdsUrl.secret`                      | Name of the secret for RDS credentials                     | `waii-rds-creds`             | String  |
| `secrets.rdsUrl.data.rdsUrl`                 | RDS URL for Waii app deployment                            | `""`                         | String  |
| `secrets.waiiSecrets.secret`                 | Name of the secret for Waii secrets                        | `waii-secrets`               | String  |
| `secrets.waiiSecrets.data.waiiDefaultApiKey` | Default API key for Waii app                               | `""`                         | String  |
| `secrets.waiiSecrets.data.openaiApiKey`      | API key for OpenAI LLM models                              | `""`                         | String  |

## Waii Image tag and digest
Set only one of two: (`image.tag`, `image.digest`). Run with either of the following:

- `--set image.tag="my-image-tag"`
- `--set image.digest="my-image-digest"`

If both are set, preference is given to `image.digest`. By default, the chart uses image tag.

## RDS Configuration

### Using RDS Parameter Overrides

This method is particularly helpful when RDS parameters contain special characters, which can result in corrupted RDS URLs.

When `env.rds.override` is set to `true`, the following RDS parameters can be configured:

| Parameter                    | Description                       | Default      |
|------------------------------|-----------------------------------|--------------|
| `env.rds.host`               | RDS host                          | `localhost`  |
| `env.rds.port`               | RDS port                          | `5432`       |
| `env.rds.user`               | RDS username                      | `postgres`   |
| `env.rds.password.secret`    | Secret name for RDS password      | `rds-secret` |
| `env.rds.password.hardcoded` | Hardcoded RDS password (fallback) | `mypassword` |
| `env.rds.database`           | RDS database name                 | `database`   |

### Using RDS URL

When `env.rds.override` is false, `secrets.rdsUrl.data.rdsUrl` can be populated with the RDS URL to be used.
Example: "postgresql://waii:password@127.0.0.1:5432/test"

If your RDS instance requires SSL certificates (verify-ca mode), modify the RDS URL with the SSL cert paths and mount those paths in the pod as volumes from a pre-existing secret (not managed by Waii chart).

### Standalone Ephemeral Postgres Mode (NOT FOR PROD)

If none of the RDS configurations are provided, a standalone ephemeral postgres database server gets deployed in the Waii container itself. This requires running the pod with elevated permissions.
The data in this standalone postgres cluster is temporary and attached to the lifecycle of the pod.

## LLM Configuration

You can configure the LLMs through one of the following:

### LLM Endpoint Config File (Recommended)

If using the in-built file in the chart, make sure your OpenAI API key has access to `gpt-4o` chat, `gpt-4` chat-reasoning and `text-embedding-ada-002` embedding model.
You can use the inbuilt file in the chart by specifying your OpenAI API key at `files.llmEndpointConfig.apiKey`, or you can provide a new custom file:

```Bash
helm install waii-release oci://registry-1.docker.io/waiilabs/waii --version 1.0.0 \
--set-file files.llmEndpointConfig.rawContent="/path/to/llm_endpoint_config/file" \
--create-namespace -n release-ns
```

If you wish to use LLMs other than OpenAI models, then LLM endpoint config file is the only way.

### OpenAI API Key (Deprecated) (NOT FOR PROD)

You can directly provide the OpenAI API key using `secrets.waiiSecrets.data.openaiApiKey`. With this method, you are restricted to OpenAI models only. There is no way to customise model endpoints and define access policies on models. Instead, using an LLM endpoint config file is recommended.
Make sure you have access to `gpt-4o` chat, `gpt-4` chat-reasoning (optional) and `text-embedding-ada-002` embedding model.

#### Use any one method; if both are specified, OpenAI API key is preferred because of legacy reasons. But the recommended way is LLM endpoint config file.

## Waii Secrets

### Docker Secrets

You can override the docker secrets using `dockerRegistry.username` and `dockerRegistry.password`.

### Waii API Key

You can provide the default Waii API key at `secrets.waiiSecrets.data.waiiDefaultApiKey`. Contact Waii for details. If running in non-authenticated mode, this is not required.

## Authentication

To start Waii app in non-authenticated mode for quick tests and demos, override the `containerArgs` to `null`.

# Quick start

Use the quick start command to run the Waii app in standalone ephemeral postgres mode. This SHOULD NOT be used in prod.

## Known issue

When running in standalone ephemeral postgres mode (not using external RDS), the container uses `sudo` (root user) to start a postgres cluster inside the container. Hence, elevated container permissions are needed.
This is STRICTLY NOT RECOMMENDED in prod. When providing an RDS URL via secrets or RDS env overrides, the default security settings in the chart should be used instead.

Here is the command (NOT FOR PROD) to run in standalone ephemeral postgres mode with elevated permissions:

```Bash
helm install waii-release . \
--set dockerRegistry.username="get_it_from_waii" \
--set dockerRegistry.password="get_it_from_waii" \
--set containerArgs=null \
--set files.llmEndpointConfig.apiKey="${OPENAI_API_KEY}" \
--set podSecurityContext.runAsNonRoot=false \
--set podSecurityContext.runAsUser=0 \
--set podSecurityContext.runAsGroup=0 \
--set podSecurityContext.fsGroup=0 \
--set podSecurityContext.seccompProfile.type=Unconfined \
--set securityContext.privileged=true \
--set securityContext.allowPrivilegeEscalation=true \
--set securityContext.readOnlyRootFilesystem=false \
--set securityContext.runAsUser=0 \
--set securityContext.capabilities.add={"ALL"} \
--set securityContext.capabilities.drop=null \
--create-namespace -n release-ns
```

If you need to override the image (say to an arm image, or another version), refer the configuration section.

# Upgrading

To upgrade the chart with the release name `waii-release`:

```Bash
helm upgrade waii-release oci://registry-1.docker.io/waiilabs/waii --version 1.0.0 -n release-ns
```

# Uninstalling

The "pre-install" hooks are intentionally marked to not get uninstalled. To uninstall the remaining chart:

```Bash
helm uninstall waii-release -n release-ns
```
