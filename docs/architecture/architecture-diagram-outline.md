# Architecture Diagram Outline

## Diagram goal
Show a production-style “private compute behind public ALB” pattern:
- ALB is public and internet-facing
- ECS tasks are private
- NAT provides egress for private subnets
- Security groups restrict ingress to tasks

## Include these components
### AWS Region
- eu-west-2 (London)

### VPC
- VPC: acme-dev-network-vpc (10.10.0.0/16)

### Availability Zones
- eu-west-2a
- eu-west-2b

### Subnets
Public:
- 10.10.0.0/24 (euw2a)
- 10.10.1.0/24 (euw2b)

Private:
- 10.10.10.0/24 (euw2a)
- 10.10.11.0/24 (euw2b)

### Gateways + routing
- Internet Gateway attached to VPC
- NAT Gateway in public subnet (euw2a) with Elastic IP
- Public route table:
  - 0.0.0.0/0 → IGW
- Private route tables:
  - 0.0.0.0/0 → NAT Gateway

### Load Balancer
- ALB: internet-facing
- Listener: HTTP:80
- Target group: IP targets (Fargate)

### ECS
- ECS Cluster (Fargate)
- ECS Service spans both private subnets
- Tasks have no public IP

### Security groups
- ALB SG: inbound 80 from 0.0.0.0/0
- Tasks SG: inbound 80 only from ALB SG

### Observability
- CloudWatch Logs group for container logs

## Arrows / flows to show
1. Internet → ALB (HTTP:80)
2. ALB → ECS tasks (HTTP:80) (SG-to-SG)
3. ECS tasks → NAT → Internet (pull images / logs)

## Notes box
- Cost-saving: single NAT gateway
- Production: NAT per AZ recommended
