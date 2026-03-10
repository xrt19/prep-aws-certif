# Jawaban — Set 20
## Domain: Cost-Optimized Architectures

---

## Soal 1 — Jawaban: C

**Reserved Instances (1-year, No Upfront)**

**Kenapa C:**
Reserved Instances (RI) memberikan diskon signifikan untuk commitment jangka panjang. Perbandingan payment options untuk 1-year term:

| Payment Option | Diskon vs On-Demand | Cara Bayar |
|---------------|--------------------|-----------|
| No Upfront | ~35–40% | Bayar per bulan |
| Partial Upfront | ~40–45% | Bayar sebagian di muka + per bulan |
| All Upfront | ~45–50% | Bayar penuh di muka |

Untuk workload yang:
- Berjalan **24/7** (steady-state, predictable)
- Minimal **1 tahun**
- Tidak bisa di-interrupt

RI adalah pilihan yang tepat. No Upfront dipilih karena soal minta "tanpa upfront payment".

**Kenapa bukan yang lain:**
- **A** — On-Demand paling mahal untuk workload steady-state. Tidak ada diskon.
- **B** — Spot bisa hemat 90%, tapi bisa di-interrupt dengan 2 menit notice. Tidak cocok untuk production yang harus running 24/7 tanpa interruption.
- **D** — Dedicated Hosts untuk compliance licensing (SQL Server, Oracle). Lebih mahal dari RI standar dan tidak diperlukan jika tidak ada multi-tenant isolation requirement.

**Konsep yang diuji:** EC2 Reserved Instances, payment options (No/Partial/All Upfront), use case untuk steady-state workloads, RI vs Spot vs On-Demand.

---

## Soal 2 — Jawaban: C

**Jadwalkan start/stop EC2**

**Kenapa C:**
Menghitung penghematan:
- Sebelum: 168 jam/minggu × harga On-Demand
- Sesudah: 45 jam/minggu (9 jam × 5 hari) × harga On-Demand
- **Penghematan: ~73%**

Cara implementasi:
1. **AWS Instance Scheduler** (AWS solution): schedule via DynamoDB config table + Lambda
2. **EventBridge + Lambda**: cron rule `cron(0 9 ? * MON-FRI *)` untuk start, `cron(0 18 ? * MON-FRI *)` untuk stop
3. **EC2 Auto Scaling Scheduled Actions** jika menggunakan ASG

Ini jauh lebih hemat dibanding Reserved Instances yang tetap di-charge 24/7 meskipun tidak running.

**Kenapa bukan yang lain:**
- **A** — Spot Instances murah tapi tidak bisa dijadwal start/stop dengan pasti — Spot bisa di-reclaim kapanpun. Untuk dev/test yang butuh reliable di jam kerja, Spot berisiko.
- **B** — Reserved Instances di-charge **per jam selama masa komitmen** meskipun instance di-stop. Stop instance tidak menghentikan billing RI. Tidak ada penghematan dari stop.
- **D** — ECS Fargate scale to zero bisa bekerja untuk stateless containers, tapi ini adalah migration besar dari EC2. Soal minta cara cost-effective yang pragmatis untuk environment yang sudah ada.

**Konsep yang diuji:** EC2 start/stop scheduling, Instance Scheduler, EventBridge cron, dev/test cost optimization, RI billing saat stopped.

---

## Soal 3 — Jawaban: D

**S3 Lifecycle Policy bertingkat**

**Kenapa D:**
S3 menyediakan beberapa storage classes dengan trade-off antara harga storage vs retrieval cost vs availability:

| Storage Class | Storage Cost | Retrieval | Min Duration | Use Case |
|--------------|-------------|-----------|--------------|----------|
| S3 Standard | $0.023/GB | Gratis | Tidak ada | Hot data, frequent access |
| S3 Standard-IA | $0.0125/GB | Per GB | 30 hari | Infrequent, > 128 KB |
| S3 Glacier Instant Retrieval | $0.004/GB | Per GB, ms | 90 hari | Quarterly access |
| S3 Glacier Flexible Retrieval | $0.0036/GB | Per GB, menit-jam | 90 hari | Yearly access, compliance |
| S3 Glacier Deep Archive | $0.00099/GB | Per GB, jam | 180 hari | Long-term archive |

Lifecycle rules memindahkan objects secara otomatis berdasarkan umur — tidak perlu manual intervention.

Untuk compliance data 7 tahun, Glacier Flexible Retrieval jauh lebih murah dari Standard (~84% lebih murah per GB).

**Kenapa bukan yang lain:**
- **A** — Simpan semua di Standard selamanya: paling mahal untuk data yang jarang diakses. Tidak optimal sama sekali.
- **B** — Hapus setelah 90 hari melanggar compliance requirement 7 tahun. Tidak bisa dilakukan.
- **C** — Glacier Instant Retrieval setelah 30 hari lebih mahal dari Glacier Flexible karena ada minimum storage duration 90 hari dan harga per GB lebih tinggi. Juga memindahkan data terlalu cepat ke Glacier — log 30–90 hari masih mungkin diakses untuk debugging.

**Konsep yang diuji:** S3 storage classes, S3 Lifecycle Policy, tiered storage, compliance archiving, cost comparison per tier.

---

## Soal 4 — Jawaban: C

**Aurora Serverless v2**

**Kenapa C:**
Aurora Serverless v2 di-charge berdasarkan **ACU (Aurora Capacity Units)** yang dikonsumsi per detik:
- Minimum ACU bisa di-set ke 0.5 ACU (hampir nol) — biaya sangat rendah saat idle
- Scale up otomatis dalam detik saat ada traffic
- Di luar jam kerja (155 jam/minggu dari 168): hanya bayar 0.5 ACU × storage
- Vs RDS instance tetap: bayar per jam meskipun idle

**Perkiraan penghematan:**
- RDS db.t3.medium: ~$0.068/jam × 168 jam = ~$11.4/minggu
- Aurora Serverless v2: ~45 jam aktif (normal ACU) + 123 jam idle (0.5 ACU minimum) → jauh lebih murah

**Kenapa bukan yang lain:**
- **A** — RI 1 tahun memberikan diskon tapi di-charge tetap per jam meskipun database idle di malam hari. Tidak menyelesaikan masalah idle cost.
- **B** — Snapshot setiap malam dan restore setiap pagi: (1) Restore RDS butuh menit-an → ada downtime; (2) Snapshot storage cost (meski murah); (3) Operational complexity; (4) Tetap bayar untuk jam yang instance running. Tidak praktis.
- **D** — DynamoDB untuk menggantikan MySQL relational database adalah perubahan arsitektur besar — data model berbeda, queries perlu diubah total. Over-engineered untuk tujuan cost saving.

**Konsep yang diuji:** Aurora Serverless v2, ACU billing, scale to minimum, intermittent workload cost optimization, vs RDS per-hour billing.

---

## Soal 5 — Jawaban: C

**S3 Intelligent-Tiering**

**Kenapa C:**
S3 Intelligent-Tiering dirancang untuk **unknown atau changing access patterns**:

**Tiers dalam Intelligent-Tiering:**
| Tier | Aktif setelah | Storage cost |
|------|-------------|-------------|
| Frequent Access | Default (baru upload) | Sama dengan S3 Standard |
| Infrequent Access | 30 hari tanpa akses | ~45% lebih murah |
| Archive Instant Access | 90 hari tanpa akses | ~68% lebih murah |
| Archive Access (optional) | 90+ hari | ~95% lebih murah |
| Deep Archive Access (optional) | 180+ hari | ~95% lebih murah |

**Key benefits:**
- **Tidak ada retrieval fee** antar tiers (Frequent ↔ Infrequent)
- **Tidak ada minimum storage duration** charge untuk Frequent Access tier
- **Monitoring fee** kecil per object ($0.0025 per 1000 objects) — worth it untuk banyak objek
- Saat objek diakses kembali → otomatis kembali ke Frequent Access tier

**Kenapa bukan yang lain:**
- **A** — S3 Standard untuk semua objek membayar harga tertinggi ($0.023/GB) meskipun banyak objek jarang diakses.
- **B** — Standard-IA memiliki **minimum storage duration 30 hari** dan **retrieval fee per GB**. Jika objek sering berganti akses pattern, retrieval fee bisa lebih mahal dari selisih storage cost. Juga perlu lifecycle rules untuk manage manual.
- **D** — S3 One Zone-IA menyimpan data di **satu AZ saja** — jika AZ tersebut down, data tidak tersedia. Untuk data production yang penting, ini bukan trade-off yang acceptable hanya untuk cost.

**Konsep yang diuji:** S3 Intelligent-Tiering, automatic tiering, access pattern monitoring, no retrieval fee, vs Standard-IA dan One Zone-IA.

---

## Soal 6 — Jawaban: C

**Spot Instances dengan Spot Fleet**

**Kenapa C:**
Batch processing jobs dengan checkpoint adalah **use case ideal untuk Spot**:
- Harga Spot bisa **70–90% lebih murah** dari On-Demand
- Spot Fleet: request capacity dari multiple instance types (c5.2xlarge, c5a.2xlarge, m5.2xlarge) dan multiple AZs → availability lebih tinggi, lebih sedikit interruption
- **Checkpoint strategy**: simpan progress ke S3 setiap N menit → jika instance di-reclaim, Spot Fleet launch instance baru, jobs resume dari checkpoint
- Target: selesai sebelum jam 06:00 → ada buffer waktu yang cukup untuk retry jika ada interruption

**Allocation strategies Spot Fleet:**
- `lowestPrice`: pilih pool dengan harga terendah → maximum savings
- `capacityOptimized`: pilih pool dengan kapasitas terbanyak → minimum interruption
- `priceCapacityOptimized`: balance harga dan capacity (recommended)

**Kenapa bukan yang lain:**
- **A** — On-Demand reliable tapi paling mahal untuk batch jobs yang toleran interruption. Tidak optimal.
- **B** — Reserved Instances commitment 1 tahun untuk jobs yang hanya berjalan 3–4 jam per malam. RI di-charge per jam sepanjang masa komitmen — waste jika hanya digunakan 3–4 jam/hari.
- **D** — Dedicated Hosts untuk dedicated physical server (licensing compliance). Paling mahal di antara semua opsi. Tidak ada keuntungan untuk batch processing jobs.

**Konsep yang diuji:** Spot Fleet, allocation strategies, checkpointing, batch processing cost optimization, Spot interruption handling.

---

## Soal 7 — Jawaban: C

**AWS Lambda**

**Kenapa C:**
Lambda adalah pilihan optimal untuk workload dengan traffic yang sangat **unpredictable**:

**Model biaya Lambda:**
- **Requests**: $0.20 per 1 juta requests
- **Duration**: $0.0000166667 per GB-second (1ms granularity)
- **Tidak ada idle cost** — tidak running, tidak bayar

**Perbandingan dengan EC2:**
- EC2 On-Demand: bayar per jam meskipun traffic 0
- EC2 dengan ASG: scale down tapi masih ada minimum instances
- Lambda: cost = 0 saat traffic 0; scale infinitely saat traffic tinggi

Untuk aplikasi web stateless yang bisa di-handle Lambda (< 15 menit execution, < 10 GB memory), ini adalah arsitektur paling cost-optimal untuk unpredictable traffic.

**Kenapa bukan yang lain:**
- **A** — RI untuk EC2 di-charge per jam sepanjang commitment period. Untuk unpredictable traffic dengan banyak idle time, RI tetap mahal karena bayar saat idle.
- **B** — Spot untuk web application production berisiko — Spot bisa di-interrupt, menyebabkan downtime singkat yang tidak acceptable untuk user-facing application.
- **D** — Fargate Spot lebih murah dari Fargate regular (~70%), tapi masih ada minimum per-task charge dan Spot containers bisa di-interrupt. Lambda lebih cost-optimal untuk truly variable workload.

**Konsep yang diuji:** Lambda pricing model, no idle cost, serverless for unpredictable traffic, Lambda vs EC2 vs Fargate cost comparison.

---

## Soal 8 — Jawaban: C

**VPC Gateway Endpoints untuk S3 dan DynamoDB**

**Kenapa C:**
NAT Gateway di-charge dua komponen:
1. **Per hour**: $0.045/jam per NAT Gateway
2. **Per GB data processed**: $0.045/GB

Traffic ke S3 dan DynamoDB via NAT Gateway di-charge **$0.045/GB** padahal traffic ini sebenarnya tidak perlu keluar ke internet — S3 dan DynamoDB adalah AWS services yang bisa diakses via **private AWS network**.

**VPC Gateway Endpoints:**
- **Gratis** — tidak ada biaya per jam atau per GB
- Traffic ke S3/DynamoDB lewat route table entry ke endpoint, bukan NAT
- Tinggal tambah route di route table private subnet: `pl-xxxxxx (com.amazonaws.us-east-1.s3) → vpce-xxxxxxxx`
- Tidak ada perubahan kode aplikasi

Dengan traffic S3/DynamoDB yang besar, penghematan bisa **sangat signifikan**.

**Kenapa bukan yang lain:**
- **A** — Pindah ke public subnet adalah security anti-pattern. Database dan application servers tidak boleh di public subnet. Tidak acceptable trade-off.
- **B** — Single NAT Gateway di satu AZ memang menghemat per-hour cost (dari 3 NAT GW menjadi 1), tapi: (1) Single point of failure — jika AZ tersebut down, semua instances kehilangan internet access; (2) Biaya data processing tetap sama. Gateway Endpoints lebih efektif.
- **D** — Kompresi mengurangi bytes yang di-transfer, tapi NAT Gateway masih di-charge untuk bytes yang diproses. Gateway Endpoints menghilangkan cost tersebut sepenuhnya untuk S3/DynamoDB traffic.

**Konsep yang diuji:** VPC Gateway Endpoints, NAT Gateway cost components, S3/DynamoDB private access, cost optimization network.

---

## Soal 9 — Jawaban: C

**EC2 Instance Store**

**Kenapa C:**
EC2 Instance Store adalah **ephemeral storage** yang:
- **Tidak di-charge terpisah** — sudah termasuk dalam harga instance (berbeda dengan EBS yang di-charge per GB-month)
- Secara fisik attached ke host server → latency sangat rendah, throughput sangat tinggi
- Data **hilang** saat instance stop, terminate, atau hardware failure
- Tersedia di instance types tertentu: i3, i4i, d2, h1, c5d, m5d, r5d (huruf `d` di akhir berarti ada NVMe instance store)

Untuk data intermediate/temporary yang hanya dibutuhkan selama processing:
- Tidak perlu durability → instance store sempurna
- Tidak bayar extra untuk storage → cost optimal
- Performa tinggi → processing lebih cepat

**Kenapa bukan yang lain:**
- **A** — S3 Standard di-charge per GB yang disimpan dan per request. Untuk data temporary beberapa jam, tetap ada biaya dan overhead latency (network call ke S3).
- **B** — EBS di-charge per GB-month (minimum 1 bulan) dan per IOPS (untuk provisioned). Untuk data yang hanya dibutuhkan beberapa jam, bayar satu bulan penuh adalah waste.
- **D** — EFS di-charge per GB yang tersimpan. Sama seperti EBS, bayar untuk durasi jauh lebih lama dari yang dibutuhkan.

**Konsep yang diuji:** EC2 Instance Store, ephemeral storage, no additional cost, intermediate data, Instance Store vs EBS vs S3 cost.

---

## Soal 10 — Jawaban: C

**Reserved Instance Sharing via AWS Organizations**

**Kenapa C:**
AWS Organizations memiliki fitur **Consolidated Billing** yang secara default mengaktifkan **RI Sharing** (sebelumnya disebut RI benefit sharing):

**Cara kerja:**
- Payer account (management account) mengaktifkan consolidated billing
- RI yang dibeli di Account A bisa di-apply ke On-Demand usage yang cocok di Account B, C, D dalam organization yang sama
- Matching criteria: instance family, region, OS, tenancy
- Prioritas: RI di-apply ke account yang membelinya dulu; jika ada unused capacity, baru di-share ke account lain
- Tidak ada konfigurasi manual — otomatis terjadi via consolidated billing

**Benefit:**
- RI utilization naik (tidak ada unused RI yang terbuang)
- Account yang pakai On-Demand tapi ada matching RI di account lain otomatis dapat diskon
- Bisa di-opt-out per account jika tidak mau

**Kenapa bukan yang lain:**
- **A** — AWS RAM untuk share AWS resources (subnet, Transit Gateway, Resource Groups, dll) antar accounts. Bukan untuk share RI pricing benefit.
- **B** — Cost Explorer untuk **analisis dan visualisasi** biaya dan usage. Bisa show RI utilization, tapi tidak bisa share RI benefit.
- **D** — AWS Budgets untuk **alerting** saat cost atau usage melebihi threshold. Berguna untuk cost governance, bukan untuk mengoptimalkan RI sharing.

**Konsep yang diuji:** AWS Organizations Consolidated Billing, RI Sharing, RI benefit across accounts, multi-account cost optimization.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | EC2 Reserved Instances, payment options, steady-state workload |
| 2 | EC2 start/stop scheduling, dev/test cost optimization |
| 3 | S3 Lifecycle Policy, tiered storage classes, compliance archiving |
| 4 | Aurora Serverless v2, ACU billing, intermittent workload |
| 5 | S3 Intelligent-Tiering, automatic tiering, unknown access patterns |
| 6 | Spot Fleet, checkpointing, batch processing cost |
| 7 | Lambda pricing, no idle cost, unpredictable traffic |
| 8 | VPC Gateway Endpoints, NAT Gateway cost reduction |
| 9 | EC2 Instance Store, ephemeral storage, no additional cost |
| 10 | RI Sharing via AWS Organizations Consolidated Billing |
