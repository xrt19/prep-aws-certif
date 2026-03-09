# Jawaban — Set 08
## Domain: Resilient Architectures

---

## Soal 1 — Jawaban: B

**RDS Multi-AZ**

**Kenapa B:**
RDS Multi-AZ membuat **synchronous standby replica** di AZ berbeda secara otomatis. Saat primary instance atau AZ gagal, RDS otomatis melakukan failover ke standby dalam **60–120 detik** — tanpa intervensi manual. DNS endpoint RDS di-update otomatis ke standby, sehingga aplikasi yang menggunakan endpoint RDS tidak perlu mengubah connection string.

**Kenapa bukan yang lain:**
- **A** — Snapshot manual adalah point-in-time backup. Restore dari snapshot bisa memakan waktu puluhan menit hingga jam — RTO sangat tinggi, tidak cocok untuk "downtime minimal".
- **C** — Read Replica menggunakan **asynchronous** replication — ada kemungkinan data lag. Promote Read Replica ke primary tidak otomatis dan butuh waktu. Multi-AZ dirancang khusus untuk HA, bukan Read Replica.
- **D** — Replikasi manual di EC2 sangat kompleks, tidak managed, dan error-prone.

**Konsep yang diuji:** RDS Multi-AZ, automatic failover, synchronous replication, HA database.

---

## Soal 2 — Jawaban: B

**Scheduled scaling**

**Kenapa B:**
Scheduled scaling memungkinkan Auto Scaling Group **menambah atau mengurangi kapasitas pada waktu yang sudah ditentukan** — misalnya "tambah ke 10 instances setiap Senin jam 07:55". Karena spike sudah diprediksi, kapasitas bisa di-pre-provision sebelum spike terjadi, menghindari delay dari warm-up EC2.

**Kenapa bukan yang lain:**
- **A** — Target tracking scale **reaktif** — baru scale setelah metrik (CPU) sudah naik. Dengan warm-up time 5 menit, user sudah merasakan degradasi sebelum instance baru siap melayani request.
- **C** — Simple scaling juga reaktif dan ada cooldown period setelah setiap scaling action — lebih lambat dari scheduled.
- **D** — Scale manual bergantung pada manusia — tidak reliable, bisa lupa, dan tidak "otomatis" sesuai requirement.

**Konsep yang diuji:** Scheduled scaling vs dynamic scaling, proactive vs reactive scaling, predictable traffic patterns.

---

## Soal 3 — Jawaban: B

**Set Visibility Timeout ke nilai yang cukup lama (35 menit)**

**Kenapa B:**
Visibility Timeout adalah periode di mana message yang sudah di-receive oleh satu consumer **tidak terlihat** oleh consumer lain. Default-nya 30 detik. Karena processing butuh hingga 30 menit, visibility timeout harus di-set lebih dari itu (misalnya 35 menit) untuk mencegah message re-appear di queue sebelum selesai diproses. Jika consumer selesai, ia delete message. Jika consumer crash, message akan re-appear setelah timeout habis.

**Kenapa bukan yang lain:**
- **A** — FIFO queue menjamin urutan dan exactly-once processing, tapi bukan yang menentukan apakah message terlihat oleh consumer lain. Visibility Timeout tetap harus di-set dengan benar di FIFO maupun Standard queue.
- **C** — Delete sebelum selesai diproses adalah anti-pattern — jika consumer crash di tengah proses, message sudah hilang dan tidak bisa di-retry.
- **D** — DLQ untuk menyimpan message yang **gagal** diproses setelah beberapa kali retry, bukan untuk hold message selama processing berlangsung.

**Konsep yang diuji:** SQS Visibility Timeout, message processing lifecycle, long-running processing pattern.

---

## Soal 4 — Jawaban: C

**Latency-based routing**

**Kenapa C:**
Latency-based routing di Route 53 mengukur **network latency aktual** dari user ke setiap region yang tersedia dan merutekan ke region dengan latency terendah — bukan berdasarkan jarak geografis. Dengan menambahkan **health checks** pada setiap record, jika region dengan latency terendah tidak healthy, Route 53 otomatis route ke region lain yang masih sehat.

**Kenapa bukan yang lain:**
- **A** — Simple routing tidak bisa distribute ke beberapa endpoints atau melakukan failover.
- **B** — Weighted routing untuk A/B testing atau gradual traffic shifting berdasarkan persentase — bukan untuk optimasi latency.
- **D** — Geolocation routing berdasarkan lokasi geografis user (negara/benua), bukan latency aktual. User di Asia bisa saja punya latency lebih rendah ke US-East dibanding Asia-Pacific dalam kondisi tertentu — geolocation tidak mempertimbangkan ini.

**Konsep yang diuji:** Route 53 latency-based routing, health checks, failover combination.

---

## Soal 5 — Jawaban: B

**Auto Scaling health checks — terminate dan replace otomatis**

**Kenapa B:**
Auto Scaling Group melakukan health checks pada instances secara periodik. Jika instance dinyatakan unhealthy (baik oleh EC2 status checks maupun ALB health checks), ASG akan:
1. **Terminate** instance yang unhealthy
2. **Launch** instance baru sebagai pengganti
3. Register instance baru ke load balancer

Semua terjadi otomatis tanpa intervensi manual — ini adalah self-healing capability dari Auto Scaling.

**Kenapa bukan yang lain:**
- **A** — CloudWatch alarm notify manusia — ada delay dan bergantung pada tindakan manual. Tidak otomatis.
- **C** — ALB health check **detect** instance unhealthy dan **stop sending traffic** ke instance tersebut, tapi ALB tidak bisa terminate atau replace instance. Itu tugas ASG.
- **D** — Route 53 health check untuk routing di level DNS, tidak integrate langsung dengan EC2 lifecycle management.

**Konsep yang diuji:** ASG health checks, self-healing architecture, EC2 vs ELB health check type di ASG.

---

## Soal 6 — Jawaban: B

**RDS Point-in-Time Recovery (PITR)**

**Kenapa B:**
RDS PITR memungkinkan restore database ke **setiap detik dalam retention period** (1–35 hari, default 7 hari). Ini dimungkinkan karena RDS menyimpan **automated backups harian + transaction logs (binary logs)** secara continuous. Dengan kombinasi keduanya, RDS bisa replay transactions hingga titik waktu yang sangat spesifik — misalnya tepat sebelum human error terjadi.

**Kenapa bukan yang lain:**
- **A** — Daily snapshots hanya bisa restore ke waktu snapshot diambil (misalnya jam 03:00). Tidak bisa restore ke jam 14:32 — ada window data loss yang besar.
- **C** — Manual snapshot per jam lebih baik dari daily, tapi masih ada window up to 1 jam data loss. PITR jauh lebih granular.
- **D** — Read Replica tidak bisa di-"rewind" ke titik waktu tertentu di masa lalu — ia hanya reflect state database saat ini.

**Konsep yang diuji:** RDS PITR, automated backups + transaction logs, granular restore, retention period.

---

## Soal 7 — Jawaban: B

**S3 Cross-Region Replication (CRR)**

**Kenapa B:**
S3 CRR adalah fitur native S3 yang **otomatis mereplikasi objects** dari source bucket ke destination bucket di region berbeda. Konfigurasi di S3 Replication rules — setiap object baru yang di-upload ke source bucket otomatis di-copy ke destination. Versioning harus aktif di kedua bucket. Mendukung filter berdasarkan prefix/tag jika tidak semua objects perlu direplikasi.

**Kenapa bukan yang lain:**
- **A** — Lambda custom solution menambah complexity, latency, dan operational overhead. Ada risiko failure jika Lambda tidak berjalan.
- **C** — DataSync untuk migration besar atau sync scheduled — tidak real-time per-object seperti CRR. Lebih cocok untuk one-time migration atau bulk transfer.
- **D** — Transfer Acceleration untuk mempercepat upload dari client ke satu bucket via CloudFront edge — tidak mereplikasi ke bucket lain.

**Konsep yang diuji:** S3 CRR, automatic replication, DR strategy, versioning requirement.

---

## Soal 8 — Jawaban: C

**Application Load Balancer (ALB)**

**Kenapa C:**
ALB beroperasi di **Layer 7 (HTTP/HTTPS)** dan mendukung:
- **Content-based routing**: route berdasarkan path (`/api/*`), host header, query string, HTTP method
- **WebSocket** dan HTTP/2 natively
- **Health checks** yang bisa dikonfigurasi per target group
- Target groups bisa berisi EC2, Lambda, IP addresses, atau containers

Ini persis fitur yang dibutuhkan soal.

**Kenapa bukan yang lain:**
- **A** — CLB (Classic) adalah generasi lama, tidak mendukung path-based routing atau target groups. Deprecated untuk use case baru.
- **B** — NLB beroperasi di **Layer 4 (TCP/UDP)** — tidak bisa inspect HTTP content untuk path-based routing. NLB untuk ultra-low latency, static IP, dan non-HTTP workloads.
- **D** — GWLB untuk deploy third-party network appliances (firewall, IDS/IPS) secara transparent — bukan untuk general web traffic routing.

**Konsep yang diuji:** ALB vs NLB vs CLB, Layer 7 routing, path-based routing, WebSocket support.

---

## Soal 9 — Jawaban: D

**Multi-Site Active-Active**

**Kenapa D:**
Empat strategi DR dari RTO/RPO tercepat ke terlambat:
| Strategi | RTO | RPO | Cost |
|----------|-----|-----|------|
| Backup & Restore | Jam | Jam | $ |
| Pilot Light | 10–30 menit | Menit | $$ |
| Warm Standby | Menit | Detik | $$$ |
| Multi-Site Active-Active | Detik / < 1 menit | ~0 | $$$$ |

Requirement RTO < 1 menit hanya bisa dipenuhi oleh **Multi-Site Active-Active** — karena traffic sudah berjalan di dua region sekaligus. Saat satu region down, Route 53 health check otomatis remove endpoint tersebut, dan semua traffic langsung ke region lain — dalam hitungan detik.

**Kenapa bukan yang lain:**
- **A** — Backup & Restore RTO bisa jam — jauh di atas requirement.
- **B** — Pilot Light butuh scale up dari minimal state saat DR — RTO menit hingga puluhan menit.
- **C** — Warm Standby lebih cepat dari Pilot Light tapi masih butuh scale up — RTO beberapa menit, masih di atas 1 menit untuk scale completion.

**Konsep yang diuji:** DR strategies, RTO/RPO, Multi-Site Active-Active, Route 53 health checks failover.

---

## Soal 10 — Jawaban: B

**SQS sebagai buffer — message tersimpan hingga 14 hari**

**Kenapa B:**
Ini adalah **core value proposition SQS** sebagai message queue: **decoupling** producer dan consumer. Ketika producer mengirim burst messages, semua tersimpan aman di queue (retention hingga 14 hari). Consumer bisa proses sesuai kapasitasnya sendiri tanpa tekanan dari producer. Tidak ada message yang hilang karena queue sudah penuh — SQS virtually unlimited capacity. Ini adalah pattern klasik untuk absorb traffic spikes.

**Kenapa bukan yang lain:**
- **A** — FIFO menjamin urutan, tapi bukan solusi untuk burst handling atau message retention. FIFO justru punya throughput lebih rendah dari Standard queue.
- **C** — DLQ untuk handle message yang **gagal** diproses berkali-kali — bukan untuk buffer burst traffic.
- **D** — Long Polling mengurangi empty API calls (consumer poll ketika tidak ada message), meningkatkan efisiensi — tapi tidak secara langsung menangani burst dari producer side.

**Konsep yang diuji:** SQS decoupling pattern, message retention, burst absorption, producer-consumer architecture.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | RDS Multi-AZ, automatic failover, HA database |
| 2 | Scheduled scaling, proactive vs reactive |
| 3 | SQS Visibility Timeout, long-running processing |
| 4 | Route 53 latency-based routing + health checks |
| 5 | ASG health checks, self-healing, terminate & replace |
| 6 | RDS Point-in-Time Recovery (PITR) |
| 7 | S3 Cross-Region Replication (CRR) |
| 8 | ALB vs NLB vs CLB, Layer 7 path-based routing |
| 9 | DR strategies: RTO/RPO, Multi-Site Active-Active |
| 10 | SQS decoupling, burst absorption, 14-day retention |
