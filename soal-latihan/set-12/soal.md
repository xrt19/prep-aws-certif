# Soal Latihan — Set 12
## Domain: Resilient Architectures

> 10 soal. Jawab semua dulu sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menggunakan ElastiCache Redis untuk session caching. Mereka khawatir jika **primary Redis node down**, semua session data hilang dan user ter-logout. Mereka butuh Redis yang bisa **otomatis failover** ke node lain tanpa kehilangan data.

Konfigurasi ElastiCache mana yang tepat?

- A. ElastiCache Memcached dengan multiple nodes
- B. ElastiCache Redis single node dengan snapshot harian
- C. **ElastiCache Redis dengan Multi-AZ dan automatic failover** — primary di satu AZ, replica di AZ lain, failover otomatis dalam hitungan menit
- D. ElastiCache Redis Cluster Mode Disabled, tambah read replicas

---

**Soal 2**

Sebuah aplikasi Lambda mengalami masalah: saat ada **spike traffic tiba-tiba**, banyak Lambda invocations gagal dengan error `TooManyRequestsException`. Investigasi menunjukkan bahwa Lambda telah mencapai batas concurrency akun.

Solusi mana yang paling tepat untuk memastikan fungsi Lambda kritis **selalu punya capacity** meskipun ada fungsi Lambda lain yang mengkonsumsi banyak concurrency?

- A. Naikkan memory Lambda function — ini juga menambah concurrency
- B. Pindahkan Lambda ke region lain yang tidak ramai
- C. **Set Reserved Concurrency** pada Lambda function kritis — mengalokasikan pool concurrency eksklusif untuk fungsi tersebut, tidak bisa diambil fungsi lain
- D. Gunakan Provisioned Concurrency agar Lambda selalu warm

---

**Soal 3**

Sebuah perusahaan menjalankan workload HPC (High Performance Computing) yang membutuhkan **shared file system dengan throughput sangat tinggi** (ratusan GB/s) dan **low latency** antar EC2 instances yang berjalan secara paralel.

Storage service mana yang paling tepat?

- A. Amazon EFS — shared file system yang managed
- B. Amazon S3 — object storage untuk data HPC
- C. **Amazon FSx for Lustre** — high-performance parallel file system, terintegrasi dengan S3, dirancang untuk HPC dan machine learning
- D. Amazon EBS Multi-Attach — share satu EBS volume ke beberapa instances

---

**Soal 4**

Sebuah perusahaan memiliki **on-premise servers** yang sering mengalami hardware failure. Mereka ingin solusi DR yang bisa **replicate on-premise servers ke AWS** secara continuous, dan jika terjadi disaster, bisa **failover ke AWS dalam hitungan menit**.

Service AWS mana yang paling tepat?

- A. AWS DataSync — sync data dari on-premise ke S3
- B. AWS Storage Gateway — extend on-premise storage ke AWS
- C. **AWS Elastic Disaster Recovery (DRS)** — continuous replication dari on-premise (atau cloud lain) ke AWS, failover dengan RTO menit
- D. AWS Backup — backup on-premise servers ke AWS secara terjadwal

---

**Soal 5**

Sebuah perusahaan punya beberapa workload AWS dengan backup requirements berbeda-beda: RDS snapshots setiap hari, EBS snapshots setiap 6 jam, DynamoDB backup mingguan. Saat ini dikelola secara manual di setiap service. Mereka ingin **satu tempat terpusat** untuk manage semua backup policies.

Service mana yang tepat?

- A. AWS Config — buat rules untuk enforce backup policies
- B. Amazon S3 Lifecycle policies — simpan semua backup di S3
- C. **AWS Backup** — centralized backup service yang bisa manage backup untuk RDS, EBS, DynamoDB, EFS, FSx, dan lainnya dari satu console
- D. AWS CloudFormation — otomasi pembuatan snapshots via stack

---

**Soal 6**

Sebuah perusahaan menggunakan DynamoDB untuk aplikasi yang traffic-nya sangat **unpredictable** — kadang hampir nol, kadang sangat tinggi dalam waktu singkat. Tim tidak ingin estimasi capacity manual yang salah menyebabkan throttling atau over-provisioning.

DynamoDB capacity mode mana yang paling tepat?

- A. Provisioned capacity dengan Auto Scaling — capacity naik otomatis
- B. **On-Demand capacity mode** — DynamoDB otomatis handle semua traffic tanpa perlu set capacity, bayar per request yang actual terjadi
- C. Provisioned capacity dengan nilai tinggi — pastikan tidak pernah throttle
- D. Reserved capacity — bayar di muka untuk discount

---

**Soal 7**

Sebuah perusahaan memiliki **Windows-based applications** yang membutuhkan **shared file system** kompatibel SMB (Server Message Block) dan mendukung **Active Directory integration** untuk file-level permissions.

Storage service mana yang tepat?

- A. Amazon EFS — NFS protocol, tidak support SMB
- B. Amazon S3 — object storage, tidak punya file system semantics
- C. **Amazon FSx for Windows File Server** — fully managed Windows native file system, support SMB, NTFS, Active Directory, DFS
- D. Amazon EBS Multi-Attach — block storage, tidak support SMB

---

**Soal 8**

Sebuah perusahaan ingin menguji **resiliency** aplikasi mereka dengan sengaja **mematikan random EC2 instances** di production untuk memastikan aplikasi tetap berjalan saat terjadi kegagalan infrastruktur — sebuah praktik yang disebut chaos engineering.

Tool AWS mana yang bisa digunakan untuk ini?

- A. AWS Config — detect instances yang mati
- B. AWS CloudTrail — log semua instance terminations
- C. Amazon CloudWatch — set alarm saat instance down
- D. **AWS Fault Injection Service (FIS)** — managed chaos engineering service untuk inject faults (terminate instances, throttle API, inject latency) secara controlled

---

**Soal 9**

Sebuah perusahaan menjalankan aplikasi di **dua region** (us-east-1 dan eu-west-1). Mereka ingin Route 53 mengirim **10% traffic ke eu-west-1** untuk canary testing versi baru, dan **90% ke us-east-1**. Jika eu-west-1 unhealthy, semua traffic harus ke us-east-1.

Kombinasi Route 53 routing yang tepat adalah?

- A. Failover routing saja — eu-west-1 sebagai secondary
- B. Latency-based routing — eu-west-1 akan dapat traffic dari user Eropa saja
- C. **Weighted routing dikombinasikan dengan health checks** — set weight 90/10, health check pada setiap record, jika satu unhealthy traffic otomatis ke yang sehat
- D. Geolocation routing — user Eropa ke eu-west-1, lainnya ke us-east-1

---

**Soal 10**

Sebuah perusahaan memiliki arsitektur multi-region dengan VPC di `us-east-1` dan `ap-southeast-1`. Mereka ingin **EC2 di kedua region** bisa berkomunikasi satu sama lain dengan **routing yang simpel dan centralized**, tanpa setup VPC Peering point-to-point yang makin kompleks seiring bertambahnya VPCs.

Solusi yang paling tepat adalah?

- A. VPC Peering antara kedua VPC — simpel untuk dua VPC
- B. **AWS Transit Gateway** di setiap region, dihubungkan dengan **Transit Gateway Peering** antar region — semua VPC cukup attach ke TGW lokal mereka
- C. AWS Direct Connect antara kedua region
- D. VPN Site-to-Site antara kedua region

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
