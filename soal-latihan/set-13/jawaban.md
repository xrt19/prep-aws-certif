# Jawaban — Set 13
## Domain: Resilient Architectures

---

## Soal 1 — Jawaban: C

**SQS Delay Queue (`DelaySeconds`)**

**Kenapa C:**
`DelaySeconds` adalah atribut queue (atau per-message via `MessageDeduplicationId`) yang membuat message **tidak visible ke consumer** selama periode yang ditentukan (0–900 detik = maksimum 15 menit). Setelah delay habis, message muncul di queue dan bisa di-consume. Ini berbeda dari Visibility Timeout yang baru aktif **setelah** message di-receive.

Timeline: Producer kirim message → message "hidden" selama 5 menit → message muncul di queue → consumer bisa receive.

**Kenapa bukan yang lain:**
- **A** — Visibility Timeout aktif setelah message **sudah di-receive** oleh consumer — bukan delay sebelum message visible. Urutan yang berbeda.
- **B** — DLQ untuk messages yang gagal diproses, bukan untuk delay delivery.
- **D** — WaitTimeSeconds di Long Polling maksimum 20 detik, dan itu adalah setting di sisi consumer (berapa lama consumer nunggu), bukan delay di message.

**Konsep yang diuji:** SQS Delay Queue, DelaySeconds, message delivery delay, perbedaan dengan Visibility Timeout.

---

## Soal 2 — Jawaban: C

**SNS Subscription Filter Policy**

**Kenapa C:**
SNS Filter Policy adalah JSON policy yang di-attach ke **setiap subscription** — bukan ke topic. SNS akan **hanya deliver message ke subscription** jika message attributes cocok dengan filter policy. Contoh:
- Subscription Finance: `{"category": ["financial"]}`
- Subscription Ops: `{"priority": ["high"]}`

Message dengan `category=financial` hanya ke-deliver ke Finance queue, tidak ke Ops atau Analytics. Ini mengurangi jumlah messages yang diterima consumer dan menghilangkan kebutuhan filtering di sisi consumer.

**Kenapa bukan yang lain:**
- **A** — Tiga SNS topics terpisah berarti producer harus publish ke tiga topics — duplikasi logika di sisi producer, tidak scalable.
- **B** — Filter di sisi consumer berarti semua messages tetap di-deliver ke semua queues — membuang bandwidth, storage, dan cost SQS.
- **D** — SQS FIFO untuk ordering, tidak ada hubungannya dengan message routing berdasarkan content.

**Konsep yang diuji:** SNS Subscription Filter Policy, message attributes, content-based delivery.

---

## Soal 3 — Jawaban: C

**Connection Draining (Deregistration Delay)**

**Kenapa C:**
Connection Draining (di ALB disebut **Deregistration Delay**) adalah mekanisme dimana saat instance di-deregister dari target group:
1. ALB **stop routing new requests** ke instance tersebut
2. ALB **menunggu** in-flight requests selesai (sampai deregistration delay habis, default 300 detik)
3. Setelah semua in-flight requests selesai atau timeout tercapai, instance di-deregister

Hasilnya: tidak ada request yang tiba-tiba terputus di tengah processing.

**Kenapa bukan yang lain:**
- **A** — Health check interval yang lebih cepat hanya mempercepat deteksi instance unhealthy, tidak melindungi in-flight requests saat instance di-terminate secara sengaja.
- **B** — ASG Lifecycle Hooks memang bisa pause termination, tapi skenario yang tepat untuk **in-flight HTTP request** adalah Connection Draining yang bekerja di level load balancer.
- **D** — Sticky sessions memastikan user selalu ke instance yang sama, tapi tidak melindungi saat instance tersebut di-terminate — justru memperparah karena semua session user ter-pin ke instance yang hilang.

**Konsep yang diuji:** Connection Draining, Deregistration Delay, in-flight request protection, graceful deregistration.

---

## Soal 4 — Jawaban: C

**ASG Mixed Instances Policy**

**Kenapa C:**
Mixed Instances Policy di ASG memungkinkan:
- Definisikan **base capacity** dari On-Demand (misalnya 2 instances) untuk baseline availability
- Sisanya dari **Spot Instances** untuk cost saving
- Specify **multiple instance types** — jika Spot untuk satu type tidak tersedia, ASG coba type lain
- **Capacity Rebalancing** — proactively replace Spot instances yang akan di-reclaim sebelum benar-benar di-interrupt

**Kenapa bukan yang lain:**
- **A** — Launch Template Spot saja berisiko: jika Spot tidak tersedia di semua type dan AZ yang di-configure, tidak ada fallback ke On-Demand.
- **B** — Dua ASG terpisah lebih complex untuk manage, tidak ada koordinasi otomatis antara keduanya.
- **D** — Spot Fleet adalah service terpisah untuk manage Spot Instances — bisa digunakan tapi tidak terintegrasi native dengan ASG scaling policies seperti Mixed Instances Policy.

**Konsep yang diuji:** ASG Mixed Instances Policy, Spot + On-Demand combination, instance type diversity, cost optimization with resilience.

---

## Soal 5 — Jawaban: C

**API Gateway WebSocket API**

**Kenapa C:**
API Gateway WebSocket API membuat **persistent, bidirectional connection** antara client dan server:
- Client connect sekali → connection tetap open
- Server bisa **push messages ke client kapanpun** via connection ID
- Client bisa kirim messages ke server kapanpun
- Ideal untuk: chat apps, real-time notifications, live trading, collaborative editing, game state sync

Berbeda dari REST API yang stateless (client selalu inisiasi request).

**Kenapa bukan yang lain:**
- **A** — Long polling adalah workaround HTTP — client terus-menerus buat new request, server delay response sampai ada data. Tidak truly bidirectional dan overhead lebih tinggi.
- **B** — Server-Sent Events (SSE) adalah **unidirectional** — server push ke client tapi client tidak bisa push ke server via same connection.
- **D** — REST API dengan caching untuk mengurangi backend load — tidak ada hubungannya dengan bidirectional real-time communication.

**Konsep yang diuji:** API Gateway WebSocket API, bidirectional communication, persistent connection, server push.

---

## Soal 6 — Jawaban: C

**`BisectOnFunctionError` dan `DestinationConfig` pada Event Source Mapping**

**Kenapa C:**
Ini adalah fitur spesifik Lambda Event Source Mapping untuk streaming sources (Kinesis, DynamoDB Streams):

- **`BisectOnFunctionError: true`** — saat batch gagal, split batch menjadi dua setengahnya dan retry masing-masing. Terus bisect sampai isolasi single record yang bermasalah.
- **`DestinationConfig` (On Failure)** — kirim records yang tetap gagal setelah max retries ke SQS queue atau SNS topic (DLQ equivalent untuk streaming)
- **`MaximumRetryAttempts`** — batasi jumlah retry sebelum skip
- **`MaximumRecordAgeInSeconds`** — skip records yang terlalu lama

Kombinasi ini mencegah **"poison pill"** (satu record bermasalah) memblokir seluruh stream.

**Kenapa bukan yang lain:**
- **A** — Visibility Timeout adalah konsep SQS, tidak ada di DynamoDB Streams atau Lambda Event Source Mapping.
- **B** — DLQ pada Lambda function hanya untuk **asynchronous invocations** (event dari S3, SNS, dll) — tidak bekerja untuk stream-based triggers (DynamoDB Streams, Kinesis) yang menggunakan Event Source Mapping.
- **D** — Menambahkan SQS antara DynamoDB Streams dan Lambda mengubah arsitektur secara signifikan dan menghilangkan manfaat streaming real-time.

**Konsep yang diuji:** Lambda Event Source Mapping, BisectOnFunctionError, streaming DLQ via DestinationConfig, poison pill prevention.

---

## Soal 7 — Jawaban: B

**ElastiCache Redis Cluster Mode Enabled**

**Kenapa B:**
Redis Cluster Mode Enabled **sharding** data ke banyak node groups (shard), masing-masing shard punya **primary + satu atau lebih replicas**:
- Dataset dibagi ke banyak shards — jika satu shard down, hanya **sebagian kecil** data yang tidak accessible sementara (bukan seluruh cache)
- Setiap shard bisa failover ke replica-nya secara independent
- Bisa scale horizontally hingga **500 nodes** (250 shards dengan 1 replica each)
- Mendukung dataset jauh lebih besar dari single node

**Kenapa bukan yang lain:**
- **A** — RDB snapshots untuk backup/restore, tidak mencegah data loss saat node crash (data yang masuk setelah snapshot terakhir tetap hilang), dan restore butuh waktu.
- **C** — Memcached memang bisa consistent hashing untuk distribusi, tapi tidak punya replication — jika node crash, data di node tersebut hilang sepenuhnya dan tidak ada failover.
- **D** — Tambah memory hanya meningkatkan kapasitas satu node — tidak menyelesaikan masalah resiliency saat node crash.

**Konsep yang diuji:** ElastiCache Redis Cluster Mode Enabled, sharding, partial failure isolation, horizontal scaling.

---

## Soal 8 — Jawaban: C

**AWS AppSync**

**Kenapa C:**
AWS AppSync adalah **managed GraphQL service** dengan fitur kunci:
- **Real-time subscriptions**: client subscribe ke data changes, AppSync otomatis push update via WebSocket saat data berubah — tanpa polling
- **Multiple data sources**: DynamoDB, Lambda, RDS, HTTP, OpenSearch dalam satu GraphQL schema
- **Offline support**: bisa queue mutations saat offline dan sync saat online — ideal untuk mobile
- **Conflict resolution** built-in untuk sync scenarios

**Kenapa bukan yang lain:**
- **A** — REST API + SNS adalah event-driven tapi tidak native GraphQL dan tidak punya built-in subscription mechanism yang seamless.
- **B** — API Gateway WebSocket + Lambda bisa dibuat tapi butuh banyak custom code untuk implement GraphQL subscription semantics — reinventing what AppSync provides out of the box.
- **D** — Amazon Cognito Sync (sekarang deprecated, digantikan AWS AppSync dan Amplify DataStore) hanya untuk sync small user profile data, bukan untuk aplikasi real-time dengan GraphQL.

**Konsep yang diuji:** AWS AppSync, managed GraphQL, real-time subscriptions, WebSocket push, mobile offline sync.

---

## Soal 9 — Jawaban: B

**Warm Standby**

**Kenapa B:**
Mapping requirement ke strategi:
| Strategi | RTO | RPO | Cost |
|----------|-----|-----|------|
| Backup & Restore | Jam | Jam | $ |
| Pilot Light | 10–30 menit | Menit | $$ |
| **Warm Standby** | **Menit** | **Detik–Menit** | $$$ |
| Multi-Site Active-Active | Detik | ~0 | $$$$ |

**Warm Standby** memenuhi RTO < 15 menit dan RPO < 1 menit:
- Environment minimal berjalan di region kedua (scaled-down tapi functional)
- Data di-replikasi continuous (RPO detik/menit)
- Saat DR: scale up environment yang sudah ada (RTO menit)
- Lebih murah dari Active-Active karena environment kedua kecil

**Kenapa bukan yang lain:**
- **A** — Backup & Restore: RTO jam dan RPO jam — tidak memenuhi kedua requirements.
- **C** — Active-Active memenuhi requirements tapi cost paling tinggi. Soal menyebut "cost harus minimal" — Warm Standby lebih cost-effective sambil tetap memenuhi RTO/RPO.
- **D** — Pilot Light: hanya core services running (misalnya database saja), RTO 30–60 menit untuk full scale-up — tidak memenuhi RTO < 15 menit.

**Konsep yang diuji:** DR strategies trade-offs, Warm Standby characteristics, RTO/RPO vs cost optimization.

---

## Soal 10 — Jawaban: B

**RDS failover + brief interruption + retry logic**

**Kenapa B:**
Ini adalah pemahaman realistis tentang RDS Multi-AZ failover:
1. AZ failure detected
2. RDS otomatis **promote standby** menjadi primary baru
3. **DNS record diupdate** ke endpoint baru (~60–120 detik total)
4. Selama proses ini ada **brief interruption** pada database connections yang sudah established
5. Aplikasi harus punya **retry logic** pada database connections — jika tidak, request akan gagal selama window failover

Ini adalah "exam trap" penting: Multi-AZ tidak berarti zero downtime sama sekali — ada brief interruption. Aplikasi yang well-designed harus implement retry/reconnect logic.

**Kenapa bukan yang lain:**
- **A** — Salah. Ada brief interruption selama DNS propagation dan failover. Aplikasi perlu reconnect.
- **C** — ASG tidak otomatis launch EC2 baru saat RDS failover — EC2 instances yang ada tetap jalan, hanya database connection yang perlu di-reconnect.
- **D** — CloudFront memang bisa serve cached static content, tapi untuk aplikasi yang butuh database (dynamic content), cache tidak menyelesaikan masalah selama database failover berlangsung.

**Konsep yang diuji:** RDS Multi-AZ failover realistis, brief interruption, DNS update, retry logic requirement.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | SQS Delay Queue, DelaySeconds vs Visibility Timeout |
| 2 | SNS Subscription Filter Policy, content-based delivery |
| 3 | Connection Draining / Deregistration Delay |
| 4 | ASG Mixed Instances Policy, Spot + On-Demand |
| 5 | API Gateway WebSocket API, bidirectional |
| 6 | Lambda Event Source Mapping, BisectOnFunctionError, poison pill |
| 7 | ElastiCache Redis Cluster Mode Enabled, sharding |
| 8 | AWS AppSync, GraphQL real-time subscriptions |
| 9 | DR strategies: Warm Standby RTO/RPO vs cost |
| 10 | RDS Multi-AZ failover realistis, retry logic requirement |
