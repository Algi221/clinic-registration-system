# Update Database - Change ADMIN Role to DOCTOR

## Masalah

Database masih menggunakan role `ADMIN` tetapi aplikasi sudah diperbarui untuk menggunakan role `DOCTOR` dan `PATIENT`.

## Solusi

### Opsi 1: Menggunakan phpMyAdmin atau MySQL Workbench

1. Buka phpMyAdmin atau MySQL Workbench
2. Pilih database `klinik_db`
3. Jalankan SQL berikut:

```sql
-- Update the User table enum
ALTER TABLE `User`
MODIFY COLUMN `role` ENUM('DOCTOR', 'PATIENT') NOT NULL DEFAULT 'PATIENT';

-- Update existing admin user to doctor role
UPDATE `User`
SET `role` = 'DOCTOR'
WHERE `email` = 'admin@klinik.com';
```

### Opsi 2: Menggunakan Command Line (Laragon/XAMPP)

#### Untuk Laragon:

```bash
C:\laragon\bin\mysql\mysql-8.0.30-winx64\bin\mysql.exe -u root -e "USE klinik_db; ALTER TABLE User MODIFY COLUMN role ENUM('DOCTOR', 'PATIENT') NOT NULL DEFAULT 'PATIENT'; UPDATE User SET role = 'DOCTOR' WHERE email = 'admin@klinik.com';"
```

#### Untuk XAMPP:

```bash
C:\xampp\mysql\bin\mysql.exe -u root -e "USE klinik_db; ALTER TABLE User MODIFY COLUMN role ENUM('DOCTOR', 'PATIENT') NOT NULL DEFAULT 'PATIENT'; UPDATE User SET role = 'DOCTOR' WHERE email = 'admin@klinik.com';"
```

### Opsi 3: Reset Database Lengkap (PALING MUDAH)

Jika data Anda tidak penting, reset database dan jalankan seed lagi:

```bash
cd be
npx prisma db push --force-reset
npm run seed
```

## Verifikasi

Setelah update, cek dengan query:

```sql
SELECT id, name, email, role FROM User;
```

Harusnya akan muncul:

- doctor@klinik.com dengan role DOCTOR
- budi@gmail.com dengan role PATIENT

## Kredensial Login Baru

- **Dokter**: doctor@klinik.com / doctor123
- **Pasien**: budi@gmail.com / patient123
