
# Observability & Security Plan

## Observability Plan
- Enable CloudWatch metrics, logs, and alarms for RDS, ECS, and the Application Load Balancer.
- For RDS, monitor CPU, memory-related indicators, storage usage, database connections, read/write latency, and free storage space. Enable enhanced monitoring and automated backups.
- For ECS Fargate, capture container stdout/stderr logs in CloudWatch Logs, and monitor CPU utilization, memory utilization, task health, restart counts, and service scaling events.
- For the Application Load Balancer, monitor request count, target response time, HTTP 4xx/5xx errors, healthy/unhealthy target counts, and latency.
- Create CloudWatch alarms for critical thresholds such as high ECS CPU/memory, unhealthy ALB targets, elevated 5xx errors, RDS storage pressure, or database connection spikes.
- Use centralised dashboards to provide a single view of infrastructure health, application health, and traffic behaviour.
- Add structured application logging with correlation/request IDs so issues can be traced across ALB, ECS, and database interactions.
- Where possible, include distributed tracing (for example OpenTelemetry/X-Ray) to improve root-cause analysis across services.
- Retain logs based on environment and compliance requirements, with longer retention for production.

## Security Plan
- Place RDS in private subnets with no public access, and allow inbound traffic only from the ECS service security group.
- Restrict ECS task networking using least-privilege security groups and tightly controlled outbound access where practical.
- Store database credentials and API keys in AWS Secrets Manager and inject them securely into ECS tasks at runtime.
- Encrypt data at rest using AWS-managed or customer-managed KMS keys for RDS, logs, and secrets.
- Encrypt data in transit using HTTPS/TLS between clients and the ALB, and secure connections from application to database where supported.
- Enable IAM least privilege for task roles, execution roles, and deployment roles. Avoid broad permissions such as `*:*`.
- Turn on CloudTrail, VPC Flow Logs, and ALB access logs for auditing and investigation.
- Protect the public entry point with WAF rules to reduce common web threats such as SQL injection, bad bots, and excessive request rates.
- Ensure container images are scanned for vulnerabilities before deployment, and keep base images patched and minimal.
- Enable automated backups, snapshot retention, and tested restore procedures for RDS to support resilience and recovery.
- Apply environment separation so dev, test, and prod workloads, secrets, and access paths are isolated.
- Use deployment approvals and change controls for production to reduce the chance of unsafe releases.

## Key Risks and Mitigations
- Risk: Database exposed publicly  
  Mitigation: Private subnets, no public endpoint, SG restriction to ECS only.
- Risk: Secret leakage in code or pipeline  
  Mitigation: Secrets Manager, secret masking, runtime secret injection.
- Risk: Service outage from unhealthy deployment  
  Mitigation: ALB health checks, rolling/blue-green deployment, CloudWatch alarms.
- Risk: Undetected failures in queue processing or application flow  
  Mitigation: Metrics, structured logs, tracing, and alerting on error rates and unhealthy tasks.
- Risk: Data loss or difficult recovery  
  Mitigation: Automated backups, snapshots, retention policies, and restore testing.
```

