# Phase 1: Project Setup & Basic Structure - Quiz

Test your understanding of Tauri project setup and structure.

---

## Multiple Choice Questions

### Question 1
What is the main advantage of Tauri over Electron in terms of bundle size?

- A) Tauri apps are typically 150MB
- B) Tauri apps are typically 600KB - 3MB
- C) Tauri apps are the same size as Electron apps
- D) Tauri apps are larger but faster

<details>
<summary>Show Answer</summary>

**B) Tauri apps are typically 600KB - 3MB**

Tauri uses the system's native webview instead of bundling Chromium, resulting in much smaller app sizes compared to Electron's ~150MB.
</details>

---

### Question 2
Which directory contains the Rust backend code in a Tauri project?

- A) `src/`
- B) `backend/`
- C) `src-tauri/`
- D) `rust/`

<details>
<summary>Show Answer</summary>

**C) `src-tauri/`**

The `src-tauri/` directory contains all Rust backend code, including `main.rs`, `lib.rs`, `Cargo.toml`, and `tauri.conf.json`.
</details>

---

### Question 3
What file is the main configuration file for a Tauri application?

- A) `package.json`
- B) `Cargo.toml`
- C) `vite.config.ts`
- D) `tauri.conf.json`

<details>
<summary>Show Answer</summary>

**D) `tauri.conf.json`**

`tauri.conf.json` is the main configuration file that controls app settings like window size, title, build commands, and security settings.
</details>

---

### Question 4
What command starts the Tauri development server with hot-reload?

- A) `pnpm dev`
- B) `pnpm start`
- C) `pnpm tauri dev`
- D) `pnpm tauri start`

<details>
<summary>Show Answer</summary>

**C) `pnpm tauri dev`**

`pnpm tauri dev` starts both the frontend dev server and the Tauri application with hot-reload enabled.
</details>

---

### Question 5
What does IPC stand for in the context of Tauri?

- A) Internal Process Control
- B) Inter-Process Communication
- C) Integrated Program Compiler
- D) Interface Protocol Connection

<details>
<summary>Show Answer</summary>

**B) Inter-Process Communication**

IPC (Inter-Process Communication) is the mechanism that allows the frontend (JavaScript) to communicate with the backend (Rust) in Tauri.
</details>

---

### Question 6
Which webview does Tauri use on Windows?

- A) WebKit
- B) Chromium
- C) WebView2
- D) Gecko

<details>
<summary>Show Answer</summary>

**C) WebView2**

Tauri uses WebView2 (Edge-based) on Windows, WebKit on macOS and Linux.
</details>

---

### Question 7
What is the purpose of `Cargo.toml` in a Tauri project?

- A) Configure the frontend build
- B) Define Rust dependencies and package metadata
- C) Configure the Tauri window
- D) Define TypeScript types

<details>
<summary>Show Answer</summary>

**B) Define Rust dependencies and package metadata**

`Cargo.toml` is the Rust package manifest that defines dependencies, package name, version, and build settings for the Rust backend.
</details>

---

### Question 8
What happens when you modify a React component while `pnpm tauri dev` is running?

- A) The app crashes
- B) You need to restart the dev server
- C) The change is hot-reloaded automatically
- D) The Rust code recompiles

<details>
<summary>Show Answer</summary>

**C) The change is hot-reloaded automatically**

Frontend changes are hot-reloaded instantly thanks to Vite's HMR (Hot Module Replacement). Rust changes require recompilation but this also happens automatically.
</details>

---

## Fill in the Blanks

### Question 9
Complete the following: The `identifier` field in `tauri.conf.json` should be in __________ format (e.g., `com.example.app`).

<details>
<summary>Show Answer</summary>

**reverse domain** (or reverse DNS)

The identifier uses reverse domain notation to ensure uniqueness across applications.
</details>

---

### Question 10
The frontend build output is placed in the __________ directory, which Tauri bundles into the final application.

<details>
<summary>Show Answer</summary>

**dist** (or `../dist` relative to `src-tauri/`)

Vite builds the frontend into the `dist/` directory, which is then bundled with the Rust binary.
</details>

---

## True or False

### Question 11
True or False: Tauri bundles its own browser engine like Electron does.

<details>
<summary>Show Answer</summary>

**False**

Tauri uses the system's native webview (WebKit on macOS/Linux, WebView2 on Windows) instead of bundling a browser engine, which is why Tauri apps are much smaller.
</details>

---

### Question 12
True or False: You can use Vue or Svelte instead of React with Tauri.

<details>
<summary>Show Answer</summary>

**True**

Tauri is frontend-agnostic. You can use React, Vue, Svelte, Angular, or even vanilla JavaScript/TypeScript.
</details>

---

## Short Answer

### Question 13
Explain the difference between `pnpm dev` and `pnpm tauri dev`.

<details>
<summary>Show Answer</summary>

- `pnpm dev`: Starts only the Vite frontend development server. The app runs in a browser without Tauri features.
- `pnpm tauri dev`: Starts both the Vite dev server AND the Tauri application. This opens the native window and enables all Tauri features like IPC, file system access, etc.

Use `pnpm dev` for quick frontend-only development, and `pnpm tauri dev` when you need to test Tauri-specific functionality.
</details>

---

### Question 14
What are the three main sections you would typically configure in `tauri.conf.json`?

<details>
<summary>Show Answer</summary>

1. **`build`**: Frontend build commands and paths (`beforeDevCommand`, `beforeBuildCommand`, `frontendDist`, `devUrl`)
2. **`app`**: Application settings including window configuration (`windows`), security settings (`security`)
3. **`bundle`**: Packaging options for distribution (`icon`, `identifier`, platform-specific settings)

Other sections include `productName`, `version`, and `identifier` at the root level.
</details>

---

### Question 15
Why is Tauri considered more secure than Electron by default?

<details>
<summary>Show Answer</summary>

Tauri is more secure because:

1. **Capability-based permissions**: Apps must explicitly request permissions for system access (file system, network, etc.)
2. **Rust's memory safety**: The backend is written in Rust, which prevents common memory vulnerabilities
3. **No Node.js**: Unlike Electron, there's no Node.js runtime with full system access in the renderer
4. **Principle of least privilege**: By default, the frontend has minimal access to system resources
5. **CSP (Content Security Policy)**: Built-in support for restricting what the webview can load
</details>

---

## Practical Exercise

### Question 16
Given the following `tauri.conf.json` snippet, identify what changes you would make to:
1. Change the window title to "My Notes App"
2. Set the initial window size to 1024x768
3. Make the window non-resizable

```json
{
  "app": {
    "windows": [
      {
        "title": "Tauri App",
        "width": 800,
        "height": 600,
        "resizable": true
      }
    ]
  }
}
```

<details>
<summary>Show Answer</summary>

```json
{
  "app": {
    "windows": [
      {
        "title": "My Notes App",
        "width": 1024,
        "height": 768,
        "resizable": false
      }
    ]
  }
}
```

Changes made:
1. `"title": "Tauri App"` → `"title": "My Notes App"`
2. `"width": 800` → `"width": 1024` and `"height": 600` → `"height": 768`
3. `"resizable": true` → `"resizable": false`
</details>

---

## Score Yourself

| Score | Rating |
|-------|--------|
| 14-16 | ⭐⭐⭐ Excellent! Ready for Phase 2 |
| 10-13 | ⭐⭐ Good! Review the materials you missed |
| 6-9 | ⭐ Fair. Re-read the learning materials |
| 0-5 | Need more practice. Start from the beginning |

---

## Next Steps

If you scored well, proceed to [Phase 2: Core Note Functionality + IPC](../phase-2-ipc/01-objectives.md)

If you need more practice, review:
- [Phase 1 Learning Materials](./02-materials.md)
- [Tauri v2 Documentation](https://v2.tauri.app/)