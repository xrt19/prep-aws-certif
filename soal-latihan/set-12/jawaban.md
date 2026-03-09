# Jawaban — Set 12
## Domain: Resilient Architectures

---

## Soal 1 — Jawaban: C

**ElastiCache Redis dengan Multi-AZ dan automatic failover**

**Kenapa C:**
ElastiCache Redis mendukung **Multi-AZ dengan automatic failover**: primary node di satu AZ, satu atau lebih read replicas di AZ berbeda. Jika primary node gagal, ElastiCache otomatis promote replica dengan lag terkecil menjadi primary baru dalam hitungan menit — tanpa intervensi manual. DNS endpoint diupdate otomatis sehingga aplikasi reconnect ke primary baru.

**Kenapa bukan yang lain:**
- **A** — Memcached adalah in-memory cache yang **tidak support replication atau persistence** — data hilang saat node restart. Tidak cocok untuk session data yang butuh durability.
- **B** — Single node dengan snapshot harian memiliki RTO lama (restore dari snapshot) dan RPO hingga 1 hari. Tidak untuk HA.
- **D** — Read replicas tanpa Multi-AZ enabled tidak ada automatic failover. Jika primary down, harus promote replica secara manual.

**Konsep yang diuji:** ElastiCache Redis Multi-AZ, automatic failover, Redis vs Memcached, session caching HA.

---

## Soal 2 — Jawaban: C

**Reserved Concurrency**

**Kenapa C:**
Lambda concurrency di AWS dibagi dalam dua mekanisme:
- **Reserved Concurrency**: **garansi** sejumlah concurrency untuk fungsi spesifik — tidak bisa diambil fungsi lain. Juga berfungsi sebagai **throttle ceiling** (fungsi tidak akan melebihi nilai ini).
- **Provisioned Concurrency**: pre-warm Lambda agar tidak ada cold start.

Untuk use case ini (pastikan fungsi kritis selalu punya capacity dari total akun), Reserved Concurrency adalah jawabannya — mengalokasikan pool eksklusif.

**Kenapa bukan yang lain:**
- **A** — Memory lebih banyak **tidak** menambah concurrency. Memory mempengaruhi CPU dan execution speed, bukan jumlah concurrent invocations.
- **B** — Pindah region tidak menyelesaikan masalah arsitektur — region baru juga punya account-level concurrency limit yang sama.
- **D** — Provisioned Concurrency untuk **menghilangkan cold start** (pre-warm), bukan untuk reserve capacity dari fungsi lain. Fungsi dengan Provisioned Concurrency masih bisa di-throttle jika total account concurrency habis.

**Konsep yang diuji:** Lambda Reserved Concurrency vs Provisioned Concurrency, account-level concurrency limit, throttling protection.

---

## Soal 3 — Jawaban: C

**Amazon FSx for Lustre**

**Kenapa C:**
Lustre adalah **parallel distributed file system** yang dirancang untuk high-performance workloads:
- Throughput hingga **ratusan GB/s** dan jutaan IOPS
- **Sub-millisecond latency**
- Bisa di-link langsung ke S3 bucket — data di S3 di-expose sebagai file system, hasil komputasi bisa di-write balik ke S3
- Ideal untuk HPC, machine learning, financial modeling, video processing

**Kenapa bukan yang lain:**
- **A** — EFS adalah general-purpose NFS file system. Throughput dan latency-nya jauh di bawah Lustre — tidak cocok untuk HPC yang butuh ratusan GB/s.
- **B** — S3 object storage, tidak punya POSIX file system semantics yang dibutuhkan HPC applications, dan latency lebih tinggi.
- **D** — EBS Multi-Attach hanya bisa di-attach ke **maksimum 16 instances** dalam satu AZ, dan hanya untuk io1/io2 volumes. Tidak scalable untuk HPC cluster besar.

**Konsep yang diuji:** FSx for Lustre, HPC storage, parallel file system, S3 integration.

---

## Soal 4 — Jawaban: C

**AWS Elastic Disaster Recovery (DRS)**

**Kenapa C:**
AWS DRS (sebelumnya CloudEndure Disaster Recovery) dirancang persis untuk use case ini:
- **Continuous replication** dari source (on-premise, AWS region lain, atau cloud lain) ke AWS menggunakan lightweight agent
- Data di-replicate ke **staging area** di AWS (biaya rendah)
- Saat disaster: **launch recovery instances** dari staging area dalam hitungan menit (RTO menit, RPO detik)
- Support berbagai OS (Windows, Linux)

**Kenapa bukan yang lain:**
- **A** — DataSync untuk **data migration** atau scheduled sync file/object — tidak continuous block-level replication, tidak bisa failover server.
- **B** — Storage Gateway untuk **extend storage** on-premise ke AWS (NFS/SMB/iSCSI ke S3) — bukan untuk DR server failover.
- **D** — AWS Backup untuk scheduled backups (snapshots) — RTO lebih lama karena harus restore dari backup, bukan continuous replication.

**Konsep yang diuji:** AWS Elastic Disaster Recovery, continuous replication, on-premise to AWS DR, RTO minutes.

---

## Soal 5 — Jawaban: C

**AWS Backup**

**Kenapa C:**
AWS Backup adalah **centralized managed backup service** yang mendukung:
- **RDS** (automated snapshots dan manual)
- **EBS** (snapshots)
- **DynamoDB** (on-demand dan continuous backup)
- **EFS, FSx, S3, EC2, Storage Gateway**
- Semua dikelola dari satu console dengan **backup plans** (schedule, retention, lifecycle ke cold storage)
- **Cross-region dan cross-account backup** support
- Audit dan compliance reporting

**Kenapa bukan yang lain:**
- **A** — Config untuk compliance monitoring konfigurasi resource, tidak bisa execute backup actions.
- **B** — S3 Lifecycle policies untuk manage objects di S3 bucket, tidak bisa trigger backup dari RDS atau EBS.
- **D** — CloudFormation untuk infrastructure provisioning, bukan backup management. Bisa dibuat custom tapi sangat complex dan bukan use case-nya.

**Konsep yang diuji:** AWS Backup, centralized backup management, multi-service backup policy.

---

## Soal 6 — Jawaban: B

**DynamoDB On-Demand capacity mode**

**Kenapa B:**
DynamoDB On-Demand adalah billing mode dimana DynamoDB **otomatis handle semua request** tanpa perlu set atau manage read/write capacity units. Karakteristik:
- **Instantly accommodates** traffic — tidak ada throttling saat spike
- Bayar per **request yang actual terjadi** (per million read/write request units)
- Tidak ada capacity planning
- Ideal untuk unpredictable, spiky, atau baru workloads

**Kenapa bukan yang lain:**
- **A** — Provisioned + Auto Scaling memang bisa scale, tapi ada delay (target tracking reactive, bukan instant) dan ada periode throttling sebelum scaling selesai. Tidak instant seperti On-Demand.
- **C** — Over-provisioning sangat mahal — bayar capacity yang tidak digunakan saat traffic rendah.
- **D** — Reserved capacity adalah commitment 1 atau 3 tahun untuk **provisioned** capacity — tidak sesuai untuk workload unpredictable.

**Konsep yang diuji:** DynamoDB On-Demand vs Provisioned, capacity planning, spiky workloads, throttling prevention.

---

## Soal 7 — Jawaban: C

**Amazon FSx for Windows File Server**

**Kenapa C:**
FSx for Windows File Server adalah **fully managed Windows native file system** dengan:
- **SMB protocol** (Windows native file sharing)
- **NTFS** file system dengan ACLs
- **Active Directory integration** — user/group permissions via AD
- **DFS (Distributed File System)** namespaces support
- Multi-AZ deployment option untuk HA
- Ideal untuk lift-and-shift Windows workloads ke AWS

**Kenapa bukan yang lain:**
- **A** — EFS menggunakan **NFS protocol** (Linux/Unix), tidak support SMB. Windows bisa mount NFS tapi butuh extra configuration dan tidak native AD integration.
- **B** — S3 adalah object storage — tidak ada file system semantics, tidak support SMB atau AD file permissions.
- **D** — EBS adalah block storage per instance, tidak bisa di-share sebagai file system antar instances via SMB.

**Konsep yang diuji:** FSx for Windows File Server, SMB, NTFS, Active Directory, Windows shared storage.

---

## Soal 8 — Jawaban: D

**AWS Fault Injection Service (FIS)**

**Kenapa D:**
AWS FIS adalah **managed chaos engineering service** yang memungkinkan inject berbagai jenis faults secara controlled ke infrastruktur AWS:
- **Terminate EC2 instances** secara random
- Inject CPU stress, memory pressure
- Throttle API calls
- Inject network latency/packet loss
- Failover RDS, ElastiCache
- Semua dengan **experiment templates** yang bisa di-control, di-stop kapanpun, dan di-monitor via CloudWatch

**Kenapa bukan yang lain:**
- **A** — Config untuk compliance monitoring, tidak bisa inject faults.
- **B** — CloudTrail untuk audit logging, tidak bisa terminate instances.
- **C** — CloudWatch untuk monitoring dan alerting — bisa detect instance down tapi tidak bisa membuatnya down secara sengaja.

**Konsep yang diuji:** AWS Fault Injection Service, chaos engineering, resiliency testing, fault injection.

---

## Soal 9 — Jawaban: C

**Weighted routing + health checks**

**Kenapa C:**
Route 53 Weighted routing dengan health checks:
- Set **weight 90** untuk us-east-1 record, **weight 10** untuk eu-west-1 record
- Attach **health check** ke setiap record
- Jika eu-west-1 unhealthy: Route 53 otomatis **remove record tersebut** dari rotation dan semua traffic (100%) ke us-east-1
- Jika keduanya healthy: traffic di-split 90/10 sesuai weight

Ini adalah pattern yang sangat common untuk canary + automatic failover.

**Kenapa bukan yang lain:**
- **A** — Failover routing hanya primary/secondary — tidak bisa split traffic 90/10. Hanya support active-passive.
- **B** — Latency-based routing route berdasarkan latency aktual — user di Eropa ke eu-west-1, user di AS ke us-east-1. Tidak bisa kontrol persentase 90/10 secara explicit.
- **D** — Geolocation routing berdasarkan negara/benua — tidak bisa split 90/10 dari user yang sama.

**Konsep yang diuji:** Route 53 weighted routing, health checks, canary traffic shifting, automatic failover via health check.

---

## Soal 10 — Jawaban: B

**Transit Gateway + Transit Gateway Peering antar region**

**Kenapa B:**
Transit Gateway adalah **regional hub** untuk VPC connectivity. Untuk multi-region:
- Setiap region punya TGW sendiri
- **TGW Peering** menghubungkan TGW antar region
- Setiap VPC hanya perlu **satu attachment ke TGW lokal** mereka
- Routing terpusat via TGW route tables
- Skala mudah: tambah VPC baru cukup attach ke TGW, tidak perlu buat peering baru ke setiap VPC lain

Dibandingkan VPC Peering: untuk N VPCs butuh N*(N-1)/2 peering connections — tidak scalable.

**Kenapa bukan yang lain:**
- **A** — VPC Peering untuk dua VPC memang simpel, tapi soal menyebut "seiring bertambahnya VPCs" — scalability adalah concern utama. TGW lebih tepat.
- **C** — Direct Connect untuk koneksi dedicated dari on-premise ke AWS, bukan untuk VPC-to-VPC komunikasi antar region.
- **D** — VPN Site-to-Site untuk koneksi terenkripsi dari on-premise ke AWS via internet, bukan untuk inter-region VPC connectivity managed. Bisa digunakan tapi tidak se-scalable dan se-performant TGW Peering.

**Konsep yang diuji:** Transit Gateway, TGW Peering inter-region, hub-and-spoke topology, scalable VPC connectivity.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | ElastiCache Redis Multi-AZ automatic failover |
| 2 | Lambda Reserved Concurrency vs Provisioned Concurrency |
| 3 | FSx for Lustre, HPC parallel file system |
| 4 | AWS Elastic Disaster Recovery, continuous replication |
| 5 | AWS Backup, centralized multi-service backup |
| 6 | DynamoDB On-Demand vs Provisioned, spiky workloads |
| 7 | FSx for Windows File Server, SMB, AD integration |
| 8 | AWS Fault Injection Service, chaos engineering |
| 9 | Route 53 weighted routing + health checks, canary |
| 10 | Transit Gateway Peering inter-region, hub-and-spoke |
