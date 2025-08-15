
Selamat datang di branch `canary` dari repositori **GCEME**. Branch ini memiliki peran khusus dalam alur kerja CI/CD kami.

## Tujuan Branch Canary

Branch `canary` digunakan untuk **Canary Release**. Ini adalah strategi di mana versi baru aplikasi dirilis ke sebagian kecil pengguna di lingkungan produksi sebelum diluncurkan sepenuhnya. Tujuannya adalah untuk menguji fitur baru atau perubahan di dunia nyata dengan risiko minimal.

Jika versi canary berjalan stabil dan tidak menunjukkan masalah, perubahan tersebut dapat digabungkan ke branch `master` untuk dirilis ke semua pengguna.

## Alur Kerja Otomatis (CI/CD)

Ketika ada *commit* baru yang di-*push* ke branch `canary`, pipeline CI/CD di Jenkins akan otomatis berjalan dan melakukan langkah-langkah berikut:

1.  **Test**: Menjalankan pengujian unit pada kode untuk memastikan tidak ada regresi.
2.  **Build & Push Image**: Membangun aplikasi menjadi sebuah Docker image baru dengan tag unik (berdasarkan nama branch dan nomor build) dan mengunggahnya ke Google Container Registry (GCR).
3.  **Deploy Canary**:
    * Mengambil file manifes Kubernetes dari direktori `k8s/canary/`.
    * Memperbarui manifes deployment (`frontend-canary.yaml` dan `backend-canary.yaml`) untuk menggunakan image Docker baru yang baru saja di-build.
    * Melakukan deployment ke klaster Kubernetes di dalam *namespace* `production`.

## Bagaimana Cara Kerjanya?

* Deployment `canary` akan berjalan **berdampingan** dengan deployment `production` yang sudah ada.
* Layanan `frontend` (`k8s/services/frontend.yaml`) dikonfigurasi untuk menyeleksi pod dengan label `app: gceme` dan `role: frontend`. Karena pod produksi dan pod canary memiliki label ini, *load balancer* akan secara otomatis mendistribusikan sebagian kecil lalu lintas pengguna ke pod `canary` yang baru.
* Ini memungkinkan tim untuk memantau kinerja dan stabilitas versi baru pada lalu lintas produksi secara langsung, tanpa memengaruhi mayoritas pengguna.

## Cara Menggunakan Branch Ini

1.  **Kembangkan Fitur**: Buat branch baru dari `master` untuk mengembangkan fitur atau perbaikan.
2.  **Gabung ke Canary**: Setelah selesai, gabungkan (*merge*) branch fitur Anda ke branch `canary`.
3.  **Push ke Remote**: Lakukan `git push origin canary`.
4.  **Pantau Pipeline**: Perhatikan pipeline Jenkins untuk memastikan semua tahapan (test, build, deploy) berhasil.
5.  **Verifikasi Deployment**: Setelah deployment berhasil, verifikasi bahwa versi canary berjalan dengan baik di lingkungan produksi. Pantau log, metrik, dan laporan kesalahan.
6.  **Promosikan ke Produksi**: Jika versi canary terbukti stabil selama periode pemantauan, gabungkan branch `canary` ke branch `master` untuk merilis perubahan ke semua pengguna.

Dengan menggunakan alur kerja ini, kita dapat merilis pembaruan dengan lebih percaya diri dan mengurangi risiko dampak negatif pada pengguna.
