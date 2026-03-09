# Jawaban — Set 11
## Domain: Resilient Architectures

---

## Soal 1 — Jawaban: C

**SQS Long Polling (`WaitTimeSeconds` hingga 20 detik)**

**Kenapa C:**
Secara default SQS menggunakan **Short Polling** — API call langsung return meskipun queue kosong. Long Polling mengubah behavior: SQS menunggu hingga ada message tersedia atau `WaitTimeSeconds` habis (max 20 detik). Hasilnya:
- Jumlah empty responses turun drastis
- Jumlah API calls berkurang → **biaya lebih rendah**
- CPU consumer tidak sibuk polling terus-menerus
- Jika ada message, langsung di-return tanpa harus nunggu timeout

Bisa dikonfigurasi di queue attribute atau di setiap `ReceiveMessage` call.

**Kenapa bukan yang lain:**
- **A** — Visibility Timeout mengatur berapa lama message tidak terlihat setelah di-receive, tidak mempengaruhi behavior polling saat queue kosong.
- **B** — DLQ untuk messages yang gagal diproses berkali-kali, tidak ada hubungannya dengan empty poll responses.
- **D** — FIFO untuk ordering dan deduplication, tidak mengurangi frekuensi polling.

**Konsep yang diuji:** SQS Long Polling vs Short Polling, WaitTimeSeconds, cost optimization polling.

---

## Soal 2 — Jawaban: B

**Amazon Kinesis Data Firehose**

**Kenapa B:**
Kinesis Data Firehose adalah **managed delivery service** — fully serverless, tidak perlu manage consumers atau scaling:
- **Auto-batching**: buffer data dan kirim ke S3 dalam batch (per size atau per interval waktu)
- **Transformasi**: bisa invoke Lambda untuk transform data sebelum delivery
- **Format konversi**: konversi ke Parquet, ORC
- **Kompresi**: GZIP, Snappy, ZIP
- **Destinations**: S3, Redshift, OpenSearch, Splunk, HTTP endpoints

Berbeda dari Kinesis Data Streams yang butuh consumer untuk read dan proses — Firehose langsung deliver ke destination.

**Kenapa bukan yang lain:**
- **A** — Kinesis Data Streams + Lambda adalah solusi custom yang butuh manage consumer, sharding, dan checkpoint — overhead lebih tinggi. Firehose lebih tepat untuk delivery pipeline ke S3.
- **C** — DataSync untuk bulk data migration atau scheduled sync dari file system, bukan untuk real-time streaming log dari aplikasi.
- **D** — SQS + Lambda untuk delivery ke S3 adalah custom solution, tidak native support transformasi Parquet atau kompresi seperti Firehose.

**Konsep yang diuji:** Kinesis Data Firehose vs Data Streams, managed delivery, S3 delivery pipeline, data transformation.

---

## Soal 3 — Jawaban: C

**Kurangi default cooldown period atau gunakan instance warm-up time yang lebih pendek**

**Kenapa C:**
Cooldown period mencegah ASG melakukan scaling action berikutnya sebelum efek dari scaling action sebelumnya "settle". Default cooldown adalah **300 detik (5 menit)**. Jika EC2 instances warm-up sebenarnya hanya butuh 90 detik, cooldown 300 detik berarti ASG menunggu 3.5 menit yang tidak perlu sebelum bisa scale out lagi. Solusinya: set cooldown period sesuai actual warm-up time (misalnya 120 detik), atau untuk target tracking gunakan **instance warm-up** setting yang lebih akurat.

**Kenapa bukan yang lain:**
- **A** — Simple Scaling memang bisa di-set tanpa cooldown, tapi ia reaktif dan per-step — kurang smooth dari Target Tracking. Mengganti policy hanya untuk menghindari cooldown adalah trade-off yang salah.
- **B** — Menaikkan target CPU ke 80% berarti ASG akan tolerir kondisi overload lebih lama sebelum scale — ini memperburuk user experience, bukan solusi.
- **D** — Scheduled Scaling bisa override tapi hanya jika spike waktunya predictable — tidak menyelesaikan masalah cooldown yang terlalu panjang.

**Konsep yang diuji:** ASG cooldown period, instance warm-up, target tracking scaling, scaling responsiveness.

---

## Soal 4 — Jawaban: D

**DynamoDB Accelerator (DAX)**

**Kenapa D:**
DAX adalah **in-memory cache yang fully compatible dengan DynamoDB API** — tidak perlu ubah application code (hanya ganti endpoint dari DynamoDB ke DAX). DAX memberikan:
- **Read latency microsecond** (vs single-digit millisecond DynamoDB)
- **Write-through cache** — write ke DAX otomatis forward ke DynamoDB
- **Cluster mode** untuk HA dan elasticity
- Sangat efektif untuk highly-repeated reads (hot items, popular catalog)

**Kenapa bukan yang lain:**
- **A** — DynamoDB Auto Scaling menambah read capacity units (RCU) secara otomatis saat throughput tinggi, tapi tidak mengubah latency. Latency tetap milidetik meskipun capacity ditambah.
- **B** — Global Tables untuk multi-region write availability, tidak mengurangi latency read dalam satu region.
- **C** — On-Demand vs Provisioned capacity adalah billing model — tidak mempengaruhi latency baca.

**Konsep yang diuji:** DynamoDB DAX, microsecond latency, in-memory cache, hot item pattern.

---

## Soal 5 — Jawaban: B

**Rolling deployment**

**Kenapa B:**
Elastic Beanstalk Rolling deployment:
- Update instances dalam **batch** (misalnya 25% sekaligus)
- Batch yang sedang diupdate temporarily out of service
- Kapasitas total **berkurang sementara** (tidak 100% selama rollout)
- **Tidak ada full downtime** — sebagian instances selalu running versi lama atau baru
- Lebih lambat dari All at Once tapi tidak butuh extra capacity

Ini memenuhi requirement: no downtime, bisa multi-AZ.

**Kenapa bukan yang lain:**
- **A** — All at once terminate semua instances lama dan deploy baru sekaligus — ada **downtime** selama deployment berlangsung.
- **C** — Immutable memang zero downtime dan lebih safe (launch instances baru, tidak touch yang lama), tapi butuh **double capacity** sementara — lebih mahal. Soal tidak menyebut butuh zero downtime dengan zero capacity reduction, hanya "no downtime".
- **D** — Blue/Green di Beanstalk deploy ke environment terpisah — valid untuk zero downtime tapi butuh full new environment, lebih complex dan mahal.

**Konsep yang diuji:** Elastic Beanstalk deployment policies, Rolling vs All-at-once vs Immutable vs Blue/Green trade-offs.

---

## Soal 6 — Jawaban: C

**EFS secara default menyimpan data redundant di multiple AZs**

**Kenapa C:**
Amazon EFS (Standard storage class) secara default menggunakan **regional storage** — data di-replikasi secara redundant di **semua AZ dalam satu region** secara otomatis, tanpa konfigurasi tambahan. Untuk akses, dibuat **mount target** di setiap AZ. EC2 di setiap AZ mount ke mount target di AZ mereka sendiri. Jika satu AZ down, EC2 di AZ lain tetap bisa akses EFS via mount target mereka — tidak ada interupsi.

**Kenapa bukan yang lain:**
- **A** — Salah. EFS Standard adalah regional, bukan single-AZ. Ada EFS One Zone (versi murah, single-AZ) tapi itu bukan default.
- **B** — Berbeda dengan RDS, EFS tidak butuh konfigurasi "Multi-AZ" — redundansi sudah built-in di Standard storage class.
- **D** — EFS tidak failover ke S3. EFS dan S3 adalah storage yang berbeda fundamental.

**Konsep yang diuji:** EFS regional redundancy, mount targets per AZ, EFS Standard vs One Zone, AZ resilience.

---

## Soal 7 — Jawaban: C

**CloudFront Origin Groups dengan origin failover**

**Kenapa C:**
CloudFront Origin Groups memungkinkan definisi **primary** dan **secondary origin**. Mekanisme failover:
1. CloudFront kirim request ke primary origin (ALB `us-east-1`)
2. Jika primary return **4xx atau 5xx errors** (bisa dikonfigurasi kode mana), CloudFront otomatis **retry ke secondary origin** (ALB `us-west-2`)
3. Respons dari secondary di-cache dan di-serve

Ini memberikan **automatic failover** di level CloudFront tanpa perubahan DNS.

**Kenapa bukan yang lain:**
- **A** — Geo-restriction untuk memblokir akses dari negara tertentu, bukan untuk origin failover.
- **B** — Route 53 Failover di depan CloudFront bisa bekerja tapi ada DNS propagation delay (60 detik+). Origin Groups lebih cepat karena failover terjadi di level CloudFront edge, tidak butuh DNS update.
- **D** — Lambda@Edge untuk customize request/response logic, bisa dibuat untuk redirect tapi lebih complex dan tidak ada native failover mechanism.

**Konsep yang diuji:** CloudFront Origin Groups, origin failover, primary/secondary origin, 5xx error handling.

---

## Soal 8 — Jawaban: B

**Geolocation routing**

**Kenapa B:**
Route 53 Geolocation routing me-route traffic berdasarkan **lokasi geografis pengirim query** (negara, benua, atau US state). Bisa dikonfigurasi:
- Country-level: Indonesia → `ap-southeast-1`, Germany → `eu-central-1`
- Continent-level: Asia → server Asia
- **Default record**: untuk IP yang tidak cocok dengan record manapun → `us-east-1`

Ini persis yang dibutuhkan soal — routing deterministic berdasarkan negara dengan default fallback.

**Kenapa bukan yang lain:**
- **A** — Latency-based routing berdasarkan measured network latency — user Indonesia mungkin saja di-route ke server lain jika latency ke sana lebih rendah. Tidak bisa guarantee "user Indonesia selalu ke ap-southeast-1".
- **C** — Weighted routing untuk distribusi traffic berdasarkan persentase, tidak berdasarkan lokasi.
- **D** — Geoproximity routing (Traffic Flow feature) berdasarkan jarak geografis ke resource, bisa di-bias. Berbeda dari Geolocation yang strict per negara/benua. Geoproximity butuh Route 53 Traffic Flow (ada biaya tambahan).

**Konsep yang diuji:** Route 53 Geolocation routing vs Latency-based vs Geoproximity, default record, country-level routing.

---

## Soal 9 — Jawaban: C

**RDS Storage Auto Scaling**

**Kenapa C:**
RDS Storage Auto Scaling secara otomatis **expand storage** ketika mendekati kapasitas penuh (default trigger: sisa storage < 10% atau < 5GB, selama lebih dari 5 menit). Proses expansion terjadi **tanpa downtime** — tidak ada restart instance. Bisa set **maximum storage threshold** untuk mencegah biaya tidak terkontrol. Mendukung semua tipe RDS (MySQL, PostgreSQL, MariaDB, SQL Server, Oracle).

**Kenapa bukan yang lain:**
- **A** — Multi-AZ standby punya storage yang sama dengan primary — tidak menambah kapasitas primary.
- **B** — Read Replica punya copy data yang sama — tidak mengurangi storage primary.
- **D** — Aurora memang auto-scale storage, tapi soal menyebutkan RDS MySQL. Solusi yang tepat adalah mengaktifkan Storage Auto Scaling pada RDS yang ada, bukan migrasi ke Aurora.

**Konsep yang diuji:** RDS Storage Auto Scaling, automatic storage expansion, no downtime, maximum threshold.

---

## Soal 10 — Jawaban: C

**ECS Cluster Auto Scaling dengan Capacity Providers**

**Kenapa C:**
Ada dua level scaling di ECS EC2:
1. **Task-level**: ECS Service Auto Scaling — menambah/mengurangi jumlah **tasks**
2. **Infrastructure-level**: ECS Cluster Auto Scaling via **Capacity Providers** — otomatis scale **EC2 instances** dalam ASG yang terhubung ke cluster

Capacity Provider menggunakan **Managed Scaling** yang menghitung berapa EC2 instances yang dibutuhkan berdasarkan task yang pending/running dan otomatis adjust ASG desired count. Ini menghilangkan kebutuhan manual scale EC2 atau CloudWatch alarm manual.

**Kenapa bukan yang lain:**
- **A** — ECS Service Auto Scaling scale **tasks** (containers), bukan underlying EC2 instances. Jika tidak ada EC2 capacity, tasks tetap pending meskipun service auto scaling sudah trigger.
- **B** — CloudWatch alarm + ASG bisa dilakukan manual tapi tidak native integrated dengan ECS scheduling state — bisa scale terlambat atau tidak tepat.
- **D** — AWS Auto Scaling Plans adalah higher-level service untuk koordinasi scaling policies, tidak spesifik untuk ECS Capacity Providers.

**Konsep yang diuji:** ECS Capacity Providers, Cluster Auto Scaling, EC2 infrastructure scaling vs task scaling.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | SQS Long Polling, WaitTimeSeconds, reduce empty responses |
| 2 | Kinesis Data Firehose, managed delivery, S3 pipeline |
| 3 | ASG cooldown period, instance warm-up, scaling responsiveness |
| 4 | DynamoDB DAX, microsecond latency, hot items |
| 5 | Elastic Beanstalk deployment policies, Rolling |
| 6 | EFS regional redundancy, multi-AZ by default |
| 7 | CloudFront Origin Groups, origin failover |
| 8 | Route 53 Geolocation routing, country-level, default record |
| 9 | RDS Storage Auto Scaling, no downtime, auto-expand |
| 10 | ECS Capacity Providers, Cluster Auto Scaling, EC2 scaling |
