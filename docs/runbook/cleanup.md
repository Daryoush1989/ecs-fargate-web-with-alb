
```md
# Full Cleanup Runbook (Delete Everything)

Goal: remove all billable resources (ALB, NAT, ECS tasks) and then delete the VPC.

## Order matters

### 1) ECS: delete service (stops tasks + releases ENIs)
- ECS → Clusters → acme-dev-ecs-cluster → Services → acme-dev-ecs-svc-web → Delete

### 2) Load Balancing
- EC2 → Load Balancers → acme-dev-ecs-alb-public → Delete
- EC2 → Target Groups → acme-dev-ecs-tg-http → Delete

### 3) NAT + Elastic IP (major cost)
- VPC → NAT gateways → acme-dev-network-nat-euw2a → Delete
- VPC → Elastic IPs → Release the NAT EIP (after it disassociates)

### 4) Internet Gateway
- VPC → Internet gateways → acme-dev-network-igw
  - Detach from VPC
  - Delete

### 5) Subnets
- VPC → Subnets → delete all 4 subnets

### 6) Route tables (non-main)
- VPC → Route tables → delete:
  - acme-dev-network-rtb-public
  - acme-dev-network-rtb-private-euw2a
  - acme-dev-network-rtb-private-euw2b

### 7) VPC last
- VPC → Your VPCs → acme-dev-network-vpc → Delete

## If you get “resource in use”
Check VPC → Network Interfaces (ENIs). Usually a NAT/ALB/task ENI still exists because a parent resource is still deleting.
