# Ritual Roast - AWS ECS Fargate DevOps Automation Platform

![AWS](https://img.shields.io/badge/Cloud-AWS-orange)
![ECS](https://img.shields.io/badge/Compute-ECS%20Fargate-blue)
![Jenkins](https://img.shields.io/badge/CI%2FCD-Jenkins-red)
![Docker](https://img.shields.io/badge/Container-Docker-blue)

------------------------------------------------------------------------

# 1. Executive Summary

This project demonstrates an end-to-end enterprise style DevOps
deployment platform built on AWS.

The objective was to automate application delivery using:

-   GitHub for source control
-   Jenkins for CI/CD automation
-   Docker for containerization
-   Trivy for vulnerability scanning
-   Amazon ECR for container image storage
-   Amazon ECS Fargate for container orchestration
-   Application Load Balancer for traffic distribution

The final outcome is an automated deployment pipeline where a developer
commit can be converted into a running production container.

------------------------------------------------------------------------

# 2. Architecture Overview

``` mermaid
flowchart LR

Developer --> GitHub
GitHub --> Jenkins
Jenkins --> Docker
Docker --> Trivy
Trivy --> ECR
ECR --> ECS

Users --> ALB
ALB --> TargetGroup
TargetGroup --> ECS
ECS --> Container
```

Traffic flow:

    Internet Users
          |
          v
    Application Load Balancer
          |
          v
    Target Group
          |
          v
    ECS Fargate Service
          |
          v
    ECS Task
          |
          v
    Docker Container

------------------------------------------------------------------------

# 3. Technology Stack

  Component            Technology
  -------------------- -------------
  Cloud                AWS
  Compute              ECS Fargate
  Container Registry   Amazon ECR
  Load Balancer        ALB
  CI/CD                Jenkins
  Source Control       GitHub
  Container            Docker
  Security Scan        Trivy
  Runtime              Java 21
  Application Server   Apache PHP

------------------------------------------------------------------------

# 4. Prerequisites

Install the following tools:

  Tool               Download
  ------------------ ---------------------------------------
  Git                https://git-scm.com/downloads
  Java Corretto 21   https://aws.amazon.com/corretto/
  Jenkins            https://www.jenkins.io/download/
  Docker             https://docs.docker.com/get-docker/
  AWS CLI            https://aws.amazon.com/cli/
  Trivy              https://aquasecurity.github.io/trivy/

Verification:

``` bash
git --version
java -version
docker --version
aws --version
trivy --version
```

------------------------------------------------------------------------

# 5. AWS Infrastructure

## VPC

Name:

    APP-VPC

CIDR:

    10.0.0.0/16

VPC ID:

    vpc-0397589c04d09f389

Purpose:

-   Network isolation
-   Application security boundary

------------------------------------------------------------------------

# 6. ECS Configuration

Cluster:

    App-Cluster

Service:

    flask-task-service-cl7l6v6l

Task Family:

    flask-task

Network Mode:

    awsvpc

Launch Type:

    FARGATE

------------------------------------------------------------------------

# 7. Docker Implementation

Dockerfile:

``` dockerfile
FROM php:7.4-apache

RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli

RUN apt-get update && apt-get upgrade -y

COPY . /var/www/html

EXPOSE 80
```

Container flow:

    Application Code
            |
            v
    Dockerfile
            |
            v
    Docker Image
            |
            v
    Amazon ECR
            |
            v
    ECS Task Definition
            |
            v
    Running Container

------------------------------------------------------------------------

# 8. Jenkins Setup

## Create Swap

``` bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## Java Upgrade

``` bash
sudo dnf remove java-17-amazon-corretto -y
sudo dnf install java-21-amazon-corretto -y
sudo systemctl restart jenkins
```

## Docker Permission

``` bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

Verification:

``` bash
sudo -u jenkins docker ps
```

------------------------------------------------------------------------

# 9. Jenkins Pipeline

Pipeline stages:

    Checkout Code
          |
    Docker Build
          |
    Trivy Security Scan
          |
    Login ECR
          |
    Push Image
          |
    Update Task Definition
          |
    Deploy ECS Service
          |
    Wait For Stability

------------------------------------------------------------------------

# 10. Important AWS Commands

Verify AWS identity:

``` bash
aws sts get-caller-identity
```

Check ECS service:

``` bash
aws ecs describe-services \
--cluster App-Cluster \
--services flask-task-service-cl7l6v6l \
--region ap-south-1
```

Check stopped tasks:

``` bash
aws ecs list-tasks \
--cluster App-Cluster \
--desired-status STOPPED \
--region ap-south-1
```

Describe task:

``` bash
aws ecs describe-tasks \
--cluster App-Cluster \
--tasks TASK_ID \
--region ap-south-1
```

------------------------------------------------------------------------

# 11. Troubleshooting Log

## Issue: Jenkins Docker Permission

Problem:

Jenkins pipeline could not execute Docker commands.

Root cause:

Jenkins user was not part of Docker group.

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

## Issue: Java Version Compatibility

Problem:

Jenkins required supported Java runtime.

Fix:

``` bash
sudo dnf remove java-17-amazon-corretto -y
sudo dnf install java-21-amazon-corretto -y
```

Verification:

``` bash
java -version
```

------------------------------------------------------------------------

## Issue: ECS Application Unavailable

Symptom:

    503 Service Unavailable

Investigation:

-   Checked ECS service
-   Checked stopped tasks
-   Checked target group health

Root cause:

Port mismatch between load balancer and container.

Resolution:

    ALB Port: 80
    Target Group Port: 80
    Container Port: 80

------------------------------------------------------------------------

# 12. ECS 503 Troubleshooting Flow

``` mermaid
flowchart TD

A[ALB 503]
B{Target Healthy?}
C[Check ECS Task]
D[Check Port Mapping]
E[Check Security Group]
F[Check Application Logs]

A --> B
B -->|No| C
C --> D
D --> E
B -->|Yes| F
```

------------------------------------------------------------------------

# 13. Why ECS Over EKS?

ECS Fargate was selected because the requirement was a reliable AWS
native container platform with lower operational overhead.

ECS advantages:

-   No Kubernetes cluster management
-   No worker node management
-   Native AWS integration
-   Faster deployment
-   Simpler operations

EKS would be preferred when:

-   Kubernetes standardization is required
-   Multi-cloud portability is needed
-   Advanced Kubernetes ecosystem features are required

------------------------------------------------------------------------

# 14. What Problem Did ECS Solve?

Before:

    Manual Deployment
           |
    Server Login
           |
    Install Dependencies
           |
    Restart Application

Problems:

-   Slow releases
-   Environment differences
-   Manual errors

After:

    Git Commit
       |
    Jenkins
       |
    Docker Image
       |
    ECR
       |
    ECS Deployment

Benefits:

-   Repeatable deployments
-   Consistent environments
-   Automated delivery
-   Scalable containers

------------------------------------------------------------------------

# 15. Production Improvements

Future enhancements:

-   Terraform infrastructure automation
-   HTTPS using ACM
-   Route53 DNS
-   ECS Auto Scaling
-   CloudWatch dashboards
-   Secrets Manager
-   Blue/Green deployment
-   Automated rollback

------------------------------------------------------------------------

# 16. Interview Summary

I built an AWS ECS Fargate based DevOps platform where Jenkins automates
the complete CI/CD lifecycle. Source code is pulled from GitHub, Docker
images are built and scanned using Trivy, pushed to ECR, and deployed
into ECS behind an Application Load Balancer.

The project demonstrates AWS networking, containerization, CI/CD
automation, troubleshooting, and production deployment practices.
