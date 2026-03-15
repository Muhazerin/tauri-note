# Phase 3: JSON File Storage + File System Access - Learning Objectives

## Overview
In this phase, you will learn how to persist notes to the file system using JSON files. You'll understand Tauri's file system plugin, permission system, and how to work with app directories.

---

## Learning Objectives

After completing this phase, you should be able to:

### 1. Use Tauri's File System Plugin
- [ ] Install and configure `@tauri-apps/plugin-fs`
- [ ] Understand Tauri's permission system for file access
- [ ] Configure capabilities in `tauri.conf.json`
- [ ] Use the plugin in both Rust and JavaScript

### 2. Work with App Directories
- [ ] Access standard app directories (data, config, cache)
- [ ] Understand cross-platform path handling
- [ ] Create and manage app-specific storage locations
- [ ] Use `BaseDirectory` enum for portable paths

### 3. Implement Persistent Storage
- [ ] Read and write JSON files in Rust
- [ ] Handle file I/O errors gracefully
- [ ] Implement auto-save functionality
- [ ] Load notes on application startup

### 4. Understand Tauri's Security Model
- [ ] Configure file system permissions
- [ ] Understand the principle of least privilege
- [ ] Scope file access to specific directories
- [ ] Avoid common security pitfalls

---

## Key Concepts to Master

| Concept | Description |
|---------|-------------|
| `tauri-plugin-fs` | Plugin for file system operations |
| `BaseDirectory` | Enum for standard app directories |
| Capabilities | Permission configuration in Tauri v2 |
| `std::fs` | Rust standard library file operations |
| App Data Directory | Platform-specific storage location |

---

## Success Criteria

You have successfully completed this phase when you can:

1. ✅ Save notes to a JSON file in the app data directory
2. ✅ Load notes from the JSON file on app startup
3. ✅ Handle file not found errors gracefully
4. ✅ Understand and configure file system permissions
5. ✅ Notes persist between app restarts