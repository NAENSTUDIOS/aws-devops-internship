# Task-1 — AWS Fundamentals and Linux Administration

## What I worked on

I used this task to build the operational foundation for the rest of the mentorship. I learned how AWS Regions and Availability Zones are organised, how the shared responsibility model applies to cloud infrastructure, and how to navigate the AWS Console without treating the console as a substitute for understanding the underlying resources.

I also practised administering Linux systems from the command line. This included moving around the filesystem, creating and inspecting files, managing permissions and users, installing packages, controlling services, and checking logs and network connections.

## What I used

- AWS Management Console
- AWS Regions and Availability Zones
- Linux shell
- `sudo` and user administration
- `systemctl` for service management
- `ping` and `curl` for reachability and HTTP checks
- `netstat` and `ss` for socket and connection inspection
- Basic file commands such as `pwd`, `ls`, `cd`, `mkdir`, `cp`, `mv`, `rm`, `cat`, and `nano`

## Commands I practised

```bash
pwd
ls -la
cd
mkdir
cp
mv
rm
cat
nano
sudo su -
systemctl status
systemctl start
systemctl stop
systemctl restart
ping <host>
curl http://<host>
netstat
ss
```

## What I learned

- I learned to verify the state of a service instead of assuming that it is running.
- I learned how basic Linux file, user, permission, and process operations support cloud administration.
- I learned to separate an application problem from a network reachability problem using simple command-line checks.
- I learned why Regions and Availability Zones matter when planning availability and disaster recovery.
- I built the command-line habits that I later used for EC2, ECS, RDS, and load-balancer troubleshooting.

## Security and operations lessons

This task showed me that cloud security starts with small operational decisions. I need to know which user is executing a command, which service is listening, which network path is reachable, and which responsibility belongs to AWS versus me. These checks became the basis for the IAM, VPC, security-group, and container work in later tasks.

## Evidence included

- [`evidence/ips.txt`](evidence/ips.txt) — recorded staging and production host addresses from the exercise.
- [`evidence/singapore-dr-estimate.pdf`](evidence/singapore-dr-estimate.pdf) — Singapore disaster-recovery cost estimate.

The evidence files are retained for portfolio context. They are not credentials and should not be used as a replacement for DNS, IAM, security-group, or service-health validation.

## Status

Task 1 is documented from the learning activities and evidence retained in this repository. Console and terminal screenshots can be added later if they become available.
