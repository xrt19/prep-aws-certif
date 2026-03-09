# Soal Latihan — Set 14
## Domain: High-Performing Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan media menyajikan **video dan gambar statis** ke user di seluruh dunia. User di Asia dan Eropa mengeluhkan loading yang lambat karena konten di-serve dari single S3 bucket di `us-east-1`.

Solusi paling tepat untuk meningkatkan performance secara global adalah?

- A. Buat S3 bucket di setiap region dan sync manual
- B. Gunakan S3 Transfer Acceleration untuk upload/download lebih cepat
- C. **Deploy Amazon CloudFront** di depan S3 — konten di-cache di edge locations di seluruh dunia, user dilayani dari edge terdekat
- D. Pindahkan S3 bucket ke region yang lebih sentral (misalnya eu-west-1)

---

**Soal 2**

Sebuah aplikasi gaming online membutuhkan **latency serendah mungkin** (single-digit milliseconds) untuk pemain di seluruh dunia yang terhubung ke server di AWS. Koneksi menggunakan **TCP dan UDP**, bukan HTTP.

Service mana yang paling tepat untuk optimasi network path ke AWS?

- A. Amazon CloudFront — cache konten di edge locations
- B. Route 53 latency-based routing — route ke region terdekat
- C. **AWS Global Accelerator** — routing traffic melalui AWS global network backbone, bukan public internet, untuk TCP/UDP dengan latency konsisten dan rendah
- D. AWS Direct Connect — koneksi dedicated dari user ke AWS

---

**Soal 3**

Sebuah perusahaan menjalankan workload **machine learning training** yang sangat compute-intensive. Jobs berjalan selama berjam-jam dan membutuhkan **GPU** yang powerful.

Family EC2 instance mana yang paling tepat?

- A. C-family (c7g, c6i) — compute optimized
- B. R-family (r6i, r7g) — memory optimized
- C. **P-family atau G-family** (p4d, g5) — GPU instances, dirancang untuk ML training dan inference
- D. M-family (m7i, m6i) — general purpose

---

**Soal 4**

Sebuah perusahaan menjalankan **HPC cluster** dengan ribuan EC2 instances yang saling berkomunikasi dengan **very low latency** dan **high throughput** antar instances. Mereka ingin meminimalkan network latency antar instances.

EC2 Placement Group tipe mana yang paling tepat?

- A. Spread Placement Group — instances di-spread ke berbagai hardware
- B. Partition Placement Group — instances di-group ke partitions terpisah
- C. **Cluster Placement Group** — instances di-pack dalam satu AZ, connected via high-bandwidth low-latency network (bisa pakai Enhanced Networking / EFA)
- D. Default placement — AWS pilihkan lokasi optimal

---

**Soal 5**

Sebuah database EC2 (self-managed) membutuhkan **storage dengan IOPS tertinggi yang konsisten** dan **latency serendah mungkin**. Database sangat I/O intensive dengan random read/write pattern.

Tipe EBS volume mana yang tepat?

- A. gp3 — general purpose SSD, bisa provision IOPS terpisah dari storage
- B. st1 — throughput optimized HDD, throughput tinggi tapi latency lebih tinggi
- C. sc1 — cold HDD, untuk infrequent access
- D. **io2 Block Express** — highest IOPS (hingga 256.000 IOPS), sub-millisecond latency, dirancang untuk I/O intensive databases

---

**Soal 6**

Sebuah aplikasi membaca **objek S3 yang sama berulang-ulang** dalam waktu singkat — ratusan requests per detik ke object yang sama. Tim menemukan S3 mulai throttle requests.

Solusi mana yang paling tepat untuk meningkatkan throughput read dari S3?

- A. Aktifkan S3 Transfer Acceleration
- B. Naikkan S3 request rate limit via AWS Support
- C. **Deploy CloudFront di depan S3** — cache object di edge, ratusan requests ke object yang sama di-serve dari cache tanpa hit S3
- D. Gunakan S3 Multipart Upload untuk object yang sering dibaca

---

**Soal 7**

Sebuah aplikasi web memiliki **database queries yang lambat** karena banyak expensive JOIN queries untuk data yang **jarang berubah** (misalnya data referensi, katalog produk). Queries ini menyebabkan RDS terbebani.

Solusi paling tepat untuk meningkatkan read performance tanpa mengubah arsitektur database secara besar-besaran?

- A. Upgrade RDS instance ke yang lebih besar
- B. Aktifkan RDS Multi-AZ untuk distribusi reads
- C. Tambahkan **RDS Read Replica** untuk offload read queries dari primary
- D. Gunakan DynamoDB sebagai pengganti RDS untuk queries tersebut

---

**Soal 8**

Sebuah EC2 instance perlu **transfer data dengan bandwidth tinggi** antar instances dalam satu VPC. Tim ingin memastikan network performance maksimal antara dua instances yang sering berkomunikasi.

Fitur EC2 mana yang harus diaktifkan?

- A. Elastic IP — assign static IP untuk komunikasi lebih stabil
- B. **Enhanced Networking (ENA)** — single root I/O virtualization (SR-IOV) yang memberikan higher bandwidth, lower latency, lower jitter dibanding traditional virtualization
- C. Elastic Fabric Adapter (EFA) — khusus untuk MPI workloads HPC
- D. Jumbo frames — naikkan MTU ke 9001 bytes

---

**Soal 9**

Sebuah startup menggunakan Lambda untuk API backend. Mereka mengeluhkan bahwa request pertama setelah periode idle selalu **lebih lambat** (1–2 detik) dibanding request berikutnya.

Apa penyebabnya, dan solusi apa yang menghilangkan masalah ini untuk fungsi production-critical?

- A. Lambda memory terlalu kecil — naikkan memory untuk mempercepat execution
- B. **Cold start** — Lambda perlu initialize execution environment saat pertama kali atau setelah idle. Solusi: aktifkan **Provisioned Concurrency** agar Lambda selalu warm
- C. VPC Lambda lebih lambat — pindahkan Lambda ke non-VPC
- D. Lambda timeout terlalu pendek — naikkan timeout

---

**Soal 10**

Sebuah perusahaan ingin meng-upload **file besar (10 GB)** ke S3 dengan **reliabilitas tinggi** — jika upload gagal di tengah jalan, tidak harus mulai ulang dari awal. Mereka juga ingin **paralelisasi upload** untuk kecepatan maksimal.

Fitur S3 mana yang tepat?

- A. S3 Transfer Acceleration — mempercepat upload via CloudFront edge
- B. S3 Presigned URL — generate URL upload yang aman
- C. **S3 Multipart Upload** — upload file dalam beberapa parts secara paralel, jika satu part gagal hanya re-upload part tersebut, parts bisa di-upload concurrent
- D. S3 Batch Operations — proses banyak objects sekaligus

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
