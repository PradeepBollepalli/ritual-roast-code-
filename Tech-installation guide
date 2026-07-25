# Ritual Roast - AWS ECS Fargate DevOps Portfolio Documentation

# Prerequisites & Required Tools Installation

Before deploying or rebuilding this project, install the following
tools.

## Required Tools

  -----------------------------------------------------------------------------------------------------------
  Tool              Version           Purpose           Official Download
  ----------------- ----------------- ----------------- -----------------------------------------------------
  Git               Latest            Source code       https://git-scm.com/downloads
                                      management        

  Java              Java 21           Jenkins runtime   https://aws.amazon.com/corretto/

  Jenkins           LTS               CI/CD automation  https://www.jenkins.io/download/

  Docker            Latest            Container         https://docs.docker.com/get-docker/
                                      creation and      
                                      execution         

  AWS CLI           Version 2         AWS resource      https://aws.amazon.com/cli/
                                      management        

  Trivy             Latest            Container         https://aquasecurity.github.io/trivy/
                                      vulnerability     
                                      scanning          

  Terraform         Latest (Optional) Infrastructure as https://developer.hashicorp.com/terraform/downloads
                                      Code              

  kubectl           Latest (Optional) Kubernetes        https://kubernetes.io/docs/tasks/tools/
                                      management        
  -----------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

# Tool Installation and Verification

## Git Installation

Download:

https://git-scm.com/downloads

Verify:

``` bash
git --version
```

------------------------------------------------------------------------

## Java 21 Installation

Java is required for Jenkins execution.

Download:

https://aws.amazon.com/corretto/

Verify:

``` bash
java -version
```

Expected:

``` text
openjdk version "21"
```

------------------------------------------------------------------------

## Jenkins Installation

Download Jenkins LTS:

https://www.jenkins.io/download/

Linux installation:

``` bash
sudo dnf install jenkins -y

sudo systemctl enable jenkins

sudo systemctl start jenkins
```

Verify:

``` bash
sudo systemctl status jenkins
```

Access Jenkins:

``` text
http://<JENKINS_SERVER_PUBLIC_IP>:8080
```

------------------------------------------------------------------------

## Docker Installation

Download:

https://docs.docker.com/get-docker/

Install:

``` bash
sudo dnf install docker -y

sudo systemctl start docker

sudo systemctl enable docker
```

Verify:

``` bash
docker --version
```

Configure Jenkins Docker access:

``` bash
sudo usermod -aG docker jenkins

sudo systemctl restart jenkins
```

Verification:

``` bash
sudo -u jenkins docker ps
```

------------------------------------------------------------------------

## AWS CLI Installation

Download:

https://aws.amazon.com/cli/

Verify:

``` bash
aws --version
```

Configure AWS credentials:

``` bash
aws configure
```

Validate access:

``` bash
aws sts get-caller-identity
```

------------------------------------------------------------------------

## Trivy Installation

Documentation:

https://aquasecurity.github.io/trivy/

Amazon Linux installation:

``` bash
sudo dnf install wget -y

sudo wget -qO /etc/yum.repos.d/trivy.repo https://aquasecurity.github.io/trivy-repo/rpm/releases/$(rpm -E %{rhel})/x86_64/trivy.repo

sudo dnf install trivy -y
```

Verify:

``` bash
trivy --version
```

------------------------------------------------------------------------

# AWS Account Requirements

The deployment requires permissions for:

-   Amazon ECS
-   Amazon ECR
-   EC2
-   IAM
-   VPC
-   Elastic Load Balancing
-   CloudWatch

Required AWS resource flow:

``` text
AWS Account

 |
 +-- VPC
 |
 +-- Subnets
 |
 +-- Security Groups
 |
 +-- IAM Roles
 |
 +-- ECR Repository
 |
 +-- ECS Cluster
 |
 +-- ECS Service
 |
 +-- ALB
 |
 +-- Target Group
```

------------------------------------------------------------------------

# Project Architecture

(Existing architecture, ECS, Jenkins, Docker, deployment and
troubleshooting sections continue below.)
