# Phase 1: Project Setup & Basic Structure - Learning Materials

## Table of Contents
1. [What is Tauri?](#what-is-tauri)
2. [Tauri v2 Architecture](#tauri-v2-architecture)
3. [Project Structure Explained](#project-structure-explained)
4. [Configuration Files](#configuration-files)
5. [Development Workflow](#development-workflow)
6. [Hands-On Exercises](#hands-on-exercises)

---

## What is Tauri?

Tauri is a framework for building tiny, fast binaries for all major desktop platforms. It allows you to use web technologies (HTML, CSS, JavaScript) for the frontend while leveraging Rust for the backend.

### Key Benefits
- **Small Bundle Size**: Tauri apps are typically 600KB - 3MB (vs Electron's 150MB+)
- **Security First**: Rust's memory safety + capability-based permissions
- **Native Performance**: Rust backend for CPU-intensive tasks
- **Cross-Platform**: Windows, macOS, Linux (and mobile in v2)
- **Web Tech Frontend**: Use React, Vue, Svelte, or vanilla JS

### Tauri vs Electron

| Feature | Tauri | Electron |
|---------|-------|----------|
| Bundle Size | ~3MB | ~150MB |
| Memory Usage | Low | High |
| Backend Language | Rust | Node.js |
| Security Model | Capability-based | Full Node.js access |
| Startup Time | Fast | Slower |

---

## Tauri v2 Architecture

### Overview
```
┌─────────────────────────────────────────────────────────┐
│                    Tauri Application                     │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────────┐    ┌─────────────────────────┐ │
│  │     Frontend        │    │       Backend           │ │
│  │  (React + Vite)     │◄──►│       (Rust)            │ │
│  │                     │IPC │                         │ │
│  │  - Components       │    │  - Tauri Commands       │ │
│  │  - Styles           │    │  - File System          │ │
│  │  - State            │    │  - System APIs          │ │
│  └─────────────────────┘    └─────────────────────────┘ │
│              │                         │                 │
│              ▼                         ▼                 │
│  ┌─────────────────────┐    ┌─────────────────────────┐ │
│  │      Webview        │    │     Native APIs         │ │
│  │  (System Browser)   │    │  (OS-specific)          │ │
│  └─────────────────────┘    └─────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Key Components

1. **Frontend (Webview)**
   - Renders your UI using the system's webview (WebKit on macOS/Linux, WebView2 on Windows)
   - No bundled browser = smaller app size
   - Full access to modern web APIs

2. **Backend (Rust Core)**
   - Handles system operations, file I/O, native features
   - Exposes functionality via Tauri Commands
   - Manages window, tray, menus

3. **IPC (Inter-Process Communication)**
   - Bridge between frontend and backend
   - `invoke()` calls from JS to Rust
   - Events for Rust to JS communication

---

## Project Structure Explained

After initializing a Tauri project, you'll have this structure:

```
tauri-note/
├── src/                      # Frontend source code
│   ├── App.tsx               # Main React component
│   ├── main.tsx              # React entry point
│   ├── index.css             # Global styles
│   └── vite-env.d.ts         # Vite type definitions
│
├── src-tauri/                # Rust backend
│   ├── src/
│   │   ├── main.rs           # Rust entry point
│   │   └── lib.rs            # Library exports
│   ├── Cargo.toml            # Rust dependencies
│   ├── tauri.conf.json       # Tauri configuration
│   ├── capabilities/         # Permission definitions
│   └── icons/                # App icons
│
├── public/                   # Static assets
├── index.html                # HTML entry point
├── package.json              # Node.js dependencies
├── vite.config.ts            # Vite configuration
└── tsconfig.json             # TypeScript configuration
```

### Frontend Files (`src/`)

| File | Purpose |
|------|---------|
| `main.tsx` | React app entry point, renders `<App />` |
| `App.tsx` | Main application component |
| `index.css` | Global styles (Tailwind imports go here) |

### Backend Files (`src-tauri/`)

| File | Purpose |
|------|---------|
| `main.rs` | Application entry point, window creation |
| `lib.rs` | Tauri command definitions, plugin setup |
| `Cargo.toml` | Rust dependencies and metadata |
| `tauri.conf.json` | App configuration (name, window, permissions) |
| `capabilities/` | Security permissions for the app |

---

## Configuration Files

### tauri.conf.json

This is the main configuration file for your Tauri app:

```json
{
  "$schema": "https://schema.tauri.app/config/2",
  "productName": "tauri-note",
  "version": "0.1.0",
  "identifier": "com.tauri-note.app",
  "build": {
    "frontendDist": "../dist",
    "devUrl": "http://localhost:5173",
    "beforeDevCommand": "pnpm dev",
    "beforeBuildCommand": "pnpm build"
  },
  "app": {
    "windows": [
      {
        "title": "Tauri Note",
        "width": 800,
        "height": 600,
        "resizable": true,
        "fullscreen": false
      }
    ],
    "security": {
      "csp": null
    }
  },
  "bundle": {
    "active": true,
    "icon": ["icons/icon.png"]
  }
}
```

#### Key Sections:

| Section | Purpose |
|---------|---------|
| `productName` | App name shown to users |
| `identifier` | Unique app ID (reverse domain) |
| `build` | Frontend build commands and paths |
| `app.windows` | Window configuration (size, title, etc.) |
| `bundle` | Packaging options for distribution |

### Cargo.toml

Rust package manifest:

```toml
[package]
name = "tauri-note"
version = "0.1.0"
edition = "2021"

[dependencies]
tauri = { version = "2", features = [] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"

[build-dependencies]
tauri-build = { version = "2", features = [] }
```

### vite.config.ts

Vite configuration for the frontend:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  clearScreen: false,
  server: {
    port: 5173,
    strictPort: true,
  },
  envPrefix: ['VITE_', 'TAURI_'],
});
```

---

## Development Workflow

### Commands

| Command | Description |
|---------|-------------|
| `pnpm tauri dev` | Start development with hot-reload |
| `pnpm tauri build` | Build production app |
| `pnpm dev` | Start only the frontend (no Tauri) |
| `pnpm tauri info` | Show environment information |

### Development Mode (`pnpm tauri dev`)

1. Starts Vite dev server (frontend)
2. Compiles Rust code
3. Opens the Tauri window pointing to dev server
4. Hot-reloads frontend changes
5. Recompiles Rust on backend changes

### Build Process (`pnpm tauri build`)

1. Builds frontend with Vite (`pnpm build`)
2. Compiles Rust in release mode
3. Bundles everything into platform-specific installer
4. Output in `src-tauri/target/release/bundle/`

---

## Hands-On Exercises

### Exercise 1: Explore the Project Structure
After initializing the project:
1. Open `src-tauri/tauri.conf.json` and identify the window settings
2. Open `src-tauri/src/main.rs` and find where the app is created
3. Open `src/App.tsx` and understand the React component

### Exercise 2: Modify Window Settings
1. Change the window title in `tauri.conf.json`
2. Change the window size to 1024x768
3. Run `pnpm tauri dev` to see your changes

### Exercise 3: Add Tailwind CSS
1. Install Tailwind CSS dependencies
2. Configure `tailwind.config.js`
3. Add Tailwind directives to `index.css`
4. Use Tailwind classes in `App.tsx`

### Exercise 4: Understand Hot Reload
1. Run `pnpm tauri dev`
2. Modify text in `App.tsx`
3. Observe the automatic update in the window
4. Note: Rust changes require recompilation (automatic but slower)

---

## Additional Resources

- [Tauri v2 Documentation](https://v2.tauri.app/)
- [Tauri Configuration Reference](https://v2.tauri.app/reference/config/)
- [Vite Documentation](https://vitejs.dev/)
- [React Documentation](https://react.dev/)
- [Tailwind CSS Documentation](https://tailwindcss.com/)

---

## Summary

In this phase, you learned:
- ✅ What Tauri is and how it compares to Electron
- ✅ The architecture of a Tauri v2 application
- ✅ The purpose of each file in the project structure
- ✅ How to configure Tauri via `tauri.conf.json`
- ✅ The development workflow with hot-reload

Next: [Phase 2 - Core Note Functionality + IPC](../phase-2-ipc/01-objectives.md)