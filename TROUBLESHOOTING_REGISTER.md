# ❌ Troubleshooting: Cannot Register - Database Connection Issue

## 🐛 Problem

```
POST /api/auth/register 400 2085.391 ms - 409
POST /api/auth/register 400 2030.839 ms - 409
```

**Root Cause:** **MySQL server is not running!**

Error details:

```
Can't reach database server at `127.0.0.1:3306`
```

---

## ✅ Solutions

### Option 1: Start MySQL Server (Recommended if you have MySQL)

#### Windows:

1. **Open Services** (`Win + R` → `services.msc`)
2. Find **MySQL** or **MySQL80** service
3. Right-click → **Start**

Or via Command Line (Run as Administrator):

```cmd
net start MySQL
```

atau

```cmd
net start MySQL80
```

#### Check if MySQL is running:

```bash
mysql -u root -p
```

---

### Option 2: Install MySQL (if not installed)

1. **Download MySQL:**
   - https://dev.mysql.com/downloads/installer/
2. **Install and Setup:**
   - Choose "Developer Default"
   - Set root password (or leave empty like in .env)
   - Start MySQL Service

3. **Create Database:**
   ```sql
   CREATE DATABASE klinik_db;
   ```

---

### Option 3: Use SQLite Instead (Quick Alternative)

If you don't want to deal with MySQL, switch to SQLite:

#### 1. Update `prisma/schema.prisma`:

```prisma
datasource db {
  provider = "sqlite"           // Change from "mysql"
  url      = env("DATABASE_URL")
}
```

#### 2. Update `.env`:

```env
DATABASE_URL="file:./dev.db"
JWT_SECRET="supersecretkey123"
PORT=5000
```

#### 3. Reinstall Prisma Client & Migrate:

```bash
npm install @prisma/client
npx prisma generate
npx prisma migrate dev --name init
npx prisma db seed
```

---

### Option 4: Use Docker MySQL

If you have Docker:

#### 1. Create `docker-compose.yml`:

```yaml
version: "3.8"
services:
  mysql:
    image: mysql:8.0
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

#### 2. Start MySQL:

```bash
docker-compose up -d
```

#### 3. Run migrations:

```bash
npx prisma migrate dev
npx prisma db seed
```

---

## 🧪 Verify Database Connection

After starting MySQL, verify connection:

```bash
node check-users.js
```

Should show:

```
📊 Total users in database: X
```

---

## 🚀 Quick Fix (If MySQL is already installed but not running)

```bash
# Windows (Run as Administrator)
net start MySQL80

# Then run migrations if needed
cd d:\Sistem Pendaftaran Klinik\be
npx prisma migrate dev

# Seed database
npx prisma db seed

# Start server
npm run dev
```

---

## 📝 Current Database Configuration

**File:** `.env`

```
DATABASE_URL="mysql://root:@127.0.0.1:3306/klinik_db"
```

This expects:

- MySQL server running on port 3306
- Username: `root`
- Password: (empty)
- Database: `klinik_db`

---

## ✅ After Fixing

Once database is connected, the registration should work:

```bash
POST /api/auth/register 201 - Success!
```

---

**Choose the option that works best for you and let me know if you need help with any step!** 🚀
