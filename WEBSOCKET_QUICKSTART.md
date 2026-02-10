# 🚀 WebSocket Quick Start Guide

## ✅ Sudah Diinstall

### Backend

- ✅ `socket.io` - WebSocket server
- ✅ Setup di `server.js`
- ✅ Utility functions di `src/utils/socketEmitter.js`
- ✅ Integration di `src/routes/pendaftaran.js`

### Frontend

- ✅ `socket.io-client` - WebSocket client
- ✅ `SocketContext` di `src/context/SocketContext.jsx`
- ✅ Custom hook `useRealtimeNotifications` di `src/hooks/`
- ✅ `NotificationBell` component sudah ada di header
- ✅ `RealtimeRegistrationList` component siap dipakai

---

## 📝 Cara Pakai

### 1. Tambah NotificationBell (Sudah ditambahkan di Layout!)

NotificationBell sudah otomatis muncul di header untuk semua user yang login.

### 2. Gunakan RealtimeRegistrationList

Di Doctor Dashboard atau Patient Dashboard:

```jsx
import RealtimeRegistrationList from "@/components/ui/RealtimeRegistrationList";

function DoctorDashboard() {
  const [registrations, setRegistrations] = useState([]);

  useEffect(() => {
    // Fetch initial data
    fetchRegistrations().then((data) => setRegistrations(data));
  }, []);

  return (
    <div>
      <h1>Dashboard</h1>
      <RealtimeRegistrationList initialRegistrations={registrations} />
    </div>
  );
}
```

### 3. Custom WebSocket Events

Jika ingin listen event custom:

```jsx
import { useSocket } from "@/context/SocketContext";

function MyComponent() {
  const { on, off } = useSocket();

  useEffect(() => {
    const handleCustomEvent = (data) => {
      console.log("Custom event:", data);
    };

    on("custom-event", handleCustomEvent);

    return () => {
      off("custom-event", handleCustomEvent);
    };
  }, []);
}
```

---

## 🔔 Events Yang Tersedia

### Backend → Frontend

| Event                        | Deskripsi                  | Target                   |
| ---------------------------- | -------------------------- | ------------------------ |
| `new-registration`           | Pendaftaran baru dibuat    | Room: `doctor`           |
| `registration-status-update` | Status pendaftaran berubah | Room: `patient-{userId}` |
| `queue-called`               | Antrian dipanggil          | Room: `patient-{userId}` |
| `queue-update`               | Update antrian             | Broadcast (semua)        |

### Frontend → Backend

| Event        | Deskripsi                  |
| ------------ | -------------------------- |
| `join-room`  | Bergabung ke room tertentu |
| `leave-room` | Keluar dari room           |

---

## 🛠️ Testing

### 1. Test Connection

1. Jalankan backend: `npm run dev` (di folder `be`)
2. Jalankan frontend: `npm run dev` (di folder `fe`)
3. Buka browser console, cari log: `✅ WebSocket connected`

### 2. Test Real-time Notifications

1. Login sebagai **Dokter** di satu browser/tab
2. Login sebagai **Pasien** di browser/tab lain
3. Buat pendaftaran baru sebagai pasien
4. Dokter akan menerima notifikasi instant! 🔔

### 3. Test Status Update

1. Dokter update status pendaftaran
2. Pasien akan menerima notifikasi instant! ✅

---

## 🐛 Troubleshooting

### ❌ WebSocket not connecting

- Check apakah backend server sudah running
- Check URL di `.env` file (frontend: `VITE_API_URL`)
- Check CORS settings di `server.js`

### ❌ Notifications tidak muncul

- Check browser permission untuk notifications
- Buka browser settings → Site Settings → Notifications
- Allow notifications untuk localhost

### ❌ Events tidak terima

- Check console log untuk error
- Pastikan user sudah join ke room yang benar
- Verify event listener terdaftar dengan `console.log`

---

## 📚 Documentation Lengkap

Lihat **WEBSOCKET_GUIDE.md** untuk dokumentasi lengkap dan advanced usage.

---

## 🎯 Next Steps

1. ✅ WebSocket sudah setup (DONE!)
2. 🔄 Integrate `RealtimeRegistrationList` ke dashboard
3. 🎨 Customize notification styles sesuai brand
4. 🧪 Test dengan multiple users
5. 🚀 Deploy to production

---

**Happy Coding! 🎉**
