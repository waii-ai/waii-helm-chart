# waii-helm-chart
Helm chart of Waii

## Pre-Install

In order to pull the Docker image, you need to setup the Docker pull secret:

```bash
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ \
  --docker-username=waiilabs \
  --docker-password=<Ask Waii Team for the password>
```

## Setup Postgres database

Waii K8s deployment needs a running Postgres database. You should make sure that you create a Postgres database, make sure it can be accessible from the K8s cluster, and get the connection string.

You need to create a database for the Waii service, for example:

```sql
CREATE DATABASE <waii_db>;
```

And enable the pgvector extension:

```sql
CREATE EXTENSION vector;
```

Run sanity check before the next step (Make sure you have psql installed):

```bash
psql postgresql://username:password@host:port/<waii_db>

=> create table test_vector (id serial primary key, vector vector);
=> select * from public.test_vector;
```

You should be able to see something like: 

```bash
 id | vector
----+--------
(0 rows)
```

Then you can move to the next step to install Helm chart. 

IMPORTANT: `postgresql://username:password@host:port/<waii_db>` will be your `rdsUrl` in the next step.

## Install Helm Chart

### Use Azure OpenAI 

If you need to use Azure OpenAI, you need to update `configMap.yaml` file, edit the config (update to use your own Azure OpenAI API Key and endpoint):

```yaml
...
data:
  azure-openai.conf: |
    - name: gpt-3.5-turbo
      endpoints:
        - api_type: azure
          api_key: <YOUR_AZURE_API_KEY>
          api_base: https://<YOUR_AZURE_ENDPOINT>.openai.azure.com/
          deployment_name: gpt-35
    - name: gpt-4
      endpoints:
        - api_key: <YOUR_AZURE_API_KEY>
          api_base: https://<YOUR_AZURE_ENDPOINT>.openai.azure.com/
          deployment_name: gpt-4
```

Install the Helm chart:

```bash
cd chart; helm upgrade --install waii . --set env.rdsUrl="postgresql://.../<db>"
```

Please note that `rdsUrl` should include the database name, for example:
`postgresql://waii:password@localhost:5432/test`

### Use OpenAI

If you need to use OpenAI endpoint, you can directly run this command:

```bash
cd chart; helm upgrade --install waii . --set env.openaiApiKey=$OPENAI_API_KEY --set env.rdsUrl="postgresql://.../<db>"
```

### Check the status

After you run the `helm upgrade ...` command, you should be able to find the following message:

```bash
Release "waii" has been upgraded. Happy Helming!
NAME: waii
LAST DEPLOYED: Mon Feb 26 15:00:20 2024
NAMESPACE: default
STATUS: deployed
REVISION: 14
TEST SUITE: None
```

## Access the service

You can access the service by using the following command:

```bash
kubectl port-forward svc/waii-svc 8080:3456
```

And open your browser to `http://localhost:8080` to access the service. You should be able to see Waii UI.