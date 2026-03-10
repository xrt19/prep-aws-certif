# Soal Latihan — Set 20
## Domain: Cost-Optimized Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menjalankan **EC2 instances** untuk aplikasi production yang harus berjalan 24/7 selama minimal 1 tahun. Mereka ingin mengurangi biaya EC2 secara signifikan dibanding On-Demand.

Purchasing option mana yang memberikan diskon terbesar dengan commitment 1 tahun tanpa upfront payment?

- A. On-Demand Instances — bayar per jam, tidak ada commitment
- B. Spot Instances — diskon hingga 90% tapi bisa di-interrupt
- C. **Reserved Instances (1-year, No Upfront)** — komitmen 1 tahun memberikan diskon hingga 40% dibanding On-Demand; No Upfront berarti tidak perlu bayar di muka; cocok untuk workload steady-state yang predictable dan harus running terus
- D. Dedicated Hosts — physical server dedicated, diskon untuk multi-year

---

**Soal 2**

Sebuah startup memiliki **development dan testing environment** yang hanya digunakan Senin–Jumat pukul 09:00–18:00 (9 jam/hari, 5 hari/minggu). Di luar jam tersebut, environment tidak digunakan tapi EC2 instances terus berjalan.

Cara paling cost-effective untuk mengelola environment ini?

- A. Pindahkan ke Spot Instances agar lebih murah
- B. Gunakan Reserved Instances untuk diskon
- C. **Jadwalkan start/stop EC2 dengan EventBridge + Lambda atau Instance Scheduler** — matikan instances di luar jam kerja; dari 168 jam/minggu menjadi 45 jam/minggu (~73% pengurangan biaya compute)
- D. Pindahkan ke ECS Fargate yang scale to zero

---

**Soal 3**

Sebuah perusahaan menyimpan **log files** di S3. Log lama (> 90 hari) jarang diakses, dan log yang sangat lama (> 1 tahun) hampir tidak pernah diakses tapi harus disimpan untuk compliance selama 7 tahun.

Konfigurasi S3 Lifecycle mana yang paling cost-optimal?

- A. Simpan semua log di S3 Standard selamanya untuk akses cepat kapanpun
- B. Hapus semua log setelah 90 hari untuk hemat biaya
- C. Pindahkan ke S3 Glacier Instant Retrieval setelah 30 hari
- D. **Gunakan S3 Lifecycle Policy bertingkat**: S3 Standard (0–90 hari) → S3 Standard-IA (90–365 hari) → S3 Glacier Flexible Retrieval (365 hari–7 tahun) → Delete setelah 7 tahun; setiap tier lebih murah dari sebelumnya untuk storage cost

---

**Soal 4**

Sebuah perusahaan memiliki **RDS MySQL** instance yang berjalan 24/7 untuk aplikasi internal yang hanya aktif pada jam kerja (Senin–Jumat, 09:00–17:00). Di luar jam kerja, database jarang diakses tapi tetap berjalan dan di-charge per jam.

Solusi paling cost-effective?

- A. Gunakan RDS Reserved Instance untuk diskon 1 tahun
- B. Snapshot database setiap malam dan restore setiap pagi
- C. **Migrate ke Aurora Serverless v2** — scale down ke minimum ACU (hampir nol) saat tidak ada traffic; charge hanya untuk ACU yang digunakan + storage; ideal untuk workload intermittent
- D. Pindahkan ke DynamoDB untuk menghindari biaya per jam RDS

---

**Soal 5**

Sebuah perusahaan memiliki **S3 bucket** dengan jutaan objek yang access pattern-nya tidak jelas — beberapa objek sering diakses, sebagian jarang, dan pola berubah-ubah dari waktu ke waktu. Tim tidak mau manage lifecycle rules secara manual.

Storage class S3 mana yang paling cost-optimal untuk scenario ini?

- A. S3 Standard — akses cepat, biaya storage lebih tinggi
- B. S3 Standard-IA — biaya storage lebih murah tapi ada minimum storage duration 30 hari
- C. **S3 Intelligent-Tiering** — otomatis pindahkan objek antar tiers (Frequent Access, Infrequent Access, Archive Instant Access) berdasarkan access patterns aktual; tidak ada retrieval fee; tidak ada operational overhead untuk manage lifecycle
- D. S3 One Zone-IA — satu AZ saja, lebih murah tapi kurang resilient

---

**Soal 6**

Sebuah perusahaan menjalankan **batch processing jobs** setiap malam menggunakan EC2 instances. Jobs membutuhkan 100 vCPU selama 3–4 jam dan toleran terhadap interruption (jobs bisa di-checkpoint dan restart). Jobs harus selesai sebelum jam 06:00 pagi.

Purchasing strategy mana yang paling cost-effective?

- A. On-Demand Instances — reliable tapi mahal untuk batch jobs
- B. Reserved Instances 1 tahun — terlalu lama untuk jobs yang hanya beberapa jam
- C. **Spot Instances dengan Spot Fleet** — request Spot capacity dengan beberapa instance types dan AZs; harga hingga 90% lebih murah; jobs di-checkpoint sehingga toleran interruption; Spot Fleet otomatis ganti instance yang di-reclaim
- D. Dedicated Hosts — paling mahal, tidak perlu untuk batch jobs

---

**Soal 7**

Sebuah perusahaan memiliki **aplikasi web** dengan traffic yang sangat tidak predictable — bisa ratusan requests per menit atau ribuan per detik. Mereka saat ini menggunakan EC2 dengan Auto Scaling tapi sering over-provision karena takut insufficient capacity.

Arsitektur mana yang paling cost-optimal untuk workload ini?

- A. EC2 dengan Reserved Instances untuk diskon
- B. EC2 dengan Spot Instances untuk harga murah
- C. **AWS Lambda** — serverless, charge per invocation dan duration (per ms); tidak ada idle cost; scale otomatis dari 0 hingga ribuan concurrent; ideal untuk unpredictable, stateless workloads
- D. ECS dengan Fargate Spot untuk harga lebih murah

---

**Soal 8**

Sebuah perusahaan menggunakan **NAT Gateway** di setiap AZ untuk EC2 instances di private subnet yang perlu akses internet. Tim menemukan biaya NAT Gateway sangat tinggi terutama karena banyak traffic ke **S3 dan DynamoDB**.

Cara paling efektif untuk mengurangi biaya ini?

- A. Pindahkan semua EC2 ke public subnet agar tidak perlu NAT Gateway
- B. Gunakan single NAT Gateway di satu AZ saja untuk hemat biaya
- C. **Buat VPC Gateway Endpoints untuk S3 dan DynamoDB** — traffic ke S3 dan DynamoDB lewat endpoint langsung via AWS network, tidak melalui NAT Gateway; Gateway Endpoints gratis; hanya traffic ke internet lain yang tetap butuh NAT
- D. Kompres semua data yang dikirim ke S3 untuk kurangi data transfer cost

---

**Soal 9**

Sebuah perusahaan memiliki **aplikasi data processing** yang perlu menyimpan hasil sementara (intermediate results) yang hanya dibutuhkan selama processing berlangsung (beberapa jam), setelah itu data tidak diperlukan lagi.

Storage solution mana yang paling cost-optimal?

- A. S3 Standard — durable tapi bayar per GB walaupun data hanya sementara
- B. EBS gp3 volume — persistent SSD, bayar per GB-month
- C. **EC2 Instance Store** — storage ephemeral yang tidak di-charge terpisah (sudah termasuk dalam harga instance); performa tinggi; ideal untuk data sementara yang tidak perlu persist
- D. EFS — shared file system, bayar per GB yang tersimpan

---

**Soal 10**

Sebuah perusahaan menggunakan **AWS Organizations** dengan banyak AWS accounts. Beberapa accounts memiliki Reserved Instances yang tidak terpakai sepenuhnya, sementara accounts lain menggunakan On-Demand yang mahal.

Fitur AWS mana yang memungkinkan berbagi benefit Reserved Instances antar accounts?

- A. AWS RAM (Resource Access Manager) — share resources antar accounts
- B. AWS Cost Explorer — analisis dan visualisasi biaya
- C. **Reserved Instance Sharing via AWS Organizations** — aktifkan RI Sharing di management account; Reserved Instances dari satu account bisa digunakan oleh accounts lain dalam organization yang sama; unused RI capacity otomatis di-apply ke On-Demand usage di accounts lain
- D. AWS Budgets — set alert ketika biaya melebihi threshold

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
