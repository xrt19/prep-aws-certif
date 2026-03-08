# Soal Latihan — Set 01

> 5 soal mixed domain. Jawab dulu semuanya sebelum buka `jawaban.md`.

---

**Soal 1**

Sebuah perusahaan menjalankan aplikasi web di EC2 yang di-deploy di belakang Application Load Balancer. Aplikasi ini menyimpan session state di memory lokal masing-masing instance. Ketika Auto Scaling menambah instance baru, user sering tiba-tiba ter-logout.

Solusi mana yang paling tepat untuk mengatasi masalah ini?

- A. Aktifkan sticky sessions di ALB
- B. Simpan session state di ElastiCache Redis
- C. Gunakan ALB dengan least outstanding requests algorithm
- D. Tambahkan lebih banyak instance di Auto Scaling Group

---

**Soal 2**

Tim security meminta agar semua object yang diupload ke S3 bucket tertentu harus dienkripsi menggunakan KMS key milik perusahaan, bukan AWS-managed key. Jika ada upload tanpa enkripsi yang sesuai, harus ditolak.

Cara yang paling tepat untuk mengimplementasikan ini adalah?

- A. Aktifkan S3 default encryption dengan SSE-KMS menggunakan CMK
- B. Tambahkan S3 bucket policy yang deny `s3:PutObject` jika header `x-amz-server-side-encryption-aws-kms-key-id` tidak sesuai
- C. Gunakan IAM policy untuk restrict upload hanya dari aplikasi tertentu
- D. Aktifkan S3 Object Lock pada bucket

---

**Soal 3**

Sebuah startup memiliki aplikasi yang traffic-nya sangat tidak menentu — bisa sangat ramai di siang hari, hampir tidak ada di malam hari. Mereka butuh database yang bisa handle ribuan request per detik saat peak, tapi ingin meminimalkan cost saat idle.

Database mana yang paling cocok?

- A. RDS MySQL dengan Multi-AZ
- B. DynamoDB dengan On-Demand capacity mode
- C. Aurora MySQL dengan Read Replicas
- D. ElastiCache Redis

---

**Soal 4**

Sebuah perusahaan memiliki on-premise data center dan ingin extend network mereka ke AWS. Mereka butuh koneksi yang **konsisten, low-latency, dan tidak melewati public internet** untuk transfer data sensitif secara rutin.

Solusi yang paling tepat adalah?

- A. Site-to-Site VPN dengan BGP routing
- B. AWS Direct Connect
- C. AWS Direct Connect dengan VPN sebagai backup
- D. VPC Peering

---

**Soal 5**

Sebuah aplikasi e-commerce menggunakan SQS untuk memproses order. Consumer (Lambda) kadang gagal memproses message karena data order tidak valid. Message yang gagal terus muncul kembali dan mengganggu pemrosesan order yang valid.

Solusi paling tepat untuk menangani message yang berulang kali gagal adalah?

- A. Naikkan visibility timeout SQS menjadi 24 jam
- B. Tambahkan retry logic di Lambda hingga 10x percobaan
- C. Konfigurasikan Dead Letter Queue (DLQ) pada SQS queue
- D. Ganti SQS Standard dengan SQS FIFO

---

> Sudah jawab semua? Buka `jawaban.md` untuk cek.
