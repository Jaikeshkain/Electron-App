# Electron Frontend (Renderer) to Backend (Main) Communication

This document explains how communication flows between the **Renderer** process (React UI) and the **Main** process (Electron backend) in your Electron app. It uses IPC (Inter-Process Communication) channels to facilitate message passing.

---

## 1. Core Concepts

### Main Process:

- Controls application lifecycle.
- Manages native windows.
- Runs Node.js environment.
- Handles OS-level integrations (e.g. filesystem, system tray, hardware).

### Renderer Process:

- Runs UI layer (your React frontend).
- Cannot access Node.js or OS APIs directly (for security).
- Communicates with Main using **IPC (Inter-Process Communication)**.

### Preload Script:

- A secure bridge between Renderer and Main.
- Uses `contextBridge` to expose limited, secure APIs to the frontend.

---

## 2. Communication Flow: Renderer to Main (e.g., Login)

### Step-by-Step:

**Renderer Code:**

```ts
const response = await window.electron.DB("_Login", {
  userName: credential.userName,
  password: credential.password,
});
```

- The frontend calls a `window.electron.DB` method.
- This is exposed via the **Preload Script** using `contextBridge`.

**Preload Script:**

```ts
contextBridge.exposeInMainWorld("electron", {
  DB: (key, payload) => ipcRenderer.invoke(key, payload),
});
```

- `DB()` is a custom wrapper calling `ipcRenderer.invoke`.

**ipcRenderer.invoke:**

- Sends the message to the main process.
- Waits for a response.

**Main Process:**

```ts
ipcMain.handle("_Login", async (event, payload) => {
  const { userName, password } = payload;
  // Verify credentials and return result
});
```

- Listens to the `_Login` key using `ipcMain.handle`.
- Processes the request and returns a value.

---

## 3. Communication Flow: Main to Renderer

### Example: After login/logout, update menu:

```ts
ipcMainOn("Login_Successfull", () => {
  createMenu(mainWindow);
});

ipcMainOn("Logout_Successfull", () => {
  createMenu(mainWindow);
  hardwareWindow?.close();
});
```

- Uses `ipcMain.on` (likely wrapped in `ipcMainOn`) to listen for Renderer-triggered events.
- Updates app menu or closes secondary windows.

---

## 4. Key Files Involved

### main.ts

- Initializes and manages main/secondary windows.
- Handles IPC with `ipcMain.handle`, `ipcMain.on`.
- Sets up Content Security Policy and logging.

### preload.ts

- Uses `contextBridge` to expose safe APIs to the frontend.
- Acts as the only secure link between Renderer and Main.

### renderer (React App)

- Uses `window.electron.DB(...)` for DB/login calls.
- Shows/hides UI based on backend responses.

---

## 5. Security Notes

- `contextIsolation: true` ensures no direct access to Node.js in frontend.
- `sandbox: true` locks down unsafe APIs.
- `contextBridge` selectively exposes only allowed methods.

---

## 6. Visual Summary

```text
Renderer (React)
  ↓ window.electron.DB("_Login", payload)
Preload Script
  ↓ ipcRenderer.invoke("_Login", payload)
Main Process
  ↓ ipcMain.handle("_Login", (event, payload) => { ... })
Returns response → back to Renderer (async)
```

---

Let me know if you’d like to add flow diagrams, visuals, or include reverse communication (Main to Renderer via `webContents.send`).

