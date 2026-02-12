# 🚀 MySQL Setup Guide untuk Sistem Pendaftaran Klinik

## 📋 Persiapan

Anda akan setup MySQL untuk project ini. Ada 3 opsi mudah:

---

## ✅ **Option 1: XAMPP** (PALING MUDAH - RECOMMENDED!)

### 1. Download & Install XAMPP

- Download: https://www.apachefriends.org/download.html
- Install di `C:\xampp` (default location)

### 2. Start MySQL dari XAMPP Control Panel

1. Buka **XAMPP Control Panel**
2. Klik **Start** di baris MySQL
3. MySQL akan running di port 3306

### 3. Create Database

1. Buka browser: http://localhost/phpmyadmin
2. Klik tab **SQL**
3. Run query ini:
   ```sql
   CREATE DATABASE klinik_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```
4. Klik **Go**

### 4. Run Prisma Migrations

```powershell
cd d:\Sistem Pendaftaran Klinik\be
npx prisma migrate deploy
npx prisma db seed
```

### 5. Test Connection

```powershell
node check-users.js
```

**✅ DONE!** MySQL siap dipakai!

---

## ✅ **Option 2: MySQL Community Server**

### 1. Download MySQL Installer

- Download: https://dev.mysql.com/downloads/installer/
- Pilih: **mysql-installer-community-X.X.X.msi**

### 2. Install MySQL

1. Run installer
2. Choose: **Developer Default** atau **Server only**
3. Next → Next → Execute

### 3. MySQL Server Configuration

1. **Type and Networking:**
   - Config Type: Development Computer
   - Port: 3306 (default)
   - ✅ Check "Open Windows Firewall ports"

2. **Authentication Method:**
   - Use Strong Password Encryption (Recommended)

3. **Accounts and Roles:**
   - MySQL Root Password: **kosongkan** (sesuai .env) atau isi password
   - Jika isi password, update `.env`:
     ```env
     DATABASE_URL="mysql://root:YOUR_PASSWORD@127.0.0.1:3306/klinik_db"
     ```

4. **Windows Service:**
   - ✅ Configure MySQL Server as a Windows Service
   - Service Name: MySQL80 (atau MySQL)
   - ✅ Start the MySQL Server at System Startup

5. Execute → Finish

### 4. Create Database via MySQL Command Line

```powershell
# Buka MySQL Command Line Client atau:
mysql -u root -p

# Di MySQL prompt:
CREATE DATABASE klinik_db;
SHOW DATABASES;
EXIT;
```

### 5. Run Prisma Migrations

```powershell
cd d:\Sistem Pendaftaran Klinik\be
npx prisma migrate deploy
npx prisma db seed
```

---

## ✅ **Option 3: Docker** (Untuk yang sudah familiar)

### 1. Install Docker Desktop

- Download: https://www.docker.com/products/docker-desktop

### 2. Create docker-compose.yml di folder `be/`:

```yaml
version: "3.8"
services:
  mysql:
    image: mysql:8.0
    container_name: klinik_mysql
    environment:
      MYSQL_DATABASE: klinik_db
      MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

### 3. Start MySQL Container

```powershell
cd d:\Sistem Pendaftaran Klinik\be
docker-compose up -d
```

### 4. Run Migrations

```powershell
npx prisma migrate deploy
npx prisma db seed
```

---

## 🔧 **After Installation - Setup Database**

Setelah MySQL running, jalankan commands ini:

```powershell
# 1. Navigate to backend folder
cd d:\Sistem Pendaftaran Klinik\be

# 2. Generate Prisma Client
npx prisma generate

# 3. Run migrations (create tables)
npx prisma migrate deploy

# 4. Seed database (insert demo data)
npx prisma db seed

# 5. Verify data
node check-users.js
```

**Expected output:**

```
📊 Total users in database: 2

👥 Users:

1. Dr. Ahmad
   Email: dokter@klinik.com
   Role: DOCTOR
   ...

2. Budi
   Email: budi@gmail.com
   Role: PATIENT
   ...
```

---

## 🧪 **Test Registration**

Setelah database ready, restart server dan test:

```powershell
# Stop current server (Ctrl+C)
# Start again
npm run dev
```

Test register dengan Postman/Thunder Client:

```json
POST http://localhost:5000/api/auth/register
Content-Type: application/json

{
  "name": "Test User",
  "email": "test@example.com",
  "password": "password123",
  "role": "PATIENT"
}
```

**Expected Response (201):**

```json
{
  "message": "User registered successfully",
  "user": {
    "id": "...",
    "name": "Test User",
    "email": "test@example.com",
    "role": "PATIENT"
  }
}
```

---

## ❓ **Troubleshooting**

### MySQL Service tidak running

```powershell
# Check service status
Get-Service -Name MySQL* | Select-Object Name, Status

# Start service (Run as Administrator)
net start MySQL80
```

### Can't reach database server

1. Cek MySQL service running
2. Cek port 3306 tidak dipakai aplikasi lain
3. Cek firewall settings

### Access denied for user 'root'

Update `.env`:

```env
DATABASE_URL="mysql://root:YOUR_PASSWORD@127.0.0.1:3306/klinik_db"
```

### Database doesn't exist

Create database:

```sql
CREATE DATABASE klinik_db;
```

---

## 🎯 **Quick Commands Reference**

```powershell
# Start MySQL (XAMPP)
# Via XAMPP Control Panel → Click Start

# Start MySQL (Service)
net start MySQL80

# Check if MySQL is running
Get-Service MySQL*

# Connect to MySQL
mysql -u root -p

# Create database
CREATE DATABASE klinik_db;

# Run migrations
npx prisma migrate deploy

# Seed database
npx prisma db seed

# Check users
node check-users.js

# Open Prisma Studio
npx prisma studio
```

---

## 🎉 **Recommendation for You**

**Install XAMPP** - Paling mudah, sudah include:

- ✅ MySQL
- ✅ phpMyAdmin (GUI untuk manage database)
- ✅ Apache (jika butuh PHP nanti)
- ✅ Control Panel yang user-friendly

Download XAMPP → Start MySQL → Create Database → Run Migrations → Done! 🚀

---

**Pilih salah satu option dan ikuti langkah-langkahnya. Setelah selesai, registration akan langsung work!**
