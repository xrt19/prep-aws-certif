# Soal Latihan — Set 17
## Domain: High-Performing Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan memiliki workload analytics yang menggunakan **Amazon Athena** untuk query data di S3. Query sering berjalan lambat dan mahal karena scan data CSV yang sangat besar. Tim ingin mengoptimalkan tanpa mengubah query SQL yang sudah ada.

Optimasi mana yang akan paling signifikan mengurangi waktu query dan biaya Athena?

- A. Pindahkan data ke S3 Intelligent-Tiering untuk akses lebih cepat
- B. Aktifkan S3 Transfer Acceleration pada bucket data
- C. Buat lebih banyak partisi S3 berdasarkan tanggal agar Athena scan lebih sedikit data
- D. **Konversi data ke format columnar (Parquet atau ORC) dan partition by date** — Athena charge per data yang di-scan; columnar format memungkinkan Athena baca hanya kolom yang dibutuhkan, bukan seluruh row; partitioning memungkinkan partition pruning sehingga scan hanya data yang relevan

---

**Soal 2**

Sebuah tim data engineering menggunakan **AWS Glue** untuk ETL pipeline. Job Glue sering lambat saat memproses dataset besar (puluhan GB). Tim ingin meningkatkan throughput ETL tanpa mengubah logika transformasi.

Cara paling efektif untuk mempercepat Glue ETL job?

- A. Ganti ke AWS Batch untuk ETL yang lebih berat
- B. Pindahkan data source ke DynamoDB dari S3
- C. Aktifkan Glue job bookmarks untuk hanya proses data baru
- D. **Tambah jumlah DPUs (Data Processing Units) atau aktifkan Glue Auto Scaling** — setiap DPU adalah 4 vCPU + 16 GB RAM; lebih banyak DPU = lebih banyak parallel workers yang memproses data secara bersamaan via Apache Spark di bawahnya

---

**Soal 3**

Sebuah perusahaan memiliki aplikasi yang menyimpan **session data user** di DynamoDB. Session dibuat saat login dan dihapus saat logout atau expired. Tim menemukan tabel terus bertambah besar karena data session lama tidak terhapus.

Cara paling tepat untuk menghapus data expired otomatis dan menjaga tabel tetap lean?

- A. Buat scheduled Lambda setiap 1 jam untuk scan dan hapus items expired
- B. Gunakan DynamoDB Streams untuk trigger Lambda saat item dibuat, set timer untuk delete
- C. Aktifkan S3 Lifecycle Policy pada tabel DynamoDB
- D. **Aktifkan DynamoDB TTL (Time to Live)** — tambahkan attribute timestamp expired pada setiap item; DynamoDB akan otomatis hapus items yang sudah melewati timestamp TTL, tanpa konsumsi Write Capacity Units (WCU), dalam waktu 48 jam setelah expired

---

**Soal 4**

Sebuah aplikasi menggunakan **ElastiCache Redis** untuk session storage. Traffic meningkat pesat dan single Redis node mulai kewalahan. Mereka ingin scale Redis untuk handle lebih banyak data dan concurrent connections.

Pendekatan scaling Redis mana yang paling tepat untuk workload ini?

- A. Upgrade ke node Redis yang lebih besar (vertical scaling)
- B. Tambah lebih banyak Read Replicas untuk mengurangi beban read
- C. **Aktifkan Redis Cluster Mode** — sharding data ke multiple shard (node groups), setiap shard handle subset keyspace, meningkatkan total capacity data dan write throughput secara horizontal
- D. Pindahkan ke Memcached yang mendukung multi-threading

---

**Soal 5**

Sebuah perusahaan menjalankan **aplikasi video streaming**. Video files (rata-rata 2 GB) di-upload oleh creators dan kemudian di-serve ke jutaan viewers via CloudFront. Upload sering gagal di tengah jalan karena koneksi tidak stabil dari creators.

Arsitektur mana yang paling tepat untuk menangani upload yang reliable?

- A. Minta creators upload langsung ke EC2 instance kemudian transfer ke S3
- B. Gunakan AWS DataSync untuk transfer file dari on-premise creators
- C. **Gunakan S3 Multipart Upload dengan S3 pre-signed URL** — creator upload langsung ke S3 menggunakan pre-signed URL yang di-generate server; multipart upload memungkinkan resume dari part yang gagal tanpa ulang dari awal; tidak ada server perantara yang perlu handle data besar
- D. Upload ke SQS dulu kemudian Lambda yang forward ke S3

---

**Soal 6**

Sebuah tim menggunakan **Amazon SageMaker** untuk melatih model ML. Training jobs membutuhkan waktu lama (8–12 jam) dan sangat mahal. Tim ingin mengurangi biaya tanpa mengorbankan waktu training secara signifikan.

Strategi mana yang paling efektif untuk mengurangi biaya SageMaker Training?

- A. Pindahkan training ke SageMaker Serverless Inference
- B. Gunakan SageMaker Ground Truth untuk label data lebih efisien
- C. Aktifkan SageMaker Experiments untuk tracking
- D. **Gunakan EC2 Spot Instances via SageMaker Managed Spot Training** — SageMaker bisa menggunakan Spot Instances untuk training jobs dengan checkpoint otomatis; jika instance di-interrupt, training resume dari checkpoint terakhir; bisa hemat hingga 90% dibanding On-Demand training instances

---

**Soal 7**

Sebuah perusahaan ingin men-serve **ML model inference** dengan latency sangat rendah (< 5ms) dan throughput tinggi (ribuan requests per detik) untuk aplikasi real-time fraud detection.

Deployment option SageMaker mana yang tepat?

- A. SageMaker Batch Transform — proses data dalam batch
- B. SageMaker Serverless Inference — scale to zero untuk cost efficiency
- C. **SageMaker Real-Time Inference dengan dedicated endpoint** — persistent endpoint dengan instance yang selalu running, latency sub-10ms, bisa konfigurasi instance type yang tepat (GPU untuk model yang butuh GPU inference, CPU untuk model ringan), support Auto Scaling
- D. AWS Lambda dengan model di-load dari S3

---

**Soal 8**

Sebuah perusahaan menjalankan workload **containerized microservices** di Amazon EKS. Beberapa pods sering mengalami **OOMKilled (Out of Memory)** meskipun node masih memiliki memory tersedia.

Apa penyebab paling mungkin dan solusinya?

- A. Tambah lebih banyak nodes ke EKS cluster
- B. Aktifkan cluster autoscaler untuk scale nodes
- C. Pindahkan ke Fargate mode untuk managed compute
- D. **Naikkan memory requests dan limits di pod/container spec** — OOMKilled terjadi ketika container melebihi memory limit yang di-set di spec-nya; Kubernetes akan kill container tersebut. Solusinya: naikkan `resources.limits.memory` di pod spec agar container punya cukup memory headroom

---

**Soal 9**

Sebuah perusahaan memiliki **data lake di S3** dengan data yang di-update setiap hari. Mereka menggunakan Athena untuk query. Mereka mengalami masalah di mana query kadang mengembalikan data lama meskipun data baru sudah di-upload karena Athena masih membaca partisi metadata lama.

Cara yang tepat untuk menyelesaikan masalah ini?

- A. Hapus dan re-create tabel di Athena setiap kali ada data baru
- B. Gunakan S3 Event Notification untuk trigger Lambda saat ada upload baru
- C. **Jalankan `MSCK REPAIR TABLE` atau tambahkan partisi baru via `ALTER TABLE ADD PARTITION`** — Athena menyimpan metadata partisi di AWS Glue Data Catalog; saat partisi baru ditambahkan ke S3, Athena tidak otomatis tahu; perlu update catalog secara manual atau otomatis via Glue Crawler
- D. Aktifkan S3 Versioning agar Athena selalu baca versi terbaru

---

**Soal 10**

Sebuah perusahaan ingin membangun **real-time analytics dashboard** yang menampilkan metrics dari ribuan IoT devices. Data streaming masuk via Kinesis Data Streams dan perlu di-agregasi (count, avg, sum) per menit per device type, dengan hasil tersedia dalam detik.

Service mana yang paling tepat untuk melakukan aggregasi real-time ini?

- A. AWS Glue Streaming ETL
- B. AWS Batch untuk periodic aggregation setiap menit
- C. Lambda yang di-trigger oleh Kinesis setiap batch
- D. **Amazon Kinesis Data Analytics (managed Apache Flink)** — jalankan SQL atau Flink applications untuk proses streaming data in-flight; bisa windowing (tumbling window 1 menit), aggregasi, join stream; output ke Kinesis, S3, atau OpenSearch untuk dashboard; latency detik dengan throughput tinggi

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
