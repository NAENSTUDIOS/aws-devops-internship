# Task-5 Commands

These commands follow the Task-5 implementation guide. Replace placeholders with values from the AWS account. Do not place credentials, passwords, access keys, or JWT secrets in this file.

## AWS CLI Setup

```bash
export AWS_REGION=ap-south-1
export AWS_PAGER=""
```

Confirm the active identity and region:

```bash
aws sts get-caller-identity
aws configure get region
```

## Systems Manager and EC2

Start a Session Manager shell:

```bash
aws ssm start-session --target <instance-id> --region "$AWS_REGION"
```

Validate the local application from the EC2 instance:

```bash
curl http://localhost:3000/health
```

Check that the RDS endpoint is reachable on PostgreSQL port `5432`:

```bash
timeout 5 bash -c '</dev/tcp/<RDS-ENDPOINT>/5432' \
  && echo open \
  || echo blocked
```

## Linux Application Preparation

```bash
sudo dnf update -y
sudo dnf install -y git docker postgresql15 postgresql15-server
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo dnf install -y nodejs
sudo systemctl enable --now docker
sudo usermod -aG docker ec2-user
sudo postgresql-setup --initdb
sudo systemctl enable --now postgresql
```

Clone and install the application:

```bash
git clone https://github.com/meysamraza/E-com-site.git app
cd app
npm install
```

Initialize the database and start the application service according to the task configuration:

```bash
npm run db:init
sudo systemctl daemon-reload
sudo systemctl enable --now shop
```

## RDS Validation

Run the database connectivity query from the application host:

```bash
PGPASSWORD='<DB-PASSWORD>' psql \
  -h <RDS-ENDPOINT> \
  -U appuser \
  -d shop \
  -c "select 1"
```

Expected result:

```text
1
```

## Docker and ECR

Authenticate Docker to the private ECR registry:

```bash
aws ecr get-login-password --region "$AWS_REGION" \
  | docker login --username AWS \
  --password-stdin <ACCOUNT-ID>.dkr.ecr.$AWS_REGION.amazonaws.com
```

Build, tag, and push the image:

```bash
docker build --platform linux/amd64 -t <REPOSITORY-NAME> .
docker tag <REPOSITORY-NAME>:latest \
  <ACCOUNT-ID>.dkr.ecr.$AWS_REGION.amazonaws.com/<REPOSITORY-NAME>:latest
docker push \
  <ACCOUNT-ID>.dkr.ecr.$AWS_REGION.amazonaws.com/<REPOSITORY-NAME>:latest
```

Verify the image:

```bash
aws ecr describe-images \
  --repository-name <REPOSITORY-NAME> \
  --region "$AWS_REGION"
```

## ECS

List clusters:

```bash
aws ecs list-clusters --region "$AWS_REGION"
```

Inspect the service:

```bash
aws ecs describe-services \
  --cluster <ECS-CLUSTER> \
  --services <ECS-SERVICE> \
  --region "$AWS_REGION"
```

List running tasks:

```bash
aws ecs list-tasks \
  --cluster <ECS-CLUSTER> \
  --service-name <ECS-SERVICE> \
  --desired-status RUNNING \
  --region "$AWS_REGION"
```

## Load Balancer Validation

Check the production ALB endpoint:

```bash
curl http://prod-alb-1957656459.ap-south-1.elb.amazonaws.com/health
```

Inspect target health:

```bash
aws elbv2 describe-target-health \
  --target-group-arn <TARGET-GROUP-ARN> \
  --region "$AWS_REGION"
```

## Auto Scaling Validation

```bash
aws application-autoscaling describe-scalable-targets \
  --service-namespace ecs \
  --region "$AWS_REGION"
```

```bash
aws application-autoscaling describe-scaling-policies \
  --service-namespace ecs \
  --region "$AWS_REGION"
```

## Troubleshooting

Review ECS service events:

```bash
aws ecs describe-services \
  --cluster <ECS-CLUSTER> \
  --services <ECS-SERVICE> \
  --query 'services[0].events[:10]' \
  --region "$AWS_REGION"
```

Inspect an ECS task:

```bash
aws ecs describe-tasks \
  --cluster <ECS-CLUSTER> \
  --tasks <TASK-ARN> \
  --region "$AWS_REGION"
```

Review application logs when the `awslogs` driver is configured:

```bash
aws logs tail <LOG-GROUP> \
  --since 30m \
  --follow \
  --region "$AWS_REGION"
```

## Security Checks

Confirm the RDS instance is not publicly accessible:

```bash
aws rds describe-db-instances \
  --db-instance-identifier <DB-IDENTIFIER> \
  --query 'DBInstances[0].PubliclyAccessible' \
  --region "$AWS_REGION"
```

Confirm the ECS service task counts:

```bash
aws ecs describe-services \
  --cluster <ECS-CLUSTER> \
  --services <ECS-SERVICE> \
  --query 'services[0].{desired:desiredCount,running:runningCount,pending:pendingCount}' \
  --region "$AWS_REGION"
```
