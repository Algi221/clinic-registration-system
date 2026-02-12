# 👨‍⚕️ Doctor-Poli Strict Relationship System

## 📋 Overview

Sistem ini mengimplementasikan relasi ketat antara dokter dan poli dengan aturan:

- **1 User (Doctor) = 1 Dokter Profile = 1 Poli**
- Email dokter bersifat unik
- Dokter hanya dapat mengakses data dari poli-nya sendiri
- Security: Poli ditentukan dari database, bukan dari frontend

---

## 🗄️ Database Schema

### User Model

```prisma
model User {
  id           String        @id @default(uuid())
  email        String        @unique       ← Email dokter
  password     String
  role         Role          @default(PATIENT)
  dokter       Dokter?       ← Optional relation
  // ...
}
```

### Dokter Model

```prisma
model Dokter {
  id         String         @id @default(uuid())
  userId     String         @unique       ← Link to User
  user       User           @relation
  nama       String
  spesialis  String
  poliId     String                       ← Link to Poli (FIXED)
  poli       Poli           @relation
  // ...
}
```

### Poli Model

```prisma
model Poli {
  id        String   @id @default(uuid())
  nama      String
  dokters   Dokter[]       ← Multiple doctors
  // ...
}
```

---

## 👥 Dokter Mapping

### Seeded Doctors

| Email             | Name        | Spesialis         | Poli            | Password  |
| ----------------- | ----------- | ----------------- | --------------- | --------- |
| arden@gmail.com   | Dr. Arden   | Dokter Kecantikan | Poli Kecantikan | dokter123 |
| alip@gmail.com    | Dr. Alip    | Dokter Kandungan  | Poli Kandungan  | dokter123 |
| algi@gmail.com    | Dr. Algi    | Dokter Umum       | Poli Umum       | dokter123 |
| chika@gmail.com   | Dr. Chika   | Dokter Gigi       | Poli Gigi       | dokter123 |
| humayra@gmail.com | Dr. Humayra | Ahli Gizi         | Poli Gizi       | dokter123 |
| dokter@klinik.com | Dr. Ahmad   | Dokter Umum       | Poli Umum       | dokter123 |

### Poli List

1. **Poli Umum** - Pelayanan kesehatan umum
2. **Poli Gigi** - Kesehatan gigi dan mulut
3. **Poli Kandungan** - Kesehatan ibu dan anak
4. **Poli Gizi** - Konsultasi gizi dan nutrisi
5. **Poli Kecantikan** - Perawatan kulit dan kecantikan

---

## 🔐 Authentication & Authorization

### Login Flow

#### 1. **User Login**

```javascript
POST /api/auth/login
{
  "email": "arden@gmail.com",
  "password": "dokter123"
}
```

#### 2. **Backend Response (for DOCTOR)**

```javascript
{
  "token": "jwt_token_here",
  "user": {
    "id": "user-uuid",
    "name": "Dr. Arden",
    "email": "arden@gmail.com",
    "role": "DOCTOR",
    "dokter": {              // ← Included for doctors
      "id": "dokter-uuid",
      "nama": "Dr. Arden",
      "spesialis": "Dokter Kecantikan",
      "poli": {
        "id": "poli-uuid",
        "nama": "Poli Kecantikan"
      }
    }
  }
}
```

#### 3. **Frontend Storage**

```javascript
// AuthContext stores user with dokter info
const user = {
  ...userData,
  dokter: {
    poli: { nama: "Poli Kecantikan" },
    spesialis: "Dokter Kecantikan",
  },
};
```

---

## 🛡️ Security & Access Control

### Backend: GET /pendaftaran

```javascript
router.get("/", authMiddleware, async (req, res) => {
  let where = {};

  // PATIENT: hanya pendaftaran mereka sendiri
  if (req.user.role === "PATIENT") {
    where = { pasienId: req.user.id };
  }

  // DOCTOR: hanya pendaftaran dari poli mereka
  if (req.user.role === "DOCTOR") {
    // Get dokter profile
    const dokter = await prisma.dokter.findUnique({
      where: { userId: req.user.id },
      select: { poliId: true },
    });

    // Filter by poli
    where = {
      jadwal: {
        dokter: {
          poliId: dokter.poliId, // ← Only their poli!
        },
      },
    };
  }

  // Fetch with filter
  const pendaftaran = await prisma.pendaftaran.findMany({ where });
  res.json(pendaftaran);
});
```

### Security Rules

✅ **Poli is NOT sent from frontend**

- Frontend NEVER sends poliId
- Backend fetches poli from database
- Uses authenticated user ID to get dokter profile

✅ **Backend validates role & poli**

- Middleware checks user.role
- Queries dokter table for poliId
- Filters data based on that poliId

✅ **Doctors can ONLY access their poli data**

- Dr. Arden (Poli Kecantikan) cannot see Poli Umum patients
- Dr. Algi (Poli Umum) cannot see Poli Gigi patients

---

## 🎨 Frontend Display

### Doctor Dashboard

#### Welcome Section

```jsx
<div>
  <h1>Dashboard Dokter</h1>
  <p>Selamat datang kembali, {user.name}</p>

  {/* Poli Badge */}
  {user?.dokter?.poli && (
    <div className="poli-badge">📍 {user.dokter.poli.nama}</div>
  )}
</div>;

{
  /* Info Pills */
}
<div>
  <div>📅 {date}</div>
  <div>📊 {stats.todayTotal} pasien hari ini</div>
  <div>🩺 {user.dokter.spesialis}</div>
</div>;
```

#### What Doctor Sees

```
╔════════════════════════════════════╗
║  Dashboard Dokter                  ║
║  Selamat datang kembali, Dr. Arden ║
║  [📍 Poli Kecantikan]              ║
║                                    ║
║  📅 Senin, 10 Feb 2026              ║
║  📊 3 pasien hari ini               ║
║  🩺 Dokter Kecantikan               ║
╚════════════════════════════════════╝

Daftar Pendaftaran Pasien
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Only patients from Poli Kecantikan
✅ Filtered automatically by backend
✅ No manual selection needed
```

---

## 🧪 Testing

### Test Login for Each Doctor

```bash
# Dr. Arden - Poli Kecantikan
POST /api/auth/login
{ "email": "arden@gmail.com", "password": "dokter123" }
→ Should see dokter.poli.nama = "Poli Kecantikan"

# Dr. Alip - Poli Kandungan
POST /api/auth/login
{ "email": "alip@gmail.com", "password": "dokter123" }
→ Should see dokter.poli.nama = "Poli Kandungan"

# Dr. Chika - Poli Gigi
POST /api/auth/login
{ "email": "chika@gmail.com", "password": "dokter123" }
→ Should see dokter.poli.nama = "Poli Gigi"
```

### Test Data Filtering

```bash
# Create test registration for Poli Kecantikan
POST /api/pendaftaran
{
  "jadwalId": "jadwal-dr-arden-id",  # Jadwal from Poli Kecantikan
  "keluhan": "Test"
}

# Login as Dr. Arden (Poli Kecantikan)
GET /api/pendaftaran
→ ✅ Should see the registration

# Login as Dr. Algi (Poli Umum)
GET /api/pendaftaran
→ ❌ Should NOT see the registration
```

---

## 📝 Migration & Seed

### Run Migration

```bash
cd be
npx prisma migrate dev --name add_dokter_user_relation
```

### Run Seed

```bash
npx prisma db seed
```

### Expected Output

```
✅ Created 5 Poli
✅ Created 6 Doctor Users
✅ Created 1 Patient User
✅ Created 6 Dokter Profiles (linked to Users & Poli)
✅ Created Jadwal Dokter

📊 SEED SUMMARY:
👨‍⚕️ Dokter Mapping:
   - arden@gmail.com → Poli Kecantikan
   - alip@gmail.com → Poli Kandungan
   - algi@gmail.com → Poli Umum
   - chika@gmail.com → Poli Gigi
   - humayra@gmail.com → Poli Gizi
```

---

## 🔑 Key Features

### ✅ Implemented

- [x] User-Dokter-Poli relational model
- [x] Unique email per doctor
- [x] One poli per doctor (fixed in database)
- [x] Login returns dokter + poli info
- [x] Backend filters pendaftaran by doctor's poli
- [x] Frontend displays poli badge
- [x] Frontend displays spesialis
- [x] Security: poli from database, not frontend
- [x] Seed with multiple doctors & poli
- [x] Documentation

### 🚀 Benefits

1. **Security**: Cannot manipulate poli from frontend
2. **Simplicity**: Doctor doesn't need to select poli
3. **Accuracy**: Auto-filtered data, no mistakes
4. **Scalability**: Easy to add more doctors/poli
5. **Maintainability**: Clear separation of concerns

---

## 🎯 Business Rules Enforced

### Database Level

- ✅ userId is UNIQUE in Dokter table (1 user = 1 dokter)
- ✅ poliId is required in Dokter table
- ✅ Foreign key constraints prevent orphaned data

### Application Level

- ✅ Login fetches dokter profile automatically
- ✅ GET pendaftaran filters by dokter's poliId
- ✅ Frontend never sends poliId
- ✅ Backend validates role before filtering

### UI Level

- ✅ Doctor sees their poli name in dashboard
- ✅ Doctor sees their spesialis
- ✅ Only relevant data displayed (their poli only)

---

## 📚 Related Files

### Backend

- `prisma/schema.prisma` - Database schema with relations
- `prisma/seed.js` - Seed data with doctor-poli mapping
- `src/routes/auth.js` - Login with dokter info
- `src/routes/pendaftaran.js` - Filtering by poli

### Frontend

- `src/pages/doctor/DoctorDashboard.jsx` - Shows poli info
- `src/context/AuthContext.jsx` - Stores user with dokter info

---

**Sistem relasi Doctor-Poli sudah complete dengan security & filtering! 🎉**
