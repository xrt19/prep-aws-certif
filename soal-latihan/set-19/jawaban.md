# Jawaban — Set 19
## Domain: High-Performing Architectures

---

## Soal 1 — Jawaban: C

**GSI dengan `category` (partition key) + `price` (sort key)**

**Kenapa C:**
DynamoDB query hanya bisa menggunakan primary key atau GSI/LSI. Tanpa index yang tepat, query filter + sort = full table scan yang mahal dan lambat.

**GSI design untuk query ini:**
```
GSI: CategoryPriceIndex
  Partition key: category (STRING)
  Sort key: price (NUMBER)
```

Query:
```
aws dynamodb query \
  --table-name Products \
  --index-name CategoryPriceIndex \
  --key-condition-expression "category = :cat" \
  --expression-attribute-values '{":cat": {"S": "electronics"}}'
```

DynamoDB otomatis return items diurutkan by `price` ascending (atau `ScanIndexForward: false` untuk descending). Ini adalah O(log n) index lookup, bukan O(n) full scan.

**Kenapa bukan yang lain:**
- **A** — DAX cache hasil query, tapi jika query-nya adalah full scan, DAX hanya cache hasil scan tersebut. Invocation pertama tetap lambat. Root cause (missing index) tidak diselesaikan.
- **B** — Migrasi ke Aurora adalah perubahan arsitektur besar dan mahal. DynamoDB GSI dirancang persis untuk use case ini — query pattern fleksibel tanpa harus pindah database.
- **D** — Parallel Scan membagi tabel menjadi beberapa segment dan scan paralel, memang lebih cepat dari sequential scan. Tapi masih O(n) — mahal dari sisi RCU dan cost. Bukan solusi optimal.

**Konsep yang diuji:** DynamoDB GSI composite key, query optimization, access pattern design, sort key untuk ordering.

---

## Soal 2 — Jawaban: C

**Aurora Machine Learning**

**Kenapa C:**
Aurora Machine Learning memungkinkan integrasi ML langsung ke dalam SQL workflow:

**Cara kerja:**
1. Aurora connect ke SageMaker endpoint atau Amazon Comprehend
2. Developer buat SQL function yang memanggil ML model:
```sql
-- Fraud detection via SageMaker
SELECT transaction_id,
       aws_sagemaker.invoke_endpoint(
           'fraud-detection-endpoint',
           NULL,
           amount, merchant_category, location
       ) AS fraud_score
FROM transactions
WHERE fraud_score > 0.8;

-- Sentiment analysis via Comprehend
SELECT review_id,
       aws_comprehend.detect_dominant_language(review_text) AS language,
       aws_comprehend.detect_sentiment(review_text, 'en') AS sentiment
FROM product_reviews;
```
3. Output ML menjadi kolom biasa dalam result set — bisa dipakai di WHERE, JOIN, dll

**Kenapa bukan yang lain:**
- **A** — Aurora Parallel Query untuk akselerasi **analytics queries** yang melakukan full scan — pushes query processing ke storage layer. Tidak ada integrasi ML.
- **B** — Aurora Serverless v2 untuk **scaling capacity** Aurora secara otomatis berdasarkan load. Tidak ada fungsi ML inference.
- **D** — Aurora Global Database untuk **cross-region replication** dan DR. Secondary region bisa digunakan untuk read, tapi tidak untuk ML inference dalam SQL.

**Konsep yang diuji:** Aurora Machine Learning, SageMaker integration, Comprehend integration, ML inference via SQL.

---

## Soal 3 — Jawaban: C

**DynamoDB Streams → Lambda → OpenSearch**

**Kenapa C:**
DynamoDB Streams adalah **change data capture** yang built-in ke DynamoDB:
- Setiap INSERT, UPDATE, DELETE di-capture sebagai stream record
- Records tersedia selama **24 jam** di stream
- **Ordered** — perubahan pada satu item di-deliver dalam urutan yang benar
- Stream record berisi: old image, new image, atau keduanya (dikonfigurasi via `StreamViewType`)

Pattern `DynamoDB Streams → Lambda → OpenSearch`:
1. DynamoDB Streams capture setiap perubahan item
2. Lambda di-trigger otomatis oleh stream (Event Source Mapping)
3. Lambda transform data dan index ke OpenSearch via HTTP API
4. OpenSearch menyimpan dokumen yang bisa di-search

Untuk audit trail: Lambda juga bisa simpan setiap perubahan ke S3 atau DynamoDB table lain sebagai immutable audit log.

**Kenapa bukan yang lain:**
- **A** — Lambda scheduled scan setiap menit: (1) Tidak bisa detect DELETE — item sudah hilang; (2) Tidak bisa track nilai sebelum UPDATE; (3) Latency hingga 1 menit; (4) Scan konsumsi RCU mahal.
- **B** — EventBridge Pipes bisa connect DynamoDB Streams ke berbagai targets, tapi tidak bisa langsung ke OpenSearch. Butuh Lambda atau step transformation. DynamoDB Streams → Lambda lebih straightforward dan lebih mature.
- **D** — Kinesis Data Firehose tidak bisa terhubung langsung ke DynamoDB sebagai source. Firehose menerima data dari Kinesis Data Streams atau direct PUT, bukan dari DynamoDB.

**Konsep yang diuji:** DynamoDB Streams, CDC, Lambda Event Source Mapping, change data capture pattern, OpenSearch sync.

---

## Soal 4 — Jawaban: D

**SageMaker Endpoint Blue/Green Deployment**

**Kenapa D:**
SageMaker Real-Time Endpoint mendukung **deployment strategies** untuk zero-downtime update:

**Blue/Green Deployment:**
- "Blue" = current production endpoint (model lama)
- "Green" = new endpoint (model baru) yang di-provision di background
- Traffic bisa di-shift secara gradual:
  - **Canary**: shift 10% traffic ke green, monitor, lalu shift 100%
  - **Linear**: shift bertahap 10% setiap N menit
  - **All-at-once**: langsung shift 100% ke green
- Jika metric (error rate, latency) di green buruk → **automatic rollback** ke blue
- Tidak ada interruption ke incoming requests selama proses

Ini identik dengan blue/green deployment di application tier, tapi di-managed oleh SageMaker.

**Kenapa bukan yang lain:**
- **A** — Batch Transform untuk offline inference — tidak ada real-time endpoint, tidak ada concept blue/green. Proses batch selesai lalu done.
- **B** — Serverless Inference: ada cold start (tidak cocok untuk < 100ms requirement), dan scaling terbatas dibanding real-time endpoint untuk ribuan req/s.
- **C** — Model Registry untuk **versioning dan governance** — track model versions, approval workflow, lineage. Penting untuk MLOps, tapi bukan yang mengeksekusi zero-downtime deployment. Model Registry dikombinasikan dengan endpoint update yang menggunakan blue/green.

**Konsep yang diuji:** SageMaker endpoint blue/green deployment, canary deployment, zero-downtime model update, traffic shifting.

---

## Soal 5 — Jawaban: C

**AWS Cloud Map + ECS Service Discovery**

**Kenapa C:**
ECS Service Discovery terintegrasi dengan **AWS Cloud Map** untuk DNS-based service discovery:

**Cara kerja:**
1. Buat Cloud Map namespace: `internal` (private DNS namespace)
2. Configure ECS service dengan service discovery — setiap service punya nama: `serviceb.internal`
3. Saat ECS task start → otomatis register IP:port ke Cloud Map
4. Saat ECS task stop/unhealthy → otomatis deregister dari Cloud Map
5. Service A query DNS `serviceb.internal` → dapat list IP task Service B yang healthy
6. Client-side load balancing: Service A pilih salah satu IP dari hasil DNS

Tidak butuh ALB untuk internal service-to-service communication → kurangi latency (tidak ada hop ke load balancer), kurangi cost.

**Kenapa bukan yang lain:**
- **A** — Route 53 Private Hosted Zone bisa digunakan, tapi tidak terintegrasi dengan ECS task lifecycle. Harus manually manage A records saat task scale up/down — tidak praktis dan error-prone.
- **B** — VPC Lattice adalah service untuk **service-to-service networking** yang lebih advanced (support cross-account, cross-VPC, policies), tapi lebih complex untuk setup. Untuk ECS internal service discovery dalam satu VPC, Cloud Map lebih sederhana dan native.
- **D** — Internal ALB memang bisa bekerja, tapi soal minta "menghilangkan dependency ke load balancer". ALB juga menambah latency (extra hop) dan biaya (per hour + per LCU).

**Konsep yang diuji:** AWS Cloud Map, ECS Service Discovery, DNS-based discovery, service-to-service communication, dynamic registration.

---

## Soal 6 — Jawaban: C

**CloudWatch PutMetricData API**

**Kenapa C:**
Untuk **custom business metrics** yang dihasilkan aplikasi (bukan OS/infrastructure metrics), cara yang tepat adalah langsung panggil CloudWatch API:

```python
import boto3
cloudwatch = boto3.client('cloudwatch')

cloudwatch.put_metric_data(
    Namespace='MyApp/Business',
    MetricData=[
        {
            'MetricName': 'OrdersPerMinute',
            'Dimensions': [
                {'Name': 'Environment', 'Value': 'production'},
                {'Name': 'Region', 'Value': 'us-east-1'}
            ],
            'Value': 142,
            'Unit': 'Count'
        }
    ]
)
```

Metrics tersedia di CloudWatch dalam < 1 menit, bisa buat alarms, dashboards, anomaly detection.

**Kenapa bukan yang lain:**
- **A** — CloudWatch Agent untuk mengumpulkan **OS-level dan application-level metrics** dari EC2 (CPU per process, memory, disk I/O, custom log-based metrics). Bukan untuk aplikasi yang ingin publish business metrics via API. CloudWatch Agent memerlukan config file dan install di setiap instance.
- **B** — Parse metrics dari logs via CloudWatch Logs Insights atau Metric Filters bisa bekerja, tapi ini indirect approach dengan latency lebih tinggi. PutMetricData langsung lebih simple dan real-time.
- **D** — CloudTrail untuk **audit trail API calls** ke AWS services. Tidak bisa menerima custom business metrics dari aplikasi.

**Konsep yang diuji:** CloudWatch custom metrics, PutMetricData API, namespaces, dimensions, business metrics vs infrastructure metrics.

---

## Soal 7 — Jawaban: D

**Amazon Athena**

**Kenapa D:**
Athena adalah pilihan ideal untuk use case ini karena:
- **Serverless** — tidak ada cluster yang perlu di-provision, manage, atau scale
- **SQL interface** — analysts yang familiar SQL bisa langsung query tanpa belajar tool baru
- **Query S3 directly** — tidak perlu load data ke database; data tetap di S3 data lake
- **Pay per query** — $5 per TB data yang di-scan; jika jarang query, tidak ada idle cost
- **Ad-hoc friendly** — bisa query sesekali tanpa commitment infrastructure
- Support berbagai format: CSV, JSON, Parquet, ORC, Avro

**Perbandingan dengan opsi lain untuk use case ini:**

| | Athena | Redshift | EMR |
|--|--------|----------|-----|
| Setup | Zero | Hours/days | Hours |
| Idle cost | $0 | Tinggi | Tinggi |
| SQL | Ya | Ya | Terbatas |
| Ad-hoc | Excellent | OK | Buruk |
| Query S3 | Native | Via Spectrum | Ya |

**Kenapa bukan yang lain:**
- **A** — Redshift butuh provision cluster (atau Redshift Serverless), loading data ke Redshift, dan ada biaya idle. Over-engineered untuk ad-hoc queries yang tidak terjadwal.
- **B** — EMR untuk large-scale data processing dengan Hadoop/Spark/Hive. Perlu manage cluster, steep learning curve untuk analysts, dan costly untuk ad-hoc.
- **C** — Glue ETL untuk **data transformation pipeline**, bukan untuk interactive queries oleh analysts.

**Konsep yang diuji:** Amazon Athena, serverless query, ad-hoc analytics, pay-per-scan, S3 data lake querying.

---

## Soal 8 — Jawaban: C

**AWS Secrets Manager dengan ECS native integration**

**Kenapa C:**
ECS memiliki **native integration dengan Secrets Manager dan SSM Parameter Store** untuk inject secrets ke container:

```json
{
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "myapp:latest",
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789:secret:prod/db/password"
        }
      ]
    }
  ]
}
```

ECS agent fetch secret dari Secrets Manager saat task launch dan inject sebagai environment variable ke container. Tidak ada credentials di task definition code atau Docker image.

**Keunggulan Secrets Manager dibanding SSM Parameter Store:**
- **Automatic rotation** built-in (Lambda function untuk rotate, support RDS natively)
- Support versioning secrets (AWSCURRENT, AWSPREVIOUS)
- Native support untuk database credentials rotation (RDS, Redshift, DocumentDB)

**Kenapa bukan yang lain:**
- **A** — S3 object untuk credentials tidak mendukung automatic rotation, tidak ada native ECS integration untuk inject ke container, dan butuh kode di aplikasi untuk download dan parse.
- **B** — SSM Parameter Store bisa digunakan (ECS juga support referensi ke SSM), tapi tidak support automatic rotation untuk database credentials seperti Secrets Manager. Untuk requirement "otomatis rotate", Secrets Manager lebih tepat.
- **D** — Enkripsi dengan KMS dan simpan di task definition masih berarti credentials (meskipun terenkripsi) ada di task definition — bukan best practice. Juga tidak support rotation.

**Konsep yang diuji:** Secrets Manager ECS integration, automatic rotation, secrets injection, secure credential management.

---

## Soal 9 — Jawaban: D

**Lambda di VPC dengan NAT Gateway**

**Kenapa D:**
Ini adalah arsitektur standar untuk Lambda yang butuh **akses ke kedua**: resources private di VPC dan internet:

```
Internet
    ↓
NAT Gateway (public subnet)
    ↑
Lambda Function (private subnet) → RDS (private subnet)
```

**Detail konfigurasi:**
- Lambda di-attach ke VPC, ditempatkan di **private subnet**
- Private subnet memiliki route table: `0.0.0.0/0 → NAT Gateway`
- NAT Gateway di **public subnet** dengan Elastic IP
- Lambda bisa akses RDS di private subnet via VPC internal routing
- Lambda bisa akses internet via NAT Gateway → Internet Gateway

**Kenapa bukan yang lain:**
- **A** — Dua Lambda functions bisa jadi solusi, tapi tidak efisien jika logika bisnis sama — duplikasi kode, dua functions yang perlu di-maintain. Juga tidak menjawab requirement "Lambda juga butuh akses ke RDS" dalam satu function.
- **B** — Ini salah. Lambda dalam VPC dengan NAT Gateway bisa akses internet. Yang tidak bisa adalah Lambda dalam VPC **tanpa NAT Gateway** (di private subnet tanpa route ke internet).
- **C** — VPC Peering untuk koneksi antar dua VPC yang berbeda. Jika Lambda dan RDS sudah di VPC yang sama, VPC Peering tidak diperlukan.

**Konsep yang diuji:** Lambda VPC configuration, NAT Gateway untuk internet access dari private subnet, Lambda + RDS architecture pattern.

---

## Soal 10 — Jawaban: D

**AWS Global Accelerator**

**Kenapa D:**
Ini adalah perbandingan **Global Accelerator vs Route 53** untuk multi-region routing:

| | Global Accelerator | Route 53 Latency-Based |
|--|-------------------|----------------------|
| IP type | Static anycast IP (2 IPs) | DNS name |
| Failover time | Detik (IP routing) | Menit (DNS TTL + propagation) |
| DNS dependency | Tidak | Ya |
| Health check | Per endpoint, per region | Ya, tapi failover lebih lambat |
| Protocol | TCP, UDP, HTTP/S | HTTP/S (untuk health check) |
| Network path | AWS backbone | Public internet |

**Key differentiator:** Global Accelerator menggunakan **anycast IP** — kedua IP selalu valid, traffic diarahkan via BGP routing ke AWS edge terdekat, lalu via AWS backbone ke region target. Tidak ada DNS TTL yang harus menunggu expire saat failover.

Route 53 failover butuh menunggu TTL record lama expire di DNS cache client (bisa menit hingga jam tergantung TTL setting dan client caching behavior).

**Kenapa bukan yang lain:**
- **A** — Route 53 latency-based dengan health checks bisa bekerja untuk multi-region routing, tapi **DNS propagation delay** saat failover bisa menit-an. Tidak memenuhi requirement "tidak ada DNS propagation delay".
- **B** — Route 53 geolocation routing berdasarkan lokasi geografis client (negara/benua), bukan berdasarkan network performance. Tidak otomatis failover berdasarkan endpoint health secepat Global Accelerator.
- **C** — CloudFront multi-origin untuk **content delivery dan caching**, bukan untuk routing application traffic ke multiple regions. CloudFront Origin Groups adalah untuk origin failover (CDN layer), bukan untuk application-level multi-region availability.

**Konsep yang diuji:** AWS Global Accelerator, anycast IP, instant failover vs DNS propagation, Global Accelerator vs Route 53, AWS backbone routing.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | DynamoDB GSI composite key, sort key untuk ordering |
| 2 | Aurora Machine Learning, SageMaker/Comprehend SQL integration |
| 3 | DynamoDB Streams → Lambda → OpenSearch, CDC pattern |
| 4 | SageMaker Blue/Green deployment, zero-downtime model update |
| 5 | AWS Cloud Map, ECS Service Discovery, DNS-based discovery |
| 6 | CloudWatch PutMetricData API, custom business metrics |
| 7 | Amazon Athena, serverless ad-hoc SQL on S3 |
| 8 | Secrets Manager ECS integration, automatic rotation |
| 9 | Lambda VPC + NAT Gateway, internet + private access |
| 10 | Global Accelerator anycast, instant failover vs DNS propagation |
