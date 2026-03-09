# Soal Latihan — Set 13
## Domain: Resilient Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menggunakan SQS untuk memproses order. Mereka ingin **menunda pemrosesan order baru selama 5 menit** setelah message masuk ke queue — memberi waktu untuk order cancellation window sebelum benar-benar diproses.

Fitur SQS mana yang tepat?

- A. Visibility Timeout — set ke 5 menit
- B. SQS Dead Letter Queue — hold message selama 5 menit
- C. **SQS Delay Queue** (`DelaySeconds`) — message tidak visible ke consumer selama periode delay yang ditentukan (0–900 detik)
- D. SQS Long Polling dengan WaitTimeSeconds 5 menit

---

**Soal 2**

Sebuah perusahaan memiliki SNS topic yang di-subscribe oleh **tiga SQS queues berbeda**: satu untuk tim Finance, satu untuk tim Ops, dan satu untuk tim Analytics. Masing-masing tim hanya ingin menerima **subset message yang relevan** — Finance hanya ingin message dengan attribute `category=financial`, Ops dengan `priority=high`.

Cara paling tepat agar setiap queue hanya menerima message yang relevan adalah?

- A. Buat tiga SNS topics terpisah, satu per tim
- B. Filter di sisi consumer — setiap tim filter setelah receive dari queue
- C. **SNS Subscription Filter Policy** — set filter policy per subscription, SNS hanya deliver message yang cocok dengan filter ke subscription tersebut
- D. Gunakan SQS FIFO agar message ter-sorted per kategori

---

**Soal 3**

Sebuah perusahaan menjalankan ALB dengan banyak EC2 instances di belakangnya. Saat scale-in atau deployment, instances di-terminate. Namun user melaporkan request mereka **tiba-tiba gagal** karena instance di-terminate saat sedang memproses request.

Fitur ALB/ASG mana yang mencegah ini?

- A. ALB health check interval yang lebih cepat
- B. ASG Lifecycle Hooks pada Terminating state
- C. **Connection Draining (Deregistration Delay)** — ALB berhenti mengirim request baru ke instance yang akan di-terminate, tapi menunggu in-flight requests selesai sebelum deregister (default 300 detik)
- D. Sticky sessions — user ter-pin ke instance yang sama

---

**Soal 4**

Sebuah perusahaan ingin mengurangi biaya EC2 untuk workload batch processing yang **toleran terhadap interruption**. Mereka ingin memanfaatkan **Spot Instances** sebanyak mungkin untuk hemat biaya, tapi tetap ada fallback ke On-Demand jika Spot tidak tersedia.

Fitur Auto Scaling mana yang mendukung ini?

- A. Launch Template dengan Spot Instance type saja
- B. Dua ASG terpisah — satu Spot, satu On-Demand
- C. **ASG Mixed Instances Policy** — definisikan campuran Spot dan On-Demand dengan fallback otomatis, AWS otomatis launch Spot jika tersedia atau On-Demand jika tidak
- D. Spot Fleet terpisah dari ASG

---

**Soal 5**

Sebuah aplikasi real-time membutuhkan **komunikasi dua arah (bidirectional)** antara server dan browser client — server perlu bisa **push data ke client** tanpa client harus polling. Aplikasi ini di-host di belakang API Gateway.

Fitur API Gateway mana yang tepat?

- A. API Gateway REST API dengan long polling
- B. API Gateway HTTP API dengan server-sent events
- C. **API Gateway WebSocket API** — mendukung persistent bidirectional connections, server bisa push message ke client kapanpun
- D. API Gateway REST API dengan caching enabled

---

**Soal 6**

Sebuah perusahaan menjalankan Lambda function yang di-trigger oleh **DynamoDB Streams**. Mereka menemukan bahwa jika Lambda gagal memproses batch records dari stream, Lambda terus retry batch yang sama berulang-ulang dan **memblokir pemrosesan records baru** yang masuk setelahnya.

Konfigurasi Lambda Event Source Mapping mana yang mencegah blocking ini?

- A. Naikkan Visibility Timeout agar batch tidak di-retry terlalu cepat
- B. Tambahkan Dead Letter Queue pada Lambda function
- C. **Set `BisectOnFunctionError` dan `DestinationConfig` pada Event Source Mapping** — jika batch gagal, bisect batch menjadi dua untuk isolasi record bermasalah, dan kirim failed records ke DLQ/SQS agar stream tidak blocked
- D. Gunakan SQS FIFO sebagai buffer antara DynamoDB Streams dan Lambda

---

**Soal 7**

Sebuah perusahaan memiliki ElastiCache Redis cluster yang menyimpan banyak data (ratusan GB). Mereka khawatir jika satu node Redis crash, **seluruh dataset yang ada di node tersebut hilang** karena cache tidak bisa di-rebuild dengan cepat.

Konfigurasi ElastiCache Redis mana yang menyelesaikan ini?

- A. Aktifkan Redis backup (RDB snapshots) setiap jam
- B. **ElastiCache Redis Cluster Mode Enabled** — data di-shard ke banyak node, setiap shard punya primary + replica, jika satu node crash hanya sebagian kecil data yang terdampak
- C. Gunakan ElastiCache Memcached dengan consistent hashing
- D. Tambahkan lebih banyak memory ke node yang ada

---

**Soal 8**

Sebuah perusahaan ingin membangun aplikasi mobile yang membutuhkan **real-time data sync** antara banyak client — ketika satu user update data, semua client lain harus otomatis mendapat data terbaru tanpa polling. Backend menggunakan GraphQL.

Service AWS mana yang paling tepat?

- A. API Gateway REST API + SNS push notification
- B. API Gateway WebSocket API + Lambda
- C. **AWS AppSync** — managed GraphQL service dengan built-in real-time subscriptions, data otomatis di-push ke semua subscribing clients saat ada perubahan
- D. Amazon Cognito Sync — sync data antar devices

---

**Soal 9**

Sebuah Solutions Architect mendesain arsitektur untuk aplikasi kritis yang harus memenuhi requirements berikut:
- **RTO < 15 menit**
- **RPO < 1 menit**
- Cost harus minimal

Strategi DR mana yang paling sesuai?

- A. Backup and Restore — RPO jam, RTO jam
- B. **Warm Standby** — lingkungan minimal berjalan di region kedua, data di-replikasi continuous (RPO detik/menit), scale up saat DR (RTO menit)
- C. Multi-Site Active-Active — RTO detik tapi cost paling tinggi
- D. Pilot Light — hanya core services running, RTO 30–60 menit untuk full scale-up

---

**Soal 10**

Sebuah perusahaan mengoperasikan aplikasi web dengan arsitektur berikut: CloudFront → ALB → EC2 (Multi-AZ, ASG) → RDS Multi-AZ. Tiba-tiba RDS primary instance mengalami AZ failure.

Urutan events yang benar saat failover terjadi adalah?

- A. EC2 instances langsung reconnect ke RDS standby tanpa ada interruption sama sekali
- B. RDS otomatis failover ke standby (60–120 detik), selama failover ada brief interruption pada database connections, setelah DNS updated aplikasi reconnect otomatis — jika aplikasi punya **retry logic pada database connections**
- C. ASG otomatis launch EC2 baru yang sudah terhubung ke standby RDS
- D. CloudFront serve cached content sehingga user tidak merasakan apapun

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
