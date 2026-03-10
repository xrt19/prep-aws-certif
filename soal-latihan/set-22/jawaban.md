# Jawaban — Set 22
## Domain: Cost-Optimized Architectures

---

## Soal 1 — Jawaban: C

**Pindahkan ke Fargate atau EC2 Spot untuk batch processing**

**Kenapa C:**
Lambda pricing: **$0.0000166667 per GB-second** (dengan 1 ms granularity).

Hitung biaya satu invocation Lambda:
- 3 GB memory × 15 menit (900 detik) = **2700 GB-seconds**
- 2700 × $0.0000166667 = **$0.045 per invocation**
- Jika 10.000 invocations/bulan = $450/bulan hanya dari Lambda duration cost

Bandingkan dengan Fargate:
- Fargate Spot 2 vCPU + 4 GB = ~$0.012/jam
- 15 menit = $0.003 per task
- 10.000 tasks/bulan = $30/bulan → **15x lebih murah**

Lambda optimal untuk **short-lived, event-driven** functions. Untuk heavy computation yang berjalan belasan menit, Fargate/ECS atau AWS Batch dengan Spot lebih hemat.

**Kenapa bukan yang lain:**
- **A** — Naikkan memory ke 10 GB memang bisa mempercepat eksekusi (lebih banyak CPU), tapi biaya per GB-second tetap sama. Jika eksekusi turun dari 15 menit menjadi 5 menit: 10 GB × 300 detik = 3000 GB-seconds — bahkan lebih mahal.
- **B** — EC2 On-Demand juga lebih murah dari Lambda untuk use case ini, tapi ada idle cost. Fargate Spot lebih optimal karena serverless + Spot pricing.
- **D** — Provisioned Concurrency untuk **eliminasi cold start**, bukan untuk mengurangi duration cost. Biaya per GB-second tetap sama, plus ada biaya tambahan untuk Provisioned Concurrency itu sendiri.

**Konsep yang diuji:** Lambda pricing model, GB-second calculation, Lambda vs Fargate cost comparison, right tool for long-running heavy tasks.

---

## Soal 2 — Jawaban: D

**Refactor ke serverless: API Gateway + Lambda**

**Kenapa D:**
EC2 t3.medium di-charge **per jam**, 24/7, meskipun idle:
- t3.medium On-Demand (us-east-1): ~$0.0416/jam
- Per bulan: ~$30/bulan meskipun hanya diakses beberapa kali per hari

Lambda + API Gateway untuk traffic sangat rendah:
- Free tier: 1 juta Lambda requests/bulan gratis, 400.000 GB-seconds gratis
- API Gateway: 1 juta REST API calls/bulan = $3.50
- Untuk aplikasi beberapa request per hari: kemungkinan **masuk free tier** atau biaya sangat kecil (< $1/bulan)

**Kenapa bukan yang lain:**
- **A** — t3.nano lebih kecil (2 vCPU shared, 0.5 GB RAM) dan lebih murah (~$0.0052/jam = ~$3.7/bulan), tapi masih ada idle cost 24/7. Juga mungkin tidak cukup untuk workload yang ada.
- **B** — RI 1 tahun memberikan diskon ~40% tapi tetap bayar ~$18/bulan untuk t3.medium RI No Upfront. Masih ada idle cost untuk aplikasi yang jarang digunakan.
- **C** — Spot Instance bisa di-interrupt — tidak cocok untuk aplikasi yang harus selalu available (meskipun jarang diakses).

**Konsep yang diuji:** Serverless cost model vs EC2 idle cost, Lambda free tier, API Gateway + Lambda pattern, right architecture for low-traffic applications.

---

## Soal 3 — Jawaban: C

**Aktifkan "Delete on Termination" flag**

**Kenapa C:**
Saat launch EC2 instance, setiap EBS volume memiliki flag **DeleteOnTermination**:
- **Root volume**: default `true` — otomatis dihapus saat instance terminate
- **Additional volumes**: default `false` — tetap ada setelah terminate

Untuk menghindari "orphaned" EBS volumes:
1. Saat launch: set `DeleteOnTermination=true` untuk semua volumes yang tidak butuh data setelah terminate
2. Untuk existing instance: bisa update via AWS CLI: `aws ec2 modify-instance-attribute --block-device-mappings "[{\"DeviceName\":\"/dev/sdf\",\"Ebs\":{\"DeleteOnTermination\":true}}]"`

EBS di-charge per GB-month meskipun tidak ter-attach ke instance ($0.08/GB-month untuk gp3) — "orphaned" volumes terus mengakumulasi biaya.

**Kenapa bukan yang lain:**
- **A** — CloudWatch alarm untuk EBS bisa monitor metrics (volume idle, dll) tapi tidak otomatis delete. Tetap butuh manual action atau Lambda trigger.
- **B** — AWS Config rule bisa detect unattached volumes dan flag sebagai non-compliant, tapi remediation (delete) perlu dikonfigurasi terpisah via SSM Automation. Lebih complex dari DeleteOnTermination flag.
- **D** — Lambda scheduled untuk scan dan delete: approach yang valid tapi lebih reactive (cleanup setelah fakta) dan butuh coding + deploy + maintain. DeleteOnTermination adalah preventive approach yang lebih elegant.

**Konsep yang diuji:** EBS DeleteOnTermination flag, orphaned EBS volumes, EBS pricing when unattached, proactive cost management.

---

## Soal 4 — Jawaban: C

**Atur retention period dan hapus manual snapshots**

**Kenapa C:**
RDS backup pricing:
- **Automated backup**: storage gratis hingga 100% ukuran database provisioned (misal DB 100 GB → 100 GB backup storage gratis). Biaya muncul jika melebihi ini.
- **Manual snapshots**: di-charge penuh untuk setiap GB yang disimpan, **tidak ada free tier**, dan **tidak otomatis dihapus** — akumulasi terus sampai dihapus manual.

Solusi:
1. Review dan hapus manual snapshots yang tidak diperlukan
2. Atur retention policy untuk manual snapshots: misal hapus snapshots lebih dari 30 hari via Lambda scheduled atau AWS Backup lifecycle policy
3. Review retention period automated backup (default 7 hari, max 35 hari) — sesuaikan dengan kebutuhan RTO/RPO

**Kenapa bukan yang lain:**
- **A** — Matikan automated backup: (1) Ini menghilangkan kemampuan Point-in-Time Recovery; (2) Meningkatkan risiko data loss secara signifikan; (3) Melanggar best practices. Tidak acceptable untuk production.
- **B** — Retention 1 hari: terlalu berisiko. Jika ada corruption atau data loss yang baru diketahui setelah 2+ hari, tidak ada backup untuk restore. Tradeoff cost vs risk yang buruk.
- **D** — RDS snapshot tidak bisa dipindahkan langsung ke Glacier. Snapshot di-store di S3 managed oleh AWS (tidak di S3 bucket user). Untuk long-term archive bisa export snapshot ke S3 kemudian transisi ke Glacier, tapi ini lebih complex dan tidak menyelesaikan masalah akumulasi snapshots.

**Konsep yang diuji:** RDS automated backup vs manual snapshot pricing, backup free tier, snapshot lifecycle management, cost optimization.

---

## Soal 5 — Jawaban: C

**Elastic IP di-charge saat TIDAK ter-asosiasikan ke running instance**

**Kenapa C:**
Elastic IP (EIP) pricing model AWS:
- EIP ter-asosiasikan ke **running instance**: **GRATIS**
- EIP ter-asosiasikan ke **stopped instance**: $0.005/jam
- EIP **tidak ter-asosiasikan** (idle/unattached): $0.005/jam
- EIP ter-asosiasikan ke network interface yang tidak attached ke instance: $0.005/jam

Logika AWS: EIP gratis saat digunakan (running instance), di-charge saat "idle" karena menghabiskan IPv4 public address yang terbatas.

**Best practices:**
- Release EIP yang tidak diperlukan
- Jangan stop instance terlalu lama jika tidak ingin bayar EIP
- Gunakan Elastic IP hanya jika benar-benar butuh static public IP

**Kenapa bukan yang lain:**
- **A** — Salah. EIP tidak gratis selamanya — di-charge saat tidak digunakan.
- **B** — Salah. EIP di-charge juga saat asosiasi ke stopped instance, bukan hanya saat running.
- **D** — Salah. EIP di-charge per jam, bukan per request/traffic.

**Konsep yang diuji:** Elastic IP pricing, when EIP is free vs charged, idle EIP cost, IPv4 address management.

---

## Soal 6 — Jawaban: C

**Consolidated Billing**

**Kenapa C:**
AWS Organizations Consolidated Billing menggabungkan usage dari semua member accounts:

**Volume discount tiers (contoh S3 storage):**
| Usage | Rate per GB |
|-------|-----------|
| 0–50 TB | $0.023 |
| 50–500 TB | $0.022 |
| > 500 TB | $0.021 |

Tanpa Consolidated Billing: Account A = 30 TB, Account B = 30 TB, Account C = 30 TB → semua di tier pertama ($0.023).

Dengan Consolidated Billing: total 90 TB → 50 TB di $0.023 + 40 TB di $0.022 → rata-rata lebih murah.

Benefit lain:
- **Satu tagihan** untuk semua accounts
- **RI sharing** (dibahas di Set 20)
- Free tier sharing (pertama di family)
- Support plan cost sharing

**Kenapa bukan yang lain:**
- **A** — Control Tower untuk automated multi-account setup, guardrails, dan governance. Tidak langsung terkait dengan billing aggregation atau volume discounts.
- **B** — Service Catalog untuk standardized product portfolios dan self-service deployment. Tidak ada fungsi billing.
- **D** — Config Aggregator untuk aggregate compliance dan configuration data dari semua accounts ke satu view. Tidak ada fungsi billing.

**Konsep yang diuji:** AWS Organizations Consolidated Billing, volume discounts, tiered pricing, multi-account billing aggregation.

---

## Soal 7 — Jawaban: D

**AWS Cost Explorer dengan Savings Plans recommendations**

**Kenapa D:**
Cost Explorer memiliki fitur rekomendasi yang sangat actionable:

**Savings Plans Recommendations:**
- Analisis historical usage (pilih lookback period: 7, 30, atau 60 hari)
- Kalkulasi berapa komitmen $/jam yang optimal
- Proyeksi savings dalam 1 tahun atau 3 tahun
- Breakdown: current On-Demand cost vs estimated cost with Savings Plan

**Output rekomendasi:**
```
Recommended commitment: $2.50/hour (Compute Savings Plans, 1-year, no upfront)
Estimated monthly savings: $450 (35% reduction)
Coverage: 78% of your On-Demand usage
```

Bisa langsung purchase dari Cost Explorer.

**Kenapa bukan yang lain:**
- **A** — Trusted Advisor cek under-utilized EC2 instances (CPU < 10% selama 14 hari), tapi tidak memberikan spesifik rekomendasi RI/Savings Plans dengan projected savings amount. Lebih general.
- **B** — Cost Explorer RI Recommendations ada dan valid, tapi soal menyebut "Savings Plans recommendations" — lebih fleksibel. Keduanya valid tapi Savings Plans recommendations mencakup lebih banyak use cases (Lambda, Fargate juga).
- **C** — Compute Optimizer untuk right-sizing instance (ganti m5.xlarge ke m5.large karena CPU rendah). Berbeda dengan pricing optimization (RI/Savings Plans). Keduanya komplementer tapi berbeda tujuan.

**Konsep yang diuji:** Cost Explorer Savings Plans recommendations, lookback period, commitment optimization, projected savings.

---

## Soal 8 — Jawaban: C

**S3 Lifecycle rules bertingkat untuk backup**

**Kenapa C:**
Lifecycle policy sesuai retention requirements:

```
Hari 0: Upload backup → S3 Standard
Hari 30: Transisi → S3 Standard-IA (backup harian tidak lagi fresh)
Hari 365: Transisi → S3 Glacier Deep Archive (hanya untuk audit/compliance)
Hari 1825 (5 tahun): Delete
```

**Perbandingan cost (per GB/bulan):**
- S3 Standard: $0.023
- S3 Standard-IA: $0.0125
- Glacier Deep Archive: $0.00099

Backup yang berumur 1–5 tahun di Deep Archive **23x lebih murah** dari Standard. Untuk volume backup yang besar, penghematan ini sangat signifikan.

Minimum storage duration harus diperhatikan:
- Standard-IA: 30 hari minimum
- Glacier Deep Archive: 180 hari minimum
- Lifecycle transition ke Standard-IA minimum setelah 30 hari dari upload

**Kenapa bukan yang lain:**
- **A** — Semua di Glacier Deep Archive: backup 0–30 hari tidak accessible dengan cepat (retrieval 12+ jam). Jika ada kebutuhan restore backup harian yang baru, ini tidak praktis. Retrieval fee juga mahal untuk frequent access.
- **B** — Intelligent-Tiering: ada monitoring fee per object. Untuk backup dengan access pattern yang **sudah jelas** (semakin tua semakin jarang diakses), lifecycle rules lebih cost-optimal dari Intelligent-Tiering.
- **D** — EBS snapshots lebih mahal dari S3 untuk storage ($0.05/GB-month vs $0.023/GB-month Standard), tidak support Glacier tiers, dan tidak sefleksibel S3 Lifecycle rules.

**Konsep yang diuji:** S3 Lifecycle rules for backup, tiered storage by age, Glacier Deep Archive for long-term, minimum storage duration constraints.

---

## Soal 9 — Jawaban: D

**AWS Compute Optimizer**

**Kenapa D:**
AWS Compute Optimizer adalah **ML-powered recommendation service** untuk right-sizing:

**Coverage:**
- **EC2 instances**: rekomendasikan instance type berdasarkan CPU, memory, network, disk I/O utilization
- **Auto Scaling groups**: rekomendasikan instance type untuk grup
- **EBS volumes**: rekomendasikan volume type dan size
- **Lambda functions**: rekomendasikan memory allocation optimal
- **ECS on Fargate**: rekomendasikan CPU dan memory untuk tasks

**Cara kerja:**
- Analisis CloudWatch metrics selama 14 hari (default) atau hingga 93 hari
- Identifikasi: Over-provisioned, Under-provisioned, atau Optimized
- Rekomendasikan instance type spesifik: "Ganti m5.2xlarge ke m5.large → hemat $120/bulan"
- Proyeksi performance risk untuk setiap rekomendasi

**Kenapa bukan yang lain:**
- **A** — Cost Explorer menunjukkan biaya dan trend, bisa filter per instance type, tapi tidak memberikan rekomendasi right-sizing spesifik berdasarkan performance metrics.
- **B** — Trusted Advisor cek EC2 Low Utilization (CPU < 10% selama 4 days atau lebih dari dua hari puncak — flag untuk review). Lebih threshold-based dan less granular dibanding Compute Optimizer.
- **C** — CloudWatch monitor metrics tapi tidak memberikan rekomendasi right-sizing. Perlu analyze sendiri dan decide.

**Konsep yang diuji:** AWS Compute Optimizer, ML-powered right-sizing, EC2 Lambda EBS Fargate coverage, over-provisioned resources.

---

## Soal 10 — Jawaban: D

**Gunakan private IP untuk komunikasi intra-VPC**

**Kenapa D:**
Data transfer pricing AWS:

| Transfer Type | Cost |
|--------------|------|
| Dalam satu AZ, via private IP | **Gratis** |
| Antar AZ, via private IP | $0.01/GB (each direction) |
| EC2 ke internet (NAT Gateway) | NAT cost $0.045/GB + data transfer out $0.09/GB |
| EC2 ke EC2 via public IP/Elastic IP (same region) | $0.01/GB each direction |

**Masalah yang terjadi:**
Services memanggil satu sama lain menggunakan **public DNS name** atau public Elastic IP. Ketika EC2 di private subnet resolve public DNS → dapat public IP → traffic keluar via NAT Gateway ke internet → masuk kembali ke AWS. Padahal destination-nya ada di VPC yang sama!

**Solusi:** Gunakan private IP atau private DNS (`ip-10-0-1-50.ec2.internal`) untuk service-to-service communication dalam VPC yang sama. Traffic via private IP dalam satu AZ = **$0**.

**Kenapa bukan yang lain:**
- **A** — VPC Peering untuk koneksi antar **dua VPC berbeda**. Jika semua services sudah di VPC yang sama, VPC Peering tidak diperlukan dan tidak menyelesaikan masalah.
- **B** — VPC Flow Logs untuk **monitoring dan analysis** traffic. Berguna untuk diagnosa tapi tidak mengurangi biaya.
- **C** — Pindah ke public subnet security anti-pattern. Public subnet expose instances ke internet directly. Tidak acceptable.

**Konsep yang diuji:** Data transfer pricing, intra-VPC private IP free, public IP routing cost, common misconfiguration causing unnecessary NAT charges.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | Lambda vs Fargate cost untuk long-running tasks, GB-second pricing |
| 2 | Serverless (Lambda+API GW) vs EC2 idle cost, low-traffic apps |
| 3 | EBS DeleteOnTermination flag, orphaned volumes |
| 4 | RDS snapshot pricing, manual vs automated backup cost |
| 5 | Elastic IP pricing, charged when unassociated |
| 6 | Consolidated Billing, volume discounts, tiered pricing |
| 7 | Cost Explorer Savings Plans recommendations |
| 8 | S3 Lifecycle rules bertingkat untuk backup retention |
| 9 | AWS Compute Optimizer, ML right-sizing |
| 10 | Intra-VPC private IP free, public IP routing NAT cost |
