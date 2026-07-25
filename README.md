# Ritual Roast - AWS ECS Fargate DevOps Automation Platform

![AWS](https://img.shields.io/badge/Cloud-AWS-orange)
![ECS](https://img.shields.io/badge/Compute-ECS%20Fargate-blue)
![Docker](https://img.shields.io/badge/Container-Docker-blue)
![Jenkins](https://img.shields.io/badge/CI%2FCD-Jenkins-red)
![Trivy](https://img.shields.io/badge/Security-Trivy-green)

# 1. Project Overview

## Objective

Build a production-style AWS DevOps deployment platform that automates
application delivery from source code commit to running ECS Fargate
containers.

The solution uses:

-   GitHub
-   Jenkins CI/CD
-   Docker
-   Trivy
-   Amazon ECR
-   ECS Fargate
-   Application Load Balancer
-   AWS VPC networking

------------------------------------------------------------------------

# 2. Executive Summary

## Problem Before Automation

    Developer
     |
    Manual Deployment
     |
    Server Login
     |
    Copy Files
     |
    Install Dependencies
     |
    Restart Application

Problems:

-   Manual errors
-   Slow releases
-   Environment mismatch
-   Difficult rollback

## Solution

    GitHub
     |
    Jenkins
     |
    Docker Build
     |
    Trivy Scan
     |
    Amazon ECR
     |
    ECS Task Definition
     |
    ECS Fargate Service
     |
    ALB
     |
    Users

------------------------------------------------------------------------

# 3. Technology Stack

  Area             Technology
  ---------------- ------------------------------------------
  Cloud            AWS
  Region           ap-south-1
  Compute          ECS Fargate
  Registry         Amazon ECR
  CI/CD            Jenkins
  Source Control   GitHub
  Container        Docker
  Security         Trivy
  Networking       VPC, ALB, Target Groups, Security Groups
  Monitoring       CloudWatch

------------------------------------------------------------------------

# 4. Prerequisites

Required tools:

  -----------------------------------------------------------------------------------------------------
  Tool                    Purpose                 Official Download
  ----------------------- ----------------------- -----------------------------------------------------
  Git                     Source control          https://git-scm.com/downloads

  Java 21                 Jenkins runtime         https://aws.amazon.com/corretto/

  Jenkins                 CI/CD automation        https://www.jenkins.io/download/

  Docker                  Container build         https://docs.docker.com/get-docker/

  AWS CLI                 AWS management          https://aws.amazon.com/cli/

  Trivy                   Image scanning          https://aquasecurity.github.io/trivy/

  Terraform               Infrastructure as Code  https://developer.hashicorp.com/terraform/downloads
  -----------------------------------------------------------------------------------------------------

Verification:

``` bash
git --version
java -version
docker --version
aws --version
trivy --version
```

------------------------------------------------------------------------

# 5. AWS Cloud Architecture

## Region

    ap-south-1 Mumbai

## VPC

    Name: APP-VPC

    CIDR: 10.0.0.0/16

    VPC ID:
    vpc-0397589c04d09f389

Purpose:

-   Network isolation
-   Controlled communication
-   Secure application hosting

------------------------------------------------------------------------

# Complete AWS Architecture Diagram

``` mermaid
flowchart TB

Users[Internet Users]

IGW[Internet Gateway]

subgraph AWS[AWS Cloud]

subgraph VPC[APP-VPC 10.0.0.0/16]

subgraph Public[Public Subnet]
ALB[Application Load Balancer]
Listener[Listener Port 80]
end

subgraph Private[Private Subnet]
TG[Target Group]
ECS[ECS Service]
Task[ECS Task]
Container[Docker Container]
end

SG1[ALB Security Group]
SG2[ECS Security Group]

end
end

Users --> IGW
IGW --> ALB
ALB --> Listener
Listener --> TG
TG --> ECS
ECS --> Task
Task --> Container

SG1 --> ALB
SG2 --> ECS
```

------------------------------------------------------------------------

# 6. Network Design

## Public Subnet

Contains:

-   Application Load Balancer

Purpose:

Receives internet traffic.

## Private Subnet

Contains:

-   ECS Fargate Tasks
-   Application containers

Purpose:

Protect application from direct internet exposure.

## Route Tables

Public route:

    0.0.0.0/0 --> Internet Gateway

Private route:

    Local VPC communication

## Security Groups

ALB:

    Inbound:
    HTTP 80 from Internet

ECS:

    Inbound:
    Port 80 from ALB Security Group

------------------------------------------------------------------------

# 7. ECS Fargate Implementation

Cluster:

    App-Cluster

Service:

    flask-task-service-cl7l6v6l

Task Definition:

    flask-task

Configuration:

    Launch Type:
    FARGATE

    Network Mode:
    awsvpc

    CPU:
    512

    Memory:
    1024 MB

    Container Port:
    80

------------------------------------------------------------------------

# 8. Docker Implementation

Dockerfile:

``` dockerfile
FROM php:7.4-apache

RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli

RUN apt-get update && apt-get upgrade -y

COPY . /var/www/html

EXPOSE 80
```

Lifecycle:

``` mermaid
flowchart LR

A[Source Code]
B[Dockerfile]
C[Docker Build]
D[Docker Image]
E[ECR]
F[ECS Task Definition]
G[Running Container]

A --> B --> C --> D --> E --> F --> G
```

------------------------------------------------------------------------

# 9. Jenkins Server Setup

## Swap Creation

``` bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## Java Upgrade

Problem:

Jenkins compatibility issue.

Fix:

``` bash
sudo dnf remove java-17-amazon-corretto -y
sudo dnf install java-21-amazon-corretto -y
sudo systemctl restart jenkins
```

## Docker Permission

Problem:

Jenkins could not run Docker.

Fix:

``` bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

Verification:

``` bash
sudo -u jenkins docker ps
```

------------------------------------------------------------------------

# 10. CI/CD Pipeline

``` mermaid
flowchart LR

Developer --> GitHub
GitHub --> Jenkins
Jenkins --> Build[Docker Build]
Build --> Scan[Trivy Scan]
Scan --> ECR
ECR --> ECS
ECS --> Production
```

Stages:

1.  Checkout
2.  Docker Build
3.  Trivy Scan
4.  ECR Login
5.  Push Image
6.  Register Task Definition
7.  Deploy ECS Service
8.  Wait For Stability

------------------------------------------------------------------------

# 11. Troubleshooting History

## Java Compatibility Issue

Symptoms:

Jenkins failed due to unsupported Java.

Investigation:

``` bash
java -version
```

Root Cause:

Old Java runtime.

Fix:

Install Java 21.

Verification:

``` bash
java -version
```

------------------------------------------------------------------------

## Jenkins Docker Permission Issue

Symptoms:

Docker daemon permission denied.

Investigation:

``` bash
groups jenkins
```

Root Cause:

Jenkins user missing docker group.

Fix:

``` bash
sudo usermod -aG docker jenkins
```

------------------------------------------------------------------------

## ECS 503 Service Unavailable

Symptoms:

ALB returned:

    503 Service Unavailable

Investigation:

Checked:

-   ECS task
-   Target health
-   Port mapping

Root Cause:

Port mismatch.

Expected:

    ALB:80
    Target Group:80
    Container:80

------------------------------------------------------------------------

# ALB 503 Troubleshooting

``` mermaid
flowchart TD

A[ALB 503]
B{Target Healthy?}
C[Check ECS Logs]
D[Check Port Mapping]
E[Check Security Group]

A --> B
B --> C
B --> D
D --> E
```

------------------------------------------------------------------------

# 12. AWS Commands

Identity:

``` bash
aws sts get-caller-identity
```

ECS Service:

``` bash
aws ecs describe-services --cluster App-Cluster --services flask-task-service-cl7l6v6l
```

Stopped Tasks:

``` bash
aws ecs list-tasks --cluster App-Cluster --desired-status STOPPED
```

------------------------------------------------------------------------

# 13. Security Design

Implemented:

-   IAM execution roles
-   Security groups
-   ECR image scanning
-   No hardcoded credentials

Recommended:

-   AWS Secrets Manager
-   IAM least privilege
-   HTTPS ACM certificates

------------------------------------------------------------------------

# 14. Monitoring

Use:

-   CloudWatch Logs
-   ECS service metrics
-   ALB health checks
-   Container health checks

------------------------------------------------------------------------

# 15. Production Improvements

Future enhancements:

-   Terraform automation
-   Route53 DNS
-   HTTPS ACM
-   ECS Auto Scaling
-   Blue/Green deployments
-   WAF
-   Enhanced monitoring

------------------------------------------------------------------------

# 16. Architecture Decision

# Why ECS Over EKS?

ECS Fargate was selected because the project required AWS-native
container deployment with lower operational overhead.

Advantages:

-   No Kubernetes cluster administration
-   No worker node management
-   Native AWS integration
-   Faster deployment
-   Simpler maintenance

EKS is preferred when:

-   Kubernetes standardization exists
-   Multi-cloud portability is required
-   Advanced Kubernetes ecosystem features are needed

------------------------------------------------------------------------

# 17. What Problem Did ECS Solve?

Before:

    Manual server deployment

After:

    GitHub
     |
    Jenkins
     |
    Docker
     |
    ECR
     |
    ECS
     |
    Production

Benefits:

-   Consistent environments
-   Automated releases
-   Faster deployments
-   Scalable containers

------------------------------------------------------------------------

# 18. Interview Explanation

## 30 Seconds

I built an AWS ECS Fargate DevOps platform where Jenkins automates
Docker image creation, security scanning, ECR publishing, and ECS
deployment behind an Application Load Balancer.

## 5 Minutes

Explain:

-   VPC architecture
-   ALB routing
-   ECS service
-   CI/CD pipeline
-   Troubleshooting

## Deep Technical

Discuss:

-   awsvpc networking
-   IAM execution roles
-   Task definitions
-   Target health checks
-   Container lifecycle
-   ECS vs EKS decision
