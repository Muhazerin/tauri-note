# Phase 3: JSON File Storage + File System Access - Quiz

Test your understanding of Tauri file system access and persistent storage.

---

## Multiple Choice Questions

### Question 1
In Tauri v2, how is file system access provided?

- A) Built into the core Tauri library
- B) Through the `tauri-plugin-fs` plugin
- C) Through Node.js fs module
- D) Through browser APIs

<details>
<summary>Show Answer</summary>

**B) Through the `tauri-plugin-fs` plugin**

Tauri v2 uses a plugin architecture. File system access requires installing and configuring the `tauri-plugin-fs` plugin.
</details>

---

### Question 2
What is the correct way to get the app data directory in a Tauri command?

- A) `std::env::var("APPDATA")`
- B) `app.path().app_data_dir()`
- C) `tauri::get_app_data()`
- D) `BaseDirectory::AppData`

<details>
<summary>Show Answer</summary>

**B) `app.path().app_data_dir()`**

Using `app.path().app_data_dir()` returns the platform-specific app data directory path.
</details>

---

### Question 3
Where do you configure file system permissions in Tauri v2?

- A) `package.json`
- B) `Cargo.toml`
- C) Capability files in `src-tauri/capabilities/`
- D) `tauri.conf.json` only

<details>
<summary>Show Answer</summary>

**C) Capability files in `src-tauri/capabilities/`**

Tauri v2 uses capability files (JSON) in the `capabilities/` directory to define permissions.
</details>

---

### Question 4
What happens if you try to write a file without the `fs:allow-write` permission?

- A) The file is written anyway
- B) A warning is logged but the operation succeeds
- C) The operation fails with a permission error
- D) The app crashes

<details>
<summary>Show Answer</summary>

**C) The operation fails with a permission error**

Tauri's capability system enforces permissions. Without the required permission, the operation will fail with an error.
</details>

---

### Question 5
What should you do before writing a file to ensure the directory exists?

- A) Nothing, directories are created automatically
- B) Call `fs::create_dir_all()` on the parent directory
- C) Check if the file exists first
- D) Use `fs::write_with_mkdir()`

<details>
<summary>Show Answer</summary>

**B) Call `fs::create_dir_all()` on the parent directory**

`fs::create_dir_all()` creates the directory and all parent directories if they don't exist. This prevents "directory not found" errors.
</details>

---

### Question 6
Which `BaseDirectory` should you use for storing user data that should persist?

- A) `BaseDirectory::Temp`
- B) `BaseDirectory::AppCache`
- C) `BaseDirectory::AppData`
- D) `BaseDirectory::Desktop`

<details>
<summary>Show Answer</summary>

**C) `BaseDirectory::AppData`**

`AppData` is the correct directory for persistent user data. `AppCache` is for temporary data, and `Temp` is for truly temporary files.
</details>

---

### Question 7
What's the best way to handle a missing notes file on first app launch?

- A) Show an error message
- B) Crash the application
- C) Return an empty array/vector
- D) Create a file with sample data

<details>
<summary>Show Answer</summary>

**C) Return an empty array/vector**

On first launch, there's no notes file yet. Returning an empty collection is the graceful way to handle this - the user simply starts with no notes.
</details>

---

### Question 8
What Rust function is used to serialize a struct to JSON?

- A) `json::encode()`
- B) `serde_json::to_string()`
- C) `struct.to_json()`
- D) `JSON.stringify()`

<details>
<summary>Show Answer</summary>

**B) `serde_json::to_string()`**

`serde_json::to_string()` (or `to_string_pretty()` for formatted output) serializes a Rust struct to a JSON string.
</details>

---

## Code Analysis

### Question 9
What's wrong with this code?

```rust
#[tauri::command]
fn save_notes(notes: Vec<Note>) -> Result<(), String> {
    let path = "/home/user/.config/myapp/notes.json";
    let json = serde_json::to_string(&notes).map_err(|e| e.to_string())?;
    fs::write(path, json).map_err(|e| e.to_string())?;
    Ok(())
}
```

<details>
<summary>Show Answer</summary>

**Problems:**

1. **Hardcoded path** - Won't work on Windows or other users' machines
2. **No directory creation** - Will fail if directory doesn't exist
3. **No AppHandle** - Can't get the proper app data directory

**Correct version:**
```rust
#[tauri::command]
fn save_notes(notes: Vec<Note>, app: tauri::AppHandle) -> Result<(), String> {
    let mut path = app.path().app_data_dir().map_err(|e| e.to_string())?;
    fs::create_dir_all(&path).map_err(|e| e.to_string())?;
    path.push("notes.json");
    let json = serde_json::to_string(&notes).map_err(|e| e.to_string())?;
    fs::write(&path, json).map_err(|e| e.to_string())?;
    Ok(())
}
```
</details>

---

### Question 10
What's missing from this capability configuration?

```json
{
  "identifier": "default",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:allow-read"
  ]
}
```

If the app needs to save notes to a file.

<details>
<summary>Show Answer</summary>

**Missing permissions:**

1. `fs:allow-write` - Required to write files
2. `fs:allow-app-data-dir-access` - Required to access the app data directory

**Complete version:**
```json
{
  "identifier": "default",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:default",
    "fs:allow-read",
    "fs:allow-write",
    "fs:allow-app-data-dir-access"
  ]
}
```
</details>

---

## Fill in the Blanks

### Question 11
Complete the code to check if a file exists before reading:

```rust
fn load_notes(path: &PathBuf) -> Result<Vec<Note>, String> {
    if !path._______() {
        return Ok(Vec::new());
    }
    
    let json = fs::read_to_string(path).map_err(|e| e.to_string())?;
    serde_json::from_str(&json).map_err(|e| e.to_string())
}
```

<details>
<summary>Show Answer</summary>

```rust
if !path.exists() {
    return Ok(Vec::new());
}
```

The `exists()` method on `PathBuf` checks if the file or directory exists.
</details>

---

### Question 12
Complete the JavaScript code to write a file using the fs plugin:

```typescript
import { _______, BaseDirectory } from '@tauri-apps/plugin-fs';

await _______('notes.json', JSON.stringify(notes), {
    baseDir: BaseDirectory._______
});
```

<details>
<summary>Show Answer</summary>

```typescript
import { writeTextFile, BaseDirectory } from '@tauri-apps/plugin-fs';

await writeTextFile('notes.json', JSON.stringify(notes), {
    baseDir: BaseDirectory.AppData
});
```
</details>

---

## True or False

### Question 13
True or False: In Tauri v2, you can access any file on the user's system without special permissions.

<details>
<summary>Show Answer</summary>

**False**

Tauri v2 uses a capability-based permission system. You must explicitly request and be granted permissions for file system access, and access can be scoped to specific directories.
</details>

---

### Question 14
True or False: `fs::create_dir_all()` will fail if the directory already exists.

<details>
<summary>Show Answer</summary>

**False**

`fs::create_dir_all()` is idempotent - it creates the directory if it doesn't exist, and does nothing (succeeds) if it already exists.
</details>

---

## Short Answer

### Question 15
Explain why we use `app.path().app_data_dir()` instead of hardcoding a path like `/home/user/.config/myapp/`.

<details>
<summary>Show Answer</summary>

**Reasons:**

1. **Cross-platform compatibility**: The app data directory is different on each OS:
   - Windows: `%APPDATA%\{app}`
   - macOS: `~/Library/Application Support/{app}`
   - Linux: `~/.local/share/{app}`

2. **User independence**: Hardcoded paths with usernames won't work for other users

3. **Tauri integration**: The path includes the app identifier from `tauri.conf.json`

4. **Best practice**: Using the system's designated app data location follows OS conventions and user expectations
</details>

---

### Question 16
What is the purpose of the capability system in Tauri v2, and how does it improve security?

<details>
<summary>Show Answer</summary>

**Purpose:**
The capability system defines what permissions an app has to access system resources (file system, network, etc.).

**Security improvements:**

1. **Principle of least privilege**: Apps only get the permissions they explicitly request
2. **Scoped access**: Permissions can be limited to specific directories or files
3. **Transparency**: Users/developers can see exactly what the app can access
4. **Defense in depth**: Even if frontend code is compromised, it can't access resources beyond its capabilities
5. **Audit trail**: Capability files document the app's security requirements

**Example**: An app that only needs to read/write to its own data directory can't access the user's documents or system files.
</details>

---

## Practical Exercise

### Question 17
Write a Rust function that:
1. Gets the app data directory
2. Creates it if it doesn't exist
3. Loads notes from `notes.json` if it exists
4. Returns an empty vector if the file doesn't exist
5. Handles all errors gracefully

<details>
<summary>Show Answer</summary>

```rust
use std::fs;
use std::path::PathBuf;
use tauri::Manager;
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Note {
    pub id: String,
    pub title: String,
    pub content: String,
}

#[tauri::command]
fn load_notes(app: tauri::AppHandle) -> Result<Vec<Note>, String> {
    // 1. Get app data directory
    let mut path = app.path()
        .app_data_dir()
        .map_err(|e| format!("Failed to get app data dir: {}", e))?;
    
    // 2. Create directory if it doesn't exist
    fs::create_dir_all(&path)
        .map_err(|e| format!("Failed to create directory: {}", e))?;
    
    // Add filename
    path.push("notes.json");
    
    // 3 & 4. Load notes or return empty vector
    if !path.exists() {
        return Ok(Vec::new());
    }
    
    // 5. Handle errors gracefully
    let json = fs::read_to_string(&path)
        .map_err(|e| format!("Failed to read file: {}", e))?;
    
    let notes: Vec<Note> = serde_json::from_str(&json)
        .map_err(|e| format!("Failed to parse JSON: {}", e))?;
    
    Ok(notes)
}
```

Key points:
- Uses `app.path().app_data_dir()` for cross-platform paths
- Creates directory with `create_dir_all()`
- Checks `path.exists()` before reading
- Returns `Ok(Vec::new())` for missing file
- Uses `map_err` for descriptive error messages
</details>

---

### Question 18
Write the capability configuration that allows:
- Reading and writing files in the app data directory only
- Creating directories
- Checking if files exist

<details>
<summary>Show Answer</summary>

```json
{
  "$schema": "https://schemas.tauri.app/config/2/capability",
  "identifier": "notes-storage",
  "description": "Permissions for notes file storage",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:default",
    "fs:allow-app-data-dir-access",
    {
      "identifier": "fs:allow-read",
      "allow": [
        { "path": "$APPDATA/**" }
      ]
    },
    {
      "identifier": "fs:allow-write",
      "allow": [
        { "path": "$APPDATA/**" }
      ]
    },
    "fs:allow-mkdir",
    "fs:allow-exists"
  ]
}
```

Key points:
- Scopes read/write to `$APPDATA/**` (app data directory and subdirectories)
- Includes `fs:allow-mkdir` for creating directories
- Includes `fs:allow-exists` for checking file existence
- Uses descriptive identifier and description
</details>

---

## Score Yourself

| Score | Rating |
|-------|--------|
| 16-18 | ⭐⭐⭐ Excellent! Ready for Phase 4 |
| 12-15 | ⭐⭐ Good! Review the materials you missed |
| 8-11 | ⭐ Fair. Re-read the learning materials |
| 0-7 | Need more practice. Start from the beginning |

---

## Next Steps

If you scored well, proceed to [Phase 4: Native Menus](../phase-4-native-menus/01-objectives.md)

If you need more practice, review:
- [Phase 3 Learning Materials](./02-materials.md)
- [Tauri FS Plugin Documentation](https://v2.tauri.app/plugin/file-system/)