# Serverless solution for monitoring websites and SSL Certs


## Intro
The key components are:
* [Blackbox exporter](https://github.com/prometheus/blackbox_exporter): websites probing tool
* [Prometheus](https://prometheus.io/): metrics storage (stateless)
* [Grafana](https://grafana.com/): dashboard visualization

Services are executed in [AWS Fargate](https://aws.amazon.com/fargate) container serverless platform and Grafana are backed by [AWS Aurora Serverless](https://aws.amazon.com/rds/aurora) database.

Database and Grafana Admin passwords are randomly generated and stored in [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)

[AWS Copilot](https://aws.github.io/copilot-cli/) creates a default environment and setup AWS CloudMap for services Service Discovery.

## Run in local (for development)

```
$ docker-compose up -d --build
```

## First setup in AWS account

Make sure you have AWS Copilot CLI and AWS CLI installed and configured in your system.

This creates environment resources (VPC, ECS Cluster, IAM Roles, Service Discovery,...):
```
$ copilot app init monitoring
$ copilot env init --name test --profile default --default-config
```
This creates the resources for the services:

```
copilot svc init --name blackbox-exporter --svc-type "Backend Service" \
                 --dockerfile ./blackbox-exporter/Dockerfile
                 
copilot svc init --name prometheus --svc-type "Backend Service" \
                 --dockerfile ./prometheus/Dockerfile
                 
copilot svc init --name grafana --svc-type "Load Balanced Web Service" \
                 --dockerfile ./grafana/Dockerfile --port 3000
```

## Deploy services

```
$ copilot svc deploy --name blackbox-exporter --env test
$ copilot svc deploy --name prometheus --env test
$ copilot svc deploy --name grafana --env test
```

After that you will get the grafana URL.

You can retrieve the grafana admin auto-generated password from SecretsManager executing this:

```
aws secretsmanager get-secret-value --secret-id monitoring-test-grafana-AdminPassword --query 'SecretString'
```

## Disclaimer

Deploying this in your AWS account will increase your AWS monthly bill (depends on your usage). Use are your own responsability.