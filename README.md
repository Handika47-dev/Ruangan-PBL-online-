# Sistem Peminjaman Ruangan Tutorial PBL — FK UNPATTI

Aplikasi React + Vite + Supabase. Fitur: jadwal publik tanpa identitas, pengajuan mahasiswa, kode + PIN pengecekan status, login admin, persetujuan/penolakan/pembatalan/perubahan jadwal, audit log, serta constraint database untuk mencegah bentrok.

## 1. Persiapan
- Node.js 20 atau lebih baru
- Akun dan project Supabase
- URL dan publishable key Supabase

## 2. Instalasi lokal
```bash
npm install
Salin `.env.example` menjadi `.env`, lalu isi URL dan publishable key project Supabase.
```
Isi `.env`:
```env
VITE_SUPABASE_URL=https://cpyoymtvywzswjgwzetz.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=publishable_key_anda
```
Jangan pernah menggunakan `service_role` key pada frontend. Publishable key memang bersifat publik; keamanan data dijaga oleh RLS dan fungsi database.

## 3. Siapkan database Supabase
1. Buka Supabase Dashboard → SQL Editor.
2. Buka file `supabase/schema.sql`, salin seluruh isinya dan jalankan.
3. Pastikan tabel `profiles`, `rooms`, `bookings`, `booking_audit` dan view telah terbentuk.
4. Di Project Settings → API/Data API, pastikan schema `public` tersedia untuk Data API.

## 4. Buat akun admin
1. Di Supabase Dashboard → Authentication → Users → Add user.
2. Buat akun email/password admin (password jangan dibagikan).
3. Salin UUID user tersebut.
4. Jalankan SQL berikut di SQL Editor:
```sql
update public.profiles set role='admin' where id='UUID_USER_ADMIN';
```
Jangan membuka pendaftaran admin publik. Buat admin hanya dari Dashboard Supabase.

## 5. Jalankan aplikasi
```bash
npm run dev
```
Buka URL lokal yang ditampilkan Vite. Untuk produksi:
```bash
npm run build
npm run preview
```

## 6. Deployment
Deploy folder proyek ke Vercel atau Netlify sebagai aplikasi Vite. Build command `npm run build`, output directory `dist`. Tambahkan environment variables `VITE_SUPABASE_URL` dan `VITE_SUPABASE_PUBLISHABLE_KEY` di pengaturan hosting. Tambahkan rewrite SPA agar semua path diarahkan ke `index.html`.

Contoh `vercel.json`:
```json
{"rewrites":[{"source":"/(.*)","destination":"/"}]}
```

## 7. Cara kerja
- Pengunjung membaca `public_room_schedule`; view hanya berisi nama ruang, tanggal, waktu, dan status.
- Mahasiswa mengirim pengajuan lewat RPC `submit_booking`. Database menghasilkan kode booking dan PIN; hanya hash PIN disimpan.
- Mahasiswa mengecek status melalui RPC `check_booking_status` dengan kode + PIN. Respons tidak mengandung NIM/nama/kelompok/tutor.
- Admin login lewat Supabase Auth. Role admin diperiksa di tabel `profiles`.
- Semua tindakan admin dijalankan lewat RPC yang memeriksa role di database dan dicatat ke `booking_audit`.
- Constraint exclusion PostgreSQL mengunci slot status `pending` dan `approved`, dengan rentang `[mulai, selesai)` sehingga jadwal berurutan diperbolehkan.

## 8. Catatan sebelum dipakai kampus
- Ini paket implementasi awal yang perlu diuji di project Supabase sebelum penggunaan resmi.
- Pastikan admin dan pengelola FK menyetujui alur data NIM/tutor dan kebijakan retensi.
- Untuk penggunaan produksi, tambahkan CAPTCHA/rate limit pada RPC publik, backup terjadwal, verifikasi domain email admin, dan uji penetrasi.
- Jam mengikuti zona waktu tanggal/jam lokal kampus. Data pribadi tidak tersedia di view publik.
- Status `pending` ikut mengunci slot. Jika pengajuan ditolak/dibatalkan/dialihkan online, slot otomatis bebas karena status tidak lagi termasuk status pengunci.
