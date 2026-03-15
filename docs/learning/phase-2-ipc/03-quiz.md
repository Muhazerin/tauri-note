# Phase 2: Core Note Functionality + IPC - Quiz

Test your understanding of Tauri IPC and command implementation.

---

## Multiple Choice Questions

### Question 1
What attribute is used to create a Tauri command in Rust?

- A) `#[command]`
- B) `#[tauri::command]`
- C) `#[tauri::invoke]`
- D) `#[ipc::command]`

<details>
<summary>Show Answer</summary>

**B) `#[tauri::command]`**

The `#[tauri::command]` attribute macro is used to expose a Rust function as a Tauri command that can be called from the frontend.
</details>

---

### Question 2
What function is used in JavaScript to call a Tauri command?

- A) `call()`
- B) `execute()`
- C) `invoke()`
- D) `run()`

<details>
<summary>Show Answer</summary>

**C) `invoke()`**

The `invoke()` function from `@tauri-apps/api/core` is used to call Tauri commands from JavaScript.
</details>

---

### Question 3
What Rust library is used for serialization/deserialization in Tauri?

- A) json
- B) serde
- C) serialize
- D) codec

<details>
<summary>Show Answer</summary>

**B) serde**

Serde is Rust's primary serialization framework, used to convert Rust structs to/from JSON for IPC communication.
</details>

---

### Question 4
What wrapper is used to make state thread-safe in Tauri?

- A) `Arc`
- B) `Box`
- C) `Mutex`
- D) `RefCell`

<details>
<summary>Show Answer</summary>

**C) `Mutex`**

`Mutex` (mutual exclusion) is used to ensure thread-safe access to shared mutable state in Tauri commands.
</details>

---

### Question 5
How do you access managed state in a Tauri command?

- A) `state: AppState`
- B) `state: &AppState`
- C) `state: State<'_, AppState>`
- D) `state: Managed<AppState>`

<details>
<summary>Show Answer</summary>

**C) `state: State<'_, AppState>`**

The `State<'_, AppState>` type from `tauri::State` is used to access managed state in Tauri commands.
</details>

---

### Question 6
What happens if you call `invoke()` with a parameter name that doesn't match the Rust function?

- A) It automatically converts the name
- B) The command fails with an error
- C) It uses a default value
- D) It ignores the parameter

<details>
<summary>Show Answer</summary>

**B) The command fails with an error**

Parameter names in JavaScript must exactly match the Rust function parameter names. A mismatch will cause the command to fail.
</details>

---

### Question 7
What derive macros are required for a struct to be used in Tauri commands?

- A) `#[derive(Debug)]`
- B) `#[derive(Clone)]`
- C) `#[derive(Serialize, Deserialize)]`
- D) `#[derive(Command)]`

<details>
<summary>Show Answer</summary>

**C) `#[derive(Serialize, Deserialize)]`**

Structs that cross the IPC boundary must derive `Serialize` and `Deserialize` from serde to be converted to/from JSON.
</details>

---

### Question 8
Where do you register Tauri commands?

- A) In `tauri.conf.json`
- B) In the `invoke_handler` using `generate_handler!`
- C) In `Cargo.toml`
- D) Commands are auto-registered

<details>
<summary>Show Answer</summary>

**B) In the `invoke_handler` using `generate_handler!`**

Commands must be registered in the Tauri builder using `.invoke_handler(tauri::generate_handler![command1, command2, ...])`.
</details>

---

## Code Analysis

### Question 9
What's wrong with this Tauri command?

```rust
#[tauri::command]
fn get_notes(state: State<'_, AppState>) -> Vec<Note> {
    let notes = state.notes.lock().unwrap();
    notes.clone()
}
```

- A) Missing `pub` keyword
- B) Should return `Result<Vec<Note>, String>` for error handling
- C) `unwrap()` can panic if the lock is poisoned
- D) Both B and C

<details>
<summary>Show Answer</summary>

**D) Both B and C**

1. Using `unwrap()` can cause a panic if the mutex is poisoned
2. Commands should return `Result` to properly handle and communicate errors to the frontend

Better version:
```rust
#[tauri::command]
fn get_notes(state: State<'_, AppState>) -> Result<Vec<Note>, String> {
    let notes = state.notes.lock().map_err(|e| e.to_string())?;
    Ok(notes.clone())
}
```
</details>

---

### Question 10
What's wrong with this JavaScript code?

```typescript
const note = await invoke('create_note', { 
    noteTitle: 'My Note',
    noteContent: 'Content here'
});
```

Given this Rust command:
```rust
#[tauri::command]
fn create_note(title: String, content: String) -> Note { ... }
```

<details>
<summary>Show Answer</summary>

**Parameter names don't match**

The JavaScript uses `noteTitle` and `noteContent`, but the Rust function expects `title` and `content`.

Correct version:
```typescript
const note = await invoke('create_note', { 
    title: 'My Note',
    content: 'Content here'
});
```
</details>

---

## Fill in the Blanks

### Question 11
Complete the code to properly handle errors when calling a Tauri command:

```typescript
_______ {
    const result = await invoke<Note>('get_note', { id: '123' });
    console.log(result);
} _______ (error) {
    console.error('Failed to get note:', error);
}
```

<details>
<summary>Show Answer</summary>

```typescript
try {
    const result = await invoke<Note>('get_note', { id: '123' });
    console.log(result);
} catch (error) {
    console.error('Failed to get note:', error);
}
```

Always use try/catch when calling Tauri commands to handle potential errors.
</details>

---

### Question 12
Complete the Rust code to add state management:

```rust
pub fn run() {
    tauri::Builder::default()
        ._______(AppState {
            notes: Mutex::new(Vec::new()),
        })
        .invoke_handler(tauri::generate_handler![get_notes])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

<details>
<summary>Show Answer</summary>

```rust
.manage(AppState {
    notes: Mutex::new(Vec::new()),
})
```

The `.manage()` method is used to add managed state to the Tauri application.
</details>

---

## True or False

### Question 13
True or False: Tauri commands are synchronous by default.

<details>
<summary>Show Answer</summary>

**True**

Tauri commands are synchronous by default. To make them async, add the `async` keyword to the function signature.
</details>

---

### Question 14
True or False: You can return any Rust type from a Tauri command without any special setup.

<details>
<summary>Show Answer</summary>

**False**

Types returned from Tauri commands must implement `Serialize` (from serde) to be converted to JSON for the frontend. Primitive types work automatically, but custom structs need the derive macro.
</details>

---

## Short Answer

### Question 15
Explain the difference between these two approaches for error handling in Tauri commands:

```rust
// Approach A
#[tauri::command]
fn get_note(id: String) -> Note { ... }

// Approach B
#[tauri::command]
fn get_note(id: String) -> Result<Note, String> { ... }
```

<details>
<summary>Show Answer</summary>

**Approach A** returns a `Note` directly. If something goes wrong (e.g., note not found), the only option is to panic, which crashes the application.

**Approach B** returns a `Result<Note, String>`:
- `Ok(note)` - Success case, returns the note
- `Err(message)` - Error case, returns an error message

The error is sent to the frontend where it can be caught with try/catch, allowing graceful error handling without crashing.

**Best Practice**: Always use `Result` for commands that can fail.
</details>

---

### Question 16
Why do we use `Mutex` for state in Tauri, and what does `lock()` do?

<details>
<summary>Show Answer</summary>

**Why Mutex?**
Tauri commands can be called from multiple threads simultaneously. `Mutex` (mutual exclusion) ensures that only one thread can access the data at a time, preventing data races and corruption.

**What does `lock()` do?**
1. Acquires exclusive access to the data inside the Mutex
2. Blocks other threads from accessing the data until the lock is released
3. Returns a `MutexGuard` that automatically releases the lock when it goes out of scope

Example:
```rust
let notes = state.notes.lock().map_err(|e| e.to_string())?;
// `notes` is now exclusively accessible
// Lock is released when `notes` goes out of scope
```
</details>

---

## Practical Exercise

### Question 17
Write a Tauri command that:
1. Takes a `Note` struct as input
2. Validates that the title is not empty
3. Adds the note to the state
4. Returns the added note or an error

<details>
<summary>Show Answer</summary>

```rust
use serde::{Deserialize, Serialize};
use std::sync::Mutex;
use tauri::State;

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Note {
    pub id: String,
    pub title: String,
    pub content: String,
}

pub struct AppState {
    pub notes: Mutex<Vec<Note>>,
}

#[tauri::command]
fn add_note(note: Note, state: State<'_, AppState>) -> Result<Note, String> {
    // Validate title is not empty
    if note.title.trim().is_empty() {
        return Err("Title cannot be empty".to_string());
    }
    
    // Lock the state and add the note
    let mut notes = state.notes.lock().map_err(|e| e.to_string())?;
    notes.push(note.clone());
    
    // Return the added note
    Ok(note)
}
```

Key points:
- Returns `Result<Note, String>` for error handling
- Validates input before processing
- Uses `map_err` to handle mutex lock errors
- Clones the note before pushing (since we need to return it)
</details>

---

### Question 18
Write the TypeScript code to call the `add_note` command and handle both success and error cases.

<details>
<summary>Show Answer</summary>

```typescript
import { invoke } from '@tauri-apps/api/core';

interface Note {
    id: string;
    title: string;
    content: string;
}

async function addNote(note: Note): Promise<Note | null> {
    try {
        const addedNote = await invoke<Note>('add_note', { note });
        console.log('Note added successfully:', addedNote);
        return addedNote;
    } catch (error) {
        console.error('Failed to add note:', error);
        // Could show a toast notification here
        return null;
    }
}

// Usage
const newNote: Note = {
    id: '123',
    title: 'My Note',
    content: 'Note content here'
};

const result = await addNote(newNote);
if (result) {
    // Update UI with the new note
}
```

Key points:
- Uses generic type `invoke<Note>` for type safety
- Wraps in try/catch for error handling
- Returns null on error (or could throw/handle differently)
- Logs both success and error cases
</details>

---

## Score Yourself

| Score | Rating |
|-------|--------|
| 16-18 | ⭐⭐⭐ Excellent! Ready for Phase 3 |
| 12-15 | ⭐⭐ Good! Review the materials you missed |
| 8-11 | ⭐ Fair. Re-read the learning materials |
| 0-7 | Need more practice. Start from the beginning |

---

## Next Steps

If you scored well, proceed to [Phase 3: JSON File Storage + File System Access](../phase-3-file-system/01-objectives.md)

If you need more practice, review:
- [Phase 2 Learning Materials](./02-materials.md)
- [Tauri Commands Documentation](https://v2.tauri.app/develop/calling-rust/)