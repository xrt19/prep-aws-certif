# Jawaban — Set 10
## Domain: Resilient Architectures

---

## Soal 1 — Jawaban: B

**ECS Service**

**Kenapa B:**
ECS Service adalah abstraksi yang memastikan **sejumlah task tertentu (desired count) selalu berjalan** di setiap waktu. Ketika sebuah task crash, exit, atau dinyatakan unhealthy oleh load balancer health check, ECS Service **otomatis detect** dan **launch task pengganti** untuk mempertahankan desired count. Ini adalah mekanisme self-healing di level ECS.

**Kenapa bukan yang lain:**
- **A** — Task Definition mendefinisikan blueprint task (image, CPU, memory, environment), bukan policy untuk restart atau self-healing.
- **C** — Auto Scaling Group adalah konsep EC2. Untuk ECS Fargate, scaling dilakukan oleh **ECS Service Auto Scaling** (bukan ASG) berdasarkan metrik seperti CPU/memory utilization.
- **D** — Lambda untuk restart task via CloudWatch adalah custom solution yang unnecessary — ECS Service sudah melakukan ini secara native.

**Konsep yang diuji:** ECS Service desired count, self-healing tasks, ECS Fargate resiliency.

---

## Soal 2 — Jawaban: C

**DynamoDB Global Tables**

**Kenapa C:**
DynamoDB Global Tables adalah fitur untuk membuat **multi-region, multi-active** (sebelumnya disebut multi-master) DynamoDB table. Karakteristik utama:
- **Write dari manapun**: setiap region bisa terima write, bukan hanya read
- **Replikasi otomatis** antar semua regions dalam hitungan detik
- **Conflict resolution** otomatis menggunakan "last writer wins"
- Dibangun di atas DynamoDB Streams

**Kenapa bukan yang lain:**
- **A** — DynamoDB tidak punya konsep "Read Replicas" seperti RDS. Ini bukan terminologi DynamoDB.
- **B** — Custom Lambda replication adalah solusi yang kompleks, ada delay, dan rawan inconsistency — anti-pattern ketika ada native solution.
- **D** — DynamoDB Streams + Lambda bisa digunakan untuk replikasi, tapi ini adalah implementasi manual yang lebih rendah dari Global Tables. Global Tables menggunakan mekanisme ini di balik layar tapi dengan jaminan konsistensi yang lebih kuat.

**Konsep yang diuji:** DynamoDB Global Tables, multi-region multi-active, cross-region replication.

---

## Soal 3 — Jawaban: B

**ALB weighted target groups**

**Kenapa B:**
ALB mendukung **weighted target groups** di listener rules — bisa assign persentase traffic ke masing-masing target group. Contoh blue/green:
- Awalnya: Blue TG 100%, Green TG 0%
- Shift bertahap: Blue 90%, Green 10% → Blue 50%, Green 50% → Blue 0%, Green 100%
- Jika ada masalah di Green: kembalikan ke Blue 100% dalam hitungan detik

Ini memungkinkan canary deployment dan blue/green yang fully controlled tanpa downtime.

**Kenapa bukan yang lain:**
- **A** — Health check menentukan apakah target healthy atau tidak, tidak mengontrol persentase traffic distribution.
- **C** — Route 53 weighted routing bisa digunakan tapi memerlukan dua ALB terpisah, lebih complex, dan ada DNS propagation delay saat rollback. Weighted target groups di ALB lebih cepat dan simpel.
- **D** — CloudFront origin groups untuk failover (primary/secondary), bukan untuk gradual traffic shifting.

**Konsep yang diuji:** ALB weighted target groups, blue/green deployment, canary release, gradual traffic shifting.

---

## Soal 4 — Jawaban: C

**Network Load Balancer**

**Kenapa C:**
NLB adalah satu-satunya load balancer AWS yang mendukung **Elastic IP per Availability Zone** — sehingga IP address-nya static dan tidak berubah. Ini penting untuk use case dimana clients (partner firewall) melakukan whitelist berdasarkan IP. Selain itu, NLB beroperasi di Layer 4 (TCP/UDP) yang sesuai dengan requirement "TCP connections".

**Kenapa bukan yang lain:**
- **A** — ALB **tidak bisa** di-assign Elastic IP. IP address ALB bersifat dinamis dan bisa berubah — hanya DNS name yang tetap. AWS merekomendasikan untuk tidak whitelist IP ALB.
- **B** — CLB juga tidak support Elastic IP dan sudah deprecated.
- **D** — GWLB untuk deploy network appliances (IDS/IPS/firewall), bukan untuk general application load balancing dengan static IP.

**Konsep yang diuji:** NLB Elastic IP, static IP load balancer, TCP Layer 4, partner IP whitelisting.

---

## Soal 5 — Jawaban: C

**DynamoDB Streams**

**Kenapa C:**
DynamoDB Streams mencatat **setiap perubahan item** (INSERT, MODIFY, DELETE) dalam tabel sebagai ordered stream of records. Setiap record berisi before/after image item. Stream ini bisa di-consume oleh:
- **Lambda** (paling umum — trigger otomatis)
- **Kinesis Data Streams** (via Kinesis Data Streams for DynamoDB)

Dengan satu Lambda trigger pada stream, bisa fanout ke update inventory, kirim email via SES, update analytics — semuanya reaktif dan real-time.

**Kenapa bukan yang lain:**
- **A** — TTL untuk otomatis expire/delete items berdasarkan waktu, tidak ada hubungannya dengan notifikasi perubahan.
- **B** — DAX adalah in-memory cache untuk read queries, bukan untuk capture changes.
- **D** — Global Tables untuk multi-region availability, bukan untuk event-driven reactions terhadap data changes dalam satu region.

**Konsep yang diuji:** DynamoDB Streams, event-driven architecture, Lambda trigger, change data capture.

---

## Soal 6 — Jawaban: C

**Aurora Serverless v2**

**Kenapa C:**
Aurora Serverless v2 dirancang persis untuk workload yang unpredictable:
- **Auto-scaling kapasitas** dalam hitungan milidetik berdasarkan load aktual
- Bisa dikonfigurasi **minimum ACU (Aurora Capacity Units)** yang sangat rendah (0.5 ACU) — hampir tidak ada biaya saat idle
- Tetap kompatibel dengan Aurora MySQL/PostgreSQL API
- Berbeda dari v1: v2 scale lebih halus (tidak "jumpy") dan bisa berjalan dalam VPC sepenuhnya

**Kenapa bukan yang lain:**
- **A** — RDS tetap bayar per jam meskipun tidak ada traffic — tidak ada auto-scale ke zero.
- **B** — DynamoDB On-Demand sangat baik untuk bursty NoSQL workloads, tapi soal menyebutkan "database" dalam konteks yang lebih umum (kemungkinan relational). Aurora Serverless v2 lebih tepat untuk menggantikan workload relational yang unpredictable.
- **D** — ElastiCache adalah caching layer, bukan database utama. Tidak bisa digunakan sebagai pengganti database relational.

**Konsep yang diuji:** Aurora Serverless v2, auto-scaling capacity, near-zero idle cost, unpredictable workloads.

---

## Soal 7 — Jawaban: C

**Amazon EventBridge**

**Kenapa C:**
EventBridge adalah **serverless event bus** yang dirancang untuk event-driven architectures:
- Menerima events dari **AWS services** (EC2, S3, CodePipeline, dll), **custom applications**, dan **third-party SaaS** (Datadog, Zendesk, dll)
- **Rules** memfilter events berdasarkan pattern (content-based routing)
- **Targets** bisa berupa Lambda, SQS, SNS, Step Functions, API Gateway, dan lainnya
- Tidak perlu manage infrastructure

**Kenapa bukan yang lain:**
- **A** — SNS untuk push notification fan-out, tidak punya kemampuan content-based routing yang sophisticated atau native integration dengan semua AWS service events.
- **B** — SQS queue semua messages untuk satu consumer — tidak bisa route berdasarkan content ke target yang berbeda-beda.
- **D** — Step Functions untuk orchestrate workflow setelah event diterima, bukan untuk routing events ke berbagai targets.

**Konsep yang diuji:** EventBridge, event-driven architecture, content-based routing, AWS service events.

---

## Soal 8 — Jawaban: C

**API Gateway caching**

**Kenapa C:**
API Gateway mendukung **response caching** di level stage — cache response dari backend (Lambda, HTTP endpoint) berdasarkan request parameters. Ketika request identik datang sebelum TTL habis, API Gateway langsung return cached response **tanpa memanggil backend** sama sekali. Hasilnya: latency lebih rendah dan biaya Lambda invocation berkurang drastis untuk data yang jarang berubah.

Cache bisa dikonfigurasi per method, TTL 300 detik default (bisa 0–3600 detik), dan bisa di-invalidate secara manual.

**Kenapa bukan yang lain:**
- **A** — Throttling membatasi jumlah request yang masuk (rate limiting), bukan mengurangi beban backend dengan caching. Request yang melebihi limit akan di-reject (429 error), bukan di-serve dari cache.
- **B** — Usage plans untuk monetization dan rate limiting per API key, tidak ada hubungannya dengan caching.
- **D** — Stages untuk environment separation (dev/staging/prod), tidak mempengaruhi performance atau caching.

**Konsep yang diuji:** API Gateway caching, response cache, TTL, mengurangi Lambda invocations.

---

## Soal 9 — Jawaban: C

**S3 Same-Region Replication (SRR)**

**Kenapa C:**
S3 SRR mereplikasi objects **antar bucket dalam region yang sama** secara otomatis. Use case umum:
- Aggregate logs dari beberapa buckets ke satu bucket
- **Replikasi ke bucket analytics** di akun atau bucket berbeda dalam region yang sama
- Maintain copy di akun berbeda untuk compliance (tanpa cross-region)

Sama seperti CRR, butuh S3 Versioning aktif di kedua bucket.

**Kenapa bukan yang lain:**
- **A** — CRR (Cross-Region Replication) hanya bisa replicate ke **region yang berbeda**. Tidak bisa digunakan untuk replikasi intra-region.
- **B** — Lambda custom solution menambah complexity dan operational overhead yang tidak perlu karena ada native solution.
- **D** — DataSync untuk bulk data migration atau sync scheduled (batch), tidak real-time per-object seperti SRR.

**Konsep yang diuji:** S3 SRR vs CRR, same-region replication, intra-region bucket copy.

---

## Soal 10 — Jawaban: C

**Step scaling**

**Kenapa C:**
Step scaling memungkinkan definisi **multiple scaling steps** berdasarkan magnitude alarm breach:
- CPU 70–80%: tambah 2 instances (scale out moderat)
- CPU >80%: tambah 5 instances (scale out agresif)
- CPU 40–50%: kurangi 1 instance (scale in lambat)

Ini memberikan **kontrol granular** atas seberapa agresif scale out vs scale in — persis yang dibutuhkan soal. Scale out agresif mencegah degradasi saat spike, scale in konservatif mencegah thrashing.

**Kenapa bukan yang lain:**
- **A** — Scheduled scaling berdasarkan waktu, bukan berdasarkan load aktual. Tidak cocok untuk menentukan agresivitas berbeda antara scale out dan scale in.
- **B** — Target tracking menggunakan single target value dan AWS menentukan sendiri seberapa banyak scale — tidak bisa di-customize secara granular. Scale in dan scale out menggunakan logic yang sama.
- **D** — Simple scaling hanya satu step per alarm (tambah N atau kurangi N), tidak bisa definisikan multiple steps berdasarkan magnitude. Juga ada cooldown period yang membuat respons lebih lambat.

**Konsep yang diuji:** Step scaling vs simple vs target tracking, scaling aggressiveness, scale out vs scale in asymmetry.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | ECS Service desired count, self-healing Fargate |
| 2 | DynamoDB Global Tables, multi-region multi-active |
| 3 | ALB weighted target groups, blue/green deployment |
| 4 | NLB Elastic IP, static IP, TCP Layer 4 |
| 5 | DynamoDB Streams, event-driven, Lambda trigger |
| 6 | Aurora Serverless v2, near-zero idle, auto-scale |
| 7 | EventBridge, content-based routing, event bus |
| 8 | API Gateway caching, TTL, reduce Lambda invocations |
| 9 | S3 SRR vs CRR, same-region replication |
| 10 | Step scaling, multi-step, scale out vs scale in asymmetry |
