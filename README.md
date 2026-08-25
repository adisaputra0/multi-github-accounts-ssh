# Setup Multi-Akun GitHub di Satu Laptop

Panduan ini menjelaskan cara mengonfigurasi 2 atau lebih akun GitHub dalam satu laptop menggunakan SSH tanpa perlu login/logout di browser atau Git CLI setiap kali berpindah project.

Metode ini cocok jika kamu memiliki:

- Akun GitHub pribadi
- Akun GitHub organisasi / tempat kerja / kampus
- Akun GitHub project freelance

## 1. Gambaran Cara Kerja

Misalkan kita memiliki dua akun:

| Akun       | Kegunaan                | Contoh Username |
| ---------- | ----------------------- | --------------- |
| Akun Kerja | Organisasi / Perusahaan | `akun-kerja`    |
| Akun Utama | Pribadi                 | `akun-pribadi`  |

Kita akan membuat SSH Key terpisah untuk masing-masing akun, lalu mendaftarkan SSH Host Alias agar Git tahu kunci mana yang harus dipakai.

```
Laptop
├── SSH Key: id_ed25519_kerja   ──► GitHub: akun-kerja   (Host: github-kerja)
└── SSH Key: id_ed25519_pribadi ──► GitHub: akun-pribadi (Host: github-pribadi)
```

Contoh SSH Remote URL:

- `git@github-kerja:akun-kerja/project-kantor.git` → Memakai kunci kerja
- `git@github-pribadi:akun-pribadi/portfolio.git` → Memakai kunci pribadi

## 2. Pemisahan Autentikasi vs Identitas Commit

Pahami dua konsep penting ini:

- **SSH Key & Remote URL**: Menentukan hak akses autentikasi ke repository GitHub (siapa yang push/pull).
- **Git Config (`user.name` & `user.email`)**: Menentukan identitas pembuat commit yang tercatat di riwayat Git.

> ⚠️ Memasang SSH key milik Akun A tidak otomatis mengubah `user.email` commit kamu menjadi Akun A. Keduanya harus disesuaikan.

## 3. Cek Prasyarat Lengkap

Buka Git Bash, lalu cek instalasi Git dan keberadaan folder SSH:

```bash
git --version
ls ~/.ssh
```

(Jika `ls ~/.ssh` menampilkan error folder tidak ditemukan, folder akan otomatis dibuat pada langkah berikutnya.)

## 4. Buat SSH Key untuk Akun Kerja

> ⚠️ **PENTING (Khusus Windows / Git Bash)**: Jangan mengetik `~/.ssh/...` secara manual di dalam prompt interaktif `ssh-keygen` karena tilde (`~`) tidak dievaluasi oleh prompt dan akan menyebabkan error. Gunakan flag `-f` langsung di perintah.

Jalankan perintah satu baris ini:

```bash
ssh-keygen -t ed25519 -C "email-kerja@example.com" -f ~/.ssh/id_ed25519_kerja
```

Tekan Enter jika tidak ingin memakai passphrase (atau masukkan passphrase untuk keamanan ekstra).

Kunci berhasil dibuat di `~/.ssh/id_ed25519_kerja`.

## 5. Buat SSH Key untuk Akun Pribadi

Jalankan perintah berikut:

```bash
ssh-keygen -t ed25519 -C "email-pribadi@example.com" -f ~/.ssh/id_ed25519_pribadi
```

## 6. Verifikasi Berkas Kunci

Cek apakah berkas kunci sudah terbentuk:

```bash
ls ~/.ssh
```

Pastikan minimal terdapat berkas berikut:

- `id_ed25519_kerja` (Private Key)
- `id_ed25519_kerja.pub` (Public Key)
- `id_ed25519_pribadi` (Private Key)
- `id_ed25519_pribadi.pub` (Public Key)

## 7. Buat Konfigurasi SSH (`~/.ssh/config`)

Buat dan buka file konfigurasi SSH:

```bash
touch ~/.ssh/config
nano ~/.ssh/config
```

Paste konfigurasi berikut:

```
# ==============================
# GitHub - Akun Kerja
# ==============================
Host github-kerja
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_kerja
    IdentitiesOnly yes

# ==============================
# GitHub - Akun Pribadi
# ==============================
Host github-pribadi
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_pribadi
    IdentitiesOnly yes
```

**Cara Simpan di Nano:**
Tekan `Ctrl + X` → Ketik `Y` → Tekan `Enter`.

Atur izin akses file config agar lebih aman:

```bash
chmod 600 ~/.ssh/config
```

## 8. Tambahkan Public Key ke Masing-Masing Akun GitHub

### A. Akun Kerja

Tampilkan dan salin isi public key:

```bash
cat ~/.ssh/id_ed25519_kerja.pub
```

Buka GitHub (Login akun kerja) → **Settings** → **SSH and GPG keys** → **New SSH key**.

Beri Judul (misal: `Laptop - Kerja`), tempel kunci ke kolom Key, lalu klik **Add SSH key**.

### B. Akun Pribadi

Tampilkan dan salin isi public key:

```bash
cat ~/.ssh/id_ed25519_pribadi.pub
```

Buka GitHub (Login akun pribadi) → **Settings** → **SSH and GPG keys** → **New SSH key**.

Beri Judul (misal: `Laptop - Pribadi`), tempel kunci ke kolom Key, lalu klik **Add SSH key**.

## 9. Uji Koneksi SSH

Jalankan perintah tes berikut di Git Bash:

```bash
ssh -T github-kerja
```

Hasil sukses: `Hi <username-kerja>! You've successfully authenticated...`

```bash
ssh -T github-pribadi
```

Hasil sukses: `Hi <username-pribadi>! You've successfully authenticated...`

## 10. Cara Menggunakan di Project (Workflow)

### A. Memclone Repository Baru

Gunakan Host Alias (`github-kerja` atau `github-pribadi`), bukan `github.com` biasa atau HTTPS.

```bash
# Clone Repo Kerja
git clone git@github-kerja:akun-kerja/project-kantor.git

# Clone Repo Pribadi
git clone git@github-pribadi:akun-pribadi/portfolio.git
```

### B. Mengubah Repository yang Sudah Ada

Jika repo sudah di-clone sebelumnya via HTTPS/SSH biasa, ubah remote URL-nya:

```bash
# Di folder project Kerja:
git remote set-url origin git@github-kerja:akun-kerja/project-kantor.git

# Di folder project Pribadi:
git remote set-url origin git@github-pribadi:akun-pribadi/portfolio.git
```

## 11. Mengatur Identitas Commit (`user.name` & `user.email`)

Masuk ke folder masing-masing project dan jalankan `git config --local` agar commit tercatat atas nama email yang sesuai.

**Untuk Project Kerja:**

```bash
cd path/to/project-kantor
git config user.name "Nama Lengkap Kerja"
git config user.email "email-kerja@example.com"
```

**Untuk Project Pribadi:**

```bash
cd path/to/portfolio
git config user.name "Nama Lengkap Pribadi"
git config user.email "email-pribadi@example.com"
```

## 12. Checklist Sebelum `git push`

Gunakan perintah ini untuk memastikan konfigurasi pada repository lokal sudah benar:

```bash
# 1. Cek Remote URL (Autentikasi Push/Pull)
git remote -v

# 2. Cek Identitas Commit Lokal
git config user.name
git config user.email
```

## Ringkasan Perintah Penting

| Aksi            | Perintah                                                           |
| --------------- | ------------------------------------------------------------------ |
| Cek SSH Key     | `ls ~/.ssh`                                                        |
| Tes Koneksi SSH | `ssh -T github-kerja`                                              |
| Cek Remote Repo | `git remote -v`                                                    |
| Set Remote Repo | `git remote set-url origin git@<HOST_ALIAS>:<USERNAME>/<REPO>.git` |
| Set Email Lokal | `git config user.email "email@example.com"`                        |
