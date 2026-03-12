
# CI/CD & Deployment Strategy

## Overview
This project should use Azure DevOps to automate build, test, infrastructure deployment, database migration, and application release in a secure and repeatable way. The overall objective is to promote the same tested artifact across environments, reduce deployment risk, and maintain clear approval and audit controls for production.

## CI/CD Pipeline Approach
The pipeline should begin with a CI stage triggered by pull requests and merges. In this stage, the TypeScript/CDK code is validated using linting, unit tests, and "cdk synth" to confirm the infrastructure compiles correctly. Security and dependency scanning should also be included to catch issues early.

Once validation passes, the pipeline should move to a build stage where the application is packaged as a Docker image, tagged with the commit SHA or build ID, and pushed to Amazon ECR. The key principle is to create an immutable artifact once and promote that same version across dev, test, QA, and production.

The deployment stage should then use Azure DevOps to assume an AWS deployment role and run "cdk deploy" for the target environment. This stage provisions or updates the required AWS resources such as RDS PostgreSQL, ECS Fargate, SQS, ALB, IAM roles, and security groups. For production, Azure DevOps environment approvals should be enabled so that deployment requires manual approval before execution.

After infrastructure deployment, the pipeline should update the ECS task definition with the new image tag and release the application using a rolling or blue/green deployment strategy behind the Application Load Balancer. Post-deployment smoke tests should confirm service health, ALB routing, queue consumption, and database connectivity before considering the release successful.

## Safe Database Migration Strategy
Database migrations should be treated as a controlled release step because schema changes carry higher risk than standard application deployments. All schema updates should be versioned and stored in source control using a migration tool such as Flyway or Liquibase.

The safest approach is to run migrations as a dedicated pipeline step after infrastructure is ready but before full application cutover. Migrations should be backward compatible wherever possible using an expand-and-contract model: add new schema elements first, deploy application code that supports both old and new structures, and remove deprecated elements only in a later release. This reduces downtime and rollback risk.

For production, the pipeline should verify that backups or snapshots exist before migration begins. A dedicated database migration user should be used with only the permissions required for schema changes. Migrations must be tested in lower environments first, and destructive schema changes should be avoided unless they are planned separately with clear rollback and recovery procedures.

## Secret Management
Secrets such as database credentials and API keys must never be hardcoded in source code, Docker images, or pipeline YAML. The preferred design is to store runtime secrets in AWS Secrets Manager, since the application runs in AWS. Non-sensitive configuration can be stored in Systems Manager Parameter Store if needed.

Azure DevOps should only hold minimal secret references or securely stored pipeline variables when necessary. The ECS task should retrieve secrets at runtime using task definition secret injection or IAM-authorized access to Secrets Manager. This ensures secrets are not embedded into build artifacts and can be rotated independently of the application code.

Access to secrets should follow least privilege. The ECS task role should only read the secrets required by that service, and the Azure DevOps deployment role should only access what is necessary for deployment. Separate secrets should be maintained for each environment to prevent cross-environment exposure.

## Recommended Release Model
A practical delivery model is:
- Dev: automatic deployment on merge
- Test/QA: gated promotion of the same built artifact
- Production: manual approval, controlled deployment, smoke validation, and monitoring

This approach provides a secure, auditable, and low-risk CI/CD process that aligns well with AWS CDK, ECS Fargate, RDS, and Azure DevOps delivery practices.

