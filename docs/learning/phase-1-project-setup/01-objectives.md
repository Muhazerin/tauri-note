# Phase 1: Project Setup & Basic Structure - Learning Objectives

## Overview
In this phase, you will set up a Tauri v2 project with SvelteKit, TypeScript, and Vite. You'll learn the fundamental structure of a Tauri application and how the frontend and backend work together.

---

## Learning Objectives

After completing this phase, you should be able to:

### 1. Understand Tauri Project Structure
- [ ] Explain the purpose of the `src-tauri/` directory and its contents
- [ ] Understand the relationship between the frontend (SvelteKit) and backend (Rust)
- [ ] Navigate and modify `tauri.conf.json` configuration file
- [ ] Identify the role of `Cargo.toml` in the Rust backend

### 2. Set Up a Tauri v2 Development Environment
- [ ] Initialize a new Tauri v2 project with a frontend framework
- [ ] Configure build tools (Vite) for Tauri development
- [ ] Run the development server with hot-reload
- [ ] Understand the difference between `pnpm tauri dev` and `pnpm dev`

### 3. Understand the Tauri Build Process
- [ ] Know how Tauri bundles the frontend with the Rust backend
- [ ] Understand the difference between development and production builds
- [ ] Identify the output artifacts of a Tauri build

### 4. Configure the Development Environment
- [ ] Set up Tailwind CSS for styling
- [ ] Configure TypeScript with proper settings
- [ ] Understand Vite's role in the development workflow

### 5. Understand SvelteKit Basics
- [ ] Understand file-based routing in SvelteKit
- [ ] Know the purpose of `+page.svelte`, `+layout.svelte`, and `+layout.ts`
- [ ] Understand how to disable SSR for Tauri (static adapter)

---

## Key Concepts to Master

| Concept | Description |
|---------|-------------|
| `tauri.conf.json` | Main configuration file for Tauri app settings |
| `Cargo.toml` | Rust package manifest with dependencies |
| `svelte.config.js` | SvelteKit configuration file |
| Webview | The browser engine that renders the frontend |
| IPC | Inter-Process Communication between frontend and backend |
| Hot Reload | Automatic refresh when code changes |
| SSR | Server-Side Rendering (disabled for Tauri) |

---

## Success Criteria

You have successfully completed this phase when you can:

1. ✅ Run `pnpm tauri dev` and see the application window
2. ✅ Make a change to the Svelte code and see it update automatically
3. ✅ Explain what each file in `src-tauri/` does
4. ✅ Modify `tauri.conf.json` to change the window title
5. ✅ Use Tailwind CSS classes in your Svelte components
6. ✅ Understand why `+layout.ts` exports `prerender` and `ssr` settings