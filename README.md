# SmartFlood - AWS Containerized Deployment

> Production-oriented deployment of SmartFlood using Docker containers and AWS cloud infrastructure.

## Overview

SmartFlood is an AI-assisted flood monitoring and relief allocation platform designed to support disaster response operations.

This project documents the migration of SmartFlood from a Google Cloud Run deployment to an AWS-based containerized architecture.

The migration focuses on:

- Containerized application services
- AWS cloud infrastructure
- Scalable service deployment
- Secure networking
- Centralized logging and monitoring
- Automated CI/CD
- Infrastructure as Code
- DevSecOps practices

---

## Project Architecture

The target deployment architecture separates SmartFlood into independently deployable services.

```text
                         Internet
                            |
                            v
                  +-------------------+
                  | Application Load  |
                  |     Balancer      |
                  +---------+---------+
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
       +-----------+  +-----------+  +-----------+
       | Frontend  |  |  Backend  |  | AI Service|
       | Container |  | Container |  | Container |
       +-----------+  +-----------+  +-----------+
             |              |              |
             +--------------+--------------+
                            |
              +-------------+-------------+
              |                           |
              v                           v
       +--------------+            +--------------+
       |   Supabase   |            |   MongoDB    |
       |   PostgreSQL |            |    Atlas     |
       +--------------+            +--------------+

                         AWS VPC
```

---

## Application Services

### Frontend

The SmartFlood frontend is responsible for the web-based administrative interface and visualization layer.

Technology:

- Next.js
- TypeScript
- React
- Tailwind CSS
- Leaflet

Responsibilities:

- Administrative dashboard
- Flood monitoring interface
- Sensor visualization
- Flood heatmap
- Resident management
- Relief inventory
- AI recommendation interface

---

### Backend

The backend provides the main application API and business logic.

Technology:

- Node.js
- Express
- TypeScript
- Supabase
- PostgreSQL

Responsibilities:

- Authentication and authorization
- Resident management
- Family management
- Relief inventory
- Application processing
- AI recommendation integration
- API communication

---

### AI Service

The AI service provides the recommendation and optimization functionality used by SmartFlood.

Technology:

- Python
- FastAPI
- AHP
- Fuzzy Logic

Responsibilities:

- Relief allocation recommendations
- Disaster response prioritization
- AI-based decision support
- Communication with the main backend

---

## AWS Infrastructure

The target AWS architecture uses managed AWS services for container deployment, networking, monitoring, and security.

### Amazon ECR

Amazon Elastic Container Registry is used to store application container images.

Expected repositories:

```text
smartflood/frontend
smartflood/backend
smartflood/ai-service
```

The deployment pipeline will build Docker images and push them to ECR before deploying new versions.

---

### Amazon ECS / AWS Fargate

Amazon Elastic Container Service is used to run the SmartFlood containers.

AWS Fargate provides serverless container execution without requiring direct management of EC2 instances.

Expected services:

```text
ECS Cluster
|
+-- frontend-service
|
+-- backend-service
|
+-- ai-service
```

Each service can be independently updated and scaled.

---

### Application Load Balancer

The Application Load Balancer provides a single entry point for HTTP and HTTPS traffic.

Expected routing:

```text
Internet
   |
   v
Application Load Balancer
   |
   +-- Frontend
   |
   +-- Backend API
   |
   +-- AI Service
```

Routing rules may be adjusted as the production architecture evolves.

---

### Amazon VPC

The application infrastructure will operate inside an AWS Virtual Private Cloud.

The VPC will provide network isolation and controlled communication between application components.

Planned components include:

- Public subnets
- Private subnets
- Internet Gateway
- Route tables
- Security groups
- Network access controls

---

### IAM

AWS Identity and Access Management will be used to control access to AWS resources.

The deployment will follow the principle of least privilege whenever possible.

Examples include:

- ECS task execution roles
- ECS task roles
- ECR access
- CloudWatch logging permissions
- CI/CD deployment permissions

---

### Amazon CloudWatch

CloudWatch will be used for application and infrastructure monitoring.

Expected functionality:

- Container logs
- Application logs
- Error monitoring
- CPU utilization
- Memory utilization
- Service health
- Deployment monitoring

---

## Cloud Run to AWS Migration

The project originally used Google Cloud Run for container deployment.

The target architecture replaces the Cloud Run deployment model with AWS services.

| Previous Architecture                         | Target Architecture      |
| --------------------------------------------- | ------------------------ |
| Google Cloud Run                              | Amazon ECS / AWS Fargate |
| Google Container Registry / Artifact Registry | Amazon ECR               |
| Cloud Run networking                          | Amazon VPC               |
| Cloud Run logs                                | Amazon CloudWatch        |
| Google IAM                                    | AWS IAM                  |
| GCP deployment workflow                       | AWS deployment workflow  |

The goal is not simply to move hosting providers.

The migration is also intended to improve infrastructure knowledge, deployment control, security practices, and DevOps experience.

---

## Containerization

Each major SmartFlood service is packaged as a Docker container.

Expected containers:

```text
Frontend
    |
    +-- Docker image
    |
    +-- Amazon ECR
    |
    +-- ECS/Fargate

Backend
    |
    +-- Docker image
    |
    +-- Amazon ECR
    |
    +-- ECS/Fargate

AI Service
    |
    +-- Docker image
    |
    +-- Amazon ECR
    |
    +-- ECS/Fargate
```

Each service should have its own Dockerfile and dependency configuration.

---

## Local Development

The services can be developed locally before being deployed to AWS.

Example project structure:

```text
smartflood/
|
+-- frontend/
|
+-- backend/
|
+-- ai-service/
|
+-- docker/
|
+-- infrastructure/
|
+-- .github/
|
+-- README.md
```

---

## Docker Development

Build the frontend container:

```bash
docker build -t smartflood-frontend ./frontend
```

Build the backend container:

```bash
docker build -t smartflood-backend ./backend
```

Build the AI service container:

```bash
docker build -t smartflood-ai ./ai-service
```

Run a container locally:

```bash
docker run --env-file .env smartflood-frontend
```

The exact ports and environment variables will depend on the final application configuration.

---

## Environment Variables

Sensitive configuration must not be committed to the repository.

Examples of environment variables include:

```text
DATABASE_URL=
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

MONGODB_URI=

AI_SERVICE_URL=

NEXT_PUBLIC_API_URL=

AWS_REGION=
AWS_ACCOUNT_ID=
```

Actual secrets should be stored using an appropriate secrets management solution rather than committed to Git.

---

## CI/CD Pipeline

The planned deployment pipeline uses GitHub Actions.

```text
Developer
    |
    v
Git Push
    |
    v
GitHub Actions
    |
    +-- Install dependencies
    |
    +-- Lint
    |
    +-- Type checking
    |
    +-- Tests
    |
    +-- Security checks
    |
    +-- Docker build
    |
    v
Amazon ECR
    |
    v
Amazon ECS
    |
    v
AWS Fargate
```

The pipeline will eventually automate the build and deployment process.

---

## DevSecOps

Security is treated as part of the deployment lifecycle rather than a separate final step.

Planned security practices include:

- Dependency scanning
- Container image scanning
- Secret management
- Least-privilege IAM
- Network segmentation
- HTTPS
- Secure environment variables
- Container hardening
- CloudWatch monitoring
- Infrastructure security review

Future security improvements may include:

- AWS WAF
- GuardDuty
- Security Hub
- Automated vulnerability scanning
- Infrastructure security scanning
- Runtime monitoring

---

## Infrastructure as Code

The AWS infrastructure is intended to be managed using Infrastructure as Code.

Planned tooling:

- Terraform

Expected structure:

```text
infrastructure/
|
+-- terraform/
    |
    +-- main.tf
    +-- variables.tf
    +-- outputs.tf
    +-- provider.tf
    +-- vpc.tf
    +-- ecr.tf
    +-- ecs.tf
    +-- alb.tf
    +-- iam.tf
    +-- cloudwatch.tf
```

Infrastructure should be reproducible rather than manually configured whenever practical.

---

## Deployment Strategy

The intended deployment process is:

```text
1. Developer changes application code
2. Changes are pushed to GitHub
3. CI pipeline validates the application
4. Docker images are built
5. Images are pushed to Amazon ECR
6. ECS task definitions are updated
7. ECS services deploy the new containers
8. Health checks verify the deployment
9. CloudWatch monitors the running services
```

---

## Monitoring

The deployment will use AWS monitoring capabilities to observe application and infrastructure health.

Important metrics include:

- CPU utilization
- Memory utilization
- Container restarts
- Application errors
- HTTP response status codes
- Request latency
- Service availability
- Deployment failures

Logs should provide enough information to troubleshoot application and infrastructure issues without exposing sensitive information.

---

## Database Architecture

SmartFlood uses external managed database services.

### Supabase

Supabase PostgreSQL is used for application data such as:

- Users
- Barangays
- Families
- Residents
- Resident applications
- Relief inventory
- AI recommendations

### MongoDB Atlas

MongoDB Atlas is used for sensor-related data and other document-oriented workloads.

The AWS application services communicate with these managed database platforms through secured connections.

---

## Security Considerations

The deployment should follow these principles:

### Least Privilege

AWS identities should receive only the permissions required to perform their responsibilities.

### Network Segmentation

Public-facing components should be separated from internal services where appropriate.

### Secret Management

Credentials and API keys must never be hardcoded into source code or Docker images.

### Container Security

Container images should use minimal base images and should avoid unnecessary packages and privileges.

### HTTPS

Production traffic should use HTTPS.

### Logging

Logs should avoid exposing:

- Passwords
- API keys
- Access tokens
- Database credentials
- Personally identifiable information

---

## Cost Considerations

AWS infrastructure introduces recurring cloud costs.

The final cost depends on:

- ECS/Fargate CPU allocation
- ECS/Fargate memory allocation
- Number of running tasks
- Application Load Balancer usage
- ECR storage
- CloudWatch logs
- Data transfer
- NAT Gateway usage
- Other AWS services

During development, resources should be sized appropriately and unused infrastructure should be shut down or removed.

---

## Migration Goals

The migration from Cloud Run to AWS aims to achieve the following:

- Containerize all major SmartFlood services
- Deploy containers using AWS
- Establish reproducible infrastructure
- Implement CI/CD
- Improve deployment visibility
- Apply DevSecOps practices
- Learn production-oriented AWS architecture
- Establish a foundation for future scalability

---

## Current Status

The project is currently undergoing migration from Google Cloud Run to AWS.

### Completed

- SmartFlood application development
- Frontend containerization planning
- Backend containerization planning
- AI service containerization planning
- AWS architecture planning

### In Progress

- Docker container implementation
- Amazon ECR setup
- ECS/Fargate deployment
- VPC configuration
- Application Load Balancer configuration
- CI/CD pipeline
- Infrastructure as Code

### Planned

- Automated security scanning
- Advanced AWS monitoring
- Infrastructure security hardening
- Production deployment optimization

---

## Future Improvements

Potential future improvements include:

- Terraform-based infrastructure provisioning
- Blue/green deployments
- Automated rollback
- AWS WAF
- AWS GuardDuty
- AWS Security Hub
- Container vulnerability scanning
- Centralized observability
- Automated disaster recovery
- Multi-environment deployment
- Infrastructure testing
- Performance testing
- Cost optimization

---

## Learning Objectives

This deployment is also a practical learning project focused on modern cloud and DevSecOps engineering.

Key learning areas include:

- Docker
- Container networking
- AWS ECS
- AWS Fargate
- Amazon ECR
- AWS VPC
- Application Load Balancer
- IAM
- CloudWatch
- Terraform
- GitHub Actions
- CI/CD
- Container security
- Cloud security
- Infrastructure as Code

---

## Technology Stack

### Application

- Next.js
- TypeScript
- React
- Tailwind CSS
- Node.js
- Express
- Python
- FastAPI

### Data

- PostgreSQL
- Supabase
- MongoDB Atlas

### Cloud

- AWS
- Amazon ECS
- AWS Fargate
- Amazon ECR
- Amazon VPC
- Application Load Balancer
- AWS IAM
- Amazon CloudWatch

### DevOps

- Docker
- Terraform
- GitHub Actions

### Security

- IAM
- Network segmentation
- Container security
- Dependency scanning
- Vulnerability scanning
- Cloud monitoring

---

## Project Philosophy

The goal of this migration is not simply to deploy SmartFlood to another cloud provider.

The goal is to understand the complete lifecycle of a modern application:

```text
Application Development
        |
        v
Containerization
        |
        v
Continuous Integration
        |
        v
Security Validation
        |
        v
Container Registry
        |
        v
Cloud Infrastructure
        |
        v
Application Deployment
        |
        v
Monitoring
        |
        v
Continuous Improvement
```

SmartFlood serves as the practical environment for applying these concepts to a real application.

---

## Author

**Mark Aquino**

Full-Stack Developer | DevSecOps Learner

SmartFlood is an academic and engineering project focused on flood monitoring, AI-assisted relief allocation, cloud infrastructure, and secure application deployment.
# Smart-Flood-V5-AWS
