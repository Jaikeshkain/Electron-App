# Electron Frontend (Renderer) and Backend (Main) Communication Explained

This document explains how the frontend (renderer process) and backend (main process) communicate in an Electron app using preload scripts, IPC (Inter-Process Communication), and the context bridge.

---

## 📌 Key Components

### 1. **Main Process**

- Runs in Node.js.
- Controls windows and handles OS-level operations.
- Entry point: `main.ts` or `main.js`.

### 2. **Renderer Process**

- Runs your frontend code (React, HTML, etc).
- Sandboxed from Node.js for security.

### 3. **Preload Script**

- Runs in a secure context before the renderer.
- Has access to both Node.js and the browser.
- Safely exposes APIs using `contextBridge`.

### 4. **IPC (Inter-Process Communication)**

- Electron's messaging system.
- Allows renderer to send/receive messages from the main process.

---

## 🔁 Communication Flow

### 1. **Renderer to Main (Frontend to Backend)**

#### Step-by-step:

1. Renderer calls `window.electron.send("channel", payload)`
2. Preload script forwards it using `ipcRenderer.send`
3. Main process listens via `ipcMain.on("channel", handler)`

#### Example:

```tsx
// In Renderer (React)
window.electron.send("saveUser", { name: "Alice" });
```

```ts
// In Preload
contextBridge.exposeInMainWorld("electron", {
  send: (channel, data) => ipcRenderer.send(channel, data)
});
```

```ts
// In Main Process
ipcMain.on("saveUser", (event, data) => {
  console.log("Received user:", data);
});
```

---

### 2. **Main to Renderer (Backend to Frontend)**

#### Step-by-step:

1. Main process sends message with `webContents.send("channel", payload)`
2. Preload exposes a listener using `ipcRenderer.on`
3. Renderer listens via `window.electron.on("channel", callback)`

#### Example:

```ts
// In Main
mainWindow.webContents.send("openModal", { type: "userCreate" });
```

```ts
// In Preload
contextBridge.exposeInMainWorld("electron", {
  on: (channel, callback) => ipcRenderer.on(channel, (_e, data) => callback(data))
});
```

```tsx
// In Renderer (React)
useEffect(() => {
  window.electron.on("openModal", (payload) => {
    console.log("Modal data:", payload);
  });
}, []);
```

---

## 📊 Architecture Diagram

```text
[Renderer (React)]
     ↓ send("channel")
[Preload Script - ipcRenderer.send]
     ↓
[Main Process - ipcMain.on]
     ↑
[Main Process - webContents.send]
     ↑
[Preload Script - ipcRenderer.on]
     ↑
[Renderer - window.electron.on("channel")]  
```

---

## 🔐 Why Use Preload and ContextBridge?

- Prevents direct access to Node.js from the UI.
- Improves app security.
- Gives you control over what features are exposed.

---

## ✅ Summary Table

| Task                      | Renderer                    | Preload                           | Main                    |
| ------------------------- | --------------------------- | --------------------------------- | ----------------------- |
| Send message to main      | `window.electron.send(...)` | `ipcRenderer.send(...)`           | `ipcMain.on(...)`       |
| Receive message from main | `window.electron.on(...)`   | `ipcRenderer.on(...)`             | `webContents.send(...)` |
| Secure API exposure       | ❌                           | `contextBridge.exposeInMainWorld` | ❌                       |

---

Let me know if you want code templates, architecture diagrams, or project scaffolding (with working examples) for this setup.
