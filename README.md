# Sistem Pendaftaran Klinik Berbasis Web

Proyek sistem pendaftaran klinik sederhana dengan fitur CRUD dan alur pendaftaran pasien.

## Tech Stack

### Frontend

- **Framework**: Vite + React
- **Styling**: Tailwind CSS + shadcn/ui (custom)
- **Library**: Axios, React Router, React Hook Form, Lucide React
- **State Management**: React Context API

### Backend

- **Runtime**: Node.js
- **Framework**: Express.js
- **ORM**: Prisma
- **Database**: MySQL
- **Auth**: JWT (JSON Web Token) & bcryptjs

## Fitur Utama

1. **Autentikasi**: Login & Register (Role: Admin & Pasien).
2. **Dashboard**: Menampilkan riwayat pendaftaran berdasarkan role.
3. **Pendaftaran**: Pasien memilih jadwal dokter/poli dan menyertakan keluhan.
4. **Manajemen Admin**: Admin dapat menerima atau menolak pendaftaran pasien.

## Cara Instalasi

### 1. Persiapan Database

- Pastikan MySQL berjalan (XAMPP/Laragon).
- Buat database baru bernama `klinik_db`.
- Sesuaikan `DATABASE_URL` di `be/.env` jika perlu.

### 2. Setup Backend

```bash
cd be
npm install
npx prisma db push
npm run seed
npm run dev
```

### 3. Setup Frontend

```bash
cd fe
npm install
npm run dev
```

## Akun Demo (Seed)

- **Dokter**: `dokter@klinik.com` / `dokter123`
- **Pasien**: `budi@gmail.com` / `patient123`

## Endpoint API Utama

- `POST /api/auth/register` - Daftar akun baru
- `POST /api/auth/login` - Masuk & dapatkan token
- `GET /api/poli` - List poli
- `GET /api/jadwal` - List jadwal dokter & poli
- `POST /api/pendaftaran` - Kirim pendaftaran baru (Pasien)
- `PATCH /api/pendaftaran/:id/status` - Update status (Admin)
