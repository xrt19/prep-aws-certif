# Jawaban — Set 02
## Domain: Secure Architectures

---

## Soal 1 — Jawaban: B

**Buat Customer Managed Key (CMK) di AWS KMS, gunakan untuk EBS encryption, aktifkan automatic key rotation**

**Kenapa B:**
CMK (Customer Managed Key) di KMS adalah cara yang tepat untuk kontrol penuh atas kunci enkripsi. Automatic key rotation di KMS akan rotate key material setiap tahun (atau sesuai konfigurasi), tanpa perlu re-encrypt data yang sudah ada — KMS menyimpan semua versi key secara internal untuk decrypt data lama.

**Kenapa bukan yang lain:**
- **A** — AWS-managed key (aws/ebs) tidak bisa dirotate secara manual atau dikontrol oleh customer. Tidak memenuhi requirement "kunci milik sendiri".
- **C** — Enkripsi di application layer lebih kompleks, tidak seamless, dan biasanya tidak direkomendasikan untuk block storage.
- **D** — CloudHSM untuk use case khusus yang butuh dedicated HSM hardware dan compliance tertentu (FIPS 140-2 Level 3). Overkill untuk kebutuhan ini, dan jauh lebih mahal.

**Konsep yang diuji:** KMS CMK vs AWS-managed key, automatic key rotation, EBS encryption.

---

## Soal 2 — Jawaban: B

**SCP yang deny `ec2:RunInstances` jika tag `Environment` tidak ada**

**Kenapa B:**
SCP dengan tag condition adalah mekanisme **preventif** yang benar untuk enforce tagging policy di level Organizations. Dengan deny jika `aws:RequestTag/Environment` tidak ada, tidak ada account manapun yang bisa membuat instance tanpa tag tersebut.

**Kenapa bukan yang lain:**
- **A** — `aws:RequestedRegion` membatasi region, bukan tag. Tidak relevan dengan kebutuhan.
- **C** — Config rule bersifat **detective** (detect setelah terjadi), bukan preventif. Instance sudah terlanjur dibuat sebelum di-remediate.
- **D** — Lambda yang scan setiap jam punya window waktu di mana instance tanpa tag sudah berjalan, dan menambah operational overhead signifikan.

**Konsep yang diuji:** SCP preventive controls, tag enforcement, `aws:RequestTag` condition key.

---

## Soal 3 — Jawaban: B

**Cognito Identity Pool dengan IAM role menggunakan `${cognito-identity.amazonaws.com:sub}` di S3 bucket policy**

**Kenapa B:**
Cognito Identity Pool memberikan temporary AWS credentials ke authenticated users. Dengan menggunakan variable `${cognito-identity.amazonaws.com:sub}` (yang berisi unique Cognito user ID) di S3 bucket policy, setiap user hanya bisa akses prefix S3 yang sesuai identity mereka — misalnya `s3://bucket/user-photos/${cognito-identity.amazonaws.com:sub}/*`.

**Kenapa bukan yang lain:**
- **A** — Membuat IAM user per user aplikasi tidak scalable untuk consumer apps dengan ribuan/jutaan users.
- **C** — Lambda authorizer di API Gateway butuh API layer di depan S3, menambah complexity dan latency. Tidak langsung akses S3.
- **D** — S3 ACL per object sulit di-manage dan tidak bisa dinamis berdasarkan user identity.

**Konsep yang diuji:** Cognito Identity Pool, identity-based access control, S3 bucket policy variable.

---

## Soal 4 — Jawaban: B

**AWS Database Activity Streams pada RDS**

**Kenapa B:**
Database Activity Streams (DAS) mencatat semua aktivitas database dalam near real-time ke Amazon Kinesis stream — termasuk query content, user yang menjalankan, timestamp, dll. Ini memungkinkan security tools untuk detect pattern mencurigakan seperti SQL injection tanpa mengubah aplikasi atau menambah overhead ke database.

**Kenapa bukan yang lain:**
- **A** — Enhanced Monitoring untuk OS-level metrics (CPU, memory, disk I/O di RDS host), bukan query-level activity.
- **C** — Performance Insights untuk analisis performance query (bottleneck, slow queries), bukan security monitoring.
- **D** — WAF tidak bisa di-deploy di depan RDS secara langsung. WAF untuk web applications (HTTP/HTTPS), bukan database connections.

**Konsep yang diuji:** Database Activity Streams, database audit, RDS security monitoring.

---

## Soal 5 — Jawaban: C

**Amazon GuardDuty**

**Kenapa C:**
GuardDuty adalah managed threat detection service yang menganalisis VPC Flow Logs, CloudTrail logs, dan DNS logs untuk mendeteksi ancaman. Port scanning antar instances dan komunikasi ke malicious IPs adalah persis jenis ancaman yang GuardDuty dirancang untuk detect dan generate findings.

**Kenapa bukan yang lain:**
- **A** — Inspector untuk vulnerability assessment — mencari known CVEs, network exposure, dan software vulnerabilities pada EC2/containers. Tidak real-time threat detection.
- **B** — Config untuk configuration change tracking, bukan threat detection.
- **D** — Security Hub adalah **aggregator** findings dari berbagai services (termasuk GuardDuty), bukan yang generate alert-nya sendiri. GuardDuty yang produce, Security Hub yang aggregate.

**Konsep yang diuji:** GuardDuty threat detection, GuardDuty findings, VPC Flow Logs analysis.

---

## Soal 6 — Jawaban: B

**IAM policy dengan `Condition: aws:SourceIp` pada semua IAM users**

**Kenapa B:**
`aws:SourceIp` adalah condition key yang membatasi akses berdasarkan IP address sumber. Dengan attach IAM policy yang deny semua actions jika IP bukan dari range kantor, console access dari luar akan ditolak. Ini bekerja untuk IAM-based authentication (console login).

**Kenapa bukan yang lain:**
- **A** — Security Groups mengontrol traffic ke/dari EC2 instances dan resources di VPC, bukan akses ke AWS Management Console (yang merupakan HTTPS request ke API endpoints AWS).
- **C** — SCP memang valid untuk restrict di level Organizations, tapi `aws:SourceIp` di SCP lebih tepat untuk API calls dari aplikasi, bukan console login dari browser yang bisa pakai IP dinamis. Namun untuk IAM users di single account, IAM policy lebih langsung.
- **D** — Network ACL mengontrol traffic di VPC level, tidak ada hubungannya dengan akses ke AWS Console.

**Konsep yang diuji:** IAM condition keys, `aws:SourceIp`, console access restriction.

---

## Soal 7 — Jawaban: B

**Interface VPC Endpoints untuk SSM, KMS, dan services yang dibutuhkan**

**Kenapa B:**
Interface VPC Endpoints (powered by AWS PrivateLink) membuat private endpoint di VPC untuk AWS services. EC2 di private subnet bisa akses SSM, KMS, dll via endpoint tersebut tanpa NAT Gateway — traffic tetap di AWS network. Ini lebih aman (tidak expose ke internet) dan menghilangkan NAT Gateway cost.

**Kenapa bukan yang lain:**
- **A** — Memindahkan ke public subnet bertentangan dengan prinsip security (least privilege network access) dan requirement soal.
- **C** — Transit Gateway untuk koneksi antar VPCs atau ke on-premise, bukan untuk akses ke AWS service endpoints.
- **D** — NAT Instance memang lebih murah dari NAT Gateway, tapi masih mengirim traffic melalui internet (public). Tidak menyelesaikan requirement "tanpa internet".

**Konsep yang diuji:** Interface VPC Endpoints, AWS PrivateLink, private subnet connectivity.

---

## Soal 8 — Jawaban: B

**AWS Security Hub**

**Kenapa B:**
Security Hub adalah managed service yang khusus dirancang untuk **aggregasi, normalisasi, dan prioritisasi security findings** dari berbagai AWS services dan third-party tools. Ia secara native integrate dengan GuardDuty, Inspector, IAM Access Analyzer, Macie, dan lainnya, serta menyediakan automated compliance checks (CIS AWS Foundations, PCI DSS, dll).

**Kenapa bukan yang lain:**
- **A** — CloudWatch untuk monitoring metrics dan logs, bisa diintegrasikan tapi bukan security aggregation platform.
- **C** — Config untuk configuration change tracking dan compliance rules pada resource configurations, bukan security findings aggregation.
- **D** — Detective untuk **investigate** security incidents (threat hunting, root cause analysis) setelah findings ditemukan, bukan untuk dashboard visibility awal.

**Konsep yang diuji:** Security Hub, security findings aggregation, AWS security services ecosystem.

---

## Soal 9 — Jawaban: B

**Policy dengan `secretsmanager:GetSecretValue` hanya untuk ARN secret yang spesifik**

**Kenapa B:**
Principle of Least Privilege mengharuskan Lambda hanya bisa akses resource yang benar-benar dibutuhkan. Dengan specify ARN spesifik di policy resource, Lambda tidak bisa mengakses secrets lain meskipun ada. Ini adalah praktik security yang benar.

```json
{
  "Effect": "Allow",
  "Action": "secretsmanager:GetSecretValue",
  "Resource": "arn:aws:secretsmanager:region:account:secret:specific-secret-name-*"
}
```

**Kenapa bukan yang lain:**
- **A** — `SecretsManagerReadWrite` adalah managed policy yang memberikan akses ke **semua** secrets dan bahkan write access. Melanggar least privilege.
- **C** — Environment variable Lambda yang di-enkripsi KMS bisa dianggap acceptable, tapi tidak sesuai requirement "stored in Secrets Manager". Juga tidak memenuhi kebutuhan rotation.
- **D** — Membuat IAM user untuk Lambda dan simpan access key-nya adalah anti-pattern. Lambda harus menggunakan IAM Role, bukan IAM user credentials.

**Konsep yang diuji:** Least privilege, IAM policy resource restriction, Lambda execution role best practices.

---

## Soal 10 — Jawaban: C

**IAM Access Analyzer**

**Kenapa C:**
IAM Access Analyzer dirancang khusus untuk dua hal: (1) **analyze resource policies** untuk identify external access yang tidak diinginkan (S3 bucket, KMS key, dll), dan (2) **generate IAM policies** berdasarkan actual CloudTrail activity (unused permissions). Fitur "unused access analysis" secara otomatis identify roles/users yang memiliki permissions tapi tidak pernah menggunakannya.

**Kenapa bukan yang lain:**
- **A** — CloudTrail merekam activity tapi tidak secara otomatis analyze atau report unused permissions. Butuh tool tambahan untuk analyze log-nya.
- **B** — Config untuk resource configuration compliance, tidak untuk permission analysis.
- **D** — GuardDuty untuk threat detection (anomaly dan malicious behavior), bukan permission review atau access analysis.

**Konsep yang diuji:** IAM Access Analyzer, unused access analysis, least privilege enforcement.

---

## Rekap Skor

| Soal | Topik |
|------|-------|
| 1 | KMS CMK, EBS encryption, automatic key rotation |
| 2 | SCP tag enforcement, preventive controls |
| 3 | Cognito Identity Pool, identity-based S3 access |
| 4 | Database Activity Streams, RDS security monitoring |
| 5 | GuardDuty threat detection |
| 6 | IAM condition keys, `aws:SourceIp` |
| 7 | Interface VPC Endpoints, AWS PrivateLink |
| 8 | Security Hub, findings aggregation |
| 9 | Least privilege, Lambda execution role |
| 10 | IAM Access Analyzer, unused access |
