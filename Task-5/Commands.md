# Task-5 Commands

Commands used or directly represented by the Task-5 implementation evidence. Replace placeholders before use and never commit credentials.

## AWS CLI Setup

```bash
export AWS_REGION=ap-south-1
export AWS_PAGER=""
```

```bash
aws sts get-caller-identity
aws configure get region
```

## EC2 and Systems Manager

Start a Session Manager shell:

```bash
aws ssm start-session \
  --target <INSTANCE-ID> \
  --region "$AWS_REGION"
```

Validate the application from the EC2 host:

```bash
curl http://localhost:3000/health
```

Expected result:

```json
{"status":"ok","database":"connected"}
```

## RDS Connectivity

Check that PostgreSQL is reachable from the application tier:

```bash
timeout 5 bash -c '</dev/tcp/<RDS-ENDPOINT>/5432' \
  && echo open \
  || echo blocked
```

Run a SQL validation query:

```bash
PGPASSWORD='<DB-PASSWORD>' psql \
  -h <RDS-ENDPOINT> \
  -U appuser \
  -d shop \
  -c "select 1"
```

## Docker and ECR

Authenticate to the private ECR registry:

```bash
aws ecr get-login-password --region "$AWS_REGION" \
  | docker login --username AWS \
  --password-stdin 584860296694.dkr.ecr.ap-south-1.amazonaws.com
```

Build, tag, and push the application image:

```bash
docker build --platform linux/amd64 -t ecom-app .
docker tag ecom-app:latest \
  584860296694.dkr.ecr.ap-south-1.amazonaws.com/ecom-app:latest
docker push \
  584860296694.dkr.ecr.ap-south-1.amazonaws.com/ecom-app:latest
```

Verify the image:

```bash
aws ecr describe-images \
  --repository-name ecom-app \
  --region "$AWS_REGION"
```

## ECS

Inspect the EC2-backed cluster:

```bash
aws ecs describe-clusters \
  --clusters ecom-ec2-cluster \
  --region "$AWS_REGION"
```

Inspect the service:

```bash
aws ecs describe-services \
  --cluster ecom-ec2-cluster \
  --services ecom-service \
  --region "$AWS_REGION"
```

List running tasks:

```bash
aws ecs list-tasks \
  --cluster ecom-ec2-cluster \
  --service-name ecom-service \
  --desired-status RUNNING \
  --region "$AWS_REGION"
```

Inspect the task definition:

```bash
aws ecs describe-task-definition \
  --task-definition ecom-task-ec2:2 \
  --region "$AWS_REGION"
```

## Load Balancer Validation

Validate the final ALB health endpoint:

```bash
curl http://ecom-alb-1812614674.ap-south-1.elb.amazonaws.com/health
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
  --resource-ids service/ecom-ec2-cluster/ecom-service \
  --region "$AWS_REGION"
```

```bash
aws application-autoscaling describe-scaling-policies \
  --service-namespace ecs \
  --resource-id service/ecom-ec2-cluster/ecom-service \
  --region "$AWS_REGION"
```

## Troubleshooting

Review ECS service events:

```bash
aws ecs describe-services \
  --cluster ecom-ec2-cluster \
  --services ecom-service \
  --query 'services[0].events[:10]' \
  --region "$AWS_REGION"
```

Inspect a task:

```bash
aws ecs describe-tasks \
  --cluster ecom-ec2-cluster \
  --tasks <TASK-ARN> \
  --region "$AWS_REGION"
```

Review CloudWatch logs when the `awslogs` driver is configured:

```bash
aws logs tail <LOG-GROUP> \
  --since 30m \
  --follow \
  --region "$AWS_REGION"
```
