# Phase 8: Database Migration (JSON to PouchDB/CouchDB) - Quiz

Test your understanding of document databases, data migration, and cross-platform data strategies.

---

## Multiple Choice Questions

### Question 1
What is the main limitation of using JSON files for data storage in a note-taking app?

- A) JSON files cannot store text data
- B) You must load the entire file to read or modify any single note
- C) JSON files are not supported in Rust
- D) JSON files cannot be read by JavaScript

<details>
<summary>Show Answer</summary>

**B) You must load the entire file to read or modify any single note**

With JSON file storage, every operation requires reading the entire file, parsing all data, making changes in memory, and writing the entire file back. This is inefficient for large datasets.
</details>

---

### Question 2
What are the two special fields that every PouchDB/CouchDB document must have?

- A) `id` and `version`
- B) `_id` and `_rev`
- C) `key` and `value`
- D) `uuid` and `timestamp`

<details>
<summary>Show Answer</summary>

**B) `_id` and `_rev`**

Every document in PouchDB/CouchDB has:
- `_id`: Unique identifier for the document
- `_rev`: Revision string for conflict detection
</details>

---

### Question 3
What is the purpose of the `_rev` field in a CouchDB document?

- A) To store the document's creation date
- B) To track who last modified the document
- C) To detect conflicts and prevent accidental overwrites
- D) To compress the document data

<details>
<summary>Show Answer</summary>

**C) To detect conflicts and prevent accidental overwrites**

The `_rev` field is a version identifier. When updating a document, you must provide the current revision. If someone else modified the document, the revisions won't match, and the update will fail with a conflict error.
</details>

---

### Question 4
What happens if you try to update a document with an incorrect `_rev` value?

- A) The update succeeds anyway
- B) The document is deleted
- C) A conflict error is returned
- D) A new document is created

<details>
<summary>Show Answer</summary>

**C) A conflict error is returned**

This is the conflict detection mechanism. If the `_rev` you provide doesn't match the current revision in the database, the operation fails, preventing you from accidentally overwriting someone else's changes.
</details>

---

### Question 5
Which Rust crate is recommended for embedded key-value storage in this learning module?

- A) `sqlite`
- B) `rocksdb`
- C) `sled`
- D) `leveldb`

<details>
<summary>Show Answer</summary>

**C) `sled`**

Sled is a pure Rust embedded database that provides:
- ACID transactions
- Zero-copy reads
- Simple API
- No external dependencies
</details>

---

### Question 6
What is the correct format for a CouchDB revision string?

- A) `abc123def456`
- B) `1-abc123def456`
- C) `v1.0.0`
- D) `2024-01-15T10:30:00Z`

<details>
<summary>Show Answer</summary>

**B) `1-abc123def456`**

The revision format is `{version}-{hash}` where:
- `version` is an incrementing number (1, 2, 3, ...)
- `hash` is a unique identifier for that version
</details>

---

### Question 7
What should happen when the app starts and finds an existing `notes.json` file?

- A) Delete the JSON file immediately
- B) Ignore the JSON file
- C) Migrate the data to the database and rename the JSON file
- D) Keep using the JSON file instead of the database

<details>
<summary>Show Answer</summary>

**C) Migrate the data to the database and rename the JSON file**

The migration process should:
1. Read the JSON file
2. Convert notes to database documents
3. Insert into the database
4. Rename the JSON file (e.g., to `.migrated`) as a backup
</details>

---

### Question 8
Why is it important to call `db.flush()` after database operations in sled?

- A) To free up memory
- B) To ensure data is written to disk (durability)
- C) To close the database connection
- D) To compress the data

<details>
<summary>Show Answer</summary>

**B) To ensure data is written to disk (durability)**

Without flushing, data might only be in memory. If the app crashes before the data is written to disk, it could be lost. `flush()` ensures durability.
</details>

---

### Question 9
What is "offline-first" design?

- A) An app that only works without internet
- B) An app that works fully offline and syncs when online
- C) An app that requires internet to start
- D) An app that caches images offline

<details>
<summary>Show Answer</summary>

**B) An app that works fully offline and syncs when online**

Offline-first means:
- The app stores data locally and works without network
- When network is available, it syncs with a server
- Users never see "no connection" errors for basic operations
</details>

---

### Question 10
Which database runs in the browser using IndexedDB as its storage backend?

- A) CouchDB
- B) MongoDB
- C) PouchDB
- D) PostgreSQL

<details>
<summary>Show Answer</summary>

**C) PouchDB**

PouchDB is a JavaScript database that:
- Runs in the browser
- Uses IndexedDB for storage
- Syncs with CouchDB-compatible servers
- Has the same API in browser and Node.js
</details>

---

## Code Analysis

### Question 11
What's wrong with this update function?

```rust
pub fn update_note(&self, id: &str, title: String, content: String) -> Result<NoteDocument, String> {
    let existing = self.get_note(id)?
        .ok_or_else(|| format!("Document not found: {}", id))?;
    
    let doc = NoteDocument {
        id: id.to_string(),
        rev: existing.rev.clone(),  // Keep same revision
        data: NoteData {
            title,
            content,
            created_at: existing.data.created_at,
            updated_at: Utc::now().to_rfc3339(),
        },
    };
    
    let json = serde_json::to_vec(&doc).map_err(|e| e.to_string())?;
    self.db.insert(id.as_bytes(), json).map_err(|e| e.to_string())?;
    
    Ok(doc)
}
```

<details>
<summary>Show Answer</summary>

**Problems:**

1. **No revision parameter** - The function doesn't accept the client's revision, so it can't detect conflicts
2. **Keeps same revision** - The revision should be incremented on update
3. **No flush** - Data might not be persisted to disk

**Correct version:**
```rust
pub fn update_note(
    &self,
    id: &str,
    rev: &str,  // Accept revision from client
    title: String,
    content: String,
) -> Result<NoteDocument, String> {
    let existing = self.get_note(id)?
        .ok_or_else(|| format!("Document not found: {}", id))?;
    
    // Check revision matches (conflict detection)
    if existing.rev != rev {
        return Err(format!("Conflict: expected rev '{}', got '{}'", existing.rev, rev));
    }
    
    // Generate NEW revision
    let current_version = Self::parse_rev_version(&existing.rev);
    let new_rev = Self::generate_rev(current_version + 1);
    
    let doc = NoteDocument {
        id: id.to_string(),
        rev: new_rev,
        // ... rest of data
    };
    
    // ... insert and flush
    self.db.flush()?;
    
    Ok(doc)
}
```
</details>

---

### Question 12
What's wrong with this frontend code?

```typescript
const updateNote = async (note: NoteDocument, newTitle: string) => {
  await invoke('update_note', {
    id: note._id,
    title: newTitle,
    content: note.content,
  });
};
```

<details>
<summary>Show Answer</summary>

**Problem:** Missing the `rev` parameter

The revision is required for conflict detection. Without it, the backend can't verify that you're updating the version you think you're updating.

**Correct version:**
```typescript
const updateNote = async (note: NoteDocument, newTitle: string) => {
  await invoke('update_note', {
    id: note._id,
    rev: note._rev,  // Include revision!
    title: newTitle,
    content: note.content,
  });
};
```
</details>

---

### Question 13
What's the issue with this migration code?

```rust
pub fn migrate_if_needed(db: &Database, app_data_dir: PathBuf) -> Result<MigrationResult, String> {
    let json_path = app_data_dir.join("notes.json");
    
    if !json_path.exists() {
        return Ok(MigrationResult { migrated: false, notes_count: 0, message: "No file".to_string() });
    }
    
    let json_content = fs::read_to_string(&json_path)?;
    let old_notes: Vec<OldNote> = serde_json::from_str(&json_content)?;
    
    for old_note in old_notes {
        db.insert_document(&convert_note(old_note))?;
    }
    
    // Delete the JSON file
    fs::remove_file(&json_path)?;
    
    Ok(MigrationResult { migrated: true, notes_count: old_notes.len(), message: "Done".to_string() })
}
```

<details>
<summary>Show Answer</summary>

**Problems:**

1. **Deletes JSON file** - Should rename/backup instead of delete. If something goes wrong, the original data is lost.
2. **No check for previous migration** - If the app crashes after partial migration, running again could create duplicates.
3. **Error handling** - Uses `?` which doesn't provide helpful error messages.

**Correct approach:**
```rust
// Check if already migrated
let backup_path = app_data_dir.join("notes.json.migrated");
if backup_path.exists() {
    return Ok(MigrationResult { migrated: false, ... });
}

// ... do migration ...

// Rename instead of delete (keeps backup)
fs::rename(&json_path, &backup_path)?;
```
</details>

---

## Fill in the Blanks

### Question 14
Complete the code to create a new document with proper ID and revision:

```rust
pub fn create_note(&self, title: String, content: String) -> Result<NoteDocument, String> {
    let now = Utc::now().to_rfc3339();
    let id = format!("note_{}", _______::new_v4());
    let rev = Self::generate_rev(_______);
    
    let doc = NoteDocument {
        id: id.clone(),
        rev,
        data: NoteData { title, content, created_at: now.clone(), updated_at: now },
    };
    
    // ... save to database
}
```

<details>
<summary>Show Answer</summary>

```rust
let id = format!("note_{}", Uuid::new_v4());
let rev = Self::generate_rev(1);  // First version is 1
```

- `Uuid::new_v4()` generates a random UUID for unique IDs
- First revision is always version `1`
</details>

---

### Question 15
Complete the PouchDB sync setup:

```typescript
const localDB = new PouchDB('notes');
const remoteDB = new PouchDB('http://localhost:5984/notes');

// Start live sync
const syncHandler = localDB._______(remoteDB, {
  _______: true,  // Keep syncing continuously
  retry: true,    // Retry on connection loss
});
```

<details>
<summary>Show Answer</summary>

```typescript
const syncHandler = localDB.sync(remoteDB, {
  live: true,
  retry: true,
});
```

- `sync()` sets up bidirectional replication
- `live: true` keeps the sync running continuously
- `retry: true` automatically retries on network errors
</details>

---

## True or False

### Question 16
True or False: In PouchDB/CouchDB, you can update a document without knowing its current revision.

<details>
<summary>Show Answer</summary>

**False**

You must provide the current `_rev` when updating. This is how the database detects conflicts. If you don't have the correct revision, the update will fail.
</details>

---

### Question 17
True or False: PouchDB can only run in web browsers.

<details>
<summary>Show Answer</summary>

**False**

PouchDB can run in:
- Web browsers (using IndexedDB)
- Node.js (using LevelDB)
- React Native
- Electron

It has the same API everywhere.
</details>

---

### Question 18
True or False: When migrating from JSON to a database, you should delete the original JSON file immediately after migration.

<details>
<summary>Show Answer</summary>

**False**

You should **rename** the file (e.g., to `notes.json.migrated`) instead of deleting it. This:
- Provides a backup if something goes wrong
- Prevents re-running migration (the check for `.migrated` file)
- Allows users to recover data if needed
</details>

---

## Short Answer

### Question 19
Explain why PouchDB/CouchDB is a good choice for an app that needs to work on both desktop (Tauri) and web.

<details>
<summary>Show Answer</summary>

**Reasons:**

1. **Same data model**: Both platforms use the same document structure (`_id`, `_rev`, fields)

2. **Offline-first**: Works without network on both platforms

3. **Built-in sync**: Can sync between desktop and web through a CouchDB server

4. **PouchDB in browser**: Native JavaScript library for web

5. **CouchDB protocol**: Desktop can use any CouchDB-compatible client or implement the same document structure

6. **Conflict resolution**: Built-in handling for when the same note is edited on multiple devices

7. **Mature ecosystem**: Well-documented, battle-tested, large community
</details>

---

### Question 20
What is the difference between a "soft delete" and a "hard delete" in a document database, and when might you use each?

<details>
<summary>Show Answer</summary>

**Soft Delete:**
- Mark document as deleted (e.g., add `deleted: true` field)
- Document still exists in database
- Can be "undeleted" or recovered
- Sync will propagate the "deleted" status

**Hard Delete:**
- Actually remove the document from database
- Cannot be recovered (unless you have backups)
- In CouchDB, creates a "tombstone" for sync purposes

**When to use:**

**Soft delete** when:
- Users might want to recover deleted items
- You need an audit trail
- Syncing with other devices (they need to know it was deleted)

**Hard delete** when:
- Privacy requirements (data must be truly gone)
- Storage space is critical
- No sync needed
</details>

---

## Practical Exercise

### Question 21
Write a function that handles update conflicts gracefully by re-fetching the document and retrying:

```typescript
// Implement this function
async function updateNoteWithRetry(
  id: string,
  updateFn: (note: NoteDocument) => { title: string; content: string },
  maxRetries: number = 3
): Promise<NoteDocument> {
  // Your implementation here
}
```

<details>
<summary>Show Answer</summary>

```typescript
async function updateNoteWithRetry(
  id: string,
  updateFn: (note: NoteDocument) => { title: string; content: string },
  maxRetries: number = 3
): Promise<NoteDocument> {
  let lastError: Error | null = null;
  
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      // 1. Fetch the latest version
      const note = await invoke<NoteDocument>('get_note', { id });
      
      if (!note) {
        throw new Error(`Note not found: ${id}`);
      }
      
      // 2. Apply the update function to get new values
      const { title, content } = updateFn(note);
      
      // 3. Try to update with current revision
      const updated = await invoke<NoteDocument>('update_note', {
        id: note._id,
        rev: note._rev,
        title,
        content,
      });
      
      return updated;
      
    } catch (error) {
      lastError = error as Error;
      
      // Check if it's a conflict error
      if (error.toString().includes('Conflict')) {
        console.log(`Conflict on attempt ${attempt + 1}, retrying...`);
        continue; // Retry with fresh data
      }
      
      // For other errors, don't retry
      throw error;
    }
  }
  
  throw new Error(`Failed after ${maxRetries} attempts: ${lastError?.message}`);
}

// Usage example:
const updated = await updateNoteWithRetry(
  'note_123',
  (note) => ({
    title: note.title + ' (edited)',
    content: note.content,
  })
);
```

Key points:
- Fetches latest version before each attempt
- Uses the update function to compute new values
- Retries only on conflict errors
- Throws immediately for other errors
- Has a maximum retry limit
</details>

---

### Question 22
Write the Rust code to check if a migration is needed and return appropriate status:

```rust
// Implement this function
pub fn check_migration_status(app_data_dir: &PathBuf) -> MigrationStatus {
    // Return one of:
    // - NotNeeded (no JSON file exists)
    // - AlreadyCompleted (backup file exists)
    // - Required (JSON file exists, needs migration)
}
```

<details>
<summary>Show Answer</summary>

```rust
#[derive(Debug, Serialize)]
pub enum MigrationStatus {
    NotNeeded,
    AlreadyCompleted,
    Required { notes_count: usize },
}

pub fn check_migration_status(app_data_dir: &PathBuf) -> Result<MigrationStatus, String> {
    let json_path = app_data_dir.join("notes.json");
    let backup_path = app_data_dir.join("notes.json.migrated");
    
    // Check if backup exists (already migrated)
    if backup_path.exists() {
        return Ok(MigrationStatus::AlreadyCompleted);
    }
    
    // Check if JSON file exists
    if !json_path.exists() {
        return Ok(MigrationStatus::NotNeeded);
    }
    
    // JSON exists, count notes to migrate
    let json_content = fs::read_to_string(&json_path)
        .map_err(|e| format!("Failed to read JSON: {}", e))?;
    
    let notes: Vec<serde_json::Value> = serde_json::from_str(&json_content)
        .map_err(|e| format!("Failed to parse JSON: {}", e))?;
    
    Ok(MigrationStatus::Required {
        notes_count: notes.len(),
    })
}

// Tauri command
#[tauri::command]
fn get_migration_status(app: tauri::AppHandle) -> Result<MigrationStatus, String> {
    let app_data_dir = app.path().app_data_dir().map_err(|e| e.to_string())?;
    check_migration_status(&app_data_dir)
}
```

Key points:
- Checks backup file first (most common case after migration)
- Returns enum with different states
- Includes note count for UI feedback
- Handles errors gracefully
</details>

---

## Score Yourself

| Score | Rating |
|-------|--------|
| 20-22 | ⭐⭐⭐ Excellent! You've mastered database migration |
| 15-19 | ⭐⭐ Good! Review the concepts you missed |
| 10-14 | ⭐ Fair. Re-read the learning materials |
| 0-9 | Need more practice. Start from the beginning |

---

## Key Takeaways

1. **Document databases** solve JSON file limitations (performance, concurrency, queries)
2. **Revisions** (`_rev`) are essential for conflict detection
3. **Always include revision** when updating or deleting documents
4. **Migration should be safe** - backup original data, check for previous migration
5. **Cross-platform design** - use same data model for desktop and web
6. **Offline-first** - app works without network, syncs when available

---

## Next Steps

Congratulations on completing Phase 8! 🎉

You now understand:
- Why and when to migrate from JSON to a database
- PouchDB/CouchDB document model and revision system
- How to implement CRUD with conflict detection
- Data migration strategies
- Cross-platform data architecture

Consider exploring:
- [PouchDB Guides](https://pouchdb.com/guides/)
- [CouchDB Documentation](https://docs.couchdb.org/)
- Building a web version of your note app with PouchDB
- Setting up CouchDB for multi-device sync