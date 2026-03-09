# Jawaban — Set 05
## Domain: Secure Architectures

---

## Soal 1 — Jawaban: B

**VPC Flow Logs**

**Kenapa B:**
VPC Flow Logs mencatat **metadata network traffic** di level VPC, subnet, atau ENI — termasuk source IP, destination IP, port, protocol, bytes transferred, dan **ACCEPT/REJECT status**. Data ini sangat berguna untuk analisis network security, troubleshooting connectivity, dan forensics. Tidak perlu akses ke dalam instance sama sekali.

**Kenapa bukan yang lain:**
- **A** — CloudTrail mencatat API calls ke AWS services (siapa yang panggil `RunInstances`, `DeleteBucket`, dll). Tidak mencatat network-level connections ke/dari EC2.
- **C** — Config mencatat perubahan konfigurasi resource, bukan network traffic.
- **D** — Inspector untuk vulnerability assessment (CVEs, network exposure), bukan logging koneksi aktual.

**Konsep yang diuji:** VPC Flow Logs, network traffic analysis, ACCEPT/REJECT logging.

---

## Soal 2 — Jawaban: B

**S3 Access Points — satu access point per departemen**

**Kenapa B:**
S3 Access Points memungkinkan pembuatan **named network endpoints dengan policy sendiri** untuk setiap use case atau tim — tanpa harus memodifikasi bucket policy yang semakin kompleks. Setiap access point punya ARN sendiri dan policy yang independent. Finance punya access point dengan policy restrict ke prefix `/finance/`, HR ke `/hr/`, dll. Ini jauh lebih manageable dari single bucket policy yang makin panjang.

**Kenapa bukan yang lain:**
- **A** — Bucket policy tunggal dengan banyak kondisi per-prefix bisa bekerja, tapi semakin kompleks dan sulit di-maintain seiring bertambahnya departemen. Bukan solusi paling tepat untuk multi-tenant access pattern.
- **C** — S3 ACL di level folder tidak ada — S3 ACL hanya untuk bucket dan object, dan sudah deprecated untuk sebagian besar use cases.
- **D** — IAM policy per departemen bisa bekerja tapi harus dimanage di sisi IAM, bukan di sisi S3. Tidak memberikan isolasi yang jelas antar departemen dalam konteks bucket.

**Konsep yang diuji:** S3 Access Points, multi-tenant S3 access, simplifikasi bucket policy.

---

## Soal 3 — Jawaban: B

**IAM Roles for Service Accounts (IRSA)**

**Kenapa B:**
IRSA adalah mekanisme yang memungkinkan pods Kubernetes di EKS mendapat **temporary AWS credentials yang di-scope per service account** — tanpa long-term credentials. Cara kerjanya: EKS menggunakan OIDC provider, pod dengan service account tertentu bisa `AssumeRoleWithWebIdentity` untuk dapat IAM role yang sesuai. Setiap pod hanya dapat permissions yang dibutuhkan (least privilege per pod).

**Kenapa bukan yang lain:**
- **A** — Kubernetes Secret untuk menyimpan IAM access key adalah long-term credentials yang bisa bocor jika secret tidak diproteksi dengan baik. Melanggar requirement.
- **C** — Attach IAM role ke node group memberikan permissions yang sama ke **semua pods** di node tersebut — melanggar least privilege. Jika satu pod compromise, semua pods lain di node ikut kena.
- **D** — Secrets Manager dengan init container lebih kompleks, dan tetap memerlukan cara pod bisa autentikasi ke Secrets Manager — membuat chicken-and-egg problem.

**Konsep yang diuji:** IRSA (IAM Roles for Service Accounts), EKS security, pod-level least privilege.

---

## Soal 4 — Jawaban: B

**Envelope encryption — KMS generate DEK, enkripsi data lokal dengan DEK**

**Kenapa B:**
KMS API hanya bisa mengenkripsi data hingga **4 KB** secara langsung. Untuk data lebih besar, digunakan **envelope encryption**:
1. KMS generate **Data Encryption Key (DEK)** — ada dua versi: plaintext DEK dan encrypted DEK
2. Enkripsi data besar menggunakan plaintext DEK **secara lokal** (tidak lewat network)
3. Buang plaintext DEK dari memory, simpan encrypted DEK bersama data
4. Untuk decrypt: minta KMS decrypt encrypted DEK → dapat plaintext DEK → decrypt data

Semua AWS SDK dan services seperti S3 SSE-KMS menggunakan envelope encryption secara otomatis.

**Kenapa bukan yang lain:**
- **A** — Split dan enkripsi per chunk sangat tidak efisien, tidak ada mekanisme natif untuk ini, dan tetap ada limit per call.
- **C** — KMS tidak bisa "enkripsi langsung dari S3". S3 yang memanggil KMS untuk DEK, bukan sebaliknya — ini pun envelope encryption di balik layar.
- **D** — CloudHSM untuk use case khusus (FIPS 140-2 Level 3, custom HSM). Pernyataan "KMS tidak bisa handle data besar" salah — justru envelope encryption adalah solusi standar KMS untuk ini.

**Konsep yang diuji:** Envelope encryption, KMS DEK, GenerateDataKey API, 4KB limit KMS.

---

## Soal 5 — Jawaban: B

**GuardDuty delegated administrator**

**Kenapa B:**
AWS Organizations mendukung **delegated administrator** untuk banyak services termasuk GuardDuty. Management account mendesignate satu member account (biasanya security account) sebagai GuardDuty administrator. Admin account ini bisa:
- Enable GuardDuty di semua member accounts otomatis
- Lihat dan manage semua findings dari satu tempat
- Configure detector settings secara terpusat

Ini adalah cara yang tepat untuk centralized security management di multi-account environment.

**Kenapa bukan yang lain:**
- **A** — Assume role per account bisa bekerja tapi tidak scalable untuk ratusan accounts dan tidak memberikan single-pane-of-glass view.
- **C** — Security Hub memang aggregate findings (termasuk dari GuardDuty), tapi soal menanyakan tentang **mengelola GuardDuty** itu sendiri (enable, configure, manage). Security Hub adalah layer di atasnya.
- **D** — Config aggregator untuk mengumpulkan configuration data dan compliance status, bukan untuk mengelola atau menampilkan GuardDuty findings.

**Konsep yang diuji:** GuardDuty delegated administrator, Organizations security management, multi-account GuardDuty.

---

## Soal 6 — Jawaban: B

**SecureString type untuk parameter sensitif — Standard tier, tidak ada biaya per parameter**

**Kenapa B:**
SSM Parameter Store memiliki dua tier (Standard dan Advanced) dan tiga tipe (String, StringList, SecureString). **SecureString** menggunakan KMS untuk enkripsi — bisa pakai AWS-managed key (gratis) atau CMK. Standard tier: **tidak ada biaya per parameter** (gratis hingga 10.000 parameter), hanya ada biaya API calls. Ini memenuhi kebutuhan: enkripsi untuk parameter sensitif + tidak ada biaya tambahan.

**Kenapa bukan yang lain:**
- **A** — Standard tier tidak punya opsi "SSE" yang terpisah. Enkripsi di Parameter Store dilakukan dengan tipe SecureString, bukan setting di tier.
- **C** — Advanced tier diperlukan untuk parameter >4KB, higher throughput, atau parameter policies (TTL/rotation). Tidak wajib hanya untuk enkripsi — overkill dan ada biaya $0.05/parameter/bulan.
- **D** — Parameter Store memang support enkripsi via SecureString. Secrets Manager berbeda: lebih mahal ($0.40/secret/bulan), tapi ada auto-rotation native.

**Konsep yang diuji:** SSM Parameter Store tiers, SecureString vs String, Standard vs Advanced tier, Parameter Store vs Secrets Manager.

---

## Soal 7 — Jawaban: B

**Satu CloudTrail trail dengan "Apply trail to all regions" diaktifkan**

**Kenapa B:**
CloudTrail mendukung **multi-region trail** — satu konfigurasi trail yang secara otomatis merekam events di semua regions yang ada maupun regions baru yang akan diluncurkan AWS di masa depan. Semua logs dikirim ke satu S3 bucket terpusat. Ini best practice yang direkomendasikan AWS untuk audit trail yang comprehensive.

**Kenapa bukan yang lain:**
- **A** — Enable per region satu per satu tidak scalable dan rawan terlewat. Jika AWS launch region baru, harus enable lagi secara manual.
- **C** — Config aggregator untuk compliance data, bukan pengganti CloudTrail untuk API activity logging. Keduanya punya fungsi berbeda.
- **D** — CloudWatch Logs untuk application/system logs dan metrics. Bukan solusi untuk centralized API audit trail.

**Konsep yang diuji:** CloudTrail multi-region trail, organization trail, centralized logging.

---

## Soal 8 — Jawaban: B

**AWS Artifact**

**Kenapa B:**
AWS Artifact adalah **self-service portal** untuk mengakses AWS compliance documentation:
- **AWS Artifact Reports**: laporan audit pihak ketiga (ISO 27001, SOC 1/2/3, PCI DSS, HIPAA, FedRAMP, dll)
- **AWS Artifact Agreements**: manage agreements seperti BAA (Business Associate Agreement) untuk HIPAA

Dokumen-dokumen ini adalah bukti resmi bahwa AWS infrastruktur memenuhi standar compliance tersebut. Bisa di-download langsung dari console.

**Kenapa bukan yang lain:**
- **A** — CloudTrail logs adalah bukti aktivitas **akun customer**, bukan compliance report AWS sebagai cloud provider.
- **C** — Config compliance report menunjukkan apakah **resource customer** comply dengan rules yang di-set, bukan laporan AWS compliance terhadap standar industri.
- **D** — Security Hub findings adalah security posture **akun customer**, bukan dokumen compliance AWS.

**Konsep yang diuji:** AWS Artifact, compliance reports, ISO 27001, SOC 2, shared responsibility model.

---

## Soal 9 — Jawaban: B

**S3 Block Public Access di level bucket**

**Kenapa B:**
S3 Block Public Access bisa diaktifkan di level **akun** (berlaku ke semua bucket) maupun di level **bucket individual**. Fitur ini **mengoverride** semua ACL dan bucket policies yang mencoba memberikan public access — bahkan jika ada explicit Allow `s3:PutBucketAcl` di IAM policy. Ini adalah preventive control yang paling kuat untuk mencegah accidental public exposure S3.

**Kenapa bukan yang lain:**
- **A** — SCP deny `s3:PutBucketAcl` mencegah perubahan ACL tapi tidak mencegah semua cara bucket bisa jadi publik (misalnya via bucket policy). S3 Block Public Access lebih comprehensive karena block semua jalur public access.
- **C** — Menghapus IAM policy tidak scalable dan akan mengganggu legitimate use cases yang mungkin butuh manage ACL untuk tujuan non-public.
- **D** — S3 Object Lock untuk immutable object (WORM), tidak ada hubungannya dengan mencegah perubahan ACL.

**Konsep yang diuji:** S3 Block Public Access level bucket vs akun, override behavior, preventive control.

---

## Soal 10 — Jawaban: C

**Aktifkan enkripsi default EFS + Config rule sebagai detective control**

**Kenapa C:**
Ini adalah kombinasi terbaik — **dua layer control**:
1. **Preventive**: Aktifkan enkripsi default di EFS per-region setting → semua EFS baru otomatis terenkripsi tanpa butuh konfigurasi per-EFS
2. **Detective**: Config rule `efs-encrypted-check` → catch jika ada yang bypass default setting (misalnya via API call yang spesifik override default)

Kombinasi preventive + detective adalah defense-in-depth yang direkomendasikan dengan operational overhead rendah.

**Kenapa bukan yang lain:**
- **A** — Lambda polling setiap 5 menit adalah custom solution dengan overhead maintenance tinggi dan ada delay deteksi.
- **B** — Config rule saja adalah detective only — EFS tanpa enkripsi sudah terlanjur dibuat sebelum terdeteksi. Kurang optimal.
- **D** — SCP dengan kondisi enkripsi untuk EFS memang bisa dibuat tapi sangat complex untuk di-implement dan maintain (butuh condition key yang spesifik). Default encryption setting jauh lebih simpel.

**Konsep yang diuji:** EFS encryption at rest, defense in depth, preventive + detective controls, EFS account-level default encryption.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | VPC Flow Logs, network traffic metadata, ACCEPT/REJECT |
| 2 | S3 Access Points, multi-tenant access pattern |
| 3 | IRSA (IAM Roles for Service Accounts), EKS security |
| 4 | Envelope encryption, KMS DEK, 4KB limit |
| 5 | GuardDuty delegated administrator, multi-account |
| 6 | SSM Parameter Store SecureString, Standard vs Advanced tier |
| 7 | CloudTrail multi-region trail, centralized logging |
| 8 | AWS Artifact, compliance reports ISO/SOC |
| 9 | S3 Block Public Access level bucket, override ACL/policy |
| 10 | EFS default encryption + Config rule, defense in depth |
