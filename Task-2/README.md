# Designing the AWS Infrastructure Foundation

## What I built

I created the AWS foundation required to run an application safely. I worked with a VPC, public and private subnets, route tables, an Internet Gateway, IAM access, security groups, and an EC2 instance. I also used the application load-balancer endpoint provided for the environment to validate the public application path.

## Infrastructure I configured

- A VPC with a defined CIDR range and DNS support.
- Public and private subnet placement for separating internet-facing and internal resources.
- Internet Gateway and route-table associations for controlled connectivity.
- Security-group rules for HTTP `80`, HTTPS `443`, SSH `22`, and application traffic.
- IAM users, roles, and policies with least-privilege access as the design goal.
- An Amazon Linux EC2 instance associated with the required security group.
- An Application Load Balancer endpoint:

```text
http://prod-alb-1957656459.ap-south-1.elb.amazonaws.com
```

## Architecture I worked with

```text
Internet user
      |
      v
Application Load Balancer
      |
      v
VPC
  +---+---+
  |       |
Public  Private
Subnet  Subnet
  |
  v
EC2 instance
```

## Validation I performed

```bash
ping <host>
curl http://prod-alb-1957656459.ap-south-1.elb.amazonaws.com
ssh <user>@<host>
```

I used these checks to confirm that the host was reachable, that the public HTTP path responded, and that I could administer the EC2 instance through the intended access method.

## What I learned

- I learned how subnet placement changes the exposure and routing of an AWS resource.
- I learned that route tables and security groups solve different problems: routes determine where traffic can go, while security groups control which traffic is allowed.
- I learned to treat IAM permissions as part of the infrastructure design rather than as an afterthought.
- I learned how an EC2 instance fits into a larger VPC and load-balancer path.
- I learned that connectivity should be tested from the actual workload environment, not only from my laptop.

## Security decisions

I separated public and private network areas, restricted administrative access through security-group rules, and used IAM roles and policies instead of embedding access keys in the application. This task gave me the networking and identity foundation used by the ECS and RDS deployments later in the repository.

## Status

Task 2 is documented from the infrastructure configuration, endpoint, and validation details retained for this environment. Additional console screenshots can be added when available.
