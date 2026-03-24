# Phase 8: Database Migration (JSON to PouchDB/CouchDB) - Learning Materials

## Table of Contents
1. [Why Migrate from JSON?](#why-migrate-from-json)
2. [PouchDB/CouchDB Overview](#pouchdbcouchdb-overview)
3. [Setting Up for Tauri Desktop](#setting-up-for-tauri-desktop)
4. [Implementing CRUD Operations](#implementing-crud-operations)
5. [Data Migration Strategy](#data-migration-strategy)
6. [Cross-Platform Considerations](#cross-platform-considerations)
7. [Optional: Sync with CouchDB](#optional-sync-with-couchdb)
8. [Hands-On Exercises](#hands-on-exercises)

---

## Why Migrate from JSON?

### Limitations of JSON File Storage

While JSON files work well for simple applications, they have significant limitations:

```
JSON File Approach:
┌─────────────────────────────────────┐
│  notes.json                         │
│  ┌─────────────────────────────────┐│
│  │ [                               ││
│  │   { "id": "1", "title": "..." },││
│  │   { "id": "2", "title": "..." },││
│  │   { "id": "3", "title": "..." },││
│  │   ... (entire file loaded)      ││
│  │ ]                               ││
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘
         │
         ▼ Every operation requires:
    1. Read entire file
    2. Parse all JSON
    3. Modify in memory
    4. Write entire file back
```

**Problems:**
1. **Performance**: Must load entire file to read one note
2. **Concurrency**: No safe way to handle simultaneous writes
3. **Queries**: No indexing - must scan all notes to search
4. **Data Loss Risk**: Crash during write = corrupted file
5. **No Sync**: Manual implementation needed for cloud sync

### Benefits of Document Databases

```
PouchDB/CouchDB Approach:
┌─────────────────────────────────────┐
│  Database                           │
│  ┌──────┐ ┌──────┐ ┌──────┐       │
│  │Doc 1 │ │Doc 2 │ │Doc 3 │ ...   │
│  │ _id  │ │ _id  │ │ _id  │       │
│  │ _rev │ │ _rev │ │ _rev │       │
│  └──────┘ └──────┘ └──────┘       │
│      │                             │
│      ▼ Indexes for fast queries    │
│  ┌─────────────────────────────────┐│
│  │ Index: by_date, by_title, etc. ││
│  └─────────────────────────────────┘│
└─────────────────────────────────────┘
```

**Benefits:**
1. **Efficient**: Read/write individual documents
2. **ACID**: Atomic operations, crash-safe
3. **Indexed Queries**: Fast searches with views
4. **Conflict Resolution**: Built-in handling
5. **Sync Ready**: Replication is a core feature

---

## PouchDB/CouchDB Overview

### What is PouchDB?

PouchDB is a JavaScript database that:
- Runs in the browser (using IndexedDB)
- Syncs with CouchDB-compatible servers
- Works offline-first
- Has the same API everywhere

### What is CouchDB?

CouchDB is a server-side database that:
- Stores JSON documents
- Provides HTTP API
- Supports multi-master replication
- Handles conflicts automatically

### Document Structure

Every document in PouchDB/CouchDB has:

```json
{
  "_id": "note_001",           // Unique identifier (required)
  "_rev": "1-abc123def456",    // Revision (managed by database)
  "title": "My Note",          // Your data
  "content": "Note content...",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

### The Revision System

The `_rev` field is crucial for conflict detection:

```
Initial Create:
  _rev: "1-abc123"

First Update:
  _rev: "2-def456"  (must provide "1-abc123" to update)

Second Update:
  _rev: "3-ghi789"  (must provide "2-def456" to update)
```

**Why revisions matter:**
- Prevents accidental overwrites
- Enables conflict detection during sync
- Allows tracking document history

---

## Setting Up for Tauri Desktop

### Approach Options

For Tauri (Rust backend), you have several options:

| Approach | Description | Pros | Cons |
|----------|-------------|------|------|
| **couch_rs** | CouchDB client for Rust | Full CouchDB protocol | Requires CouchDB server |
| **sled + custom** | Embedded DB with CouchDB-like API | Pure Rust, embedded | Manual implementation |
| **PouchDB via JS** | Run PouchDB in webview | Full PouchDB API | Data in webview, not Rust |

### Recommended: Hybrid Approach

For this learning module, we'll use a **hybrid approach**:
- **Rust backend**: Custom document store using `sled` (embedded key-value store)
- **CouchDB-compatible structure**: Same document format as PouchDB
- **Future-ready**: Can sync with CouchDB when needed

### Installation

**1. Add Rust dependencies:**

```bash
cd src-tauri
cargo add sled
cargo add serde_json
cargo add uuid --features v4
cargo add chrono --features serde
```

Or add to `Cargo.toml`:

```toml
[dependencies]
sled = "0.34"
serde_json = "1.0"
uuid = { version = "1.0", features = ["v4"] }
chrono = { version = "0.4", features = ["serde"] }
```

**2. Create the database module:**

```rust
// src-tauri/src/database.rs

use serde::{Deserialize, Serialize};
use sled::Db;
use std::path::PathBuf;
use uuid::Uuid;
use chrono::Utc;

/// Document wrapper with CouchDB-compatible fields
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Document<T> {
    #[serde(rename = "_id")]
    pub id: String,
    #[serde(rename = "_rev")]
    pub rev: String,
    #[serde(flatten)]
    pub data: T,
}

/// Note data structure
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct NoteData {
    pub title: String,
    pub content: String,
    pub created_at: String,
    pub updated_at: String,
}

pub type NoteDocument = Document<NoteData>;

/// Database wrapper
pub struct Database {
    db: Db,
}

impl Database {
    /// Open or create database at the specified path
    pub fn open(path: PathBuf) -> Result<Self, String> {
        let db = sled::open(path).map_err(|e| e.to_string())?;
        Ok(Self { db })
    }

    /// Generate a new revision string
    fn generate_rev(version: u32) -> String {
        let hash: String = Uuid::new_v4().to_string()[..8].to_string();
        format!("{}-{}", version, hash)
    }

    /// Parse revision to get version number
    fn parse_rev_version(rev: &str) -> u32 {
        rev.split('-')
            .next()
            .and_then(|v| v.parse().ok())
            .unwrap_or(0)
    }
}
```

**3. Initialize in your Tauri app:**

```rust
// src-tauri/src/lib.rs

mod database;

use database::Database;
use std::sync::Mutex;
use tauri::Manager;

pub struct AppState {
    pub db: Mutex<Option<Database>>,
}

#[tauri::command]
fn init_database(app: tauri::AppHandle, state: tauri::State<'_, AppState>) -> Result<(), String> {
    let mut db_path = app.path().app_data_dir().map_err(|e| e.to_string())?;
    db_path.push("notes.db");
    
    let database = Database::open(db_path)?;
    
    *state.db.lock().map_err(|e| e.to_string())? = Some(database);
    
    Ok(())
}

pub fn run() {
    tauri::Builder::default()
        .manage(AppState {
            db: Mutex::new(None),
        })
        .invoke_handler(tauri::generate_handler![
            init_database,
            // ... other commands
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

---

## Implementing CRUD Operations

### Create Document

```rust
// src-tauri/src/database.rs

impl Database {
    /// Create a new note document
    pub fn create_note(&self, title: String, content: String) -> Result<NoteDocument, String> {
        let now = Utc::now().to_rfc3339();
        let id = format!("note_{}", Uuid::new_v4());
        let rev = Self::generate_rev(1);
        
        let doc = NoteDocument {
            id: id.clone(),
            rev: rev.clone(),
            data: NoteData {
                title,
                content,
                created_at: now.clone(),
                updated_at: now,
            },
        };
        
        let json = serde_json::to_vec(&doc).map_err(|e| e.to_string())?;
        
        self.db
            .insert(id.as_bytes(), json)
            .map_err(|e| e.to_string())?;
        
        // Flush to ensure durability
        self.db.flush().map_err(|e| e.to_string())?;
        
        Ok(doc)
    }
}

// Tauri command
#[tauri::command]
fn create_note(
    title: String,
    content: String,
    state: tauri::State<'_, AppState>,
) -> Result<NoteDocument, String> {
    let db_guard = state.db.lock().map_err(|e| e.to_string())?;
    let db = db_guard.as_ref().ok_or("Database not initialized")?;
    
    db.create_note(title, content)
}
```

### Read Documents

```rust
impl Database {
    /// Get a single note by ID
    pub fn get_note(&self, id: &str) -> Result<Option<NoteDocument>, String> {
        match self.db.get(id.as_bytes()).map_err(|e| e.to_string())? {
            Some(data) => {
                let doc: NoteDocument = serde_json::from_slice(&data)
                    .map_err(|e| e.to_string())?;
                Ok(Some(doc))
            }
            None => Ok(None),
        }
    }
    
    /// Get all notes
    pub fn get_all_notes(&self) -> Result<Vec<NoteDocument>, String> {
        let mut notes = Vec::new();
        
        for result in self.db.iter() {
            let (key, value) = result.map_err(|e| e.to_string())?;
            
            // Only process note documents
            let key_str = String::from_utf8_lossy(&key);
            if key_str.starts_with("note_") {
                let doc: NoteDocument = serde_json::from_slice(&value)
                    .map_err(|e| e.to_string())?;
                notes.push(doc);
            }
        }
        
        // Sort by created_at descending (newest first)
        notes.sort_by(|a, b| b.data.created_at.cmp(&a.data.created_at));
        
        Ok(notes)
    }
}

// Tauri commands
#[tauri::command]
fn get_note(id: String, state: tauri::State<'_, AppState>) -> Result<Option<NoteDocument>, String> {
    let db_guard = state.db.lock().map_err(|e| e.to_string())?;
    let db = db_guard.as_ref().ok_or("Database not initialized")?;
    
    db.get_note(&id)
}

#[tauri::command]
fn get_all_notes(state: tauri::State<'_, AppState>) -> Result<Vec<NoteDocument>, String> {
    let db_guard = state.db.lock().map_err(|e| e.to_string())?;
    let db = db_guard.as_ref().ok_or("Database not initialized")?;
    
    db.get_all_notes()
}
```

### Update Document

```rust
impl Database {
    /// Update an existing note (requires correct revision)
    pub fn update_note(
        &self,
        id: &str,
        rev: &str,
        title: String,
        content: String,
    ) -> Result<NoteDocument, String> {
        // Get existing document
        let existing = self.get_note(id)?
            .ok_or_else(|| format!("Document not found: {}", id))?;
        
        // Check revision matches (conflict detection)
        if existing.rev != rev {
            return Err(format!(
                "Conflict: expected rev '{}', got '{}'",
                existing.rev, rev
            ));
        }
        
        // Generate new revision
        let current_version = Self::parse_rev_version(&existing.rev);
        let new_rev = Self::generate_rev(current_version + 1);
        let now = Utc::now().to_rfc3339();
        
        let doc = NoteDocument {
            id: id.to_string(),
            rev: new_rev,
            data: NoteData {
                title,
                content,
                created_at: existing.data.created_at,
                updated_at: now,
            },
        };
        
        let json = serde_json::to_vec(&doc).map_err(|e| e.to_string())?;
        
        self.db
            .insert(id.as_bytes(), json)
            .map_err(|e| e.to_string())?;
        
        self.db.flush().map_err(|e| e.to_string())?;
        
        Ok(doc)
    }
}

#[tauri::command]
fn update_note(
    id: String,
    rev: String,
    title: String,
    content: String,
    state: tauri::State<'_, AppState>,
) -> Result<NoteDocument, String> {
    let db_guard = state.db.lock().map_err(|e| e.to_string())?;
    let db = db_guard.as_ref().ok_or("Database not initialized")?;
    
    db.update_note(&id, &rev, title, content)
}
```

### Delete Document

```rust
impl Database {
    /// Delete a note (requires correct revision)
    pub fn delete_note(&self, id: &str, rev: &str) -> Result<(), String> {
        // Get existing document
        let existing = self.get_note(id)?
            .ok_or_else(|| format!("Document not found: {}", id))?;
        
        // Check revision matches
        if existing.rev != rev {
            return Err(format!(
                "Conflict: expected rev '{}', got '{}'",
                existing.rev, rev
            ));
        }
        
        self.db
            .remove(id.as_bytes())
            .map_err(|e| e.to_string())?;
        
        self.db.flush().map_err(|e| e.to_string())?;
        
        Ok(())
    }
}

#[tauri::command]
fn delete_note(
    id: String,
    rev: String,
    state: tauri::State<'_, AppState>,
) -> Result<(), String> {
    let db_guard = state.db.lock().map_err(|e| e.to_string())?;
    let db = db_guard.as_ref().ok_or("Database not initialized")?;
    
    db.delete_note(&id, &rev)
}
```

### Frontend Usage

Create a service module for database operations:

```typescript
// src/lib/services/notes.ts
import { invoke } from '@tauri-apps/api/core';

export interface NoteDocument {
  _id: string;
  _rev: string;
  title: string;
  content: string;
  created_at: string;
  updated_at: string;
}

export async function createNote(title: string, content: string): Promise<NoteDocument> {
  return await invoke('create_note', { title, content });
}

export async function getAllNotes(): Promise<NoteDocument[]> {
  return await invoke('get_all_notes');
}

export async function getNote(id: string): Promise<NoteDocument | null> {
  return await invoke('get_note', { id });
}

export async function updateNote(
  id: string,
  rev: string,
  title: string,
  content: string
): Promise<NoteDocument> {
  return await invoke('update_note', { id, rev, title, content });
}

export async function deleteNote(id: string, rev: string): Promise<void> {
  return await invoke('delete_note', { id, rev });
}
```

Create a Svelte store for notes state:

```typescript
// src/lib/stores/notes.ts
import { writable } from 'svelte/store';
import * as notesService from '$lib/services/notes';
import type { NoteDocument } from '$lib/services/notes';

function createNotesStore() {
  const { subscribe, set, update } = writable<NoteDocument[]>([]);

  return {
    subscribe,
    
    load: async () => {
      const notes = await notesService.getAllNotes();
      set(notes);
    },
    
    create: async (title: string, content: string) => {
      const note = await notesService.createNote(title, content);
      update(notes => [note, ...notes]);
      return note;
    },
    
    update: async (id: string, rev: string, title: string, content: string) => {
      const updated = await notesService.updateNote(id, rev, title, content);
      update(notes => notes.map(n => n._id === id ? updated : n));
      return updated;
    },
    
    delete: async (id: string, rev: string) => {
      await notesService.deleteNote(id, rev);
      update(notes => notes.filter(n => n._id !== id));
    }
  };
}

export const notes = createNotesStore();
```

Using in a Svelte component:

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { notes } from '$lib/stores/notes';
  import type { NoteDocument } from '$lib/services/notes';

  let newTitle = '';
  let newContent = '';
  let error = '';

  onMount(() => {
    notes.load();
  });

  async function handleCreate() {
    if (!newTitle.trim()) return;
    
    try {
      await notes.create(newTitle, newContent);
      newTitle = '';
      newContent = '';
    } catch (e) {
      error = String(e);
    }
  }

  async function handleDelete(note: NoteDocument) {
    try {
      await notes.delete(note._id, note._rev);
    } catch (e) {
      if (String(e).includes('Conflict')) {
        // Handle conflict - refresh and retry
        await notes.load();
        error = 'Note was modified. Please try again.';
      } else {
        error = String(e);
      }
    }
  }
</script>

<main class="p-4">
  {#if error}
    <p class="text-red-500 mb-4">{error}</p>
  {/if}

  <form on:submit|preventDefault={handleCreate} class="mb-6">
    <input bind:value={newTitle} placeholder="Title" class="border p-2 mr-2" />
    <textarea bind:value={newContent} placeholder="Content" class="border p-2 mr-2"></textarea>
    <button type="submit" class="bg-blue-500 text-white px-4 py-2">Add Note</button>
  </form>

  {#each $notes as note (note._id)}
    <div class="border p-4 mb-2 rounded">
      <h2 class="font-bold">{note.title}</h2>
      <p>{note.content}</p>
      <p class="text-sm text-gray-500">Rev: {note._rev}</p>
      <button on:click={() => handleDelete(note)} class="text-red-500 mt-2">
        Delete
      </button>
    </div>
  {/each}
</main>
```

---

## Data Migration Strategy

### Migration Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Migration Flow                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Check if migration needed                               │
│     ┌─────────────┐                                        │
│     │ notes.json  │ exists?                                │
│     │ exists?     │──── No ──→ Skip migration              │
│     └──────┬──────┘                                        │
│            │ Yes                                           │
│            ▼                                               │
│  2. Read JSON data                                         │
│     ┌─────────────┐                                        │
│     │ Parse JSON  │                                        │
│     │ file        │                                        │
│     └──────┬──────┘                                        │
│            │                                               │
│            ▼                                               │
│  3. Convert to documents                                   │
│     ┌─────────────┐                                        │
│     │ Add _id,    │                                        │
│     │ _rev fields │                                        │
│     └──────┬──────┘                                        │
│            │                                               │
│            ▼                                               │
│  4. Insert into database                                   │
│     ┌─────────────┐                                        │
│     │ Batch       │                                        │
│     │ insert      │                                        │
│     └──────┬──────┘                                        │
│            │                                               │
│            ▼                                               │
│  5. Rename/backup JSON file                                │
│     ┌─────────────┐                                        │
│     │ notes.json  │ → notes.json.migrated                  │
│     └─────────────┘                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Migration Implementation

```rust
// src-tauri/src/migration.rs

use crate::database::{Database, NoteData, NoteDocument};
use serde::{Deserialize, Serialize};
use std::fs;
use std::path::PathBuf;
use uuid::Uuid;

/// Old note format from JSON file
#[derive(Debug, Deserialize)]
struct OldNote {
    id: String,
    title: String,
    content: String,
    #[serde(default)]
    created_at: Option<String>,
    #[serde(default)]
    updated_at: Option<String>,
}

/// Migration result
#[derive(Debug, Serialize)]
pub struct MigrationResult {
    pub migrated: bool,
    pub notes_count: usize,
    pub message: String,
}

/// Check if migration is needed and perform it
pub fn migrate_if_needed(
    db: &Database,
    app_data_dir: PathBuf,
) -> Result<MigrationResult, String> {
    let json_path = app_data_dir.join("notes.json");
    let backup_path = app_data_dir.join("notes.json.migrated");
    
    // Check if JSON file exists and hasn't been migrated
    if !json_path.exists() {
        return Ok(MigrationResult {
            migrated: false,
            notes_count: 0,
            message: "No migration needed - no JSON file found".to_string(),
        });
    }
    
    // Check if already migrated
    if backup_path.exists() {
        return Ok(MigrationResult {
            migrated: false,
            notes_count: 0,
            message: "Migration already completed".to_string(),
        });
    }
    
    // Read and parse JSON file
    let json_content = fs::read_to_string(&json_path)
        .map_err(|e| format!("Failed to read JSON file: {}", e))?;
    
    let old_notes: Vec<OldNote> = serde_json::from_str(&json_content)
        .map_err(|e| format!("Failed to parse JSON: {}", e))?;
    
    let notes_count = old_notes.len();
    
    // Convert and insert each note
    for old_note in old_notes {
        let now = chrono::Utc::now().to_rfc3339();
        
        // Create document with CouchDB-compatible structure
        let doc = NoteDocument {
            id: format!("note_{}", old_note.id),
            rev: format!("1-{}", &Uuid::new_v4().to_string()[..8]),
            data: NoteData {
                title: old_note.title,
                content: old_note.content,
                created_at: old_note.created_at.unwrap_or_else(|| now.clone()),
                updated_at: old_note.updated_at.unwrap_or_else(|| now.clone()),
            },
        };
        
        db.insert_document(&doc)?;
    }
    
    // Rename JSON file to mark as migrated
    fs::rename(&json_path, &backup_path)
        .map_err(|e| format!("Failed to backup JSON file: {}", e))?;
    
    Ok(MigrationResult {
        migrated: true,
        notes_count,
        message: format!("Successfully migrated {} notes", notes_count),
    })
}
```

### Add Insert Method to Database

```rust
// Add to src-tauri/src/database.rs

impl Database {
    /// Insert a document directly (used for migration)
    pub fn insert_document(&self, doc: &NoteDocument) -> Result<(), String> {
        let json = serde_json::to_vec(doc).map_err(|e| e.to_string())?;
        
        self.db
            .insert(doc.id.as_bytes(), json)
            .map_err(|e| e.to_string())?;
        
        Ok(())
    }
}
```

### Tauri Command for Migration

```rust
// src-tauri/src/lib.rs

mod migration;

use migration::{migrate_if_needed, MigrationResult};

#[tauri::command]
fn run_migration(
    app: tauri::AppHandle,
    state: tauri::State<'_, AppState>,
) -> Result<MigrationResult, String> {
    let db_guard = state.db.lock().map_err(|e| e.to_string())?;
    let db = db_guard.as_ref().ok_or("Database not initialized")?;
    
    let app_data_dir = app.path().app_data_dir().map_err(|e| e.to_string())?;
    
    migrate_if_needed(db, app_data_dir)
}
```

### Frontend Migration Flow

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { invoke } from '@tauri-apps/api/core';
  import { notes } from '$lib/stores/notes';

  interface MigrationResult {
    migrated: boolean;
    notes_count: number;
    message: string;
  }

  let loading = true;
  let migrationStatus = '';
  let error = '';

  onMount(async () => {
    try {
      // 1. Initialize database
      await invoke('init_database');
      
      // 2. Run migration if needed
      const result = await invoke<MigrationResult>('run_migration');
      
      if (result.migrated) {
        migrationStatus = `Migrated ${result.notes_count} notes from JSON`;
      }
      
      // 3. Load notes
      await notes.load();
      
    } catch (e) {
      console.error('Initialization failed:', e);
      error = String(e);
    } finally {
      loading = false;
    }
  });
</script>

{#if loading}
  <div class="p-4">Initializing...</div>
{:else if error}
  <div class="p-4 text-red-500">Error: {error}</div>
{:else}
  {#if migrationStatus}
    <div class="p-4 bg-green-100 text-green-800 mb-4">
      {migrationStatus}
    </div>
  {/if}
  
  <!-- Rest of your app -->
  {#each $notes as note (note._id)}
    <div class="border p-4 mb-2">
      <h2 class="font-bold">{note.title}</h2>
      <p>{note.content}</p>
    </div>
  {/each}
{/if}
```

---

## Cross-Platform Considerations

### Same Data Model

Use the same document structure for both desktop and web:

```typescript
// shared/types.ts (can be shared between projects)

export interface NoteDocument {
  _id: string;
  _rev: string;
  title: string;
  content: string;
  created_at: string;
  updated_at: string;
}
```

### Web Implementation with PouchDB

For the web version, use PouchDB directly:

```typescript
// web/src/database.ts
import PouchDB from 'pouchdb';

const db = new PouchDB('notes');

export async function createNote(title: string, content: string): Promise<NoteDocument> {
  const now = new Date().toISOString();
  const doc = {
    _id: `note_${crypto.randomUUID()}`,
    title,
    content,
    created_at: now,
    updated_at: now,
  };
  
  const result = await db.put(doc);
  return { ...doc, _rev: result.rev };
}

export async function getAllNotes(): Promise<NoteDocument[]> {
  const result = await db.allDocs({ include_docs: true });
  return result.rows
    .filter(row => row.id.startsWith('note_'))
    .map(row => row.doc as NoteDocument);
}

export async function updateNote(
  id: string,
  rev: string,
  title: string,
  content: string
): Promise<NoteDocument> {
  const existing = await db.get(id);
  const updated = {
    ...existing,
    title,
    content,
    updated_at: new Date().toISOString(),
  };
  
  const result = await db.put(updated);
  return { ...updated, _rev: result.rev };
}

export async function deleteNote(id: string, rev: string): Promise<void> {
  await db.remove(id, rev);
}
```

### Abstraction Layer

Create a unified interface:

```typescript
// shared/dataAccess.ts

export interface DataAccess {
  createNote(title: string, content: string): Promise<NoteDocument>;
  getAllNotes(): Promise<NoteDocument[]>;
  getNote(id: string): Promise<NoteDocument | null>;
  updateNote(id: string, rev: string, title: string, content: string): Promise<NoteDocument>;
  deleteNote(id: string, rev: string): Promise<void>;
}

// Desktop implementation (uses Tauri invoke)
export class TauriDataAccess implements DataAccess {
  async createNote(title: string, content: string): Promise<NoteDocument> {
    return await invoke('create_note', { title, content });
  }
  // ... other methods
}

// Web implementation (uses PouchDB)
export class PouchDBDataAccess implements DataAccess {
  private db = new PouchDB('notes');
  
  async createNote(title: string, content: string): Promise<NoteDocument> {
    // PouchDB implementation
  }
  // ... other methods
}
```

---

## Optional: Sync with CouchDB

### Setting Up CouchDB

**Using Docker:**

```bash
docker run -d --name couchdb \
  -e COUCHDB_USER=admin \
  -e COUCHDB_PASSWORD=password \
  -p 5984:5984 \
  couchdb:latest
```

**Create a database:**

```bash
curl -X PUT http://admin:password@localhost:5984/notes
```

### Enable CORS

```bash
curl -X PUT http://admin:password@localhost:5984/_node/_local/_config/httpd/enable_cors \
  -d '"true"'

curl -X PUT http://admin:password@localhost:5984/_node/_local/_config/cors/origins \
  -d '"*"'

curl -X PUT http://admin:password@localhost:5984/_node/_local/_config/cors/methods \
  -d '"GET, PUT, POST, HEAD, DELETE"'

curl -X PUT http://admin:password@localhost:5984/_node/_local/_config/cors/credentials \
  -d '"true"'
```

### PouchDB Sync (Web)

```typescript
import PouchDB from 'pouchdb';

const localDB = new PouchDB('notes');
const remoteDB = new PouchDB('http://admin:password@localhost:5984/notes');

// One-time sync
await localDB.sync(remoteDB);

// Live sync (continuous)
const syncHandler = localDB.sync(remoteDB, {
  live: true,
  retry: true,
})
.on('change', (info) => {
  console.log('Sync change:', info);
})
.on('error', (err) => {
  console.error('Sync error:', err);
});

// Stop sync when needed
// syncHandler.cancel();
```

### Rust Sync (Desktop)

For the desktop app, you can use the `couch_rs` crate:

```rust
// Cargo.toml
[dependencies]
couch_rs = "0.9"
```

```rust
use couch_rs::Client;
use couch_rs::types::find::FindQuery;

async fn sync_to_server(local_db: &Database) -> Result<(), String> {
    let client = Client::new("http://localhost:5984", "admin", "password")
        .map_err(|e| e.to_string())?;
    
    let remote_db = client.db("notes").await.map_err(|e| e.to_string())?;
    
    // Get all local notes
    let local_notes = local_db.get_all_notes()?;
    
    // Push to remote
    for note in local_notes {
        let doc = serde_json::to_value(&note).map_err(|e| e.to_string())?;
        remote_db.upsert(&doc).await.map_err(|e| e.to_string())?;
    }
    
    Ok(())
}
```

---

## Hands-On Exercises

### Exercise 1: Set Up Database
1. Add sled dependency to your Tauri project
2. Create the database module
3. Initialize database on app startup
4. Verify database file is created in app data directory

### Exercise 2: Implement CRUD
1. Implement `create_note` command
2. Implement `get_all_notes` command
3. Implement `update_note` with revision checking
4. Implement `delete_note` with revision checking
5. Test all operations from the frontend

### Exercise 3: Data Migration
1. Create a test `notes.json` file with sample data
2. Implement the migration function
3. Run migration on app startup
4. Verify notes are in the database
5. Verify JSON file is renamed to `.migrated`

### Exercise 4: Handle Conflicts
1. Try to update a note with wrong revision
2. Implement proper error handling in frontend
3. Show user-friendly error message
4. Implement retry logic (re-fetch and retry)

### Exercise 5: Cross-Platform Prep
1. Create shared TypeScript types
2. Create data access abstraction
3. Implement PouchDB version for web
4. Test both implementations work the same

---

## Common Pitfalls

### 1. Forgetting Revision on Update

```typescript
// ❌ Wrong - missing revision
await invoke('update_note', { id, title, content });

// ✅ Correct - include revision
await invoke('update_note', { id, rev: note._rev, title, content });
```

### 2. Not Handling Conflicts

```typescript
// ❌ Wrong - no conflict handling
const updateNote = async (note: NoteDocument) => {
  await invoke('update_note', { ...note });
};

// ✅ Correct - handle conflicts
const updateNote = async (note: NoteDocument) => {
  try {
    return await invoke('update_note', { ...note });
  } catch (error) {
    if (error.includes('Conflict')) {
      // Re-fetch and retry or show user
      const latest = await invoke('get_note', { id: note._id });
      // Handle conflict...
    }
    throw error;
  }
};
```

### 3. Not Flushing Database

```rust
// ❌ Wrong - data might be lost on crash
self.db.insert(id.as_bytes(), json)?;

// ✅ Correct - ensure durability
self.db.insert(id.as_bytes(), json)?;
self.db.flush()?;
```

---

## Summary

In this phase, you learned:
- ✅ Why document databases are better than JSON files for this use case
- ✅ PouchDB/CouchDB concepts: documents, revisions, conflicts
- ✅ How to set up an embedded database in Tauri
- ✅ How to implement CRUD operations with revision handling
- ✅ How to migrate existing JSON data to the database
- ✅ How to design for cross-platform (desktop + web)
- ✅ (Optional) How to sync with a CouchDB server

---

## Additional Resources

- [PouchDB Documentation](https://pouchdb.com/guides/)
- [CouchDB Documentation](https://docs.couchdb.org/)
- [Sled Documentation](https://docs.rs/sled/)
- [couch_rs Crate](https://crates.io/crates/couch_rs)

---

## Next Steps

Test your knowledge: [Phase 8 Quiz](./03-quiz.md)