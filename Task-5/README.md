# Task-5 — Production Environment Build in Mumbai

## Objective

Build a production-style AWS environment for the e-commerce application in **Asia Pacific (Mumbai), `ap-south-1`**. The design combines network segmentation, private application and database tiers, managed identity, EC2-based application preparation, an Application Load Balancer, Amazon RDS PostgreSQL, Amazon ECR, Amazon ECS, and service auto scaling.

## Task Scope

Task 5 is the transition from a working cloud deployment to a repeatable production environment built from the network layer upward:

1. Create the VPC, subnets, gateways, and route tables.
2. Define separate security groups for the load balancer, application, and database.
3. Use Systems Manager instead of SSH key-pair access for EC2 administration.
4. Prepare the application on EC2 and create a reusable AMI.
5. Place two application instances behind an internet-facing ALB.
6. Move the database to a private, Multi-AZ RDS PostgreSQL deployment.
7. Publish the container image to a private ECR repository.
8. Run the application on ECS with EC2 capacity.
9. Configure ECS service auto scaling between two and four tasks.

The implementation results and screenshots will be added to this directory as evidence becomes available.

## Target Architecture

```text
Internet
    |
    v
Application Load Balancer
    |
    v
Public subnets
    |
    v
Private application subnets
    |
  +---+-------------------+
  |                       |
EC2 application        ECS service
preparation            on EC2 capacity
  |                       |
  +-----------+-----------+
              |
              v
       Private RDS subnets
              |
              v
       Multi-AZ PostgreSQL

Docker image -> Private ECR -> ECS task definition

RDS backups -> Singapore replication destination
```

## Network Design

The assignment specifies a VPC with:

- CIDR range: `10.0.0.0/16`
- Two Availability Zones
- Two public subnets
- Four private subnets
- Two NAT gateways, one per Availability Zone
- DNS hostnames and DNS resolution enabled
- No VPC endpoints in this task design

Subnet allocation:

| Tier | CIDR |
|---|---|
| Public subnet 1 | `10.0.0.0/24` |
| Public subnet 2 | `10.0.1.0/24` |
| Application subnet 1 | `10.0.2.0/24` |
| Application subnet 2 | `10.0.3.0/24` |
| Database subnet 1 | `10.0.4.0/24` |
| Database subnet 2 | `10.0.5.0/24` |

## Security Model

Three security groups are used to keep traffic directional:

| Security group | Allowed traffic |
|---|---|
| ALB security group | HTTP `80` from the internet |
| Application security group | TCP `3000` and ephemeral response ports from the ALB security group |
| Database security group | PostgreSQL `5432` from the application security group |

This creates an explicit trust path:

```text
Internet -> ALB -> Application -> RDS
```

The database is not intended to receive direct public traffic. EC2 administration is performed through AWS Systems Manager Session Manager, avoiding an inbound SSH requirement and avoiding a key pair for the instances.

## Identity and Access

The task uses separate roles for separate responsibilities:

- **EC2 preparation role:** trusted by EC2 and attached to `AmazonSSMManagedInstanceCore`.
- **ECS container-instance role:** trusted by EC2 and attached to the ECS container-instance policy and Systems Manager policy.
- **ECS task execution role:** trusted by `ecs-tasks.amazonaws.com` and attached to `AmazonECSTaskExecutionRolePolicy`.
- **ECR publishing identity:** used only for image publication and removed after the push, according to the task instructions.

Long-lived access keys, passwords, and JWT secrets must not be committed to this repository or included in screenshots.

## EC2 Application Preparation

The first Amazon Linux 2023 instance is created in an application subnet without a public IP address. It is accessed through Session Manager and prepared with:

- Git
- Docker
- Node.js 20
- PostgreSQL client and server packages
- A local application checkout
- A systemd service for the shop application

The application is configured to listen on port `3000`. The initial health check must report database connectivity before the instance is converted into an AMI.

## AMI and Application Redundancy

After the first EC2 application instance is validated:

1. Create an AMI from the configured instance.
2. Wait until the AMI is available.
3. Launch a second instance from that AMI in the other application subnet and Availability Zone.
4. Validate the second instance through Session Manager.
5. Confirm the application health endpoint responds locally on both instances.

This provides an auditable progression from one configured host to a repeatable application image.

## Application Load Balancer

The load-balancing layer uses:

- Internet-facing scheme
- Public subnets in both Availability Zones
- ALB security group
- HTTP listener on port `80`
- Target group protocol and port: HTTP `3000`
- Health check path: `/health`
- Both EC2 application instances registered as targets

The supplied production ALB endpoint from the earlier infrastructure work is:

```text
http://prod-alb-1957656459.ap-south-1.elb.amazonaws.com
```

The Task-5 proof will confirm the final target group and application response for this task.

## RDS PostgreSQL

The database design is:

- PostgreSQL 15
- Production template
- Multi-AZ DB instance
- Instance class: `db.t4g.micro`
- Storage: `20 GiB` gp3
- Storage autoscaling enabled
- Private DB subnet group using only the two database subnets
- Public access disabled
- Database security group only
- Initial database: `shop`
- Master user: `appuser`
- Automated backup retention: `7` days
- Backup replication destination: Singapore

The application connects to RDS on port `5432`. Connectivity is validated from the private application tier before the ECS deployment proceeds.

## Container Registry and ECS

The Docker image is published to a private ECR repository in `ap-south-1`. The ECS cluster uses:

- EC2 launch type
- On-demand EC2 capacity
- Amazon Linux 2023 container instances
- `t3.small` capacity instances
- Desired capacity range of two to four instances
- Application subnets
- Application security group

The task definition uses bridge networking, maps container port `3000` to a dynamic host port, loads configuration through environment variables, and writes container logs to CloudWatch Logs.

The ECS service is configured as a replica service with:

- Desired tasks: `2`
- Minimum healthy percentage: `50`
- Maximum percentage: `100`
- Health-check grace period: `90` seconds
- Rolling deployment
- ALB integration on listener port `80`

## Auto Scaling

ECS service auto scaling is configured with:

- Minimum tasks: `2`
- Maximum tasks: `4`
- Policy: target tracking
- Metric: ECS service average CPU utilization
- Target: `50%`
- Scale-out cooldown: `60` seconds
- Scale-in cooldown: `120` seconds

The final evidence should demonstrate two healthy tasks and the active scaling policy.

## Validation Plan

The following checks are required for completion:

1. VPC resource map shows the expected subnets, gateways, and NAT gateways.
2. EC2 application health endpoint reports `"database":"connected"`.
3. The AMI reaches `Available` state.
4. The ALB target group reports healthy EC2 targets.
5. RDS configuration confirms Multi-AZ, private subnets, and the database security group.
6. RDS connectivity checks return `open` and SQL returns `1`.
7. The private ECR repository contains the pushed image.
8. ECS service shows `2/2` tasks running.
9. ECS targets are healthy behind the ALB.
10. The auto scaling policy is active.
11. The ALB `/health` endpoint reports database connectivity.

## Evidence Status

The Task-5 PDF defines the implementation and required deliverables. The 12 screenshots will be stored in `Task-5/screenshots/` and used to replace any pending evidence references with verified results.

Expected evidence slots:

```text
Task-5/screenshots/
├── 01-vpc-resource-map.png
├── 02-ec2-health-database-connected.png
├── 03-ami-available.png
├── 04-ec2-targets-healthy.png
├── 05-rds-configuration.png
├── 06-rds-connectivity-and-query.png
├── 07-ecr-image.png
├── 08-ecs-service-2-of-2-running.png
├── 09-ecs-targets-healthy.png
├── 10-ecs-autoscaling-policy.png
├── 11-alb-health-database-connected.png
└── 12-submission-details.png
```

## Learning Outcomes

- Designed a two-AZ VPC with separate public, application, and database tiers.
- Applied security-group rules based on service-to-service trust rather than broad network access.
- Used Systems Manager Session Manager for private-instance administration.
- Converted a configured EC2 application host into a reusable AMI.
- Built an ALB-backed, redundant application tier.
- Configured a private, Multi-AZ PostgreSQL database with backup replication.
- Published and consumed a private container image through ECR and ECS.
- Operated ECS on EC2 capacity with rolling deployment and target tracking.
- Validated infrastructure through health checks, SQL, target health, and service state.

## Project Summary

Task 5 is a full production-environment build rather than a single-service exercise. It connects the security boundary, private networking, compute preparation, database availability, image supply chain, container orchestration, and scaling policy into one operating system for the application.

The strongest evidence for this task will be the relationship between the controls: the ALB can reach the application, the application can reach RDS, the ECS service can replace tasks, and the database and backup design reduce the impact of a single failure.

## Suggested Commit Messages

```text
docs(task-5): document production environment design
feat(task-5): build multi-AZ VPC and security boundaries
feat(task-5): prepare EC2 application AMI
feat(task-5): configure ALB and private RDS PostgreSQL
feat(task-5): publish image to private ECR
feat(task-5): deploy ECS service with EC2 capacity
feat(task-5): configure ECS service target tracking
test(task-5): add production validation evidence
```
