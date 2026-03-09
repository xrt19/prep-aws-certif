# Soal Latihan — Set 16
## Domain: High-Performing Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menjalankan batch processing jobs yang membutuhkan ribuan vCPU selama beberapa jam, kemudian tidak dibutuhkan lagi. Budget sangat terbatas dan mereka mau toleransi interruption asalkan jobs bisa di-resume.

EC2 purchasing option mana yang paling tepat untuk use case ini?

- A. On-Demand Instances — bayar per jam tanpa commitment
- B. Reserved Instances — diskon besar untuk commitment 1–3 tahun
- C. **Spot Instances** — harga hingga 90% lebih murah dari On-Demand, cocok untuk fault-tolerant batch workloads yang bisa di-checkpoint dan di-resume saat interruption
- D. Dedicated Hosts — dedicated physical server untuk licensing compliance

---

**Soal 2**

Sebuah aplikasi DynamoDB sering melakukan query seperti:
- `WHERE status = 'PENDING' ORDER BY created_at DESC`
- `WHERE user_id = 123 AND status = 'ACTIVE'`

Tabel memiliki `order_id` sebagai partition key. Query-query ini tidak bisa menggunakan table's primary key secara efisien.

Fitur DynamoDB mana yang tepat untuk mengoptimalkan queries ini?

- A. DynamoDB DAX — in-memory cache untuk read latency
- B. DynamoDB Streams — capture item-level changes
- C. **Global Secondary Index (GSI)** — buat index dengan partition key dan sort key yang berbeda dari tabel utama, memungkinkan query patterns yang fleksibel
- D. DynamoDB On-Demand capacity — scale otomatis saat ada traffic spike

---

**Soal 3**

Sebuah tim data analytics perlu **query data CSV langsung dari S3** menggunakan SQL tanpa perlu load data ke database atau ETL pipeline. Dataset CSV berukuran besar (beberapa GB) dan mereka hanya butuh subset kolom/baris tertentu.

Fitur S3 mana yang tepat?

- A. S3 Byte-Range Fetches — retrieve sebagian file berdasarkan byte offset
- B. **S3 Select** — jalankan SQL query langsung pada objek S3 (CSV, JSON, Parquet), hanya data yang match filter yang dikirim ke client, mengurangi data transfer drastis
- C. S3 Batch Operations — jalankan operasi pada banyak objek sekaligus
- D. S3 Inventory — generate report daftar objek di bucket

---

**Soal 4**

Sebuah perusahaan memiliki **data warehouse di Amazon Redshift**. Query analytics tertentu sangat lambat karena melakukan full table scan pada tabel besar. Tim ingin mengoptimalkan tanpa menambah node Redshift.

Teknik optimasi Redshift mana yang paling tepat untuk kasus ini?

- A. Tambah Read Replica Redshift untuk query analytics
- B. Aktifkan Redshift Serverless untuk auto-scaling
- C. **Pilih distribution key dan sort key yang tepat** — distribution key menentukan bagaimana data di-distribute ke nodes (mengurangi data shuffling antar nodes), sort key memungkinkan zone maps untuk skip blocks yang tidak relevan
- D. Pindahkan data ke S3 dan gunakan Athena

---

**Soal 5**

Sebuah perusahaan menjalankan **search functionality** di aplikasi e-commerce mereka. Mereka butuh full-text search, faceted search (filter by category, price range), dan real-time search suggestions. Data product perlu diindex dan bisa dicari dalam milidetik.

AWS service mana yang paling tepat?

- A. RDS MySQL dengan LIKE query dan FULLTEXT index
- B. DynamoDB dengan GSI untuk setiap filter attribute
- C. **Amazon OpenSearch Service** — managed Elasticsearch/OpenSearch untuk full-text search, faceted search, dan analytics pada semi-structured data
- D. Amazon Redshift untuk complex SQL analytics queries

---

**Soal 6**

Sebuah aplikasi menggunakan **API Gateway + Lambda** sebagai backend. Banyak API calls melakukan query yang sama ke database dengan parameter yang sama. Tim ingin mengurangi latency dan beban database untuk repeated identical requests.

Fitur API Gateway mana yang tepat?

- A. API Gateway Usage Plans — batasi jumlah requests per API key
- B. API Gateway Custom Domain — gunakan domain sendiri
- C. **API Gateway caching** — aktifkan response caching di stage level, responses di-cache berdasarkan method + path + query parameters, TTL bisa dikonfigurasi, mengurangi invocations ke Lambda dan queries ke database
- D. API Gateway WebSocket API — ganti ke persistent connection

---

**Soal 7**

Sebuah Lambda function melakukan banyak komputasi CPU-intensive (image processing). Saat ini dikonfigurasi dengan 512 MB memory dan waktu eksekusi rata-rata 10 detik. Tim ingin mengurangi waktu eksekusi tanpa mengubah kode.

Cara paling efektif untuk meningkatkan performance Lambda ini?

- A. Tambah Lambda Layers untuk split kode menjadi lebih modular
- B. Pindahkan ke runtime terbaru (Python 3.12 dari Python 3.9)
- C. **Naikkan memory allocation** — Lambda mendapat CPU proporsional dengan memory yang di-alokasikan. Naikkan memory (misalnya ke 2048 MB atau 3008 MB) secara otomatis meningkatkan CPU power yang tersedia, bisa significantly mengurangi execution time
- D. Gunakan Lambda Reserved Concurrency untuk prioritisasi

---

**Soal 8**

Sebuah perusahaan menjalankan ECS tasks di Fargate. Beberapa task container melakukan preprocessing data yang sangat CPU-intensive. Task sering timeout karena CPU throttled meskipun memory masih tersedia banyak.

Apa yang harus diubah?

- A. Tambah lebih banyak tasks untuk distribusi beban
- B. Pindahkan ke EC2 launch type untuk lebih banyak control
- C. Naikkan memory limit task definition
- D. **Naikkan CPU unit allocation di task definition** — di Fargate, CPU dan memory di-define di level task. CPU throttling terjadi ketika CPU units kurang — naikkan CPU units (misalnya dari 256 ke 1024 atau 2048 vCPU units) agar container mendapat lebih banyak compute power

---

**Soal 9**

Sebuah perusahaan menggunakan **Auto Scaling dengan target tracking policy** untuk aplikasi yang memiliki traffic pattern sangat predictable — traffic tinggi setiap weekday pukul 09:00–17:00 dan rendah di luar jam tersebut. Tim menginginkan scaling yang proaktif, bukan reaktif.

Strategi scaling mana yang paling cocok dikombinasikan dengan target tracking?

- A. Step Scaling — scale berdasarkan threshold metric yang berbeda-beda
- B. Simple Scaling — scale dengan cooldown period setelah setiap scaling activity
- C. **Predictive Scaling** — analisis historical traffic patterns untuk forecast dan pre-scale sebelum traffic naik, ideal untuk cyclical patterns yang predictable, dikombinasikan dengan target tracking untuk handling unexpected spikes
- D. Lifecycle Hooks — pause scaling untuk custom action

---

**Soal 10**

Sebuah aplikasi HPC (High-Performance Computing) menggunakan EC2 instances dalam **Cluster Placement Group** dengan workload **MPI (Message Passing Interface)** — ribuan nodes perlu berkomunikasi dengan low latency dan high bandwidth antar-node. Standard Enhanced Networking tidak cukup untuk kebutuhan mereka.

Network adapter apa yang direkomendasikan?

- A. Enhanced Networking (ENA) — SR-IOV untuk bandwidth tinggi
- B. Jumbo Frames (MTU 9001) — meningkatkan transfer efficiency
- C. AWS Direct Connect — dedicated connection bandwidth tinggi
- D. **Elastic Fabric Adapter (EFA)** — network interface khusus untuk HPC/MPI workloads, memungkinkan bypass OS kernel network stack (OS-bypass) untuk komunikasi antar-node dengan latency sangat rendah, mendukung RDMA (Remote Direct Memory Access)

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
