# ECS Fargate Web App Behind an Application Load Balancer

## Overview

This project documents a console-built AWS lab for running a containerized web application on Amazon ECS Fargate in private subnets and exposing it through a public Application Load Balancer.

It is a beginner-friendly but technically realistic AWS networking and container platform project focused on VPC design, security groups, load balancing, ECS service configuration, and cleanup planning.

## Business Problem

Cloud engineers often need to run web workloads without exposing compute resources directly to the internet. This project documents the common AWS pattern of keeping application tasks private while allowing controlled public access through an ALB.

## Architecture

Traffic flow:

```text
Internet
  -> Public Application Load Balancer
  -> ECS Fargate tasks in private subnets
  -> NAT Gateway for outbound internet access
  -> CloudWatch Logs for container logging
```

Key design points:

- ECS tasks have no public IP addresses.
- The ALB is the only public entry point.
- The task security group accepts inbound traffic only from the ALB security group.
- Public and private subnets span two availability zones.
- One NAT Gateway is used for cost control in the lab design.

## AWS Services Used

- Amazon ECS
- AWS Fargate
- Elastic Load Balancing / Application Load Balancer
- Amazon VPC
- Public and private subnets
- Internet Gateway
- NAT Gateway
- Amazon CloudWatch Logs
- AWS IAM
- Amazon EC2 security groups

## Tools Used

- AWS Management Console
- Markdown documentation
- Mermaid architecture diagram
- Git and GitHub

## Security Features

- Private ECS tasks with public IP disabled
- ALB security group as the controlled ingress point
- Task security group restricted to ALB-originated traffic
- Private subnet placement for application tasks
- IAM task execution role for ECS image pulls and logging

## Deployment Summary

The README and supporting docs describe a console-built AWS lab. The project does not currently include Terraform, CDK, or application source code. It is best presented as a hands-on AWS Console networking and ECS deployment project.

No AWS deployment commands were run during this README refresh.

## Testing and Validation

Validation includes:

- Confirming ECS tasks reach `RUNNING`
- Confirming target group health checks pass
- Opening the ALB DNS name, represented as `http://<domain-name>` or `http://<alb-dns-name>`
- Reviewing ECS service events for task placement or image pull issues
- Reviewing CloudWatch Logs for container output

## Evidence / Screenshots

Architecture documentation is stored under `docs/architecture`, and cleanup guidance is stored under `docs/runbook`. The current documentation uses example CIDR ranges and placeholder ALB DNS values rather than real account identifiers.

## Cost Control

This lab can incur cost from the ALB, NAT Gateway, Fargate tasks, CloudWatch Logs, and Elastic IPs. The single NAT Gateway design is cost-aware but less resilient than one NAT Gateway per availability zone.

## Cleanup

Follow `docs/runbook/cleanup.md` after testing. Cleanup should remove the ECS service and task definition, ALB and target group, NAT Gateway, Elastic IP, route tables, subnets, security groups, CloudWatch log groups, and IAM roles created for the lab.

## Lessons Learned

- Public ALBs and private ECS tasks are a strong baseline for containerized web workloads.
- Security groups should express trust boundaries between layers.
- NAT Gateway design is a balance between cost and resilience.
- Console-built labs are useful for fundamentals, but IaC is better for repeatability.

## Future Improvements

- Add Terraform or CDK for repeatable deployment.
- Add HTTPS with ACM and HTTP-to-HTTPS redirect.
- Add ECR-based application image deployment.
- Add GitHub Actions CI/CD.
- Add VPC endpoints for ECR and CloudWatch Logs to reduce NAT dependency.
- Add CloudWatch alarms and dashboard evidence.
