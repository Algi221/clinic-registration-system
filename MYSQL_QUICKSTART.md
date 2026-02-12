# ⚡ Quick Start - MySQL Setup

## 🎯 Langkah Cepat (Pilih Salah Satu)

### Option A: XAMPP (PALING MUDAH!) ⭐

1. **Download XAMPP:** https://www.apachefriends.org/download.html
2. **Install** → Next, Next, Finish
3. **Buka XAMPP Control Panel**
4. **Klik START** di baris MySQL
5. **Buka browser:** http://localhost/phpmyadmin
6. **Klik tab SQL**, paste ini:
   ```sql
   CREATE DATABASE klinik_db;
   ```
7. **Klik Go**

**Done! Lanjut ke Step Berikutnya di bawah ↓**

---

### Option B: MySQL Installer

1. **Download:** https://dev.mysql.com/downloads/installer/
2. **Install** → Developer Default
3. **Set password root** (kosongkan saja)
4. **MySQL akan auto-start**

**Done! Lanjut ke Step Berikutnya ↓**

---

## 🚀 Setup Database (Jalankan Ini Setelah MySQL Running)

```powershell
# 1. Masuk ke folder backend
cd d:\Sistem Pendaftaran Klinik\be

# 2. Generate Prisma Client
npx prisma generate

# 3. Run migrations (buat tabel)
npx prisma migrate deploy

# 4. Seed database (isi data demo)
npx prisma db seed

# 5. Cek data berhasil
node check-users.js
```

**Expected Output:**

```
📊 Total users in database: 2

👥 Users:

1. Dr. Ahmad
   Email: dokter@klinik.com
   Role: DOCTOR

2. Budi Santoso
   Email: budi@gmail.com
   Role: PATIENT
```

---

## ✅ Test Login

Restart server dan test login:

```powershell
npm run dev
```

**Login sebagai Dokter:**

- Email: `dokter@klinik.com`
- Password: `dokter123`

**Login sebagai Pasien:**

- Email: `budi@gmail.com`
- Password: `patient123`

---

## 🎉 Done!

MySQL siap, database terisi, registration & login akan work! 🚀

**Butuh help?** Lihat `MYSQL_SETUP_GUIDE.md` untuk panduan lengkap.
