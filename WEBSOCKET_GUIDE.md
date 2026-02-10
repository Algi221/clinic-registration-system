# WebSocket Implementation Guide

## 🚀 Implementasi WebSocket untuk Real-time Features

Sistem pendaftaran klinik ini menggunakan **Socket.IO** untuk komunikasi real-time antara server dan client.

---

## 📋 Features Real-time

### 1. **Notifikasi Pendaftaran Baru** (untuk Dokter)

- Dokter menerima notifikasi instan saat ada pendaftaran baru
- Event: `new-registration`

### 2. **Update Status Pendaftaran** (untuk Pasien)

- Pasien menerima notifikasi saat status pendaftaran berubah
- Event: `registration-status-update`

### 3. **Pemanggilan Antrian** (untuk semua)

- Broadcast saat nomor antrian dipanggil
- Event: `queue-called`, `queue-update`

---

## 🔧 Backend Implementation

### Setup Server (server.js)

```javascript
const { Server } = require("socket.io");
const io = new Server(server, {
  cors: {
    origin: process.env.CLIENT_URL || "http://localhost:5173",
    methods: ["GET", "POST"],
  },
});

// Connection handler
io.on("connection", (socket) => {
  console.log(`New client connected: ${socket.id}`);

  socket.on("join-room", (room) => {
    socket.join(room);
  });

  socket.on("disconnect", () => {
    console.log(`Client disconnected: ${socket.id}`);
  });
});
```

### Emit Events dari Backend

```javascript
const { emitNewRegistration } = require("../utils/socketEmitter");

// Saat pendaftaran baru dibuat
const pendaftaran = await prisma.pendaftaran.create({...});
emitNewRegistration(pendaftaran);
```

### Socket Rooms

- `doctor` - Room untuk semua dokter
- `patient-{userId}` - Room untuk pasien tertentu
- `admin` - Room untuk admin (jika ada)

---

## 💻 Frontend Implementation

### 1. Setup SocketProvider (App.jsx)

```jsx
import { SocketProvider } from "./context/SocketContext";

function App() {
  return (
    <Router>
      <AuthProvider>
        <SocketProvider>{/* Your app content */}</SocketProvider>
      </AuthProvider>
    </Router>
  );
}
```

### 2. Menggunakan WebSocket Hook

```jsx
import { useSocket } from "../context/SocketContext";

function MyComponent() {
  const { socket, isConnected, joinRoom, on, off } = useSocket();

  useEffect(() => {
    // Join room
    joinRoom("doctor");

    // Listen to events
    const handleNewReg = (data) => {
      console.log("New registration:", data);
    };

    on("new-registration", handleNewReg);

    // Cleanup
    return () => {
      off("new-registration", handleNewReg);
    };
  }, [socket, isConnected]);
}
```

### 3. Menggunakan Notification Hook

```jsx
import { useRealtimeNotifications } from "../hooks/useRealtimeNotifications";
import { useAuth } from "../context/AuthContext";

function Dashboard() {
  const { user } = useAuth();
  const { notifications, unreadCount, markAsRead } = useRealtimeNotifications(
    user?.role,
    user?.id,
  );

  return (
    <div>
      <p>Unread: {unreadCount}</p>
      {notifications.map((notif) => (
        <div key={notif.id} onClick={() => markAsRead(notif.id)}>
          {notif.message}
        </div>
      ))}
    </div>
  );
}
```

### 4. Notification Bell Component

```jsx
import NotificationBell from "../components/ui/NotificationBell";

function Header() {
  return (
    <header>
      <NotificationBell />
    </header>
  );
}
```

---

## 🎯 Cara Menggunakan

### Backend

1. Server otomatis mendengarkan WebSocket connections
2. Gunakan fungsi dari `socketEmitter.js` untuk emit events
3. Events otomatis terkirim ke room yang sesuai

### Frontend

1. Aplikasi otomatis connect ke WebSocket saat load
2. Hook `useRealtimeNotifications` otomatis join room sesuai role
3. Browser notifications muncul otomatis (jika permission granted)
4. Komponen `NotificationBell` bisa ditambahkan di header/navbar

---

## 📦 Available Functions

### Backend (socketEmitter.js)

```javascript
emitToAll(event, data); // Emit ke semua client
emitToRoom(room, event, data); // Emit ke room tertentu
emitNewRegistration(registrationData); // Emit pendaftaran baru
emitRegistrationStatusUpdate(id, status, patientId); // Emit update status
emitQueueCall(queueData); // Emit pemanggilan antrian
```

### Frontend (useSocket hook)

```javascript
socket; // Socket instance
isConnected; // Connection status
joinRoom(room); // Join room
leaveRoom(room); // Leave room
on(event, callback); // Listen to event
off(event, callback); // Stop listening
emit(event, data); // Emit event to server
```

### Frontend (useRealtimeNotifications hook)

```javascript
notifications; // Array of notifications
unreadCount; // Jumlah notifikasi belum dibaca
markAsRead(id); // Tandai satu notifikasi dibaca
markAllAsRead(); // Tandai semua dibaca
clearNotifications(); // Hapus semua notifikasi
```

---

## 🔔 Browser Notifications

Browser notifications otomatis muncul untuk:

- Pendaftaran baru (dokter)
- Update status pendaftaran (pasien)
- Pemanggilan antrian (pasien yang dipanggil)

Permission browser notification akan diminta otomatis saat pertama kali load.

---

## ⚙️ Environment Variables

### Backend (.env)

```env
CLIENT_URL=http://localhost:5173
PORT=5000
```

### Frontend (.env)

```env
VITE_API_URL=http://localhost:5000
```

---

## 🧪 Testing

### Test Connection

1. Buka browser console
2. Check log: "✅ WebSocket connected: [socket-id]"
3. Jika error, check CORS settings dan URL

### Test Events

1. Buat pendaftaran baru sebagai pasien
2. Dokter harus menerima notifikasi instant
3. Update status pendaftaran sebagai dokter
4. Pasien harus menerima notifikasi instant

---

## 🐛 Troubleshooting

### Connection Failed

- Pastikan backend server running
- Check CORS settings di server.js
- Verify VITE_API_URL di .env frontend

### Events Tidak Terima

- Check apakah user sudah join room (check console log)
- Verify event listener terdaftar dengan benar
- Check browser console untuk error

### Browser Notifications Tidak Muncul

- Check permission di browser settings
- Pastikan browser support notifications
- Check console untuk permission errors

---

## 📚 Additional Resources

- [Socket.IO Documentation](https://socket.io/docs/v4/)
- [React Hook Best Practices](https://react.dev/reference/react)
- [Browser Notification API](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)

---

## 🎨 Customization

### Menambah Event Baru

#### 1. Backend

```javascript
// socketEmitter.js
const emitCustomEvent = (data) => {
  emitToRoom("target-room", "custom-event", data);
};
```

#### 2. Frontend

```javascript
// useRealtimeNotifications.js
const handleCustomEvent = (data) => {
  // Handle event
};

on("custom-event", handleCustomEvent);
```

---

**Happy Coding! 🚀**
