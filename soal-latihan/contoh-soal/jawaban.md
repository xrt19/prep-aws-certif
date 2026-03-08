# Jawaban — Set 01

---

## Soal 1 — Jawaban: B

**Simpan session state di ElastiCache Redis**

**Kenapa B:**
Session state harus disimpan di tempat yang bisa diakses oleh semua instance — bukan di memory lokal. ElastiCache Redis adalah solusi standar untuk shared session store di arsitektur horizontally-scaled.

**Kenapa bukan yang lain:**
- **A (Sticky sessions)** — ini workaround, bukan solusi. Kalau instance yang "dipegang" user di-terminate oleh Auto Scaling, user tetap logout. Selain itu sticky sessions mengurangi efektivitas load balancing.
- **C (Least outstanding requests)** — ini cuma mengubah algoritma routing, tidak menyelesaikan masalah session.
- **D (Tambah instance)** — tidak relevan sama sekali dengan masalah session.

**Konsep yang diuji:** Stateless architecture, shared session store.

---

## Soal 2 — Jawaban: B

**S3 bucket policy yang deny PutObject jika header KMS key tidak sesuai**

**Kenapa B:**
Default encryption (opsi A) memang akan encrypt object, tapi tidak *menolak* upload yang tidak menggunakan CMK yang benar — object tetap bisa diupload dengan SSE-S3 atau CMK yang berbeda. Untuk *enforce* penggunaan CMK spesifik, harus pakai bucket policy dengan explicit Deny.

Contoh kondisi policy:
```json
"Condition": {
  "StringNotEquals": {
    "s3:x-amz-server-side-encryption-aws-kms-key-id": "arn:aws:kms:..."
  }
}
```

**Kenapa bukan yang lain:**
- **A (Default encryption)** — encrypt by default, tapi tidak enforce CMK spesifik.
- **C (IAM policy)** — restrict siapa yang bisa upload, bukan *bagaimana* cara upload.
- **D (Object Lock)** — untuk immutability (WORM), bukan enkripsi.

**Konsep yang diuji:** S3 bucket policy, enforce encryption, KMS CMK.

---

## Soal 3 — Jawaban: B

**DynamoDB dengan On-Demand capacity mode**

**Kenapa B:**
DynamoDB On-Demand otomatis scale sesuai traffic tanpa perlu provisioning. Bayar per request — saat idle hampir tidak ada cost, saat peak bisa handle ribuan request per detik tanpa konfigurasi tambahan.

**Kenapa bukan yang lain:**
- **A (RDS Multi-AZ)** — bayar per jam meskipun idle, tidak bisa auto-scale compute.
- **C (Aurora + Read Replicas)** — Read Replicas tetap jalan (dan bayar) meskipun idle. Aurora Serverless v2 lebih cocok jika tetap mau pakai relational DB.
- **D (ElastiCache)** — ini caching layer, bukan primary database.

**Konsep yang diuji:** DynamoDB capacity modes, cost optimization untuk variable workload.

---

## Soal 4 — Jawaban: B

**AWS Direct Connect**

**Kenapa B:**
Requirement-nya jelas: konsisten, low-latency, tidak lewat public internet. Hanya Direct Connect yang memenuhi ketiganya — ini koneksi fisik dedicated dari on-premise ke AWS.

**Kenapa bukan yang lain:**
- **A (Site-to-Site VPN)** — melewati public internet (encrypted, tapi tetap internet). Latency variable.
- **C (Direct Connect + VPN backup)** — ini arsitektur yang bagus untuk production, tapi soal tidak menyebut kebutuhan redundancy/backup. Jawaban paling tepat dan simpel adalah B.
- **D (VPC Peering)** — untuk koneksi antar VPC di AWS, bukan on-premise ke AWS.

**Konsep yang diuji:** Hybrid connectivity, Direct Connect vs VPN trade-offs.

---

## Soal 5 — Jawaban: C

**Konfigurasikan Dead Letter Queue (DLQ)**

**Kenapa C:**
DLQ adalah mekanisme standar untuk menangani "poison pill messages" — message yang berulang kali gagal diproses. Setelah melebihi `maxReceiveCount`, message otomatis dipindah ke DLQ sehingga tidak menghalangi message lain yang valid. Tim bisa investigasi message di DLQ secara terpisah.

**Kenapa bukan yang lain:**
- **A (Visibility timeout 24 jam)** — message tetap di queue, hanya "disembunyikan" lebih lama. Tidak menyelesaikan masalah, malah memperburuk karena message valid lain juga bisa terdampak.
- **B (Retry 10x di Lambda)** — kalau data memang tidak valid, retry sebanyak apapun tetap gagal. Ini membuang resource.
- **D (Ganti ke FIFO)** — FIFO menjamin ordering dan exactly-once delivery, tapi tidak menyelesaikan masalah message gagal berulang. Justru di FIFO, satu message gagal bisa **block seluruh queue**.

**Konsep yang diuji:** SQS DLQ, poison pill pattern, message processing failure handling.

---

## Rekap Skor

| Soal | Topik | Domain |
|------|-------|--------|
| 1 | Stateless architecture, session store | Resilient |
| 2 | S3 bucket policy, KMS enforce | Secure |
| 3 | DynamoDB capacity modes | Cost + Performance |
| 4 | Direct Connect vs VPN | High-Performing |
| 5 | SQS DLQ | Resilient |
