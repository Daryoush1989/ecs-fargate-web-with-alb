# Full Cleanup Runbook

Goal: remove the billable lab resources, including the ALB, NAT Gateway, ECS tasks, and VPC resources.

## Order Matters

### 1. ECS Service

- ECS -> Clusters -> `acme-dev-ecs-cluster` -> Services -> `acme-dev-ecs-svc-web` -> Delete
- Wait for the service tasks to stop and for task ENIs to be released.

### 2. Load Balancing

- EC2 -> Load Balancers -> `acme-dev-ecs-alb-public` -> Delete
- EC2 -> Target Groups -> `acme-dev-ecs-tg-http` -> Delete

### 3. NAT Gateway and Elastic IP

- VPC -> NAT gateways -> `acme-dev-network-nat-euw2a` -> Delete
- VPC -> Elastic IPs -> release the NAT Elastic IP after it disassociates

### 4. Internet Gateway

- VPC -> Internet gateways -> `acme-dev-network-igw`
- Detach it from the VPC.
- Delete it.

### 5. Subnets

- VPC -> Subnets -> delete all four lab subnets.

### 6. Route Tables

Delete the non-main route tables:

- `acme-dev-network-rtb-public`
- `acme-dev-network-rtb-private-euw2a`
- `acme-dev-network-rtb-private-euw2b`

### 7. VPC

- VPC -> Your VPCs -> `acme-dev-network-vpc` -> Delete

## If You Get "Resource In Use"

Check VPC -> Network Interfaces. A NAT Gateway, ALB, or ECS task ENI may still exist because a parent resource is still deleting. Wait a few minutes and retry after the dependent resource is gone.
