# Soal Latihan — Set 10
## Domain: Resilient Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menjalankan aplikasi di **Amazon ECS dengan Fargate**. Mereka ingin memastikan bahwa jika satu task crash, task baru **otomatis di-launch** sebagai pengganti tanpa intervensi manual.

Fitur ECS mana yang menjamin ini?

- A. ECS Task Definition — definisikan restart policy di sana
- B. **ECS Service** — mendefinisikan desired count tasks dan otomatis replace task yang gagal untuk mempertahankan desired count
- C. Auto Scaling Group — scale ECS tasks berdasarkan CPU
- D. CloudWatch alarm yang trigger Lambda untuk restart task

---

**Soal 2**

Sebuah perusahaan global memiliki aplikasi dengan DynamoDB sebagai database. Mereka butuh data DynamoDB **tersedia dan bisa di-write dari beberapa regions sekaligus** (bukan hanya read), dengan replikasi otomatis antar regions.

Fitur DynamoDB mana yang tepat?

- A. DynamoDB Read Replicas di setiap region
- B. DynamoDB Cross-Region Replication manual via Lambda
- C. **DynamoDB Global Tables** — multi-region, multi-active table yang otomatis sinkron ke semua regions, setiap region bisa read dan write
- D. DynamoDB Streams dengan Lambda yang forward ke region lain

---

**Soal 3**

Sebuah tim ingin melakukan **blue/green deployment** untuk aplikasi mereka di belakang ALB. Versi baru (green) di-deploy ke target group baru, dan traffic di-shift secara bertahap dari blue ke green. Jika ada masalah, traffic bisa langsung dikembalikan ke blue.

Fitur ALB mana yang memungkinkan ini?

- A. ALB health checks — otomatis shift traffic jika green healthy
- B. **ALB weighted target groups** — tentukan persentase traffic ke setiap target group, shift secara bertahap
- C. Route 53 weighted routing antar dua ALB
- D. CloudFront origin groups dengan failover

---

**Soal 4**

Sebuah perusahaan menjalankan aplikasi yang memerlukan **IP address tetap (static)** di load balancer mereka, karena beberapa partner whitelist IP tersebut di firewall mereka. Aplikasi menggunakan TCP connections.

Load balancer mana yang tepat?

- A. Application Load Balancer — assign Elastic IP
- B. Classic Load Balancer — sudah punya IP tetap
- C. **Network Load Balancer** — mendukung Elastic IP per AZ, static IP address
- D. Gateway Load Balancer — assign static IP untuk setiap AZ

---

**Soal 5**

Sebuah aplikasi e-commerce menggunakan DynamoDB. Ketika ada perubahan pada tabel (misalnya order baru dibuat), sistem lain perlu **bereaksi secara real-time** — misalnya update inventory, trigger email, dan update analytics.

Fitur DynamoDB mana yang memungkinkan ini?

- A. DynamoDB TTL — set expiry pada items yang berubah
- B. DynamoDB Accelerator (DAX) — cache perubahan data
- C. **DynamoDB Streams** — capture item-level changes secara real-time sebagai ordered stream, bisa di-consume oleh Lambda atau Kinesis
- D. DynamoDB Global Tables — replicate perubahan ke region lain

---

**Soal 6**

Sebuah startup memiliki workload database yang sangat tidak menentu — **bisa tidak ada traffic sama sekali selama berjam-jam**, lalu tiba-tiba ada banyak request. Mereka tidak ingin bayar untuk database yang idle, tapi perlu performa yang cukup saat dibutuhkan.

Database AWS mana yang paling cocok?

- A. RDS MySQL dengan instance kecil — biaya rendah saat idle
- B. DynamoDB On-Demand — hanya bayar per request
- C. **Aurora Serverless v2** — otomatis scale kapasitas dari minimum hingga maksimum, bisa scale ke near-zero saat idle
- D. ElastiCache — in-memory database yang hemat cost

---

**Soal 7**

Sebuah perusahaan ingin merespons **event dari berbagai sumber AWS** (misalnya EC2 instance state change, S3 object upload, CodePipeline stage failure) dan route event tersebut ke target yang berbeda-beda berdasarkan **content event**-nya.

Service mana yang paling tepat?

- A. Amazon SNS — publish event ke topic dan subscribe
- B. Amazon SQS — queue semua events dan proses
- C. **Amazon EventBridge** — event bus yang bisa route events berdasarkan rules/patterns ke berbagai targets
- D. AWS Step Functions — orchestrate responses ke setiap event

---

**Soal 8**

Sebuah API Gateway menerima traffic yang sangat tinggi. Banyak request meminta **data yang sama berulang-ulang** (misalnya daftar produk, konfigurasi aplikasi). Tim ingin mengurangi beban backend Lambda dan mempercepat response time.

Fitur API Gateway mana yang tepat?

- A. API Gateway throttling — batasi jumlah request per detik
- B. API Gateway usage plans — batasi request per API key
- C. **API Gateway caching** — cache response dari backend selama TTL tertentu, request identik tidak sampai ke Lambda
- D. API Gateway stages — deploy ke stage yang berbeda untuk isolasi

---

**Soal 9**

Sebuah perusahaan memiliki S3 bucket di `us-east-1` sebagai source of truth. Mereka ingin **copy semua objects ke bucket lain di region yang sama** untuk digunakan oleh tim analytics, tanpa replikasi ke region lain.

Solusi yang tepat adalah?

- A. Gunakan S3 Cross-Region Replication (CRR) dengan destination di region yang sama
- B. Buat Lambda yang copy object setiap ada upload baru
- C. **S3 Same-Region Replication (SRR)** — replikasi otomatis antar bucket dalam region yang sama
- D. Gunakan AWS DataSync untuk sync kedua bucket secara periodik

---

**Soal 10**

Sebuah perusahaan ingin Auto Scaling Group **scale out lebih agresif saat CPU tinggi** (tambah banyak instance sekaligus) tapi **scale in secara perlahan** (kurangi sedikit-sedikit) untuk menghindari thrashing.

Jenis scaling policy mana yang paling tepat untuk mengkonfigurasi behavior ini?

- A. Scheduled scaling — set jadwal berbeda untuk scale out dan scale in
- B. Target tracking — akan otomatis agresif saat CPU tinggi
- C. **Step scaling** — definisikan multiple steps: CPU 70–80% tambah 2 instances, CPU >80% tambah 5 instances, scale in selalu 1 instance
- D. Simple scaling — satu action untuk scale out, satu untuk scale in

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
