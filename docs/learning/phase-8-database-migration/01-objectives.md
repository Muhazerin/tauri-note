# Phase 8: Database Migration (JSON to PouchDB/CouchDB) - Learning Objectives

## Overview
In this phase, you will learn how to migrate from JSON file storage to a proper document database using PouchDB/CouchDB. This approach enables offline-first functionality, cross-platform compatibility (desktop and web), and optional cloud synchronization.

---

## Why This Phase?

After learning JSON file storage in Phase 3, you might wonder: "Why do I need a database?" Here's why:

| JSON Files | PouchDB/CouchDB |
|------------|-----------------|
| Simple for small data | Scales to large datasets |
| No query capabilities | Rich querying with indexes |
| Manual conflict handling | Built-in conflict resolution |
| Desktop only | Works on desktop, web, and mobile |
| No sync | Built-in replication/sync |
| Load entire file | Efficient partial reads |

---

## Learning Objectives

After completing this phase, you should be able to:

### 1. Understand Document Database Concepts
- [ ] Explain the difference between JSON files and document databases
- [ ] Understand PouchDB/CouchDB architecture
- [ ] Describe the revision system and why it matters
- [ ] Explain offline-first design principles

### 2. Set Up PouchDB in Tauri
- [ ] Choose the right approach for Rust backend
- [ ] Configure database storage location
- [ ] Initialize the database on app startup
- [ ] Handle database connections properly

### 3. Implement CRUD Operations
- [ ] Create new documents (notes)
- [ ] Read documents by ID and query all documents
- [ ] Update documents with proper revision handling
- [ ] Delete documents (soft delete vs. hard delete)

### 4. Migrate Existing Data
- [ ] Detect if migration is needed
- [ ] Read existing JSON data
- [ ] Convert JSON to PouchDB documents
- [ ] Handle migration errors gracefully
- [ ] Clean up after successful migration

### 5. Design for Cross-Platform
- [ ] Structure data for desktop and web compatibility
- [ ] Understand PouchDB in browser vs. Rust backend
- [ ] Create a consistent data access pattern

### 6. (Optional) Configure Sync
- [ ] Set up a CouchDB server
- [ ] Configure replication between local and remote
- [ ] Handle sync conflicts

---

## Key Concepts to Master

| Concept | Description |
|---------|-------------|
| Document | A JSON object with `_id` and `_rev` fields |
| Revision (`_rev`) | Version identifier for conflict detection |
| Replication | Syncing data between databases |
| Conflict | When the same document is modified in multiple places |
| View/Index | Pre-computed query results for fast lookups |
| Offline-First | App works without network, syncs when available |

---

## PouchDB/CouchDB Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Your App                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐              ┌─────────────────┐      │
│  │  Desktop App    │              │    Web App      │      │
│  │  (Tauri/Rust)   │              │   (Browser)     │      │
│  │                 │              │                 │      │
│  │  ┌───────────┐  │              │  ┌───────────┐  │      │
│  │  │  Rust DB  │  │              │  │  PouchDB  │  │      │
│  │  │  Layer    │  │              │  │ (IndexedDB│  │      │
│  │  │           │  │              │  │  backend) │  │      │
│  │  └─────┬─────┘  │              │  └─────┬─────┘  │      │
│  └────────┼────────┘              └────────┼────────┘      │
│           │                                │               │
│           └──────────────┬─────────────────┘               │
│                          │                                 │
│                          ▼ (Optional Sync)                 │
│               ┌─────────────────────┐                      │
│               │   CouchDB Server    │                      │
│               │   (Self-hosted or   │                      │
│               │    Cloud Service)   │                      │
│               └─────────────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

---

## Success Criteria

You have successfully completed this phase when you can:

1. ✅ Explain why PouchDB/CouchDB is better than JSON files for this use case
2. ✅ Store and retrieve notes using the database
3. ✅ Successfully migrate existing JSON notes to the database
4. ✅ Handle document revisions correctly during updates
5. ✅ App continues to work offline
6. ✅ Understand how to extend this to a web app
7. ✅ (Bonus) Set up sync with a CouchDB server

---

## Prerequisites

Before starting this phase, ensure you have completed:
- Phase 2: IPC & Commands (for Tauri command patterns)
- Phase 3: File System (for understanding current JSON storage)

---

## Estimated Time

| Section | Time |
|---------|------|
| Understanding concepts | 30 min |
| Setting up database | 30 min |
| Implementing CRUD | 1 hour |
| Data migration | 45 min |
| Testing & debugging | 30 min |
| (Optional) Sync setup | 1 hour |
| **Total** | **2-4 hours** |

---

## Next Steps

Ready to learn? Continue to [Learning Materials](./02-materials.md)