# Jawaban — Set 14
## Domain: High-Performing Architectures

---

## Soal 1 — Jawaban: C

**Amazon CloudFront**

**Kenapa C:**
CloudFront adalah **Content Delivery Network (CDN)** dengan 450+ edge locations di seluruh dunia. Cara kerja:
1. Request user di Asia atau Eropa masuk ke edge location terdekat
2. Jika konten ada di cache edge (cache hit) → langsung di-serve, sangat cepat
3. Jika cache miss → CloudFront fetch dari S3 origin, cache di edge, serve ke user

Hasilnya: latency turun drastis karena user tidak perlu round-trip ke `us-east-1`.

**Kenapa bukan yang lain:**
- **A** — Buat bucket di setiap region dan sync manual adalah sangat complex, tidak scalable, dan butuh manajemen konsistensi data yang rumit.
- **B** — S3 Transfer Acceleration mempercepat **upload ke S3** dari klien yang jauh, bukan untuk delivery/download konten ke end user.
- **D** — Pindah ke eu-west-1 hanya membantu user Eropa, tapi user Asia masih jauh. CloudFront cover semua regions sekaligus.

**Konsep yang diuji:** CloudFront CDN, edge locations, cache, global content delivery, S3 origin.

---

## Soal 2 — Jawaban: C

**AWS Global Accelerator**

**Kenapa C:**
Global Accelerator memberikan **2 static anycast IP addresses** yang menjadi entry point global. Traffic dari user masuk ke **AWS edge location terdekat**, lalu dirouting melalui **AWS private global network backbone** (bukan public internet) ke endpoint di region target. Keuntungan:
- **Consistent low latency** — AWS backbone jauh lebih stabil dari public internet
- Support **TCP dan UDP** (tidak hanya HTTP seperti CloudFront)
- **Instant failover** jika endpoint di satu region down
- Tidak ada caching — sesuai untuk real-time interactive applications

**Kenapa bukan yang lain:**
- **A** — CloudFront adalah CDN dengan caching — lebih tepat untuk static content HTTP/HTTPS. Tidak cache-friendly untuk real-time gaming state yang selalu dinamis. Juga tidak native support UDP.
- **B** — Route 53 latency-based routing meminimalisir DNS lookup time ke region terdekat, tapi traffic tetap melewati public internet setelah itu — tidak ada optimasi network path.
- **D** — Direct Connect untuk dedicated private connection dari **on-premise/office** ke AWS, bukan untuk end-user consumers yang tersebar di seluruh dunia.

**Konsep yang diuji:** Global Accelerator, AWS backbone network, TCP/UDP, anycast IP, vs CloudFront.

---

## Soal 3 — Jawaban: C

**P-family atau G-family (GPU instances)**

**Kenapa C:**
AWS menyediakan GPU instances dalam dua keluarga utama:
- **P-family (p4d, p3)**: NVIDIA A100/V100 GPUs — untuk **ML training** yang butuh compute power sangat tinggi, multi-GPU interconnect via NVLink
- **G-family (g5, g4dn)**: NVIDIA A10G/T4 GPUs — untuk **ML inference**, graphics rendering, video encoding

Untuk ML training yang compute-intensive dan berjam-jam, P-family adalah pilihan utama.

**Kenapa bukan yang lain:**
- **A** — C-family (Compute Optimized) untuk CPU-intensive workloads seperti batch processing, web servers, scientific modeling yang tidak butuh GPU.
- **B** — R-family (Memory Optimized) untuk workloads yang butuh RAM sangat besar: in-memory databases, big data analytics, real-time processing large datasets.
- **D** — M-family (General Purpose) untuk balanced workloads — tidak optimal untuk ML training yang butuh specialized GPU hardware.

**Konsep yang diuji:** EC2 instance families, GPU instances P/G family, ML training vs inference, compute vs memory vs GPU optimization.

---

## Soal 4 — Jawaban: C

**Cluster Placement Group**

**Kenapa C:**
Cluster Placement Group menempatkan instances secara **fisik berdekatan** dalam satu AZ di same underlying hardware rack atau dekat satu sama lain. Hasilnya:
- **Network latency terendah** antar instances (single-digit microseconds)
- **Bandwidth tertinggi** (hingga 100 Gbps dengan instances yang support Enhanced Networking)
- Bisa dikombinasikan dengan **Elastic Fabric Adapter (EFA)** untuk MPI/HPC workloads
- Trade-off: semua instances di satu AZ → tidak redundant

**Kenapa bukan yang lain:**
- **A** — Spread Placement Group menempatkan instances di **hardware yang berbeda** untuk maximize availability (toleran hardware failure) — tapi ini meningkatkan jarak fisik antar instances, meningkatkan latency. Kebalikan dari yang dibutuhkan.
- **B** — Partition Placement Group membagi instances ke partitions berbeda (tiap partition = rack berbeda) untuk fault isolation — untuk distributed systems seperti Hadoop/Cassandra, bukan untuk low-latency HPC communication.
- **D** — Default placement tidak ada guarantee tentang co-location — instances bisa berada di hardware yang jauh satu sama lain.

**Konsep yang diuji:** EC2 Placement Groups, Cluster vs Spread vs Partition, HPC networking, EFA.

---

## Soal 5 — Jawaban: D

**io2 Block Express**

**Kenapa D:**
io2 Block Express adalah tipe EBS tertinggi performanya:
- **IOPS**: hingga **256,000 IOPS** per volume (vs io1 maksimum 64,000 IOPS)
- **Latency**: **sub-millisecond** (< 1 ms)
- **Throughput**: hingga 4,000 MB/s
- **Durability**: 99.999% (vs 99.8–99.9% untuk tipe lain)
- Ideal untuk: Oracle, SQL Server, SAP HANA, MySQL/PostgreSQL dengan I/O sangat tinggi

**Kenapa bukan yang lain:**
- **A** — gp3 bisa provision IOPS hingga 16,000 IOPS — jauh di bawah io2 Block Express. Tepat untuk most workloads tapi bukan untuk "highest IOPS yang konsisten".
- **B** — st1 (throughput optimized HDD) untuk sequential read/write large blocks (big data, log processing, data warehouses) — latency lebih tinggi dari SSD, tidak cocok untuk random I/O database.
- **C** — sc1 (cold HDD) untuk infrequently accessed data — IOPS dan throughput sangat rendah.

**Konsep yang diuji:** EBS volume types, io2 Block Express, IOPS comparison, latency, database storage selection.

---

## Soal 6 — Jawaban: C

**CloudFront di depan S3**

**Kenapa C:**
Ini adalah **caching problem** — ratusan requests ke object yang sama. Solusinya adalah menempatkan cache di depan S3:
- CloudFront cache object di edge location
- Ratusan requests berikutnya untuk object yang sama di-serve dari **CloudFront cache** — tidak ada request yang masuk ke S3
- S3 request rate throttle tidak lagi menjadi bottleneck
- Bonus: latency lebih rendah karena served dari edge

**Kenapa bukan yang lain:**
- **A** — Transfer Acceleration mempercepat **upload** ke S3 via edge network — tidak membantu dengan throttling pada read/download requests.
- **B** — S3 default sudah cukup tinggi: **3,500 PUT/COPY/POST/DELETE** dan **5,500 GET/HEAD** requests **per second per prefix**. Tapi throttling bisa terjadi jika melampaui ini. Naik limit tidak menyelesaikan root cause — caching lebih efektif.
- **D** — Multipart Upload untuk mengupload file besar — tidak ada hubungannya dengan read throttling.

**Konsep yang diuji:** CloudFront as S3 cache, S3 request rate limits, caching pattern untuk high read throughput.

---

## Soal 7 — Jawaban: C

**RDS Read Replica**

**Kenapa C:**
RDS Read Replica adalah **asynchronous replica** dari primary database yang bisa melayani **read-only queries**:
- Offload read traffic dari primary → primary bebas untuk writes
- Bisa buat hingga 15 Read Replicas untuk Aurora (5 untuk RDS MySQL/PostgreSQL)
- Bisa di-region berbeda (cross-region Read Replica)
- Aplikasi perlu di-configure untuk send read queries ke replica endpoint, write ke primary endpoint

Ideal untuk use case ini: read-heavy workload dengan data yang jarang berubah.

**Kenapa bukan yang lain:**
- **A** — Upgrade instance menambah resource tapi tidak mengurangi jumlah queries ke primary. Scaling vertical ada batasnya.
- **B** — **Exam trap klasik**: RDS Multi-AZ standby **tidak melayani read queries**. Standby hanya untuk failover. Ini sudah dibahas di Set 09 tapi sering muncul di exam.
- **D** — Migrasi ke DynamoDB adalah perubahan arsitektur besar — data model berbeda (NoSQL vs relational), queries perlu diubah, dan mungkin tidak cocok untuk semua use cases.

**Konsep yang diuji:** RDS Read Replica untuk offload reads, Multi-AZ tidak serve reads (exam trap), read scaling patterns.

---

## Soal 8 — Jawaban: B

**Enhanced Networking (ENA)**

**Kenapa B:**
Enhanced Networking menggunakan **SR-IOV (Single Root I/O Virtualization)** untuk memberikan network performance yang mendekati bare-metal:
- **Higher bandwidth**: hingga 100 Gbps (dengan instances yang support)
- **Lower latency**: lebih rendah dari traditional virtualized networking
- **Lower CPU overhead**: network processing lebih efisien
- **Lower jitter**: konsistensi latency lebih baik

Enhanced Networking (via ENA driver) sudah otomatis tersedia di most modern instance types — tidak perlu konfigurasi khusus, otomatis aktif.

**Kenapa bukan yang lain:**
- **A** — Elastic IP untuk static public IP address, tidak mempengaruhi bandwidth atau latency komunikasi antar instances dalam VPC.
- **C** — EFA (Elastic Fabric Adapter) adalah level di atas Enhanced Networking — untuk **MPI (Message Passing Interface)** workloads HPC yang butuh bypass OS kernel untuk communicate antar nodes. Overkill untuk general high-bandwidth inter-instance communication.
- **D** — Jumbo frames (MTU 9001) meningkatkan efficiency untuk large transfers tapi butuh konfigurasi di kedua sisi dan tidak secara fundamental meningkatkan raw bandwidth seperti Enhanced Networking.

**Konsep yang diuji:** Enhanced Networking, ENA, SR-IOV, ENA vs EFA, network performance optimization.

---

## Soal 9 — Jawaban: B

**Cold start — solusi: Provisioned Concurrency**

**Kenapa B:**
**Cold start** terjadi ketika Lambda perlu:
1. Download function code/container image
2. Start execution environment (runtime, dependencies)
3. Run initialization code (koneksi DB, load config)

Proses ini butuh waktu — biasanya 100ms hingga beberapa detik tergantung runtime dan dependencies. Terjadi saat: pertama kali di-invoke, setelah idle period, atau saat scale out ke instance baru.

**Provisioned Concurrency** pre-warms sejumlah execution environments — mereka selalu siap, tidak ada initialization delay saat request masuk.

**Kenapa bukan yang lain:**
- **A** — Memory lebih banyak memang mempercepat execution (Lambda dapat CPU proporsional dengan memory), tapi tidak menghilangkan initialization time yang menyebabkan cold start. Cold start tetap ada.
- **C** — Lambda dalam VPC dulu memang lebih lambat cold start karena perlu setup ENI. Tapi sejak 2020, AWS sudah fix ini dengan shared ENI — Lambda VPC tidak lagi signifikan lebih lambat dari non-VPC.
- **D** — Timeout adalah batas maksimum execution time — tidak ada hubungannya dengan cold start latency pada awal invocation.

**Konsep yang diuji:** Lambda cold start, Provisioned Concurrency vs Reserved Concurrency, initialization overhead.

---

## Soal 10 — Jawaban: C

**S3 Multipart Upload**

**Kenapa C:**
S3 Multipart Upload memecah file besar menjadi **parts** (minimum 5 MB per part, maksimum 10,000 parts):
- **Paralelisasi**: upload banyak parts secara bersamaan → memanfaatkan full network bandwidth
- **Resiliency**: jika satu part gagal, hanya re-upload part tersebut — tidak dari awal
- **Better throughput**: aggregate bandwidth dari parallel connections
- AWS **merekomendasikan** Multipart Upload untuk file > 100 MB, dan **wajib** untuk file > 5 GB

**Kenapa bukan yang lain:**
- **A** — Transfer Acceleration mempercepat transfer menggunakan CloudFront edge (routing yang lebih optimal), tapi tidak memberikan resiliency "tidak harus mulai ulang" dan tidak memparalelisasi upload secara native. Keduanya bisa digabungkan tapi Multipart adalah yang menjawab requirement utama.
- **B** — Presigned URL untuk mengamankan akses upload (URL yang ber-expire) — tidak ada hubungannya dengan reliability atau paralelisasi untuk file besar.
- **D** — Batch Operations untuk execute operations (copy, tag, restore) pada **existing objects** dalam S3 dalam skala besar — bukan untuk upload file baru.

**Konsep yang diuji:** S3 Multipart Upload, parallel upload, fault tolerance, large file upload best practices.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | CloudFront CDN, edge locations, global content delivery |
| 2 | Global Accelerator, AWS backbone, TCP/UDP, vs CloudFront |
| 3 | EC2 GPU instances P/G family, ML training |
| 4 | Cluster Placement Group, HPC low latency |
| 5 | EBS io2 Block Express, highest IOPS sub-ms latency |
| 6 | CloudFront as S3 cache, read throttling solution |
| 7 | RDS Read Replica offload reads (Multi-AZ tidak serve reads) |
| 8 | Enhanced Networking ENA, SR-IOV, ENA vs EFA |
| 9 | Lambda cold start, Provisioned Concurrency |
| 10 | S3 Multipart Upload, parallel, resume on failure |
