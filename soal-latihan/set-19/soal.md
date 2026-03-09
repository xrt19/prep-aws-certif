# Soal Latihan — Set 19
## Domain: High-Performing Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah aplikasi e-commerce memiliki **product catalog** yang disimpan di DynamoDB. Ada dua query pattern utama:
1. `GET /products/{product_id}` — fetch satu produk by primary key (cepat)
2. `GET /products?category=electronics&sort=price_asc` — filter by category, sort by price (lambat, full scan)

Struktur tabel: `product_id` (partition key). Bagaimana cara mengoptimalkan query pattern kedua?

- A. Aktifkan DynamoDB DAX untuk cache query hasil scan
- B. Pindahkan ke Aurora MySQL untuk support ORDER BY dan WHERE clause
- C. **Buat GSI dengan `category` sebagai partition key dan `price` sebagai sort key** — query pattern kedua bisa dieksekusi langsung via GSI tanpa full scan; DynamoDB query GSI dengan `KeyConditionExpression` bisa filter by category dan sort by price secara efisien
- D. Gunakan DynamoDB Parallel Scan untuk mempercepat full table scan

---

**Soal 2**

Sebuah perusahaan menggunakan **Amazon Aurora MySQL**. Mereka ingin menjalankan **machine learning inference** langsung dari dalam SQL query tanpa mengekspor data ke luar database, untuk use case seperti fraud detection per transaksi saat di-INSERT.

Fitur mana yang memungkinkan ini?

- A. Aurora Parallel Query — eksekusi query di storage layer
- B. Aurora Serverless v2 — scale otomatis untuk workload bervariasi
- C. **Aurora Machine Learning** — integrasi native antara Aurora dan SageMaker atau Amazon Comprehend; bisa panggil ML model via SQL function (`aws_comprehend.detect_sentiment()`, `aws_sagemaker.invoke_endpoint()`); output ML muncul sebagai kolom dalam result set
- D. Aurora Global Database — replikasi ke secondary region untuk ML workload

---

**Soal 3**

Sebuah startup memiliki aplikasi yang menggunakan **DynamoDB** dengan write-heavy workload. Mereka ingin track setiap perubahan item (INSERT, UPDATE, DELETE) untuk membangun **audit trail** dan **sync ke OpenSearch** secara real-time untuk search functionality.

Arsitektur paling tepat?

- A. Lambda scheduled setiap menit untuk scan DynamoDB dan detect perubahan
- B. EventBridge Pipes untuk koneksi langsung DynamoDB ke OpenSearch
- C. **DynamoDB Streams → Lambda → OpenSearch** — DynamoDB Streams capture setiap item-level change dalam urutan kronologis; Lambda di-trigger oleh stream untuk proses dan index ke OpenSearch; reliable, ordered, dan near real-time
- D. Kinesis Data Firehose terhubung langsung ke DynamoDB table

---

**Soal 4**

Sebuah perusahaan ingin membangun **real-time recommendation engine** di aplikasi mereka. Model ML sudah di-train di SageMaker. Mereka butuh serve recommendations dengan latency < 100ms untuk ribuan requests per detik, dan model perlu di-update (re-deploy) tanpa downtime.

Fitur SageMaker mana yang mendukung zero-downtime model update?

- A. SageMaker Batch Transform — proses semua user sekaligus secara periodik
- B. SageMaker Serverless Inference — scale to zero untuk cost efficiency
- C. SageMaker Model Registry untuk versioning model
- D. **SageMaker Endpoint dengan Blue/Green deployment** — deploy model baru ke "green" environment sambil "blue" masih serving traffic; shift traffic secara bertahap (canary) atau sekaligus; rollback instan jika ada masalah; zero downtime

---

**Soal 5**

Sebuah perusahaan memiliki **microservices** yang di-deploy di ECS Fargate. Service A perlu memanggil Service B dan Service C. Saat ini menggunakan internal ALB dengan DNS discovery. Tim ingin menghilangkan dependency ke load balancer dan menggunakan **service-to-service communication** yang lebih efisien dengan built-in load balancing dan health checking.

AWS feature mana yang tepat?

- A. Route 53 Private Hosted Zone dengan A records per service
- B. VPC Lattice untuk service-to-service networking
- C. **AWS Cloud Map + ECS Service Discovery** — ECS otomatis register task IP ke Cloud Map saat task start dan deregister saat stop; Service A bisa discover Service B via DNS (`serviceb.local`) atau API; tidak butuh load balancer untuk internal communication
- D. Application Load Balancer internal untuk setiap service

---

**Soal 6**

Sebuah perusahaan memiliki **EC2 fleet** yang berjalan di multiple Availability Zones. Mereka ingin mengumpulkan **custom application metrics** (business metrics seperti orders per minute, cart abandonment rate) dan kirim ke CloudWatch untuk alerting dan dashboards.

Cara paling tepat untuk publish custom metrics ke CloudWatch?

- A. Aktifkan CloudWatch Agent di setiap EC2 instance
- B. Gunakan CloudWatch Logs dan parse metrics dari log entries
- C. **Gunakan CloudWatch PutMetricData API** — aplikasi memanggil PutMetricData API (via SDK) untuk publish custom metrics dengan namespace, dimensions, dan value; metrics tersedia di CloudWatch dalam hitungan detik untuk alerting dan dashboard
- D. Kirim metrics ke CloudTrail untuk analysis

---

**Soal 7**

Sebuah perusahaan memiliki **data analytics platform** di AWS. Tim data ingin menjalankan **interactive SQL queries** pada data di S3 data lake tanpa setup atau manage infrastructure apapun. Queries dijalankan ad-hoc oleh analysts, tidak terjadwal.

Service mana yang paling tepat?

- A. Amazon Redshift — data warehouse untuk analytics
- B. Amazon EMR — managed Hadoop/Spark cluster
- C. AWS Glue ETL jobs — serverless data transformation
- D. **Amazon Athena** — serverless interactive query service; query data di S3 langsung dengan SQL (Presto engine); tidak ada infrastructure untuk di-manage; bayar per data yang di-scan ($5/TB); perfect untuk ad-hoc queries oleh analysts

---

**Soal 8**

Sebuah perusahaan menjalankan **containerized application** di ECS yang membutuhkan akses ke secret database credentials. Saat ini credentials di-hardcode di environment variables di task definition. Tim ingin menghilangkan hardcoded credentials dengan cara yang aman dan otomatis rotate.

Solusi paling tepat?

- A. Simpan credentials di S3 object dengan SSE-KMS encryption
- B. Gunakan AWS Systems Manager Parameter Store dengan SecureString
- C. **Gunakan AWS Secrets Manager dengan ECS native integration** — di task definition, referensikan secret ARN dari Secrets Manager; ECS otomatis inject secret sebagai environment variable saat task launch; Secrets Manager handle automatic rotation; tidak ada credentials di task definition code
- D. Enkripsi credentials menggunakan KMS dan simpan di ECS task definition

---

**Soal 9**

Sebuah perusahaan memiliki **Lambda function** yang perlu melakukan network call ke third-party API di internet. Lambda saat ini tidak di VPC. Tim kemudian juga butuh Lambda akses ke RDS yang berada di private VPC subnet.

Bagaimana konfigurasi yang tepat?

- A. Buat dua Lambda functions — satu untuk API call (non-VPC), satu untuk RDS (VPC)
- B. Lambda non-VPC tidak bisa akses RDS di private subnet — harus pilih salah satu
- C. Gunakan VPC Peering antara Lambda VPC dan RDS VPC
- D. **Attach Lambda ke VPC dengan subnet yang memiliki NAT Gateway** — Lambda dalam VPC bisa akses RDS di private subnet; untuk akses internet, gunakan NAT Gateway di public subnet; Lambda di private subnet route internet traffic via NAT Gateway

---

**Soal 10**

Sebuah perusahaan ingin meningkatkan **availability dan performance** aplikasi web mereka yang di-host di multiple AWS regions. Mereka butuh solusi yang bisa:
- Route users ke region terdekat berdasarkan network performance
- Otomatis failover jika satu region down
- Tidak ada DNS propagation delay saat failover

Service mana yang tepat?

- A. Route 53 latency-based routing dengan health checks
- B. Route 53 geolocation routing ke region berbeda
- C. CloudFront multi-origin dengan failover
- D. **AWS Global Accelerator** — anycast static IP yang secara otomatis route traffic ke endpoint AWS terdekat dan paling healthy via AWS backbone; failover dalam detik tanpa DNS propagation delay karena menggunakan IP routing bukan DNS TTL; support health checks per endpoint

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
