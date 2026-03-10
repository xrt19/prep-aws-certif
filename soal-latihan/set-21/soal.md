# Soal Latihan — Set 21
## Domain: Cost-Optimized Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menjalankan **data warehouse** di Amazon Redshift dengan cluster yang berjalan 24/7. Cluster digunakan intensif pada jam kerja untuk query analytics, tapi hampir tidak digunakan di malam hari dan weekend. Mereka ingin mengurangi biaya Redshift tanpa mengorbankan performance saat jam kerja.

Opsi mana yang paling cost-optimal?

- A. Snapshot cluster setiap malam dan delete cluster, restore setiap pagi
- B. Gunakan Redshift RA3 nodes dengan managed storage
- C. **Redshift Serverless** — bayar hanya saat ada queries yang berjalan (per RPU-second); tidak ada biaya saat idle; scale otomatis; cocok untuk intermittent analytics workload
- D. Beli Reserved Nodes Redshift untuk diskon

---

**Soal 2**

Sebuah perusahaan memiliki **banyak EC2 instances** dengan berbagai ukuran dan family. Mereka ingin mendapatkan diskon mirip Reserved Instances tapi dengan **fleksibilitas** untuk berubah instance type, OS, atau region tanpa penalty.

Purchasing option mana yang paling tepat?

- A. Standard Reserved Instances — diskon tertinggi, tidak fleksibel
- B. Spot Instances — diskon besar tapi bisa di-interrupt
- C. **Compute Savings Plans** — diskon hingga 66% vs On-Demand; berlaku untuk any EC2 instance family, size, OS, tenancy, dan region; juga berlaku untuk Lambda dan Fargate; lebih fleksibel dari RI
- D. Convertible Reserved Instances — bisa ubah instance attributes tapi diskon lebih kecil

---

**Soal 3**

Sebuah perusahaan memiliki **CloudFront distribution** yang melayani konten ke seluruh dunia. Mereka menemukan bahwa sebagian besar traffic berasal dari **US dan Europe** saja. CloudFront saat ini dikonfigurasi dengan price class yang mencakup semua edge locations termasuk Asia, South America, dan Australia yang mahal.

Cara mengurangi biaya CloudFront?

- A. Kurangi TTL cache agar lebih sedikit requests ke origin
- B. Pindahkan konten ke S3 Standard-IA untuk kurangi origin cost
- C. Aktifkan CloudFront compression untuk kurangi data transfer
- D. **Ubah CloudFront Price Class ke Price Class 100 (US, Europe, Israel)** — Price Class menentukan edge locations yang digunakan; Price Class 100 hanya gunakan edge locations di US dan Europe (region termurah); jika user di Asia, dilayani dari edge US/Europe dengan sedikit latency tambahan tapi cost jauh lebih murah

---

**Soal 4**

Sebuah perusahaan menjalankan **workload containerized** di ECS. Beberapa services adalah stateless dan toleran interruption (background jobs, image processing). Tim ingin mengurangi biaya Fargate secara signifikan untuk services ini.

Opsi mana yang paling tepat?

- A. Pindahkan ke EC2 launch type untuk lebih murah
- B. Kurangi CPU dan memory allocation di task definition
- C. **Gunakan Fargate Spot** — diskon hingga 70% dari harga Fargate regular; AWS bisa interrupt tasks dengan 2 menit notice; ideal untuk fault-tolerant, interruptible workloads seperti batch processing dan background jobs
- D. Gunakan ECS Capacity Providers dengan Spot EC2

---

**Soal 5**

Sebuah perusahaan memiliki **data di S3** yang sering diakses dalam 30 hari pertama setelah upload, kemudian sangat jarang diakses. Mereka sudah menggunakan S3 Standard, dan ingin mengurangi biaya storage tanpa kehilangan kemampuan akses cepat saat dibutuhkan.

Opsi mana yang paling tepat dengan pertimbangan minimum storage duration?

- A. S3 Intelligent-Tiering — monitoring fee per object
- B. **S3 Standard-IA (Infrequent Access)** — storage cost ~45% lebih murah dari Standard; minimum storage duration 30 hari; retrieval fee per GB; ideal untuk data yang sudah pasti jarang diakses setelah periode tertentu yang jelas
- C. S3 Glacier Instant Retrieval — lebih murah tapi minimum 90 hari
- D. S3 One Zone-IA — murah tapi data hanya di satu AZ

---

**Soal 6**

Sebuah perusahaan menggunakan **AWS Cost Explorer** dan menemukan bahwa biaya **data transfer** sangat tinggi. Setelah ditelusuri, banyak traffic keluar dari EC2 instances di `us-east-1` ke internet untuk men-serve file statis ke users di seluruh dunia.

Solusi paling cost-effective untuk mengurangi data transfer cost?

- A. Pindahkan semua EC2 ke region yang lebih murah data transfer-nya
- B. Kompres semua response HTTP sebelum dikirim ke client
- C. **Deploy CloudFront di depan EC2** — CloudFront cache responses di edge; data transfer dari EC2 ke CloudFront (origin fetch) jauh lebih murah dari data transfer langsung EC2 ke internet; setelah di-cache, tidak ada lagi origin fetch untuk requests yang sama
- D. Gunakan S3 Transfer Acceleration untuk semua file statis

---

**Soal 7**

Sebuah perusahaan memiliki **DynamoDB table** dengan provisioned capacity mode. Mereka membeli 1000 WCU dan 2000 RCU, tapi dari CloudWatch metrics terlihat average utilization hanya 20% WCU dan 15% RCU. Mereka membayar untuk capacity yang tidak terpakai.

Solusi mana yang paling cost-optimal?

- A. Aktifkan DynamoDB DAX untuk kurangi RCU consumption
- B. Buat Global Table untuk distribusikan load ke multiple regions
- C. **Switch ke DynamoDB On-Demand capacity mode** — bayar per actual request ($1.25 per million write request units, $0.25 per million read request units); tidak ada unused capacity yang terbuang; ideal untuk low atau unpredictable utilization
- D. Kurangi manual provisioned capacity ke 200 WCU dan 300 RCU

---

**Soal 8**

Sebuah perusahaan menggunakan banyak **AWS services** dan ingin mendapatkan visibilitas penuh tentang di mana biaya terbesar berasal, breakdown per service, per team, dan per environment (prod/dev/staging). Mereka juga ingin set alert jika biaya melebihi budget tertentu.

Tools AWS mana yang harus digunakan? (pilih kombinasi yang tepat)

- A. AWS CloudTrail — audit semua API calls untuk track resource usage
- B. AWS Config — track konfigurasi resource untuk compliance
- C. Amazon CloudWatch — monitor metrics dan set alarms
- D. **AWS Cost Explorer + AWS Budgets + Cost Allocation Tags** — Cost Explorer untuk analisis dan visualisasi spend; Budgets untuk alert threshold; Cost Allocation Tags untuk breakdown biaya per team/environment/project

---

**Soal 9**

Sebuah perusahaan memiliki **Amazon RDS** instance dengan storage yang terus bertumbuh. Mereka menggunakan `gp2` EBS volume untuk RDS storage dan mulai merasa biaya storage terlalu tinggi, terutama karena mereka memprovision lebih dari yang diperlukan untuk mendapatkan IOPS yang cukup (karena gp2 IOPS bergantung pada ukuran volume).

Cara mengoptimalkan cost RDS storage?

- A. Pindahkan ke Aurora yang memiliki storage auto-scaling
- B. Snapshot dan restore ke instance yang lebih kecil
- C. **Migrasi storage ke gp3** — gp3 memungkinkan provision IOPS dan throughput secara independent dari ukuran volume; bisa provision 3000 IOPS pada volume kecil (vs gp2 yang IOPS-nya bergantung storage size); biaya gp3 ~20% lebih murah dari gp2 per GB
- D. Gunakan Magnetic (standard) storage untuk lebih murah

---

**Soal 10**

Sebuah perusahaan menjalankan **Amazon EMR cluster** untuk nightly data processing jobs. Cluster di-provision dengan 10 core nodes + 20 task nodes. Jobs selesai dalam 4 jam. Setelah jobs selesai, cluster tetap running selama 2 jam sebelum ada yang terminate.

Cara paling tepat untuk mengoptimalkan biaya EMR ini?

- A. Pindahkan ke AWS Glue yang lebih murah untuk ETL
- B. Gunakan Reserved Instances untuk EMR instances
- C. Kurangi jumlah task nodes menjadi 10
- D. **Aktifkan EMR Auto Termination dan gunakan Spot Instances untuk task nodes** — Auto Termination otomatis terminate cluster setelah idle X menit (menghilangkan 2 jam waste); Spot Instances untuk task nodes bisa hemat 60–90% di node yang toleran interruption; core nodes tetap On-Demand untuk stability

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
