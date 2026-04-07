# CICD Static Site to AWS ECS

This project is a simple static site deployed through GitHub Actions to Amazon ECS. The pipeline lint-checks the HTML, scans the repository and image with Trivy, builds a Docker image, pushes it to Amazon ECR, and triggers a fresh ECS deployment.

## What this pipeline does

1. Checks out the repository on every push to `main`.
2. Runs `tidy` against `index.html` as a basic quality gate.
3. Runs a Trivy filesystem scan to catch high and critical issues before build.
4. Assumes an AWS IAM role using GitHub OIDC instead of long-lived AWS keys.
5. Logs in to Amazon ECR and builds the image with both `latest` and commit-SHA tags.
6. Runs a Trivy image scan on the built image.
7. Pushes the image to ECR and forces a new ECS deployment.

## Required GitHub configuration

Create these before running the workflow:

### Secrets

- `AWS_ROLE_ARN`

### Variables

- `ECR_REPOSITORY`
- `ECS_CLUSTER`
- `ECS_SERVICE`

Use these example values:

```text
AWS_ROLE_ARN=arn:aws:iam::123456789012:role/github-actions-ecs-deploy
ECR_REPOSITORY=myapp
ECS_CLUSTER=my-ecs-cluster
ECS_SERVICE=my-ecs-service
```

`ECR_REPOSITORY` should be the repository name inside ECR, not the full registry URL. The workflow reads the registry URL automatically from the ECR login step.

## AWS side setup

You need these resources ready in AWS:

- An ECR repository
- An ECS cluster
- An ECS service already configured to pull from your ECR image
- An IAM role trusted by GitHub OIDC

The IAM role should allow only what this workflow needs:

- `ecr:GetAuthorizationToken`
- `ecr:BatchCheckLayerAvailability`
- `ecr:InitiateLayerUpload`
- `ecr:UploadLayerPart`
- `ecr:CompleteLayerUpload`
- `ecr:PutImage`
- `ecs:UpdateService`
- `ecs:DescribeServices`

## Why OIDC is better

This workflow uses `AWS_ROLE_ARN` with `aws-actions/configure-aws-credentials`. That means GitHub receives short-lived AWS credentials at runtime instead of storing permanent access keys in repository secrets.

This is the safer approach because:

- there are no long-lived AWS keys to rotate or leak
- access is temporary
- the IAM role can be tightly scoped

## Top 1% improvements

If you want this project to feel more production-grade, these are the next upgrades:

1. Deploy a new ECS task definition revision for each image instead of relying on `latest`.
2. Add `Checkov` for Terraform, GitHub Actions, and Dockerfile policy scanning.
3. Add branch protection so deployment only happens after pull request review.
4. Add a separate pull request workflow for linting and scans before merge.
5. Publish Trivy results to GitHub Security or keep SARIF reports as artifacts.
6. Add image signing, SBOM generation, and dependency review.
7. Use separate AWS accounts or environments for `dev`, `stage`, and `prod`.

## Local Docker commands

```powershell
docker build -t myapp .
docker run -d -p 8080:80 myapp
```

Open `http://localhost:8080` to test locally.

## Important note

The current deploy step uses `aws ecs update-service --force-new-deployment`, which works well for a learning project and for services already configured around the same repository image. For stronger production control, move to task-definition based deployments with immutable image tags only.
