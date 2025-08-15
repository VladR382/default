# GCEME (Contoh Aplikasi Go di Google Cloud)

Repositori ini berisi kode sumber untuk **GCEME**, sebuah aplikasi web contoh yang ditulis dalam bahasa Go. Aplikasi ini dirancang untuk didemonstrasikan pada Google Cloud Platform (GCP) dan di-deploy menggunakan Kubernetes.

Aplikasi ini memiliki dua mode operasi:

1.  **Frontend**: Menampilkan antarmuka web yang mengambil data dari layanan backend.
2.  **Backend**: Menyediakan data dalam format JSON tentang instance Google Compute Engine (GCE) tempat aplikasi berjalan.

## Arsitektur

Aplikasi ini dibagi menjadi dua komponen utama:

  * **Frontend**: Sebuah layanan yang menampilkan halaman HTML dengan informasi yang diterima dari backend.
  * **Backend**: Sebuah layanan yang mengembalikan informasi detail tentang lingkungan server (seperti ID instance, zona, proyek, dll.) dalam format JSON.

Keduanya dirancang untuk berjalan sebagai *deployment* terpisah di dalam sebuah klaster Kubernetes.

## Fitur

  * **Aplikasi Berbasis Go**: Sederhana dan efisien.
  * **Mode Ganda**: Dapat dijalankan sebagai frontend atau backend.
  * **Integrasi GCE**: Secara otomatis mendeteksi dan menampilkan metadata dari instance GCE.
  * **Containerized**: Siap untuk di-build sebagai gambar Docker.
  * **Orkestrasi Kubernetes**: Dilengkapi dengan manifes Kubernetes untuk berbagai lingkungan (development, canary, production).
  * **CI/CD Pipeline**: Termasuk `Jenkinsfile` untuk otomatisasi proses testing, build, dan deployment.

## Prasyarat

Sebelum memulai, pastikan Anda telah menginstal perangkat berikut:

  * Go (versi 1.10 atau lebih baru)
  * Docker
  * kubectl
  * Google Cloud SDK
  * Jenkins (opsional, untuk menjalankan pipeline CI/CD)

## Memulai

### 1\. Menjalankan Secara Lokal

**Backend**

Untuk menjalankan server dalam mode backend pada port 8080:

```bash
go run . -port=8080
```

**Frontend**

Untuk menjalankan server dalam mode frontend, Anda perlu mengarahkan ke layanan backend:

```bash
go run . -frontend -port=8080 -backend-service=http://localhost:8081
```

### 2\. Membangun dengan Docker

Repositori ini sudah menyertakan `Dockerfile` untuk membangun aplikasi.

```bash
# Clone repositori
git clone <URL_REPOSITORI_ANDA>
cd <NAMA_DIREKTORI>

# Build Docker image
docker build -t gceme:1.0.0 .
```

### 3\. Deployment ke Kubernetes

File manifes Kubernetes tersedia di direktori `k8s/`. Terdapat konfigurasi untuk lingkungan `dev`, `canary`, dan `production`.

**Contoh Deployment ke Lingkungan Development:**

1.  Pastikan `kubectl` sudah terkonfigurasi untuk menunjuk ke klaster Anda.

2.  Buat *namespace* jika belum ada (misalnya, untuk cabang `dev-feature-1`):

    ```bash
    kubectl create ns dev-feature-1
    ```

3.  Terapkan manifes layanan dan deployment:

    ```bash
    # Terapkan layanan (backend & frontend)
    kubectl apply -f k8s/services/ -n dev-feature-1

    # Terapkan aplikasi backend dan frontend untuk dev
    kubectl apply -f k8s/dev/ -n dev-feature-1
    ```

## Pipeline CI/CD

Proyek ini menggunakan `Jenkinsfile` untuk mendefinisikan pipeline *Continuous Integration* dan *Continuous Deployment* (CI/CD).

Tahapan pipeline meliputi:

1.  **Test**: Menjalankan pengujian unit menggunakan `go test`.
2.  **Build and Push Image**: Membangun gambar Docker dan mendorongnya ke Google Container Registry (GCR) menggunakan Google Cloud Build.
3.  **Deploy**: Melakukan deployment aplikasi ke Kubernetes berdasarkan cabang Git:
      * Cabang `master` akan di-deploy ke lingkungan **production**.
      * Cabang `canary` akan di-deploy ke lingkungan **canary**.
      * Cabang lainnya akan di-deploy ke *namespace* **development** mereka sendiri.

## Lisensi

Proyek ini dilisensikan di bawah Lisensi Apache, Versi 2.0. Anda dapat menemukan detail lengkapnya di file `LICENSE`.
