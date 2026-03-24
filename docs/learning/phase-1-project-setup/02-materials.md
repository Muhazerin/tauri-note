# Phase 1: Project Setup & Basic Structure - Learning Materials

## Table of Contents
1. [What is Tauri?](#what-is-tauri)
2. [Tauri v2 Architecture](#tauri-v2-architecture)
3. [Project Structure Explained](#project-structure-explained)
4. [Configuration Files](#configuration-files)
5. [SvelteKit Basics for Tauri](#sveltekit-basics-for-tauri)
6. [Development Workflow](#development-workflow)
7. [Hands-On Exercises](#hands-on-exercises)

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
│  │  (SvelteKit + Vite) │◄──►│       (Rust)            │ │
│  │                     │IPC │                         │ │
│  │  - Components       │    │  - Tauri Commands       │ │
│  │  - Routes           │    │  - File System          │ │
│  │  - Stores           │    │  - System APIs          │ │
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

After initializing a Tauri project with SvelteKit, you'll have this structure:

```
tauri-note/
├── src/                      # Frontend source code
│   ├── app.html              # HTML template
│   ├── routes/               # SvelteKit routes
│   │   ├── +page.svelte      # Home page component
│   │   ├── +layout.svelte    # Root layout (optional)
│   │   └── +layout.ts        # Layout config (SSR disabled)
│   └── lib/                  # Shared code
│       ├── components/       # Reusable Svelte components
│       ├── stores/           # Svelte stores
│       └── types/            # TypeScript types
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
├── static/                   # Static assets
├── package.json              # Node.js dependencies
├── svelte.config.js          # SvelteKit configuration
├── vite.config.js            # Vite configuration
└── tsconfig.json             # TypeScript configuration
```

### Frontend Files (`src/`)

| File | Purpose |
|------|---------|
| `app.html` | HTML template with `%sveltekit.head%` and `%sveltekit.body%` |
| `routes/+page.svelte` | Home page component |
| `routes/+layout.ts` | Disables SSR and enables prerendering for Tauri |
| `lib/` | Shared components, stores, and utilities |

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
    "frontendDist": "../build",
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

### svelte.config.js

SvelteKit configuration for Tauri:

```javascript
import adapter from '@sveltejs/adapter-static';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
  preprocess: vitePreprocess(),
  kit: {
    adapter: adapter({
      fallback: 'index.html'
    })
  }
};

export default config;
```

**Important**: We use `adapter-static` because Tauri loads static files, not a Node.js server.

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

### vite.config.js

Vite configuration for the frontend:

```javascript
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [sveltekit()],
  clearScreen: false,
  server: {
    port: 5173,
    strictPort: true,
  },
  envPrefix: ['VITE_', 'TAURI_'],
});
```

---

## SvelteKit Basics for Tauri

### Why SvelteKit?

SvelteKit provides:
- **File-based routing**: Create routes by adding files to `src/routes/`
- **Component-based architecture**: Reusable `.svelte` components
- **Built-in stores**: Reactive state management
- **TypeScript support**: Full type safety
- **Vite integration**: Fast development with HMR

### Disabling SSR for Tauri

Tauri loads static files, so we need to disable Server-Side Rendering. Create `src/routes/+layout.ts`:

```typescript
// This disables SSR for the entire app
export const prerender = true;
export const ssr = false;
```

### File-Based Routing

| File | Route |
|------|-------|
| `src/routes/+page.svelte` | `/` |
| `src/routes/about/+page.svelte` | `/about` |
| `src/routes/notes/+page.svelte` | `/notes` |
| `src/routes/notes/[id]/+page.svelte` | `/notes/:id` |

### Basic Svelte Component

```svelte
<script lang="ts">
  // TypeScript code here
  let count = $state(0);
  
  function increment() {
    count++;
  }
</script>

<button onclick={increment}>
  Count: {count}
</button>

<style>
  button {
    padding: 1rem;
  }
</style>
```

### Svelte Stores for State Management

```typescript
// src/lib/stores/counter.ts
import { writable } from 'svelte/store';

export const count = writable(0);
```

```svelte
<script lang="ts">
  import { count } from '$lib/stores/counter';
</script>

<button onclick={() => $count++}>
  Count: {$count}
</button>
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
3. Open `src/routes/+page.svelte` and understand the Svelte component

### Exercise 2: Modify Window Settings
1. Change the window title in `tauri.conf.json`
2. Change the window size to 1024x768
3. Run `pnpm tauri dev` to see your changes

### Exercise 3: Add Tailwind CSS
1. Install Tailwind CSS: `pnpm add -D tailwindcss postcss autoprefixer`
2. Initialize Tailwind: `npx tailwindcss init -p`
3. Configure `tailwind.config.js`:
   ```javascript
   export default {
     content: ['./src/**/*.{html,js,svelte,ts}'],
     theme: { extend: {} },
     plugins: [],
   }
   ```
4. Create `src/app.css` with Tailwind directives:
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```
5. Import in `src/routes/+layout.svelte`:
   ```svelte
   <script>
     import '../app.css';
   </script>
   
   <slot />
   ```

### Exercise 4: Understand Hot Reload
1. Run `pnpm tauri dev`
2. Modify text in `src/routes/+page.svelte`
3. Observe the automatic update in the window
4. Note: Rust changes require recompilation (automatic but slower)

### Exercise 5: Create a New Route
1. Create `src/routes/about/+page.svelte`:
   ```svelte
   <h1>About</h1>
   <p>This is the about page.</p>
   <a href="/">Back to Home</a>
   ```
2. Add a link from the home page to `/about`
3. Test navigation in the Tauri window

---

## Additional Resources

- [Tauri v2 Documentation](https://v2.tauri.app/)
- [Tauri Configuration Reference](https://v2.tauri.app/reference/config/)
- [Vite Documentation](https://vitejs.dev/)
- [Svelte Documentation](https://svelte.dev/docs)
- [SvelteKit Documentation](https://svelte.dev/docs/kit)
- [Tailwind CSS Documentation](https://tailwindcss.com/)

---

## Summary

In this phase, you learned:
- ✅ What Tauri is and how it compares to Electron
- ✅ The architecture of a Tauri v2 application
- ✅ The purpose of each file in the project structure
- ✅ How to configure Tauri via `tauri.conf.json`
- ✅ SvelteKit basics: routing, components, and stores
- ✅ Why SSR must be disabled for Tauri
- ✅ The development workflow with hot-reload

Next: [Phase 2 - Core Note Functionality + IPC](../phase-2-ipc/01-objectives.md)