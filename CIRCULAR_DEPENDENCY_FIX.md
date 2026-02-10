# Circular Dependency Fix - Socket.IO

## ⚠️ Problem

Saat menjalankan server, muncul warning:

```
(node:11136) Warning: Accessing non-existent property 'io' of module exports inside circular dependency
```

## 🔍 Root Cause

**Circular Dependency Pattern:**

1. `server.js` membuat dan export `io` instance
2. `server.js` import `src/routes/pendaftaran.js`
3. `src/routes/pendaftaran.js` import `src/utils/socketEmitter.js`
4. `src/utils/socketEmitter.js` import `io` dari `server.js` ⚠️ **CIRCULAR!**

Node.js mendeteksi circular dependency dan memberikan warning.

## ✅ Solution

Menggunakan **Lazy Initialization Pattern** dengan Socket Instance Holder.

### Implementation

#### 1. Create Socket Instance Holder (`src/utils/socket.js`)

```javascript
let io = null;

const setSocketIO = (socketInstance) => {
  io = socketInstance;
};

const getSocketIO = () => {
  if (!io) {
    console.warn("Socket.IO instance not initialized yet");
  }
  return io;
};

module.exports = { setSocketIO, getSocketIO };
```

#### 2. Update Server (`server.js`)

```javascript
const { setSocketIO } = require("./src/utils/socket");

// Create Socket.IO instance
const io = new Server(server, {
  /* config */
});

// ⭐ Set socket instance BEFORE importing routes
setSocketIO(io);

// Now safe to import routes
const pendaftaranRoutes = require("./src/routes/pendaftaran");
```

#### 3. Update Socket Emitter (`src/utils/socketEmitter.js`)

```javascript
// Before (CIRCULAR):
// const { io } = require("../../server");

// After (FIXED):
const { getSocketIO } = require("./socket");

const emitToAll = (event, data) => {
  const io = getSocketIO();
  if (io) {
    io.emit(event, data);
  }
};
```

## 🎯 How It Works

### Old Flow (Circular)

```
server.js
  ↓ import
routes/pendaftaran.js
  ↓ import
utils/socketEmitter.js
  ↓ import io from
server.js ← CIRCULAR!
```

### New Flow (Fixed)

```
server.js
  ↓ create io
  ↓ setSocketIO(io)
utils/socket.js (stores io)
  ↓
server.js continues
  ↓ import
routes/pendaftaran.js
  ↓ import
utils/socketEmitter.js
  ↓ getSocketIO() from
utils/socket.js ✅ NO CIRCULAR!
```

## 📊 Benefits

1. ✅ **No more circular dependency warnings**
2. ✅ **Cleaner separation of concerns**
3. ✅ **Socket instance is globally accessible**
4. ✅ **Easy to mock for testing**
5. ✅ **Safe initialization order**

## 🧪 Verification

Run server and check:

```bash
npm run dev
```

**Before:**

```
Warning: Accessing non-existent property 'io' of module exports...
Server is running on port 5000
```

**After:**

```
Server is running on port 5000
WebSocket server is ready
```

✅ No warnings!

## 📝 Files Changed

1. **Created:** `src/utils/socket.js` - Socket instance holder
2. **Modified:** `server.js` - Import and set socket instance
3. **Modified:** `src/utils/socketEmitter.js` - Use getSocketIO()

## 💡 Pattern for Future

When you need to access Socket.IO instance from anywhere:

```javascript
// Import the getter
const { getSocketIO } = require("./path/to/utils/socket");

// Use it
function yourFunction() {
  const io = getSocketIO();
  if (io) {
    io.emit("your-event", data);
  }
}
```

This pattern avoids circular dependencies while keeping code clean and maintainable! 🚀

---

**Status:** ✅ **FIXED** - Circular dependency warning resolved!
