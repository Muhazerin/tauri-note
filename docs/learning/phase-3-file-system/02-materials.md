# Phase 3: JSON File Storage + File System Access - Learning Materials

## Table of Contents
1. [Tauri File System Plugin](#tauri-file-system-plugin)
2. [App Directories](#app-directories)
3. [Configuring Permissions](#configuring-permissions)
4. [Reading and Writing Files](#reading-and-writing-files)
5. [Implementing Persistent Storage](#implementing-persistent-storage)
6. [Hands-On Exercises](#hands-on-exercises)

---

## Tauri File System Plugin

### Why a Plugin?

In Tauri v2, file system access is provided through a plugin rather than being built-in. This follows the principle of least privilege - your app only gets the capabilities it explicitly requests.

### Installation

**1. Add the Rust dependency:**

```bash
cd src-tauri
cargo add tauri-plugin-fs
```

Or add to `Cargo.toml`:
```toml
[dependencies]
tauri-plugin-fs = "2"
```

**2. Add the JavaScript package:**

```bash
pnpm add @tauri-apps/plugin-fs
```

**3. Register the plugin in Rust:**

```rust
// src-tauri/src/lib.rs
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_fs::init())  // Add this line
        .invoke_handler(tauri::generate_handler![...])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

---

## App Directories

### Standard Directories

Tauri provides access to standard application directories that work across platforms:

| Directory | Windows | macOS | Linux |
|-----------|---------|-------|-------|
| App Data | `%APPDATA%\{app}` | `~/Library/Application Support/{app}` | `~/.local/share/{app}` |
| App Config | `%APPDATA%\{app}` | `~/Library/Application Support/{app}` | `~/.config/{app}` |
| App Cache | `%LOCALAPPDATA%\{app}` | `~/Library/Caches/{app}` | `~/.cache/{app}` |

### BaseDirectory Enum

In Rust, use the `BaseDirectory` enum:

```rust
use tauri_plugin_fs::BaseDirectory;

// Available directories
BaseDirectory::AppData      // App-specific data
BaseDirectory::AppConfig    // App configuration
BaseDirectory::AppCache     // Temporary cache
BaseDirectory::Desktop      // User's desktop
BaseDirectory::Document     // User's documents
BaseDirectory::Download     // User's downloads
// ... and more
```

### Getting the App Data Path

```rust
use tauri::Manager;

#[tauri::command]
fn get_data_path(app: tauri::AppHandle) -> Result<String, String> {
    let path = app.path()
        .app_data_dir()
        .map_err(|e| e.to_string())?;
    
    Ok(path.to_string_lossy().to_string())
}
```

---

## Configuring Permissions

### Capabilities in Tauri v2

Tauri v2 uses a capability-based permission system. You must explicitly grant permissions for file system access.

**1. Create a capability file:**

```json
// src-tauri/capabilities/default.json
{
  "$schema": "https://schemas.tauri.app/config/2/capability",
  "identifier": "default",
  "description": "Default capabilities for the app",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:default",
    "fs:allow-app-data-dir-access",
    "fs:allow-read",
    "fs:allow-write"
  ]
}
```

**2. Reference in tauri.conf.json:**

```json
{
  "app": {
    "security": {
      "capabilities": ["default"]
    }
  }
}
```

### Permission Scopes

You can scope permissions to specific directories:

```json
{
  "permissions": [
    {
      "identifier": "fs:allow-read",
      "allow": [
        { "path": "$APPDATA/**" }
      ]
    },
    {
      "identifier": "fs:allow-write",
      "allow": [
        { "path": "$APPDATA/notes.json" }
      ]
    }
  ]
}
```

### Available Permissions

| Permission | Description |
|------------|-------------|
| `fs:default` | Basic file system access |
| `fs:allow-read` | Allow reading files |
| `fs:allow-write` | Allow writing files |
| `fs:allow-app-data-dir-access` | Access app data directory |
| `fs:allow-exists` | Check if files exist |
| `fs:allow-mkdir` | Create directories |

---

## Reading and Writing Files

### Using Rust's std::fs

For simple file operations in Tauri commands:

```rust
use std::fs;
use std::path::PathBuf;
use tauri::Manager;

#[tauri::command]
fn save_notes(notes: Vec<Note>, app: tauri::AppHandle) -> Result<(), String> {
    // Get app data directory
    let mut path = app.path()
        .app_data_dir()
        .map_err(|e| e.to_string())?;
    
    // Create directory if it doesn't exist
    fs::create_dir_all(&path).map_err(|e| e.to_string())?;
    
    // Add filename
    path.push("notes.json");
    
    // Serialize and write
    let json = serde_json::to_string_pretty(&notes)
        .map_err(|e| e.to_string())?;
    
    fs::write(&path, json).map_err(|e| e.to_string())?;
    
    Ok(())
}

#[tauri::command]
fn load_notes(app: tauri::AppHandle) -> Result<Vec<Note>, String> {
    let mut path = app.path()
        .app_data_dir()
        .map_err(|e| e.to_string())?;
    
    path.push("notes.json");
    
    // Check if file exists
    if !path.exists() {
        return Ok(Vec::new());  // Return empty vec if no file
    }
    
    // Read and deserialize
    let json = fs::read_to_string(&path)
        .map_err(|e| e.to_string())?;
    
    let notes: Vec<Note> = serde_json::from_str(&json)
        .map_err(|e| e.to_string())?;
    
    Ok(notes)
}
```

### Using the Plugin from JavaScript

You can also use the plugin directly from JavaScript:

```typescript
import { writeTextFile, readTextFile, exists, createDir, BaseDirectory } from '@tauri-apps/plugin-fs';

// Write a file
await writeTextFile('notes.json', JSON.stringify(notes), {
    baseDir: BaseDirectory.AppData
});

// Read a file
const content = await readTextFile('notes.json', {
    baseDir: BaseDirectory.AppData
});
const notes = JSON.parse(content);

// Check if file exists
const fileExists = await exists('notes.json', {
    baseDir: BaseDirectory.AppData
});

// Create directory
await createDir('notes', {
    baseDir: BaseDirectory.AppData,
    recursive: true
});
```

---

## Implementing Persistent Storage

### Complete Storage Module

```rust
// src-tauri/src/storage.rs

use crate::Note;
use std::fs;
use std::path::PathBuf;
use tauri::Manager;

const NOTES_FILE: &str = "notes.json";

pub struct Storage {
    path: PathBuf,
}

impl Storage {
    pub fn new(app: &tauri::AppHandle) -> Result<Self, String> {
        let mut path = app.path()
            .app_data_dir()
            .map_err(|e| e.to_string())?;
        
        // Ensure directory exists
        fs::create_dir_all(&path).map_err(|e| e.to_string())?;
        
        path.push(NOTES_FILE);
        
        Ok(Self { path })
    }
    
    pub fn save(&self, notes: &[Note]) -> Result<(), String> {
        let json = serde_json::to_string_pretty(notes)
            .map_err(|e| e.to_string())?;
        
        fs::write(&self.path, json)
            .map_err(|e| e.to_string())?;
        
        Ok(())
    }
    
    pub fn load(&self) -> Result<Vec<Note>, String> {
        if !self.path.exists() {
            return Ok(Vec::new());
        }
        
        let json = fs::read_to_string(&self.path)
            .map_err(|e| e.to_string())?;
        
        serde_json::from_str(&json)
            .map_err(|e| e.to_string())
    }
}
```

### Integrating with App State

```rust
// src-tauri/src/lib.rs

use std::sync::Mutex;
use tauri::{Manager, State};

mod storage;
use storage::Storage;

pub struct AppState {
    pub notes: Mutex<Vec<Note>>,
    pub storage: Mutex<Option<Storage>>,
}

#[tauri::command]
fn init_storage(app: tauri::AppHandle, state: State<'_, AppState>) -> Result<(), String> {
    let storage = Storage::new(&app)?;
    let notes = storage.load()?;
    
    // Update state
    *state.storage.lock().map_err(|e| e.to_string())? = Some(storage);
    *state.notes.lock().map_err(|e| e.to_string())? = notes;
    
    Ok(())
}

#[tauri::command]
fn create_note(title: String, content: String, state: State<'_, AppState>) -> Result<Note, String> {
    let note = Note::new(title, content);
    
    // Add to state
    let mut notes = state.notes.lock().map_err(|e| e.to_string())?;
    notes.push(note.clone());
    
    // Save to file
    let storage = state.storage.lock().map_err(|e| e.to_string())?;
    if let Some(ref s) = *storage {
        s.save(&notes)?;
    }
    
    Ok(note)
}
```

### Loading Notes on Startup

In your React app:

```typescript
// src/App.tsx
import { useEffect, useState } from 'react';
import { invoke } from '@tauri-apps/api/core';

function App() {
    const [notes, setNotes] = useState<Note[]>([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        async function initApp() {
            try {
                // Initialize storage and load notes
                await invoke('init_storage');
                const loadedNotes = await invoke<Note[]>('get_all_notes');
                setNotes(loadedNotes);
            } catch (error) {
                console.error('Failed to initialize:', error);
            } finally {
                setLoading(false);
            }
        }
        
        initApp();
    }, []);

    if (loading) {
        return <div>Loading...</div>;
    }

    return (
        // Your app UI
    );
}
```

---

## Error Handling Best Practices

### Graceful Degradation

```rust
#[tauri::command]
fn load_notes(app: tauri::AppHandle) -> Result<Vec<Note>, String> {
    let path = match app.path().app_data_dir() {
        Ok(p) => p,
        Err(e) => {
            eprintln!("Failed to get app data dir: {}", e);
            return Ok(Vec::new());  // Return empty instead of error
        }
    };
    
    let file_path = path.join("notes.json");
    
    if !file_path.exists() {
        return Ok(Vec::new());  // No file yet, that's OK
    }
    
    match fs::read_to_string(&file_path) {
        Ok(json) => {
            serde_json::from_str(&json).map_err(|e| {
                format!("Failed to parse notes: {}", e)
            })
        }
        Err(e) => {
            eprintln!("Failed to read notes file: {}", e);
            Ok(Vec::new())  // Return empty on read error
        }
    }
}
```

### Backup Before Write

```rust
fn save_with_backup(path: &PathBuf, content: &str) -> Result<(), String> {
    // Create backup if file exists
    if path.exists() {
        let backup_path = path.with_extension("json.bak");
        fs::copy(path, &backup_path).map_err(|e| e.to_string())?;
    }
    
    // Write new content
    fs::write(path, content).map_err(|e| e.to_string())?;
    
    Ok(())
}
```

---

## Hands-On Exercises

### Exercise 1: Set Up File System Plugin
1. Install the fs plugin (Rust and JavaScript)
2. Configure capabilities for app data access
3. Verify the plugin is working

### Exercise 2: Implement Save/Load
1. Create a `save_notes` command
2. Create a `load_notes` command
3. Test saving and loading notes

### Exercise 3: Auto-Save
1. Modify `create_note` to auto-save
2. Modify `update_note` to auto-save
3. Modify `delete_note` to auto-save

### Exercise 4: Load on Startup
1. Create an `init_storage` command
2. Call it when the app starts
3. Display loaded notes in the UI

### Exercise 5: Error Handling
1. Handle missing file gracefully
2. Handle corrupted JSON gracefully
3. Implement backup before save

---

## Common Pitfalls

### 1. Forgetting to Create Directory
```rust
// ❌ Will fail if directory doesn't exist
fs::write(&path, content)?;

// ✅ Create directory first
fs::create_dir_all(path.parent().unwrap())?;
fs::write(&path, content)?;
```

### 2. Missing Permissions
```json
// ❌ Missing write permission
"permissions": ["fs:allow-read"]

// ✅ Include both read and write
"permissions": ["fs:allow-read", "fs:allow-write"]
```

### 3. Hardcoded Paths
```rust
// ❌ Hardcoded path - won't work cross-platform
let path = "/home/user/.config/myapp/notes.json";

// ✅ Use app directories
let path = app.path().app_data_dir()?.join("notes.json");
```

---

## Additional Resources

- [Tauri FS Plugin Documentation](https://v2.tauri.app/plugin/file-system/)
- [Tauri Capabilities Guide](https://v2.tauri.app/security/capabilities/)
- [Rust std::fs Documentation](https://doc.rust-lang.org/std/fs/)

---

## Summary

In this phase, you learned:
- ✅ How to install and configure the file system plugin
- ✅ How to work with app directories
- ✅ How to configure file system permissions
- ✅ How to read and write JSON files
- ✅ How to implement persistent storage for notes

Next: [Phase 4 - Native Menus](../phase-4-native-menus/01-objectives.md)