# Phase 2: Core Note Functionality + IPC - Learning Objectives

## Overview
In this phase, you will learn how to create Tauri commands in Rust and call them from the Svelte frontend using IPC (Inter-Process Communication). You'll build the core CRUD functionality for the note app.

---

## Learning Objectives

After completing this phase, you should be able to:

### 1. Implement Tauri Commands
- [ ] Create Rust functions with the `#[tauri::command]` attribute
- [ ] Understand how commands are registered in the Tauri app builder
- [ ] Handle different return types (Result, Option, custom structs)
- [ ] Use async commands for non-blocking operations

### 2. Call Rust from JavaScript
- [ ] Use the `invoke()` function to call Tauri commands
- [ ] Pass arguments from JavaScript to Rust
- [ ] Handle async responses with async/await
- [ ] Implement proper error handling with try/catch

### 3. Serialize Data Between Frontend and Backend
- [ ] Use `serde` for JSON serialization in Rust
- [ ] Understand how TypeScript types map to Rust structs
- [ ] Handle complex data structures across the IPC boundary
- [ ] Debug serialization issues

### 4. Manage Application State
- [ ] Use Tauri's state management for shared data
- [ ] Understand thread-safe state with `Mutex`
- [ ] Access state in Tauri commands
- [ ] Update state from multiple commands

---

## Key Concepts to Master

| Concept | Description |
|---------|-------------|
| `#[tauri::command]` | Macro that exposes a Rust function to the frontend |
| `invoke()` | JavaScript function to call Tauri commands |
| `serde` | Rust library for serialization/deserialization |
| `tauri::State` | Managed application state accessible in commands |
| `Mutex` | Thread-safe wrapper for mutable state |

---

## Success Criteria

You have successfully completed this phase when you can:

1. ✅ Create a Tauri command that returns a string
2. ✅ Call the command from Svelte and display the result
3. ✅ Pass parameters from Svelte to a Rust command
4. ✅ Create a Note struct that serializes to/from JSON
5. ✅ Implement CRUD operations (Create, Read, Update, Delete) for notes
6. ✅ Store notes in application state
7. ✅ Use Svelte stores for frontend state management
