# waii-helm-chart
Helm chart of Waii

## Install

In order to pull the Docker image, you need to setup the Docker pull secret:

```bash
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ \
  --docker-username=waiilabs \
  --docker-password=<Ask Waii Team for the password>
```

Then, you can install the Helm chart:

```bash
cd chart/
helm upgrade --install waii . --set env.openaiApiKey=$OPENAI_API_KEY \
    --set env.rdsUrl="postgresql://<url to your PG instance>/<db name for waii>" \
```

Please note that `rdsUrl` should include the database name, for example:
`postgresql://waii:password@localhost:5432/test`

## Access the service 

