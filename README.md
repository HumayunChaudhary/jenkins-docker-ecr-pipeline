# ECS/Fargate Microservices Deployment with Jenkins, ECR, ALB & Cloud Map

## Overview

This project demonstrates a production-style deployment of a containerized full-stack application on AWS using:

- Amazon ECS with AWS Fargate
- Amazon ECR
- Application Load Balancer (ALB)
- AWS Cloud Map Service Discovery
- Jenkins CI/CD Pipeline
- Nginx Reverse Proxy
- Private/Public Subnet Architecture

The frontend and backend applications are containerized using Docker. Jenkins builds and pushes the Docker images to Amazon ECR, after which ECS/Fargate deploys the updated containers.

---

# Architecture

```text
Internet
   ↓
Application Load Balancer (Public Subnets)
   ↓
Frontend ECS/Fargate Service (Private Subnets)
   ↓
Nginx Reverse Proxy
   ↓
backend.ecsworkshop.local
   ↓
AWS Cloud Map Service Discovery
   ↓
Backend ECS/Fargate Service (Private Subnets)
```

---

# Key AWS Services Used

| Service | Purpose |
|---|---|
| Amazon ECS | Container orchestration |
| AWS Fargate | Serverless container runtime |
| Amazon ECR | Docker image registry |
| Application Load Balancer | External traffic distribution |
| AWS Cloud Map | Internal service discovery |
| Amazon VPC | Network isolation |
| NAT Gateway | Outbound internet access for private subnets |
| IAM | Permissions management |
| CloudWatch Logs | Container logging |
| Jenkins | CI/CD automation |

---

# Why Cloud Map Was Used

Backend ECS task IPs are dynamic and may change whenever tasks restart or scale.

AWS Cloud Map provides service discovery so applications can communicate using stable DNS names instead of hardcoded IP addresses.

Example:

```text
backend.ecsworkshop.local
```

The frontend Nginx container dynamically resolves this DNS name through the AWS VPC DNS resolver.

---

# VPC Architecture

The deployment uses:

- 2 Public Subnets
- 2 Private Subnets
- Internet Gateway
- NAT Gateway
- Public Route Table
- Private Route Table

## Public Subnets

Used for:

- Application Load Balancer
- NAT Gateway

## Private Subnets

Used for:

- ECS/Fargate Tasks

This ensures containers are not directly exposed to the internet.

---

# Security Groups

## ALB Security Group

Allows:

```text
HTTP 80 from 0.0.0.0/0
```

## ECS Tasks Security Group

Allows:

```text
HTTP 80 from ALB security group
TCP 3000 from ECS security group
```

---

# Jenkins CI/CD Pipeline

The Jenkins pipeline performs the following steps:

1. Authenticate Docker to Amazon ECR
2. Build frontend Docker image
3. Build backend Docker image
4. Push images to Amazon ECR
5. Register new ECS task definition revisions
6. Update ECS services
7. Trigger rolling deployment on ECS/Fargate

---

# Docker Image Tagging Strategy

Docker images are tagged using:

```text
$GIT_COMMIT
```

Example:

```text
frontend-a1b2c3d
backend-a1b2c3d
```

This provides:

- Traceability
- Immutable deployments
- Easier rollback capability
- Version-specific deployments

---

# ECS Service Discovery

Backend ECS service is registered into AWS Cloud Map:

```text
Service Name: backend
Namespace: ecsworkshop.local
```

Resulting DNS:

```text
backend.ecsworkshop.local
```

---

# Nginx Reverse Proxy Configuration

Frontend Nginx proxies API requests to the backend service using Cloud Map DNS.

```nginx
server {
    listen 80;

    resolver 10.0.0.2 valid=10s ipv6=off;

    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        set $backend "backend.ecsworkshop.local:3000";
        proxy_pass http://$backend;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

# Important Learning Outcomes

This project helped reinforce practical understanding of:

- ECS task definitions
- ECS services vs tasks
- ECS/Fargate networking
- Cloud Map service discovery
- Application Load Balancer behavior
- Nginx reverse proxying
- Jenkins-based CI/CD pipelines
- Docker image lifecycle
- ECR authentication
- IAM roles and permissions
- Public vs private subnet architecture
- NAT Gateway routing
- CloudWatch logging

---

# ECS Concepts Learned

## Task Definition

A blueprint that defines:

- Docker image
- CPU/memory
- ports
- logging
- networking
- environment variables

## ECS Service

Responsible for:

- Maintaining desired task count
- Restarting failed tasks
- Registering tasks with ALB target groups
- Rolling deployments

## ECS Task

A running instance of a task definition.

---

# Cloud Map Concept Learned

AWS Cloud Map is a service discovery mechanism that allows applications and microservices to discover dynamically changing service endpoints using DNS or API-based lookup.

In ECS/Fargate environments, Cloud Map allows services to communicate using stable DNS names instead of temporary task IP addresses.

---

# Deployment Flow

```text
Developer pushes code to GitHub
            ↓
Jenkins pipeline triggers
            ↓
Docker images are built
            ↓
Images pushed to Amazon ECR
            ↓
New ECS task definition revisions created
            ↓
ECS services updated
            ↓
Fargate launches updated tasks
            ↓
ALB routes traffic to healthy frontend tasks
            ↓
Frontend proxies API calls to backend service
```

---

# Future Improvements

Possible improvements for this project:

- Infrastructure as Code using Terraform/CloudFormation
- HTTPS using ACM certificates
- Route53 custom domain
- ECS Auto Scaling
- Internal ALB for backend services
- Blue/Green deployments
- GitHub Actions CI/CD
- Monitoring with Prometheus/Grafana
- WAF integration

---

# Conclusion

This project demonstrates a complete microservices deployment workflow on AWS using ECS/Fargate, Jenkins CI/CD, ECR, ALB, and Cloud Map service discovery.

The architecture follows modern cloud-native deployment practices using private networking, container orchestration, and automated deployments.
