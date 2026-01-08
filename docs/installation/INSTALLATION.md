# Instalasi Cepat untuk Non-IT — Yayasan Management System (YMS)

Panduan langkah demi langkah agar dapat menjalankan proyek ini di Windows (lokal) — ditulis sederhana untuk non-IT. Ikuti urutan langkah, dan gunakan bagian "Troubleshooting" bila menemui masalah.

Prasyarat dasar

-   Komputer Windows (Admin user disarankan)
-   Koneksi Internet
-   Minimal 8GB disk tersedia

Ringkasan singkat proses

1. Install XAMPP (Apache + MySQL)
2. Install Composer (PHP package manager)
3. Install Node.js & pnpm (frontend build)
4. Clone repository dan salin .env
5. Buat database MySQL (atau import SQL dump)
6. Install dependensi (composer & npm)
7. Jalankan migrasi &/atau import data
8. Buat storage symlink & build assets
9. Jalankan server dan buka di browser

1) Install XAMPP (Apache + MySQL)

-   Buka https://www.apachefriends.org/download.html
-   Download versi XAMPP untuk Windows (PHP 8.x yang kompatibel).
-   Jalankan installer → Next → Next. Saat selesai, buka XAMPP Control Panel.
-   Start: Apache dan MySQL.
-   Cek: buka http://localhost → halaman XAMPP muncul.

Catatan:

-   Jika Apache tidak bisa start: port 80/443 sudah dipakai. Ubah port Apache di XAMPP Control Panel → Config → httpd.conf (ubah Listen 80 ke Listen 8080) lalu restart. Akan mengubah alamat menjadi http://localhost:8080.

2. Install Composer (PHP dependencies)

-   Download installer Windows: https://getcomposer.org/Composer-Setup.exe
-   Jalankan dan biarkan menambahkan ke PATH.
-   Verifikasi di PowerShell / CMD:
    -   composer -V

3. Install Node.js & pnpm (frontend build)

-   Download LTS dari https://nodejs.org/ (mis. 18.x atau 20.x LTS)
-   Instal dan verifikasi:

    -   node -v
    -   npm -v

-   pnpm (direkomendasikan di proyek ini)

    -   instal pnpm global:
        -   npm install -g pnpm
    -   Verifikasi:
        -   pnpm -v

-   Catatan: pnpm mengganti npm/yarn — semua perintah npm di panduan ini dapat dijalankan dengan pnpm (pnpm install, pnpm dev, pnpm build).

4. Siapkan project lokal (Git)

-   Buka PowerShell, masuk folder tujuan:
    -   cd C:\path\to\projects
-   Clone repository:
    -   git clone <repo-url> yayasan-management-system
    -   cd yayasan-management-system
-   (Jika Anda sudah punya source di d:\coding\cms_yayasan, langsung gunakan folder itu.)

5. Copy .env dan atur konfigurasi

-   Salin file contoh:
    -   PowerShell:
        -   copy .env.example .env
    -   CMD:
        -   copy .env.example .env
-   Buka .env di text editor (VS Code/Notepad). Ubah bagian penting:
    -   APP_URL=http://localhost (atau http://localhost:8000 jika pakai artisan serve)
    -   DB_CONNECTION=mysql
    -   DB_HOST=127.0.0.1
    -   DB_PORT=3306 (atau 3307 jika MySQL XAMPP menggunakan 3307)
    -   DB_DATABASE= yms_local
    -   DB_USERNAME= root
    -   DB_PASSWORD= (kosong jika default XAMPP)
-   Simpan.

6. Buat database di phpMyAdmin (atau import SQL)

Opsi A — Buat database kosong (pakai migrasi/seeder)

-   Buka http://localhost/phpmyadmin
-   Klik "New" → beri nama database (contoh: yms_local) → Create
-   Atur .env agar menunjuk ke database ini (DB_DATABASE, DB_USERNAME, DB_PASSWORD)

Opsi B — Pakai file SQL hasil export (jika ada halaman/data dibuat langsung di DB)

-   Jika tim memberikan file dump (contoh: yms_export.sql), import ke MySQL:
    -   Menggunakan phpMyAdmin: buka database baru → Import → pilih file .sql → Go
    -   Menggunakan CLI (lebih cepat, di PowerShell/CMD):
        -   mysql -u root -p
        -   CREATE DATABASE yms_local CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
        -   exit
        -   mysql -u root -p yms_local < C:\path\to\yms_export.sql
-   Setelah import, periksa tabel penting (pages, users, publications).
-   Jika dump hanya berisi data (tanpa struktur), jalankan migrate dulu:
    -   php artisan migrate
    -   lalu import data.
-   Catatan file media:
    -   Dump SQL tidak menyertakan file di storage/public. Jika ada gambar/foto di DB yang menunjuk storage paths, minta juga folder storage/app/public export atau zip dari instance sumber dan taruh ke storage/app/public lalu jalankan:
        -   php artisan storage:link
        -   pastikan permission/paths sesuai.

7. Install PHP & JS dependencies

-   Dari root project (folder yang berisi composer.json & package.json) jalankan:
    -   composer install
    -   pnpm install # <- gunakan pnpm (pengganti npm)
-   Jika Anda lebih nyaman menggunakan npm, gunakan:
    -   npm install
-   Jika composer gagal karena memory: jalankan:

    -   php -d memory_limit=-1 composer install

-   Catatan pnpm:
    -   pnpm membuat node_modules berbeda (pinned store) — biasanya bekerja tanpa konfigurasi tambahan.
    -   Jika ada masalah package resolution, coba `pnpm install --shamefully-hoist` atau hapus node_modules & pnpm-lock.yaml lalu ulangi.

8. Generate APP_KEY & migrasi database (jika menggunakan migrasi)

-   Di terminal:
    -   php artisan key:generate
    -   Jika Anda memilih migrasi: php artisan migrate --seed
    -   Jika Anda sudah import full SQL dump (schema + data), tidak perlu migrate; cukup cek tabel sudah ada.

9. Buat storage symlink (file publik)

-   php artisan storage:link

10. Jalankan build frontend

-   Dengan pnpm (direkomendasikan):

    -             - Untuk development (hot reload):
        -               - pnpm run dev
        -               - atau pnpm dev  (tergantung script di package.json)
    -             - Untuk production build:
        -               - pnpm run build

-   Dengan npm (jika tidak pakai pnpm):

    -             - npm run dev
    -             - npm run build

-   Jika Vite menampilkan error port, tambahkan -- --port <port> atau ubah PORT env (contoh: pnpm run dev -- --port 5174).

11. Jalankan server Laravel

-   php artisan serve
-   Buka http://127.0.0.1:8000

Mail testing dengan MailHog (local)

-   Opsi Docker (paling mudah):
    -   jalankan: docker run -d -p 1025:1025 -p 8025:8025 --name mailhog mailhog/mailhog
    -   akses UI MailHog: http://localhost:8025
-   Opsi Windows binary (tanpa Docker):
    -   Unduh MailHog binary atau gunakan choco jika tersedia.
    -   Jalankan MailHog, pastikan mendengarkan port 1025 dan UI 8025.
-   Konfigurasi .env Laravel:
    -   MAIL_MAILER=smtp
    -   MAIL_HOST=127.0.0.1
    -   MAIL_PORT=1025
    -   MAIL_USERNAME=
    -   MAIL_PASSWORD=
    -   MAIL_ENCRYPTION=null
    -   MAIL_FROM_ADDRESS=no-reply@local.test
    -   MAIL_FROM_NAME="${APP_NAME}"
-   Kirim test email dari aplikasi — cek di http://localhost:8025.

Troubleshooting umum

-   Composer tidak ditemukan: restart terminal setelah instal Composer.
-   Error "Access denied" ke database: periksa DB_USERNAME & DB_PASSWORD di .env.
-   Apache tidak mau start: matikan layanan lain (IIS, Skype) atau ubah port Apache.
-   npm run dev error port Vite: ubah PORT env atau jalankan npm run dev -- --port 5174
-   Migrasi gagal karena missing extension: aktifkan php_pdo_mysql di php.ini (pada XAMPP, File → PHP (php.ini)).
-   Permission issues (Windows): jalankan terminal sebagai Administrator.

-   Masalah terkait pnpm:
    -   Paket tidak ditemukan: coba `pnpm install` ulang dan hapus `node_modules` & `pnpm-lock.yaml` lalu `pnpm install`.
    -   Konflik native addons: jalankan `pnpm rebuild` atau gunakan npm jika perlu sementara.
    -   Jika server target tidak pakai pnpm, Anda bisa tetap menggunakan npm saat deployment.

Keamanan lokal

-   Jangan gunakan kredensial produksi di mesin lokal.
-   Jangan commit .env ke repo (gitignore ada .env).

Tambahan opsional

-   Mail testing: MailHog seperti di atas.
-   Jika Anda menggunakan dump DB dari sumber lain, pastikan juga menyalin storage (media) dan environment (APP_URL) agar link gambar berfungsi.
-   Cron & Queue: untuk fitur background, gunakan Supervisor (Linux) — tidak diperlukan untuk pengembangan lokal.

Tips akhir

-   Ikuti langkah secara berurutan.
-   Simpan perubahan .env hanya di lokal.
-   Jika mau, saya bisa tambahkan: skrip import SQL otomatis, contoh perintah Docker Compose (database + mailhog), atau panduan vhost step-by-step untuk Windows.

Butuh saya tambahkan panduan vhost step-by-step atau versi untuk Linux/macOS?
