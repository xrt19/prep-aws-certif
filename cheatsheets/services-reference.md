# Cheat Sheet — AWS Services Quick Reference

> One-liner untuk setiap service yang sering keluar di exam.

---

## Compute

| Service | One-liner |
|---------|-----------|
| **EC2** | Virtual machines di cloud |
| **Lambda** | Serverless functions, max 15 menit, event-driven |
| **ECS** | Container orchestration (AWS-native) |
| **EKS** | Managed Kubernetes |
| **Fargate** | Serverless compute untuk container (ECS/EKS) |
| **Elastic Beanstalk** | PaaS — deploy app tanpa manage infra |
| **Batch** | Managed batch computing jobs |
| **Lightsail** | Simple VPS untuk workload kecil |

## Storage

| Service | One-liner |
|---------|-----------|
| **S3** | Object storage, unlimited, 99.999999999% durability |
| **EBS** | Block storage untuk EC2 (single AZ, satu instance) |
| **EFS** | Managed NFS, bisa di-mount multi-AZ dan multi-instance |
| **FSx for Windows** | Managed Windows File Server (SMB) |
| **FSx for Lustre** | High-performance file system, terintegrasi S3 |
| **Storage Gateway** | Hybrid storage: on-premise ↔ AWS |
| **Snow Family** | Physical device untuk migrate/transfer data besar |

## Database

| Service | One-liner |
|---------|-----------|
| **RDS** | Managed relational DB (MySQL, PostgreSQL, Oracle, SQL Server) |
| **Aurora** | AWS-native relational DB, 5x faster than MySQL |
| **DynamoDB** | Serverless NoSQL, single-digit ms latency |
| **ElastiCache** | In-memory caching (Redis / Memcached) |
| **Redshift** | Data warehouse, OLAP, petabyte-scale |
| **DocumentDB** | MongoDB-compatible managed DB |
| **Neptune** | Managed graph database |
| **QLDB** | Immutable ledger database |
| **Keyspaces** | Managed Cassandra-compatible DB |

## Networking

| Service | One-liner |
|---------|-----------|
| **VPC** | Isolated virtual network di AWS |
| **Route 53** | DNS + domain registration + routing policies |
| **CloudFront** | CDN — cache content di edge locations global |
| **API Gateway** | Managed API (REST/HTTP/WebSocket), throttling, auth |
| **ELB (ALB/NLB)** | Load balancing untuk EC2, containers, Lambda |
| **Global Accelerator** | Static IPs + Anycast routing via AWS backbone |
| **Direct Connect** | Dedicated physical connection ke AWS |
| **Transit Gateway** | Hub untuk koneksi banyak VPC |
| **PrivateLink** | Private endpoint untuk services AWS / third-party |

## Security & Identity

| Service | One-liner |
|---------|-----------|
| **IAM** | Users, roles, policies — who can do what |
| **STS** | Short-term credentials via AssumeRole |
| **KMS** | Managed encryption keys |
| **Secrets Manager** | Store & auto-rotate secrets |
| **SSM Parameter Store** | Config & secrets store (gratis untuk standard) |
| **ACM** | Free SSL/TLS certificates |
| **GuardDuty** | Threat detection via ML |
| **Inspector** | Vulnerability scanning (EC2, Lambda, ECR) |
| **Macie** | Sensitive data discovery di S3 |
| **WAF** | Layer 7 firewall (SQLi, XSS, rules) |
| **Shield** | DDoS protection (Standard=free, Advanced=paid) |
| **Cognito** | User authentication untuk web/mobile apps |
| **Organizations** | Multi-account management + SCPs |

## Messaging & Integration

| Service | One-liner |
|---------|-----------|
| **SQS** | Message queue — decouple producers & consumers |
| **SNS** | Pub/sub — push notifications ke multiple subscribers |
| **EventBridge** | Event bus — route events antar services |
| **Step Functions** | Serverless workflow orchestration |
| **Kinesis Data Streams** | Real-time data streaming |
| **Kinesis Firehose** | Load streaming data ke S3/Redshift/OpenSearch |
| **MSK** | Managed Apache Kafka |
| **AppSync** | Managed GraphQL API |

## Developer Tools & Monitoring

| Service | One-liner |
|---------|-----------|
| **CloudWatch** | Metrics, logs, alarms, dashboards |
| **CloudTrail** | API call logging (who did what, when) |
| **Config** | Track resource configuration changes over time |
| **X-Ray** | Distributed tracing untuk aplikasi |
| **CodePipeline** | CI/CD pipeline orchestration |
| **CodeBuild** | Managed build service |
| **CodeDeploy** | Automated deployment ke EC2/Lambda/ECS |
| **CloudFormation** | IaC — provision AWS resources via templates |
| **CDK** | Define CloudFormation via code (TypeScript, Python, etc) |

## Analytics

| Service | One-liner |
|---------|-----------|
| **Athena** | SQL query langsung ke S3 (serverless) |
| **Glue** | ETL service + Data Catalog |
| **EMR** | Managed Hadoop/Spark cluster |
| **QuickSight** | Business intelligence & visualization |
| **OpenSearch** | Search & analytics (ElasticSearch managed) |
| **Lake Formation** | Setup & manage data lake di S3 |

---

## GCP → AWS Service Mapping

| GCP | AWS Equivalent |
|-----|---------------|
| GCE | EC2 |
| GCS | S3 |
| Cloud SQL | RDS |
| Cloud Spanner | Aurora (closest) |
| BigQuery | Redshift + Athena |
| GKE | EKS |
| Cloud Run | Fargate + Lambda |
| Cloud Functions | Lambda |
| Cloud IAM | IAM |
| Cloud KMS | KMS |
| Pub/Sub | SNS + SQS |
| Cloud Logging | CloudWatch Logs |
| Cloud Monitoring | CloudWatch |
| VPC | VPC |
| Cloud Armor | WAF + Shield |
| Cloud CDN | CloudFront |
| Cloud DNS | Route 53 |
| Dataflow | Kinesis + Glue |
| Firestore | DynamoDB |
