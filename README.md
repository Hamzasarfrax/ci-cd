# CI/CD Static Site Deployment to AWS ECS

This project demonstrates a clean CI/CD pipeline for a containerized static website using GitHub Actions, Amazon ECR, and Amazon ECS. Every push to `main` runs quality checks, builds a Docker image, pushes it to ECR, and triggers a fresh ECS deployment.

It is designed as a practical learning project, but the structure is strong enough to present professionally in a portfolio, resume, or interview discussion.

## Architecture Overview

The delivery flow is straightforward:

1. GitHub Actions starts on every push to `main`.
2. The workflow checks out the code and runs HTML validation.
3. Trivy scans the repository for high and critical findings.
4. Docker builds the application image.
5. GitHub Actions assumes an AWS IAM role using OIDC.
6. The workflow logs in to Amazon ECR and pushes the image.
7. Amazon ECS is instructed to force a new deployment of the service.

## Current Workflow Capabilities

- Automated deployment on every push to `main`
- HTML validation using `tidy`
- Repository scanning with Trivy
- Docker image build and push to Amazon ECR
- Secure AWS authentication using GitHub OIDC
- ECS service refresh using `aws ecs update-service --force-new-deployment`

## Tech Stack

- GitHub Actions
- Docker
- Amazon ECR
- Amazon ECS
- AWS IAM OIDC federation
- Trivy
- HTML, CSS, JavaScript

## Repository Structure

```text
.
|-- .github/
|   `-- workflows/
|       `-- deploy.yml
|-- Dockerfile
|-- index.html
|-- style.css
|-- script.js
`-- README.md
```

## GitHub Actions Configuration

The workflow expects the following repository secrets:

- `AWS_REGION`
- `ECR_REPOSITORY`
- `ECS_CLUSTER`
- `ECS_SERVICE`

Example values:

```text
AWS_REGION=us-east-1
ECR_REPOSITORY=myapp
ECS_CLUSTER=my-ecs-cluster
ECS_SERVICE=my-ecs-service
```

`ECR_REPOSITORY` should contain only the ECR repository name, such as `myapp`, not the full registry URL.

## AWS Requirements

Before running the pipeline, make sure the following resources already exist in AWS:

- An Amazon ECR repository
- An Amazon ECS cluster
- An Amazon ECS service
- An IAM role trusted by GitHub OIDC

The IAM role used by GitHub Actions should allow at least:

- `ecr:GetAuthorizationToken`
- `ecr:BatchCheckLayerAvailability`
- `ecr:InitiateLayerUpload`
- `ecr:UploadLayerPart`
- `ecr:CompleteLayerUpload`
- `ecr:PutImage`
- `ecs:UpdateService`
- `ecs:DescribeServices`

## Why This Project Stands Out

This project goes beyond a basic "push code and deploy" demo. It shows:

- Secure cloud authentication without long-lived AWS access keys
- Automated validation and security scanning before deployment
- Real container registry integration with ECR
- Real service deployment behavior with ECS
- A practical CI/CD pattern that mirrors real DevOps workflows

For learners, it proves hands-on understanding of cloud deployment pipelines. For hiring managers, it shows applied knowledge rather than tutorial-only familiarity.

## Local Development

Build and run the container locally:

```powershell
docker build -t myapp .
docker run -d -p 8080:80 myapp
```

Then open:

```text
http://localhost:8080
```

## Production-Level Next Steps

If you want to push this project toward a more advanced, production-style setup, these are the best upgrades:

1. Tag images with both `latest` and the Git commit SHA.
2. Deploy a new ECS task definition revision instead of relying only on `force-new-deployment`.
3. Add a pull request workflow for validation before merge.
4. Publish Trivy scan results as artifacts or SARIF reports.
5. Introduce environment separation such as `dev`, `staging`, and `prod`.
6. Add branch protection and required status checks.
7. Add image signing, SBOM generation, and policy scanning.

## Notes

The current deployment strategy is intentionally simple and effective for a learning or portfolio project. It assumes the ECS service is already configured to use the ECR repository being updated by the pipeline.

For stricter release control in production, the next logical step is moving to immutable image tags and task-definition based deployments.
