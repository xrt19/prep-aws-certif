# Tracking — Soal Latihan SAA-C03

> Total target: **250 soal** | 10 soal per set | 25 set

---

## Cara Lanjut Setelah Token Reset

Jika sesi terhenti di tengah jalan, gunakan prompt ini di sesi baru:

```
Lanjutkan membuat soal latihan SAA-C03. 
Baca soal-latihan secara keseluruhan (untuk tahu mana yang sudah dan belum), 
kemudian baca juga soal-latihan/tracking.md
untuk tahu set mana yang harus dibuat berikutnya dan topik apa yang sudah
dipakai. Lanjutkan dari set yang statusnya "In Progress" dulu, atau set
berikutnya yang statusnya "Belum". Setelah selesai, update tracking.md.
```

---

## Ringkasan Progress

| Domain | Target | Dibuat | Sisa |
|--------|--------|--------|------|
| Secure Architectures (30%) | 75 | 75 | 0 |
| Resilient Architectures (26%) | 65 | 60 | 5 |
| High-Performing (24%) | 60 | 60 | 0 |
| Cost-Optimized (20%) | 50 | 40 | 10 |
| Mixed | 0 | 0 | 0 |
| **Total** | **250** | **235** | **15** |

---

## Detail Per Set

### Secure Architectures (Set 01–07)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 01 | Selesai | IAM Identity Center/SSO, credential compromise, IAM Role EC2, S3 Object Lock, AWS Config, WAF, SCP Organizations, Secrets Manager rotation, CloudTrail, VPC Gateway Endpoint |
| 02 | Selesai | KMS CMK + EBS encryption, SCP tag enforcement, Cognito Identity Pool S3 access, Database Activity Streams, GuardDuty, IAM SourceIp condition, Interface VPC Endpoints, Security Hub, Lambda least privilege, IAM Access Analyzer |
| 03 | Selesai | Amazon Macie PII discovery, NACL vs SG explicit deny, ACM auto-renewal, Cross-account IAM Role, Permission Boundaries, S3 Block Public Access, Network Firewall DPI, IAM policy evaluation order, Lambda resource-based policy, ALB HTTP→HTTPS redirect |
| 04 | Selesai | AWS Shield Standard/Advanced, SSM Session Manager logging, AWS RAM resource sharing, S3 pre-signed URL, Amazon Inspector CVE scan, AWS Firewall Manager, KMS key policy precedence, Cognito User Pool + Identity Pool, Config auto-remediation SSM, RDS encryption at rest immutable |
| 05 | Selesai | VPC Flow Logs, S3 Access Points, IRSA EKS, Envelope encryption + KMS DEK, GuardDuty delegated admin, SSM Parameter Store SecureString/tiers, CloudTrail multi-region trail, AWS Artifact compliance reports, S3 Block Public Access level bucket, EFS default encryption + Config rule |
| 06 | Selesai | AWS Control Tower landing zone, S3 Glacier Vault Lock, Amazon Detective incident investigation, Customer Managed vs Inline Policy, STS session policy, SG referencing cross-VPC, WAF rate-based rules, S3 Object Ownership bucket owner enforced, Trusted Advisor security checks, Multi-tier SG chaining |
| 07 | Selesai | AWS Private CA, AWS Verified Access, CORS, AD Connector vs Managed AD, IAM role chaining 1 jam, CloudFront Signed URLs/Cookies, Gateway+Interface Endpoints, Service-Linked Role, VPC Peering vs TGW, multi-service security mapping, CloudTrail org trail + S3 Lock, KMS audit via CloudTrail, Lambda least privilege per-ARN, Config SG-attached-to-ENI, Config access-keys-rotated |

### Resilient Architectures (Set 08–13)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 08 | Selesai | RDS Multi-AZ failover, Scheduled scaling, SQS Visibility Timeout, Route 53 latency-based routing, ASG self-healing health checks, RDS PITR, S3 CRR, ALB vs NLB vs CLB, DR strategies RTO/RPO, SQS decoupling burst absorption |
| 09 | Selesai | RDS Proxy connection pooling, Aurora auto-scaling storage + failover, SNS fan-out pattern, SQS FIFO ordering + exactly-once, ASG Lifecycle Hooks, Aurora Global Database cross-region, Route 53 Failover routing + health check, Kinesis Data Streams replay, Step Functions orchestration, ElastiCache caching layer |
| 10 | Selesai | ECS Service self-healing Fargate, DynamoDB Global Tables multi-active, ALB weighted target groups blue/green, NLB Elastic IP static, DynamoDB Streams event-driven, Aurora Serverless v2, EventBridge content-based routing, API Gateway caching, S3 SRR vs CRR, Step scaling asymmetric scale |
| 11 | Selesai | SQS Long Polling reduce empty responses, Kinesis Firehose managed delivery S3, ASG cooldown period warm-up, DynamoDB DAX microsecond latency, Elastic Beanstalk Rolling deployment, EFS regional multi-AZ by default, CloudFront Origin Groups failover, Route 53 Geolocation routing, RDS Storage Auto Scaling, ECS Capacity Providers Cluster scaling |
| 12 | Selesai | ElastiCache Redis Multi-AZ failover, Lambda Reserved vs Provisioned Concurrency, FSx for Lustre HPC, AWS Elastic Disaster Recovery, AWS Backup centralized, DynamoDB On-Demand spiky workloads, FSx for Windows SMB AD, AWS Fault Injection Service chaos engineering, Route 53 weighted + health checks canary, Transit Gateway Peering inter-region |
| 13 | Selesai | SQS Delay Queue DelaySeconds, SNS Subscription Filter Policy, Connection Draining Deregistration Delay, ASG Mixed Instances Policy Spot+On-Demand, API Gateway WebSocket API, Lambda Event Source Mapping BisectOnFunctionError, ElastiCache Redis Cluster Mode sharding, AWS AppSync GraphQL real-time, DR Warm Standby RTO/RPO vs cost, RDS Multi-AZ failover brief interruption retry logic |

### High-Performing Architectures (Set 14–19)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 14 | Selesai | CloudFront CDN global delivery, Global Accelerator AWS backbone TCP/UDP, EC2 GPU P/G family ML, Cluster Placement Group HPC, EBS io2 Block Express highest IOPS, CloudFront as S3 cache read throttling, RDS Read Replica offload reads, Enhanced Networking ENA SR-IOV, Lambda cold start Provisioned Concurrency, S3 Multipart Upload parallel resume |
| 15 | Selesai | Redis vs Memcached feature comparison, DynamoDB hot partition key design, EC2 Instance Store ephemeral, Kinesis shard splitting throughput, Aurora Parallel Query storage-layer, Lambda@Edge vs CloudFront Functions, S3 Byte-Range Fetches partial retrieval, VPC Gateway Endpoint routing NAT bypass, ASG instance warm-up target tracking, Database indexing full scan vs index |
| 16 | Selesai | EC2 Spot Instances, DynamoDB GSI, S3 Select, Redshift distribution/sort key, Amazon OpenSearch, API Gateway caching, Lambda power tuning memory-CPU, ECS Fargate CPU units, Predictive Scaling, EFA OS-bypass RDMA MPI |
| 17 | Selesai | Athena Parquet/ORC partitioning, AWS Glue DPU, DynamoDB TTL, ElastiCache Redis Cluster Mode sharding, S3 Multipart Upload pre-signed URL, SageMaker Managed Spot Training, SageMaker Real-Time Inference, Kubernetes OOMKilled resource limits, Athena MSCK REPAIR TABLE Glue catalog, Kinesis Data Analytics Flink tumbling window |
| 18 | Selesai | RDS Read Replica dedicated analytics, CloudFront Functions simple redirect, AWS DMS CDC heterogeneous migration, Lambda Layers shared dependencies, Amazon MSK managed Kafka, RDS Proxy connection pooling Lambda, S3 Event Notification SQS Lambda pipeline, CloudFront in front of ALB static caching, FSx for Lustre HPC parallel file system, RDS Performance Insights Top SQL |
| 19 | Selesai | DynamoDB GSI composite sort key, Aurora Machine Learning SageMaker SQL, DynamoDB Streams Lambda OpenSearch CDC, SageMaker Blue/Green deployment, AWS Cloud Map ECS Service Discovery, CloudWatch PutMetricData custom metrics, Amazon Athena serverless ad-hoc SQL, Secrets Manager ECS integration rotation, Lambda VPC NAT Gateway internet+private, Global Accelerator anycast failover vs Route 53 |

### Cost-Optimized Architectures (Set 20–24)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 20 | Selesai | EC2 Reserved Instances payment options, EC2 start/stop scheduling dev/test, S3 Lifecycle Policy tiered storage classes, Aurora Serverless v2 ACU billing, S3 Intelligent-Tiering auto tiering, Spot Fleet checkpointing batch, Lambda no idle cost unpredictable traffic, VPC Gateway Endpoints NAT cost reduction, EC2 Instance Store ephemeral no cost, RI Sharing Organizations Consolidated Billing |
| 21 | Selesai | Redshift Serverless RPU billing, Compute Savings Plans flexibility, CloudFront Price Class, Fargate Spot interruptible, S3 Standard-IA minimum duration, CloudFront data transfer cost EC2-to-CloudFront free, DynamoDB On-Demand vs Provisioned, Cost Explorer Budgets Cost Allocation Tags, EBS gp3 vs gp2 independent IOPS, EMR Auto Termination Spot task nodes |
| 22 | Selesai | Lambda GB-second pricing vs Fargate long-running, serverless vs EC2 idle low-traffic, EBS DeleteOnTermination orphaned volumes, RDS snapshot manual vs automated cost, Elastic IP unassociated charge, Consolidated Billing volume discounts, Cost Explorer Savings Plans recommendations, S3 Lifecycle backup retention tiers, Compute Optimizer right-sizing, intra-VPC private IP free vs public IP NAT cost |
| 23 | Selesai | EKS Spot worker nodes Karpenter, CloudWatch Logs retention policy, SQS Long Polling API request cost, ElastiCache caching reduce RDS cost, S3 small objects aggregation request pricing, RI baseline + Spot burst mixed strategy, S3 cross-region transfer pricing, Budgets actions automated governance, Spot Fleet capacityOptimized HPC diversification, WAF Managed Rules vs custom rules |
| 24 | Belum | |

### Mixed (Set 25)

| Set | Status | Topik Soal |
|-----|--------|-----------|
| 25 | Belum | |

---

## Topik Yang Sudah Dipakai

> Diupdate setiap set selesai dibuat. Digunakan untuk menghindari duplikasi soal antar set.

### Secure Architectures
**Set 01:** IAM Identity Center/SSO, credential compromise response, IAM Role untuk EC2, S3 Object Lock Compliance mode, AWS Config change detection, AWS WAF SQLi/XSS, SCP Organizations, Secrets Manager auto-rotation, CloudTrail audit trail, VPC Gateway Endpoint S3

**Set 02:** KMS CMK + EBS encryption + key rotation, SCP tag enforcement (`aws:RequestTag`), Cognito Identity Pool + identity-based S3 access, Database Activity Streams RDS, GuardDuty threat detection, IAM `aws:SourceIp` condition, Interface VPC Endpoints PrivateLink, Security Hub findings aggregation, Lambda execution role least privilege, IAM Access Analyzer unused access

**Set 03:** Amazon Macie PII/sensitive data discovery, NACL explicit deny vs Security Groups, ACM auto-renewal + ALB, cross-account IAM Role + STS AssumeRole, IAM Permission Boundaries, S3 Block Public Access level akun, AWS Network Firewall DPI + Suricata, IAM policy evaluation order (Explicit Deny), Lambda resource-based policy cross-account, ALB HTTP→HTTPS redirect listener rule

**Set 04:** AWS Shield Standard vs Advanced, SSM Session Manager session logging, AWS RAM resource sharing, S3 pre-signed URL time-limited access, Amazon Inspector vulnerability assessment, AWS Firewall Manager centralized WAF, KMS key policy vs IAM policy precedence, Cognito User Pool + Identity Pool combined, AWS Config auto-remediation + SSM Automation, RDS encryption immutable at launch

**Set 05:** VPC Flow Logs network traffic metadata, S3 Access Points multi-tenant, IRSA IAM Roles for Service Accounts EKS, envelope encryption + KMS DEK 4KB limit, GuardDuty delegated administrator multi-account, SSM Parameter Store SecureString + Standard/Advanced tier, CloudTrail multi-region trail, AWS Artifact compliance reports ISO/SOC, S3 Block Public Access level bucket, EFS default encryption + Config rule defense-in-depth

**Set 06:** AWS Control Tower landing zone + account factory, S3 Glacier Vault Lock WORM compliance, Amazon Detective incident investigation graph analysis, Customer Managed Policy vs Inline Policy reusability, STS session policy AssumeRole effective permissions intersection, Security Group cross-VPC referencing, WAF rate-based rules credential stuffing, S3 Object Ownership bucket owner enforced, AWS Trusted Advisor security checks, multi-tier SG chaining defense in depth

**Set 07:** AWS Private CA internal TLS mTLS, AWS Verified Access zero-trust tanpa VPN, CORS cross-origin API Gateway/S3, AD Connector vs AWS Managed Microsoft AD, IAM role chaining 1 jam session limit, CloudFront Signed URLs vs Signed Cookies, Gateway Endpoints + Interface Endpoints kombinasi, Service-Linked Role AWS service permissions, VPC Peering non-transitive vs Transit Gateway, multi-service security architecture mapping, CloudTrail organization trail + S3 Object Lock Log Archive, KMS API calls audit via CloudTrail, Lambda execution role least privilege per-ARN, Config rule ec2-security-group-attached-to-eni, Config rule access-keys-rotated 90 hari

### Resilient Architectures
**Set 08:** RDS Multi-AZ automatic failover, scheduled scaling proactive, SQS Visibility Timeout long-running processing, Route 53 latency-based routing + health checks, ASG self-healing terminate & replace, RDS Point-in-Time Recovery, S3 Cross-Region Replication, ALB vs NLB vs CLB Layer 7 path routing, DR strategies RTO/RPO Multi-Site Active-Active, SQS decoupling burst absorption 14-day retention

**Set 09:** RDS Proxy connection pooling Lambda, Aurora auto-scaling storage + failover < 30 detik, SNS fan-out one-to-many pattern, SQS FIFO ordering + exactly-once MessageGroupId, ASG Lifecycle Hooks pre-terminate custom action, Aurora Global Database cross-region < 1 detik lag, Route 53 Failover routing + health check requirement, Kinesis Data Streams replay multiple consumers, Step Functions workflow orchestration state machine, ElastiCache caching layer sub-millisecond (Multi-AZ tidak serve reads)

**Set 10:** ECS Service desired count self-healing Fargate, DynamoDB Global Tables multi-region multi-active, ALB weighted target groups blue/green canary, NLB Elastic IP static TCP Layer 4, DynamoDB Streams event-driven Lambda trigger, Aurora Serverless v2 near-zero idle, EventBridge content-based routing event bus, API Gateway caching TTL reduce Lambda invocations, S3 SRR vs CRR same-region replication, Step scaling multi-step asymmetric scale out vs scale in

**Set 11:** SQS Long Polling WaitTimeSeconds reduce empty responses, Kinesis Data Firehose managed delivery transform S3, ASG cooldown period instance warm-up, DynamoDB DAX microsecond latency hot items, Elastic Beanstalk Rolling deployment no downtime, EFS regional redundancy multi-AZ by default, CloudFront Origin Groups origin failover, Route 53 Geolocation routing country-level default record, RDS Storage Auto Scaling no downtime, ECS Capacity Providers Cluster Auto Scaling EC2 infrastructure

**Set 12:** ElastiCache Redis Multi-AZ automatic failover, Lambda Reserved Concurrency vs Provisioned Concurrency, FSx for Lustre HPC parallel file system, AWS Elastic Disaster Recovery continuous replication on-premise, AWS Backup centralized multi-service, DynamoDB On-Demand vs Provisioned spiky workloads, FSx for Windows File Server SMB NTFS AD integration, AWS Fault Injection Service chaos engineering, Route 53 weighted routing + health checks canary failover, Transit Gateway Peering inter-region hub-and-spoke

**Set 13:** SQS Delay Queue DelaySeconds vs Visibility Timeout, SNS Subscription Filter Policy content-based delivery, Connection Draining Deregistration Delay in-flight protection, ASG Mixed Instances Policy Spot+On-Demand fallback, API Gateway WebSocket API bidirectional persistent, Lambda Event Source Mapping BisectOnFunctionError poison pill, ElastiCache Redis Cluster Mode Enabled sharding partial failure, AWS AppSync managed GraphQL real-time subscriptions, DR Warm Standby RTO/RPO vs cost trade-off, RDS Multi-AZ failover brief interruption retry logic requirement

### High-Performing Architectures
**Set 14:** CloudFront CDN edge locations global delivery, Global Accelerator AWS backbone TCP/UDP anycast vs CloudFront, EC2 GPU P/G family ML training vs inference, Cluster Placement Group HPC low latency EFA, EBS io2 Block Express highest IOPS sub-ms, CloudFront as S3 cache read throttle solution, RDS Read Replica offload reads (Multi-AZ tidak serve reads), Enhanced Networking ENA SR-IOV vs EFA, Lambda cold start Provisioned Concurrency, S3 Multipart Upload parallel parts resume on failure

**Set 15:** Redis vs Memcached feature comparison data structures persistence pub/sub, DynamoDB hot partition key design cardinality write sharding, EC2 Instance Store ephemeral high-performance scratch space, Kinesis shard splitting throughput scaling Enhanced Fan-Out read vs write, Aurora Parallel Query storage-layer processing OLAP, Lambda@Edge vs CloudFront Functions edge personalization, S3 Byte-Range Fetches partial object retrieval HTTP Range, VPC Gateway Endpoint route table DynamoDB S3 NAT bypass, ASG instance warm-up period target tracking metric accuracy, Database indexing full table scan vs index scan query optimization

**Set 16:** EC2 Spot Instances fault-tolerant batch workloads interruption checkpoint, DynamoDB GSI Global Secondary Index query pattern access pattern design projection, S3 Select SQL query CSV JSON Parquet push-down filter data transfer reduction, Redshift distribution key DISTKEY DISTYLE sort key zone maps MPP architecture query optimization, Amazon OpenSearch full-text search faceted search inverted index real-time suggestions, API Gateway caching response cache TTL cache key Lambda invocation reduction, Lambda power tuning memory-CPU proportional relationship execution time cost optimization, ECS Fargate task definition CPU units CPU throttling CFS container performance, Predictive Scaling forecast ML cyclical traffic patterns proactive pre-scaling, EFA Elastic Fabric Adapter OS-bypass RDMA MPI HPC inter-node communication

**Set 17:** Athena query optimization columnar format Parquet ORC partitioning partition pruning data scan cost, AWS Glue DPU Data Processing Units Apache Spark parallelism Auto Scaling, DynamoDB TTL Time to Live automatic expiry no WCU consumption, ElastiCache Redis Cluster Mode Enabled sharding hash slots horizontal scaling, S3 Multipart Upload pre-signed URL direct-to-S3 upload resume on failure, SageMaker Managed Spot Training checkpoint interruption recovery cost optimization, SageMaker Real-Time Inference vs Serverless Inference vs Batch Transform latency requirements, Kubernetes OOMKilled resource requests limits memory management pod spec, Athena partition management MSCK REPAIR TABLE ALTER TABLE ADD PARTITION Glue Data Catalog, Kinesis Data Analytics Apache Flink managed streaming tumbling window aggregation IoT analytics

**Set 18:** RDS Read Replica dedicated per workload analytics isolation instance type flexibility, CloudFront Functions vs Lambda@Edge simple redirect URL rewriting cost comparison, AWS DMS Database Migration Service CDC Change Data Capture heterogeneous migration Oracle Aurora minimal downtime, Lambda Layers shared dependencies binary large packages size limits 250MB, Amazon MSK Managed Streaming Kafka API compatibility self-managed migration, RDS Proxy connection pooling Lambda too many connections VPC, S3 Event Notification SQS Lambda event-driven pipeline buffer retry decoupling, CloudFront in front of ALB static dynamic content caching cache behaviors, FSx for Lustre HPC high-performance parallel file system POSIX S3 integration sub-ms, RDS Performance Insights DB load Top SQL wait events query monitoring

**Set 19:** DynamoDB GSI composite key sort key for ordering query optimization, Aurora Machine Learning SageMaker Comprehend SQL function ML inference in-database, DynamoDB Streams Lambda OpenSearch CDC change data capture audit trail real-time sync, SageMaker endpoint blue/green deployment canary traffic shifting zero-downtime model update, AWS Cloud Map ECS Service Discovery DNS-based dynamic registration deregistration, CloudWatch PutMetricData API custom business metrics namespaces dimensions, Amazon Athena serverless ad-hoc interactive SQL S3 data lake pay-per-scan, Secrets Manager ECS native integration secrets injection automatic rotation database credentials, Lambda VPC configuration NAT Gateway private subnet internet access RDS pattern, Global Accelerator anycast static IP instant failover AWS backbone vs Route 53 DNS propagation delay

**Set 20:** EC2 Reserved Instances No Partial All Upfront payment options steady-state workload 1-year 3-year, EC2 start/stop scheduling Instance Scheduler EventBridge Lambda dev/test cost optimization RI billing when stopped, S3 Lifecycle Policy tiered storage Standard Standard-IA Glacier Flexible Deep Archive compliance archiving 7-year retention, Aurora Serverless v2 ACU billing scale to minimum intermittent workload vs RDS per-hour, S3 Intelligent-Tiering automatic tier migration Frequent Infrequent Archive no retrieval fee unknown access patterns, Spot Fleet allocation strategies checkpointing batch processing 90% savings, Lambda pricing per invocation per ms no idle cost unpredictable traffic serverless, VPC Gateway Endpoints S3 DynamoDB NAT Gateway cost components data processing per GB, EC2 Instance Store ephemeral storage no additional charge intermediate temporary data, RI Sharing AWS Organizations Consolidated Billing multi-account benefit sharing

**Set 21:** Redshift Serverless RPU per-second billing idle cost elimination intermittent analytics, Compute Savings Plans vs Standard RI vs Convertible RI flexibility EC2 Lambda Fargate, CloudFront Price Class 100 200 All edge location cost vs latency trade-off, Fargate Spot 70% discount interruptible fault-tolerant container workloads 2-minute notice, S3 Standard-IA minimum 30-day storage duration retrieval fee known access patterns lifecycle, CloudFront data transfer pricing EC2-to-CloudFront free origin fetch caching cost reduction, DynamoDB On-Demand vs Provisioned break-even utilization 20% threshold unused capacity, AWS Cost Explorer AWS Budgets Cost Allocation Tags cost visibility governance, EBS gp3 vs gp2 independent IOPS provisioning baseline 3000 IOPS cost 20% cheaper, EMR Auto Termination idle timeout Spot task nodes vs On-Demand core nodes HDFS safety

**Set 22:** Lambda GB-second pricing long-running heavy tasks vs Fargate Spot cost comparison, API Gateway Lambda serverless vs EC2 idle cost low-traffic infrequent applications, EBS DeleteOnTermination flag orphaned unattached volumes cost prevention, RDS manual snapshot vs automated backup pricing free tier manual snapshot accumulation, Elastic IP unassociated charge $0.005/hour vs free when attached running instance, AWS Organizations Consolidated Billing volume discounts tiered pricing S3 data transfer aggregation, Cost Explorer Savings Plans recommendations lookback period commitment optimization projected savings, S3 Lifecycle rules backup retention harian mingguan bulanan tiered by age Glacier Deep Archive, AWS Compute Optimizer ML right-sizing EC2 Lambda EBS Fargate ASG over-provisioned, intra-VPC private IP data transfer free vs public IP NAT Gateway misconfiguration charge

**Set 23:** EKS cost optimization Karpenter Cluster Autoscaler Spot worker nodes On-Demand mix node diversification, CloudWatch Logs retention policy per log group storage cost Never Expire anti-pattern, SQS Long Polling WaitTimeSeconds 20 short polling vs long polling API request cost reduction, ElastiCache caching layer reduce RDS load downsize instance cost cache-aside pattern, S3 API request pricing small objects aggregation ZIP Parquet PUT GET cost per million, RI baseline capacity Spot burst capacity ASG Mixed Instances Policy On-Demand fallback, S3 cross-region data transfer $0.02/GB vs same-region free S3 CRR alternative, AWS Budgets budget actions IAM SCP enforcement automated cost governance dev environment sprawl, Spot Fleet capacityOptimized strategy diversification multiple pools cliff event prevention HPC checkpoint, AWS WAF Managed Rules rule groups flat pricing vs custom rules per-rule pricing security cost

### Mixed
*(belum ada)*
