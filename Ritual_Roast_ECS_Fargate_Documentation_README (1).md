# Ritual Roast - AWS ECS Fargate CI/CD Platform

## Executive Summary

Ritual Roast is an enterprise-style DevOps implementation that
demonstrates an end-to-end automated application deployment workflow
using AWS ECS Fargate, Amazon ECR, Jenkins, Docker, GitHub, Trivy, and
Application Load Balancer.

The objective of this project was to build a production-style CI/CD
pipeline where source code changes are automatically built, security
scanned, containerized, stored, and deployed into AWS.

## Architecture Overview

``` mermaid
flowchart LR
User[Users] --> ALB[Application Load Balancer HTTP:80]
ALB --> TG[Target Group]
TG --> ECS[ECS Fargate Service]
ECS --> Task[ECS Task Definition]
Task --> Container[Docker Container PHP Apache Port 80]

GitHub[GitHub Repository] --> Jenkins[Jenkins Pipeline]
Jenkins --> Docker[Docker Build]
Docker --> Trivy[Trivy Scan]
Trivy --> ECR[Amazon ECR]
ECR --> ECS
```

## Project Objective

The project achieves:

-   Automated Docker image creation
-   Security scanning before deployment
-   Private container image storage
-   Serverless container deployment
-   Load balanced application access
-   Repeatable deployment workflow

## Technology Stack

  Component            Technology
  -------------------- ---------------------------
  Cloud                AWS
  Container Platform   ECS Fargate
  Registry             Amazon ECR
  Load Balancing       Application Load Balancer
  CI/CD                Jenkins
  Source Control       GitHub
  Containerization     Docker
  Security Scan        Trivy
  Application Server   Apache
  Runtime              PHP 7.4

# AWS Architecture

## VPC

VPC Name: APP-VPC

CIDR:

    10.0.0.0/16

The VPC provides isolated networking for application resources.

## Traffic Flow

    Internet Users

          |

    Application Load Balancer

          |

    Target Group

          |

    ECS Service

          |

    Fargate Task

          |

    Docker Container Port 80

## ECS Architecture

    ECS Cluster
     |
     |
    ECS Service
     |
     |
    Task Definition
     |
     |
    Fargate Task
     |
     |
    Container

## Container Configuration

Container:

    Name:
    flask-container

    Port:
    80

    Protocol:
    TCP

## Docker Implementation

Dockerfile:

``` dockerfile
FROM php:7.4-apache

RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli

RUN apt-get update && apt-get upgrade -y

COPY . /var/www/html

EXPOSE 80
```

## CI/CD Pipeline

``` mermaid
flowchart TD
Developer --> GitHub
GitHub --> Jenkins
Jenkins --> Checkout
Checkout --> DockerBuild
DockerBuild --> Trivy
Trivy --> ECR
ECR --> ECS
ECS --> Production
```

## Jenkins Pipeline Stages

1.  Checkout source code

2.  Build Docker image

3.  Run Trivy vulnerability scan

4.  Authenticate with ECR

5.  Push image to ECR

6.  Update ECS task definition

7.  Register new task revision

8.  Deploy ECS service

9.  Wait for deployment stability

## Deployment Commands

AWS authentication:

``` bash
aws sts get-caller-identity
```

Register task definition:

``` bash
aws ecs register-task-definition \
--cli-input-json file://task-definition.json \
--region ap-south-1
```

Update ECS service:

``` bash
aws ecs update-service \
--cluster App-Cluster \
--service flask-task-service-cl7l6v6l \
--task-definition flask-task \
--force-new-deployment \
--region ap-south-1
```

Check ECS service:

``` bash
aws ecs describe-services \
--cluster App-Cluster \
--services flask-task-service-cl7l6v6l \
--region ap-south-1
```

# Troubleshooting Log

## Incident 1: Jenkins Java Compatibility

Problem:

Jenkins service required compatible Java version.

Resolution:

``` bash
sudo dnf remove java-17-amazon-corretto -y

sudo dnf install java-21-amazon-corretto -y

sudo systemctl restart jenkins
```

Verification:

``` bash
java -version

systemctl status jenkins
```

------------------------------------------------------------------------

## Incident 2: Jenkins Docker Permission

Problem:

Jenkins user could not execute Docker commands.

Resolution:

``` bash
sudo usermod -aG docker jenkins

sudo systemctl restart jenkins
```

Verification:

``` bash
sudo -u jenkins docker ps
```

------------------------------------------------------------------------

## Incident 3: ECS Service Running But Application Unavailable

Symptom:

    503 Service Temporarily Unavailable

ECS showed:

    desired:1
    running:1
    pending:0

Root cause:

Container port mismatch.

Old configuration:

    Container Port:5000

ALB Target Group expected:

    Port:80

Resolution:

Updated:

    containerPort:80
    hostPort:80

Verification:

Target group:

    Healthy:1
    Unhealthy:0

------------------------------------------------------------------------

## Incident 4: ECS Target Health Failure Decision Flow

``` mermaid
flowchart TD
A[ALB returns 503] --> B{Target Healthy?}

B -->|No| C[Check Port Mapping]
C --> D[Check ECS Task Definition]
D --> E[Check Container Port]

B -->|Yes| F[Check Application Logs]
```

# Master Command Reference

## Docker

``` bash
docker build -t image-name .
docker run -d -p 80:80 image-name
docker ps
```

## ECS Debugging

``` bash
aws ecs list-clusters --region ap-south-1

aws ecs list-task-definitions \
--family-prefix flask-task \
--region ap-south-1

aws ecs describe-task-definition \
--task-definition flask-task \
--region ap-south-1
```

## Git

``` bash
git add .
git commit -m "changes"
git push origin main
```

# Interview Explanation

## 30 Second Explanation

I built an automated ECS Fargate deployment platform where developers
push code to GitHub, Jenkins builds Docker images, Trivy performs
vulnerability scanning, images are stored in ECR, and ECS Fargate
deploys containers behind an Application Load Balancer.

## Key Technical Questions

### Why ECS Fargate?

-   No server management
-   AWS managed container runtime
-   Easy scaling

### Why ECR?

-   Private Docker registry
-   IAM integration
-   Secure image management

### How did you troubleshoot 503?

I checked:

1.  ECS service status
2.  Running tasks
3.  Target group health
4.  Container port mapping
5.  ALB listener configuration

# Future Improvements

Future enhancements:

-   Terraform infrastructure as code
-   CloudWatch centralized logging
-   ECS autoscaling
-   AWS Secrets Manager integration
-   Blue/Green deployment
-   WAF protection
-   Monitoring dashboards

# Conclusion

This project demonstrates a complete enterprise DevOps workflow from
source code commit to production deployment using AWS cloud-native
services.
