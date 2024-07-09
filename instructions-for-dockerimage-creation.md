# FastCVE Deployment Guide

## Environment Configuration

Ensure the environment-specific configuration files are present at the following locations:
- `config/setenv`
- `src/config/setenv`

### Export Environment Variables

Before starting the FastCVE container, export the relevant environment variables. Below is an example for the `qa-new` environment:

```bash
export FCDB_NAME=vuln_db
export FCDB_PASS=<scqp_rds_db_password>
export FCDB_USER=<scqp_rds_db_username>
export INP_ENV_NAME=qa-new
export POSTGRES_PASSWORD=password
export DOCKER_TAG=qa-new

export FCDB_HOST=<env_pgbouncer_IP>
export FCDB_PORT=5432
export FCDB_REMOTE_NAME=<db_name_for_env_rds_configured_in_pgbouncer>

export FCDB_WEB_PARAMS='--host 0.0.0.0 --port 8000 --workers 4'
```

### Build and Push Docker Image to ECR
#### Build Docker Image

```bash
docker compose build --no-cache
```

#### Authenticate Docker with AWS ECR

```bash
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 091235034633.dkr.ecr.us-east-2.amazonaws.com
```

#### Tag and Push the Docker Image
```bash
docker tag fastcve:$DOCKER_TAG 091235034633.dkr.ecr.us-east-2.amazonaws.com/fastcve:$DOCKER_TAG
docker push 091235034633.dkr.ecr.us-east-2.amazonaws.com/fastcve:$DOCKER_TAG
```

### Local Testing of the Built Image
#### Start the FastCVE Container
```bash
docker compose up
```

#### Verify if the API is Running
From another terminal, execute:
```bash
curl -vi http://localhost:8000/api/search/cve
```
#### Access the Container and Check Logs
```bash
docker exec -it fastcve /bin/sh
```
Inside the container, you can inspect the logs and troubleshoot any issues.

### Testing the Image on ECS

#### Run the Task on ECS
```bash
aws ecs run-task \
    --cluster <cluster-name>  \
    --task-definition dev-bonsai-fastcve:27 \
    --network-configuration "awsvpcConfiguration={subnets=['<subnet>'],securityGroups=['<sg>'],assignPublicIp=DISABLED}" \
    --enable-execute-command \
    --launch-type FARGATE \
    --region us-east-2
```

#### Execute Commands on the Running Task
```bash
aws ecs execute-command  \
    --region us-east-2 \
    --cluster <cluster-name> \
    --task <task_id> \
    --container fastcve \
    --command "/bin/bash" \
    --interactive
```

#### Update and Test API
Inside the container:
```bash
apk update
apk upgrade
apk add curl 

curl -vi http://localhost:8000/api/search/cve
```

## Important Notes
- Ensure all environment-specific variables are correctly set before starting the container.
- Verify the container is running and the API is functional by using curl.
- For any issues, access the container and inspect the logs to troubleshoot.