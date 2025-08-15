Selamat datang di branch `new-feature`. Branch ini adalah lingkungan pengembangan terisolasi untuk membangun dan menguji fitur baru sebelum siap untuk diuji lebih lanjut di lingkungan *canary* atau digabungkan ke `master`.

## Tujuan Branch Ini

Tujuan utama dari branch fitur adalah:

  * **Pengembangan Terisolasi**: Memungkinkan developer untuk mengerjakan fitur baru tanpa mengganggu kode di branch `master` atau `canary`.
  * **Deployment Otomatis**: Setiap *commit* yang di-*push* ke branch ini akan secara otomatis di-*deploy* ke *namespace* Kubernetes-nya sendiri. Ini menciptakan lingkungan pengujian yang mandiri dan bersih untuk setiap fitur.
  * **Validasi Awal**: Memberikan kesempatan untuk meninjau dan menguji fungsionalitas fitur di lingkungan yang mirip produksi sebelum dipromosikan ke tahap selanjutnya.

## Alur Kerja Otomatis (CI/CD)

Berdasarkan `Jenkinsfile` dalam repositori ini, setiap kali Anda melakukan `git push` ke branch `new-feature`, pipeline CI/CD akan melakukan langkah-langkah berikut:

1.  **Test**: Menjalankan pengujian unit untuk memastikan kode baru tidak merusak fungsionalitas yang ada.
2.  **Build & Push Image**: Membangun aplikasi menjadi Docker image dengan tag unik (misalnya, `gcr.io/qwiklabs-gcp-01-dbdf2a62c37b/gceme:new-feature.BUILD_NUMBER`) dan mengunggahnya ke Google Container Registry (GCR).
3.  **Deploy ke Namespace Development**:
      * Pipeline akan membuat *namespace* baru di Kubernetes yang namanya sama dengan branch ini (yaitu, `new-feature`) jika belum ada.
      * Manifes Kubernetes dari direktori `k8s/dev/` dan `k8s/services/` akan digunakan untuk deployment.
      * Gambar Docker di dalam manifes deployment akan diperbarui untuk menggunakan gambar yang baru saja dibuat.
      * Layanan frontend akan di-deploy sebagai tipe `ClusterIP`, artinya tidak akan terekspos ke internet publik, melainkan hanya bisa diakses dari dalam klaster.

## Cara Mengakses Aplikasi Anda

Karena layanan di-*deploy* sebagai `ClusterIP`, Anda tidak bisa mengaksesnya melalui alamat IP eksternal. Untuk mengakses dan menguji fitur Anda, ikuti langkah-langkah berikut:

1.  Pastikan `kubectl` Anda sudah terkonfigurasi untuk klaster yang benar.
2.  Jalankan perintah `kubectl proxy` di terminal Anda:
    ```bash
    kubectl proxy
    ```
3.  Buka browser dan akses URL berikut:
    ```
    http://localhost:8001/api/v1/proxy/namespaces/new-feature/services/gceme-frontend:80/
    ```

URL ini akan meneruskan permintaan Anda melalui proxy `kubectl` langsung ke layanan frontend yang berjalan di *namespace* `new-feature`.

## Langkah Selanjutnya

Setelah Anda puas dengan fitur yang dikembangkan di branch ini:

1.  Pastikan semua pengujian berhasil.
2.  Gabungkan (*merge*) branch `new-feature` ini ke branch `canary` untuk pengujian lebih lanjut dengan sebagian kecil lalu lintas produksi.
