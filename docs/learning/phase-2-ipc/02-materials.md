# Phase 2: Core Note Functionality + IPC - Learning Materials

## Table of Contents
1. [Understanding IPC in Tauri](#understanding-ipc-in-tauri)
2. [Creating Tauri Commands](#creating-tauri-commands)
3. [Calling Commands from JavaScript](#calling-commands-from-javascript)
4. [Data Serialization with Serde](#data-serialization-with-serde)
5. [Application State Management](#application-state-management)
6. [Building the Note CRUD Operations](#building-the-note-crud-operations)
7. [Hands-On Exercises](#hands-on-exercises)

---

## Understanding IPC in Tauri

### What is IPC?

IPC (Inter-Process Communication) is the mechanism that allows the frontend (JavaScript/TypeScript running in the webview) to communicate with the backend (Rust).

```
┌─────────────────┐         IPC          ┌─────────────────┐
│    Frontend     │ ◄──────────────────► │     Backend     │
│   (React/JS)    │                      │     (Rust)      │
│                 │   invoke('cmd')      │                 │
│                 │ ─────────────────►   │  #[command]     │
│                 │                      │  fn cmd() {}    │
│                 │   ◄───────────────   │                 │
│                 │   Result/Response    │                 │
└─────────────────┘                      └─────────────────┘
```

### Two-Way Communication

1. **Frontend → Backend**: Using `invoke()` to call Tauri commands
2. **Backend → Frontend**: Using events (covered in later phases)

---

## Creating Tauri Commands

### Basic Command

A Tauri command is a Rust function decorated with `#[tauri::command]`:

```rust
// src-tauri/src/lib.rs

#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}
```

### Registering Commands

Commands must be registered in the Tauri app builder:

```rust
// src-tauri/src/lib.rs

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![greet])  // Register here
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Command with Multiple Parameters

```rust
#[tauri::command]
fn add_numbers(a: i32, b: i32) -> i32 {
    a + b
}
```

### Command Returning Result

For operations that can fail, return a `Result`:

```rust
#[tauri::command]
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("Cannot divide by zero".to_string())
    } else {
        Ok(a / b)
    }
}
```

### Async Commands

For I/O operations, use async commands:

```rust
#[tauri::command]
async fn fetch_data() -> Result<String, String> {
    // Simulating async operation
    tokio::time::sleep(std::time::Duration::from_secs(1)).await;
    Ok("Data fetched!".to_string())
}
```

---

## Calling Commands from JavaScript

### Basic Invoke

```typescript
import { invoke } from '@tauri-apps/api/core';

// Call a command with no arguments
const result = await invoke('greet', { name: 'World' });
console.log(result); // "Hello, World!"
```

### With Type Safety

```typescript
import { invoke } from '@tauri-apps/api/core';

// Define the expected return type
const result = await invoke<string>('greet', { name: 'World' });
```

### Handling Errors

```typescript
import { invoke } from '@tauri-apps/api/core';

try {
    const result = await invoke<number>('divide', { a: 10, b: 0 });
    console.log(result);
} catch (error) {
    console.error('Error:', error); // "Cannot divide by zero"
}
```

### Parameter Naming Convention

**Important**: Parameter names in JavaScript must match the Rust function parameter names:

```rust
// Rust
#[tauri::command]
fn greet(name: &str) -> String { ... }
```

```typescript
// JavaScript - parameter name must be "name"
invoke('greet', { name: 'World' });  // ✅ Correct
invoke('greet', { n: 'World' });     // ❌ Wrong - parameter name mismatch
```

---

## Data Serialization with Serde

### What is Serde?

Serde is Rust's serialization/deserialization framework. It converts Rust structs to JSON (and vice versa) for IPC communication.

### Adding Serde to Cargo.toml

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

### Creating a Serializable Struct

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Note {
    pub id: String,
    pub title: String,
    pub content: String,
    pub created_at: String,
    pub updated_at: String,
}
```

### Using Structs in Commands

```rust
#[tauri::command]
fn get_note(id: &str) -> Result<Note, String> {
    // Return a Note struct - automatically serialized to JSON
    Ok(Note {
        id: id.to_string(),
        title: "My Note".to_string(),
        content: "Note content here".to_string(),
        created_at: "2024-01-01T00:00:00Z".to_string(),
        updated_at: "2024-01-01T00:00:00Z".to_string(),
    })
}
```

### TypeScript Interface

Create a matching TypeScript interface:

```typescript
// src/types/Note.ts
export interface Note {
    id: string;
    title: string;
    content: string;
    created_at: string;  // Note: snake_case to match Rust
    updated_at: string;
}
```

### Calling with Types

```typescript
import { invoke } from '@tauri-apps/api/core';
import { Note } from './types/Note';

const note = await invoke<Note>('get_note', { id: '123' });
console.log(note.title); // "My Note"
```

---

## Application State Management

### Why State Management?

Commands are stateless by default. To persist data between command calls, use Tauri's state management.

### Creating State

```rust
use std::sync::Mutex;

// Define your state structure
pub struct AppState {
    pub notes: Mutex<Vec<Note>>,
}

// Initialize state in the app builder
pub fn run() {
    tauri::Builder::default()
        .manage(AppState {
            notes: Mutex::new(Vec::new()),
        })
        .invoke_handler(tauri::generate_handler![...])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Accessing State in Commands

```rust
use tauri::State;

#[tauri::command]
fn get_all_notes(state: State<'_, AppState>) -> Result<Vec<Note>, String> {
    let notes = state.notes.lock().map_err(|e| e.to_string())?;
    Ok(notes.clone())
}

#[tauri::command]
fn add_note(note: Note, state: State<'_, AppState>) -> Result<Note, String> {
    let mut notes = state.notes.lock().map_err(|e| e.to_string())?;
    notes.push(note.clone());
    Ok(note)
}
```

### Understanding Mutex

`Mutex` (mutual exclusion) ensures thread-safe access to shared data:

```rust
// Lock the mutex to access data
let notes = state.notes.lock().unwrap();

// The lock is automatically released when `notes` goes out of scope
```

**Important**: Always handle the potential error from `lock()` in production code.

---

## Building the Note CRUD Operations

### Complete Example

```rust
// src-tauri/src/lib.rs

use serde::{Deserialize, Serialize};
use std::sync::Mutex;
use tauri::State;
use uuid::Uuid;
use chrono::Utc;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Note {
    pub id: String,
    pub title: String,
    pub content: String,
    pub created_at: String,
    pub updated_at: String,
}

pub struct AppState {
    pub notes: Mutex<Vec<Note>>,
}

// CREATE
#[tauri::command]
fn create_note(title: String, content: String, state: State<'_, AppState>) -> Result<Note, String> {
    let now = Utc::now().to_rfc3339();
    let note = Note {
        id: Uuid::new_v4().to_string(),
        title,
        content,
        created_at: now.clone(),
        updated_at: now,
    };
    
    let mut notes = state.notes.lock().map_err(|e| e.to_string())?;
    notes.push(note.clone());
    
    Ok(note)
}

// READ ALL
#[tauri::command]
fn get_all_notes(state: State<'_, AppState>) -> Result<Vec<Note>, String> {
    let notes = state.notes.lock().map_err(|e| e.to_string())?;
    Ok(notes.clone())
}

// READ ONE
#[tauri::command]
fn get_note(id: String, state: State<'_, AppState>) -> Result<Note, String> {
    let notes = state.notes.lock().map_err(|e| e.to_string())?;
    notes
        .iter()
        .find(|n| n.id == id)
        .cloned()
        .ok_or_else(|| "Note not found".to_string())
}

// UPDATE
#[tauri::command]
fn update_note(id: String, title: String, content: String, state: State<'_, AppState>) -> Result<Note, String> {
    let mut notes = state.notes.lock().map_err(|e| e.to_string())?;
    
    let note = notes
        .iter_mut()
        .find(|n| n.id == id)
        .ok_or_else(|| "Note not found".to_string())?;
    
    note.title = title;
    note.content = content;
    note.updated_at = Utc::now().to_rfc3339();
    
    Ok(note.clone())
}

// DELETE
#[tauri::command]
fn delete_note(id: String, state: State<'_, AppState>) -> Result<(), String> {
    let mut notes = state.notes.lock().map_err(|e| e.to_string())?;
    let initial_len = notes.len();
    notes.retain(|n| n.id != id);
    
    if notes.len() == initial_len {
        Err("Note not found".to_string())
    } else {
        Ok(())
    }
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .manage(AppState {
            notes: Mutex::new(Vec::new()),
        })
        .invoke_handler(tauri::generate_handler![
            create_note,
            get_all_notes,
            get_note,
            update_note,
            delete_note
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Frontend Usage

```typescript
// src/hooks/useNotes.ts
import { invoke } from '@tauri-apps/api/core';
import { Note } from '../types/Note';

export const useNotes = () => {
    const createNote = async (title: string, content: string): Promise<Note> => {
        return invoke<Note>('create_note', { title, content });
    };

    const getAllNotes = async (): Promise<Note[]> => {
        return invoke<Note[]>('get_all_notes');
    };

    const getNote = async (id: string): Promise<Note> => {
        return invoke<Note>('get_note', { id });
    };

    const updateNote = async (id: string, title: string, content: string): Promise<Note> => {
        return invoke<Note>('update_note', { id, title, content });
    };

    const deleteNote = async (id: string): Promise<void> => {
        return invoke('delete_note', { id });
    };

    return { createNote, getAllNotes, getNote, updateNote, deleteNote };
};
```

---

## Hands-On Exercises

### Exercise 1: Create Your First Command
1. Create a simple command that returns "Hello from Rust!"
2. Call it from React and display the result
3. Add a parameter to personalize the greeting

### Exercise 2: Work with Structs
1. Create a `Note` struct with id, title, and content
2. Create a command that returns a hardcoded Note
3. Display the note in a React component

### Exercise 3: Implement State
1. Add `AppState` with a `Vec<Note>`
2. Create `add_note` and `get_all_notes` commands
3. Test adding and retrieving notes

### Exercise 4: Complete CRUD
1. Implement `update_note` command
2. Implement `delete_note` command
3. Create a simple UI to test all operations

### Exercise 5: Error Handling
1. Add validation to `create_note` (title required)
2. Handle the error in the frontend
3. Display appropriate error messages to the user

---

## Common Pitfalls

### 1. Parameter Name Mismatch
```typescript
// ❌ Wrong
invoke('greet', { n: 'World' });

// ✅ Correct
invoke('greet', { name: 'World' });
```

### 2. Forgetting to Register Commands
```rust
// ❌ Command exists but not registered
.invoke_handler(tauri::generate_handler![greet])

// ✅ All commands registered
.invoke_handler(tauri::generate_handler![greet, create_note, get_all_notes])
```

### 3. Missing Serde Derives
```rust
// ❌ Missing derives
struct Note { ... }

// ✅ With derives
#[derive(Serialize, Deserialize)]
struct Note { ... }
```

### 4. Not Handling Mutex Errors
```rust
// ❌ Can panic
let notes = state.notes.lock().unwrap();

// ✅ Proper error handling
let notes = state.notes.lock().map_err(|e| e.to_string())?;
```

---

## Additional Resources

- [Tauri Commands Documentation](https://v2.tauri.app/develop/calling-rust/)
- [Serde Documentation](https://serde.rs/)
- [Rust Mutex Documentation](https://doc.rust-lang.org/std/sync/struct.Mutex.html)

---

## Summary

In this phase, you learned:
- ✅ How to create Tauri commands with `#[tauri::command]`
- ✅ How to call commands from JavaScript using `invoke()`
- ✅ How to serialize data with Serde
- ✅ How to manage application state with `Mutex`
- ✅ How to implement CRUD operations for notes

Next: [Phase 3 - JSON File Storage + File System Access](../phase-3-file-system/01-objectives.md)