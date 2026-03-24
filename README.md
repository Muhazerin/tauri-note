# Tauri Note

A note-taking application built with Tauri v2, SvelteKit, and Rust. This project serves as a learning resource for Tauri desktop application development.

## 🚀 Tech Stack

- **Frontend**: [SvelteKit](https://svelte.dev/docs/kit) with TypeScript
- **Backend**: [Rust](https://www.rust-lang.org/) with [Tauri v2](https://v2.tauri.app/)
- **Styling**: [Tailwind CSS](https://tailwindcss.com/)
- **Build Tool**: [Vite](https://vitejs.dev/)

## 📁 Project Structure

```
tauri-note/
├── src/                    # SvelteKit frontend
│   ├── routes/             # SvelteKit routes
│   ├── lib/                # Shared components, stores, utilities
│   └── app.html            # HTML template
├── src-tauri/              # Rust backend
│   ├── src/                # Rust source code
│   ├── Cargo.toml          # Rust dependencies
│   └── tauri.conf.json     # Tauri configuration
├── static/                 # Static assets
└── docs/                   # Learning materials
```

## 🛠️ Prerequisites

- [Node.js](https://nodejs.org/) (v18+)
- [Rust](https://www.rust-lang.org/tools/install)
- [pnpm](https://pnpm.io/) (recommended) or npm
- System dependencies for Tauri - see [Tauri Prerequisites](https://v2.tauri.app/start/prerequisites/)

## 🏃 Getting Started

1. **Clone the repository**
   ```bash
   git clone https://github.com/Muhazerin/tauri-note.git
   cd tauri-note
   ```

2. **Install dependencies**
   ```bash
   cd tauri-note
   pnpm install
   ```

3. **Run in development mode**
   ```bash
   pnpm tauri dev
   ```

4. **Build for production**
   ```bash
   pnpm tauri build
   ```

## 📚 Learning Materials

This project includes comprehensive learning materials organized into 8 phases:

| Phase | Topic | Description |
|-------|-------|-------------|
| 1 | [Project Setup](./docs/learning/phase-1-project-setup/) | Initialize Tauri v2 with SvelteKit |
| 2 | [IPC & Commands](./docs/learning/phase-2-ipc/) | Create Tauri commands, use invoke() |
| 3 | [File System](./docs/learning/phase-3-file-system/) | Persist data with JSON files |
| 4 | [Native Menus](./docs/learning/phase-4-native-menus/) | Create menus, keyboard shortcuts |
| 5 | [System Tray](./docs/learning/phase-5-system-tray/) | Add tray icon and menu |
| 6 | [Custom Window](./docs/learning/phase-6-custom-window/) | Custom titlebar, window controls |
| 7 | [Auto-Updates](./docs/learning/phase-7-auto-updates/) | Configure updater, GitHub releases |
| 8 | [Database Migration](./docs/learning/phase-8-database-migration/) | Migrate JSON to document database |

Start learning: [📖 Learning Materials](./docs/learning/README.md)

## 🧪 Development

### Commands

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start SvelteKit dev server only |
| `pnpm tauri dev` | Start Tauri app with hot-reload |
| `pnpm build` | Build SvelteKit frontend |
| `pnpm tauri build` | Build production Tauri app |
| `pnpm tauri info` | Show environment information |

### Project Conventions

- **Svelte components**: `PascalCase.svelte`
- **TypeScript utilities**: `camelCase.ts`
- **Rust modules**: `snake_case.rs`
- **Tauri commands**: `snake_case` (e.g., `get_all_notes`)

See [Development Standards](./.clinerules/development-standards.md) for more details.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🔗 Resources

- [Tauri v2 Documentation](https://v2.tauri.app/)
- [SvelteKit Documentation](https://svelte.dev/docs/kit)
- [Svelte Documentation](https://svelte.dev/docs)
- [Rust Book](https://doc.rust-lang.org/book/)