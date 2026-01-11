# ECS Fargate Web App Behind a Public ALB (VPC + NAT + Private Subnets)

A beginner-friendly, production-style AWS reference project that deploys a containerized web app on **Amazon ECS (Fargate)** in **private subnets**, accessible only through a **public Application Load Balancer (ALB)**. Outbound internet access for the private tasks is provided via a **NAT Gateway**.

This project was built **via the AWS Console** to focus on fundamentals: networking, security groups, load balancing, and ECS service wiring.

---
## Repository structure

```text
.
├── README.md
├── .gitignore
└── docs
    ├── architecture
    │   ├── architecture.mmd
    │   └── architecture-diagram-outline.png
    └── runbook
        └── cleanup.md

## Architecture

**Traffic flow (high level):**
- Internet → **Public ALB** (public subnets)
- ALB → **ECS tasks (Fargate)** (private subnets)
- ECS tasks → outbound internet (image pulls/logs) → **NAT Gateway** (public subnet) → Internet Gateway

**Key properties:**
- ECS tasks have **no public IPs**
- Tasks accept inbound traffic **only from the ALB security group**
- Multi-AZ subnets (2 public + 2 private)

> Note: This project uses **one NAT Gateway** (cost-saving). A production setup typically uses **one NAT per AZ**.

---

## AWS Region

- `eu-west-2` (London)

---

## Naming Conventions

Environment: `dev`  
Company example: `acme`

Core resource names used:
- VPC: `acme-dev-network-vpc`
- Public subnets:
  - `acme-dev-network-subnet-public-euw2a` (`10.10.0.0/24`)
  - `acme-dev-network-subnet-public-euw2b` (`10.10.1.0/24`)
- Private subnets:
  - `acme-dev-network-subnet-private-euw2a` (`10.10.10.0/24`)
  - `acme-dev-network-subnet-private-euw2b` (`10.10.11.0/24`)
- IGW: `acme-dev-network-igw`
- NAT: `acme-dev-network-nat-euw2a`
- Public RT: `acme-dev-network-rtb-public`
- Private RTs:
  - `acme-dev-network-rtb-private-euw2a`
  - `acme-dev-network-rtb-private-euw2b`
- Security groups:
  - ALB SG: `acme-dev-ecs-sg-alb`
  - Tasks SG: `acme-dev-ecs-sg-tasks`
- Target group: `acme-dev-ecs-tg-http`
- ALB: `acme-dev-ecs-alb-public`
- ECS:
  - Cluster: `acme-dev-ecs-cluster`
  - Task definition: `acme-dev-ecs-task-web`
  - Service: `acme-dev-ecs-svc-web`
  - Execution role: `acme-dev-ecs-role-task-execution`

---

## Prerequisites

- An AWS account with permissions for: VPC, EC2 (ALB/Target Groups/Security Groups), ECS, IAM, CloudWatch Logs
- AWS Console access
- (Optional) Git + VS Code if you’re documenting your build in this repo

---

## Deployment Summary (Console)

### 1) Networking (VPC)
- Create VPC `10.10.0.0/16`
- Enable DNS resolution + DNS hostnames
- Create 2 public + 2 private subnets across 2 AZs
- Create/attach Internet Gateway (IGW)
- Create public route table:
  - `0.0.0.0/0 → IGW`
  - Associate both public subnets
- Create NAT Gateway in public subnet (euw2a) with an Elastic IP
- Create private route tables (per AZ) and route:
  - `0.0.0.0/0 → NAT Gateway` (both private RTs point to the single NAT)

### 2) Security Groups
- **ALB SG**: inbound HTTP 80 from `0.0.0.0/0`; outbound allow all
- **Tasks SG**: inbound HTTP 80 from **ALB SG**; outbound allow all

### 3) Load Balancing
- Create target group:
  - Target type: **IP addresses** (required for Fargate)
  - Port: 80, Health check path: `/`
- Create ALB:
  - Internet-facing, IPv4
  - Subnets: both **public** subnets
  - SG: ALB SG
  - Listener: HTTP:80 → forward to target group

### 4) ECS (Fargate)
- Create IAM execution role (attach `AmazonECSTaskExecutionRolePolicy`)
- Create ECS cluster
- Create task definition (Fargate)
  - Container port: 80
  - Logging: CloudWatch (awslogs)
  - Image example: `amazon/amazon-ecs-sample:latest`
- Create ECS service
  - Desired tasks: 2
  - Subnets: both **private** subnets
  - Public IP: **Disabled**
  - SG: Tasks SG
  - Attach to existing ALB + listener + target group

---

## Test the Application

1. Go to **EC2 → Load Balancers → `acme-dev-ecs-alb-public`**
2. Copy the **DNS name**
3. Open in your browser:
   - `http://<ALB_DNS_NAME>`

Expected result: a simple sample application page.

---

## Troubleshooting

### Target group shows “No targets”
- ECS tasks are not in RUNNING state yet. Check ECS service **Events** and task **Stopped reason**.

### CannotPullContainerError / image not found
- Confirm the container image URI is valid (typos are common).
- If tasks are in private subnets, ensure NAT + private route tables are correct.

### Targets are unhealthy
- Confirm:
  - Tasks SG allows inbound **80 from ALB SG**
  - Container listens on port **80**
  - Target group health check path is correct (`/` for sample app)

---

## Cost Notes

This lab can incur charges if left running:
- NAT Gateway (notable hourly cost)
- ALB (hourly + usage)
- ECS tasks (Fargate compute)
- Elastic IP (if allocated and unused)

---

## Cleanup

See: `docs/runbook/cleanup.md`

---

## Improvements (Next Steps)
- Use 2 NAT Gateways (one per AZ) for resilience
- Add HTTPS with ACM certificate + HTTP→HTTPS redirect
- Infrastructure as Code (Terraform/CloudFormation/CDK)
- Add CI/CD pipeline (GitHub Actions)
- PrivateLink/VPC endpoints for ECR/CloudWatch Logs to reduce NAT dependency

---


