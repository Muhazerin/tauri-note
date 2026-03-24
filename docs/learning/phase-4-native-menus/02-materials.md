# Phase 4: Native Menus - Learning Materials

## Table of Contents
1. [Introduction to Native Menus](#introduction-to-native-menus)
2. [Creating Menus in Tauri v2](#creating-menus-in-tauri-v2)
3. [Keyboard Shortcuts](#keyboard-shortcuts)
4. [Handling Menu Events](#handling-menu-events)
5. [Complete Example](#complete-example)
6. [Hands-On Exercises](#hands-on-exercises)

---

## Introduction to Native Menus

Native menus provide a familiar interface for users and follow platform conventions. In Tauri, menus are created in Rust and can trigger actions in both the backend and frontend.

### Menu Types
- **Application Menu**: The main menu bar (File, Edit, View, Help)
- **Context Menu**: Right-click menus
- **Tray Menu**: System tray icon menus (covered in Phase 5)

---

## Creating Menus in Tauri v2

### Basic Menu Structure

```rust
use tauri::menu::{Menu, MenuItem, Submenu};

fn create_menu(app: &tauri::AppHandle) -> Result<Menu<tauri::Wry>, tauri::Error> {
    let menu = Menu::new(app)?;
    
    // Create File submenu
    let file_menu = Submenu::new(app, "File", true)?;
    file_menu.append(&MenuItem::new(app, "New", true, Some("CmdOrCtrl+N"))?)?;
    file_menu.append(&MenuItem::new(app, "Save", true, Some("CmdOrCtrl+S"))?)?;
    file_menu.append(&MenuItem::new(app, "Quit", true, Some("CmdOrCtrl+Q"))?)?;
    
    menu.append(&file_menu)?;
    
    Ok(menu)
}
```

### Menu Item Types

```rust
use tauri::menu::{MenuItem, CheckMenuItem, IconMenuItem, PredefinedMenuItem};

// Regular menu item
let new_item = MenuItem::new(app, "New Note", true, Some("CmdOrCtrl+N"))?;

// Check menu item (toggleable)
let auto_save = CheckMenuItem::new(app, "Auto Save", true, true, None)?;

// Predefined items (platform-specific)
let separator = PredefinedMenuItem::separator(app)?;
let quit = PredefinedMenuItem::quit(app, Some("Quit"))?;
let copy = PredefinedMenuItem::copy(app, Some("Copy"))?;
let paste = PredefinedMenuItem::paste(app, Some("Paste"))?;
```

### Setting the Menu

```rust
// src-tauri/src/lib.rs
use tauri::Manager;

pub fn run() {
    tauri::Builder::default()
        .setup(|app| {
            let menu = create_menu(app.handle())?;
            app.set_menu(menu)?;
            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

---

## Keyboard Shortcuts

### Accelerator Syntax

| Modifier | Windows/Linux | macOS |
|----------|---------------|-------|
| `CmdOrCtrl` | Ctrl | Cmd |
| `Ctrl` | Ctrl | Ctrl |
| `Cmd` | - | Cmd |
| `Alt` | Alt | Option |
| `Shift` | Shift | Shift |

### Common Shortcuts

```rust
// Cross-platform shortcuts
Some("CmdOrCtrl+N")  // New
Some("CmdOrCtrl+S")  // Save
Some("CmdOrCtrl+O")  // Open
Some("CmdOrCtrl+Q")  // Quit
Some("CmdOrCtrl+Z")  // Undo
Some("CmdOrCtrl+Shift+Z")  // Redo

// Combinations
Some("CmdOrCtrl+Shift+S")  // Save As
Some("Alt+F4")  // Close (Windows)
```

### Platform-Specific Menus

```rust
#[cfg(target_os = "macos")]
fn create_macos_menu(app: &tauri::AppHandle) -> Result<Menu<tauri::Wry>, tauri::Error> {
    let menu = Menu::new(app)?;
    
    // macOS requires an app menu with the app name
    let app_menu = Submenu::new(app, "Tauri Note", true)?;
    app_menu.append(&PredefinedMenuItem::about(app, Some("About Tauri Note"), None)?)?;
    app_menu.append(&PredefinedMenuItem::separator(app)?)?;
    app_menu.append(&PredefinedMenuItem::quit(app, Some("Quit Tauri Note"))?)?;
    
    menu.append(&app_menu)?;
    
    Ok(menu)
}
```

---

## Handling Menu Events

### Using Menu Item IDs

```rust
use tauri::menu::{Menu, MenuItem, MenuId};

fn create_menu(app: &tauri::AppHandle) -> Result<Menu<tauri::Wry>, tauri::Error> {
    let menu = Menu::new(app)?;
    
    let file_menu = Submenu::new(app, "File", true)?;
    
    // Create items with specific IDs
    let new_item = MenuItem::with_id(app, "new_note", "New Note", true, Some("CmdOrCtrl+N"))?;
    let save_item = MenuItem::with_id(app, "save_note", "Save", true, Some("CmdOrCtrl+S"))?;
    let quit_item = MenuItem::with_id(app, "quit", "Quit", true, Some("CmdOrCtrl+Q"))?;
    
    file_menu.append(&new_item)?;
    file_menu.append(&save_item)?;
    file_menu.append(&PredefinedMenuItem::separator(app)?)?;
    file_menu.append(&quit_item)?;
    
    menu.append(&file_menu)?;
    
    Ok(menu)
}
```

### Listening for Menu Events

```rust
use tauri::Manager;

pub fn run() {
    tauri::Builder::default()
        .setup(|app| {
            let menu = create_menu(app.handle())?;
            app.set_menu(menu)?;
            
            // Listen for menu events
            app.on_menu_event(|app, event| {
                match event.id().as_ref() {
                    "new_note" => {
                        println!("New note clicked!");
                        // Emit event to frontend
                        app.emit("menu-new-note", ()).unwrap();
                    }
                    "save_note" => {
                        println!("Save clicked!");
                        app.emit("menu-save-note", ()).unwrap();
                    }
                    "quit" => {
                        app.exit(0);
                    }
                    _ => {}
                }
            });
            
            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Listening in the Frontend

```typescript
import { listen } from '@tauri-apps/api/event';

// Listen for menu events
listen('menu-new-note', () => {
    console.log('New note from menu');
    // Create new note
});

listen('menu-save-note', () => {
    console.log('Save from menu');
    // Save current note
});
```

---

## Complete Example

### Full Menu Setup

```rust
// src-tauri/src/menu.rs

use tauri::menu::{Menu, MenuItem, Submenu, PredefinedMenuItem, CheckMenuItem};
use tauri::Manager;

pub fn create_app_menu(app: &tauri::AppHandle) -> Result<Menu<tauri::Wry>, tauri::Error> {
    let menu = Menu::new(app)?;
    
    // ===== File Menu =====
    let file_menu = Submenu::new(app, "File", true)?;
    file_menu.append(&MenuItem::with_id(app, "new_note", "New Note", true, Some("CmdOrCtrl+N"))?)?;
    file_menu.append(&MenuItem::with_id(app, "save_note", "Save", true, Some("CmdOrCtrl+S"))?)?;
    file_menu.append(&MenuItem::with_id(app, "delete_note", "Delete", true, Some("CmdOrCtrl+D"))?)?;
    file_menu.append(&PredefinedMenuItem::separator(app)?)?;
    file_menu.append(&MenuItem::with_id(app, "quit", "Quit", true, Some("CmdOrCtrl+Q"))?)?;
    
    // ===== Edit Menu =====
    let edit_menu = Submenu::new(app, "Edit", true)?;
    edit_menu.append(&PredefinedMenuItem::undo(app, Some("Undo"))?)?;
    edit_menu.append(&PredefinedMenuItem::redo(app, Some("Redo"))?)?;
    edit_menu.append(&PredefinedMenuItem::separator(app)?)?;
    edit_menu.append(&PredefinedMenuItem::cut(app, Some("Cut"))?)?;
    edit_menu.append(&PredefinedMenuItem::copy(app, Some("Copy"))?)?;
    edit_menu.append(&PredefinedMenuItem::paste(app, Some("Paste"))?)?;
    edit_menu.append(&PredefinedMenuItem::select_all(app, Some("Select All"))?)?;
    
    // ===== View Menu =====
    let view_menu = Submenu::new(app, "View", true)?;
    view_menu.append(&CheckMenuItem::with_id(app, "auto_save", "Auto Save", true, true, None)?)?;
    view_menu.append(&PredefinedMenuItem::separator(app)?)?;
    view_menu.append(&MenuItem::with_id(app, "zoom_in", "Zoom In", true, Some("CmdOrCtrl+Plus"))?)?;
    view_menu.append(&MenuItem::with_id(app, "zoom_out", "Zoom Out", true, Some("CmdOrCtrl+Minus"))?)?;
    
    // ===== Help Menu =====
    let help_menu = Submenu::new(app, "Help", true)?;
    help_menu.append(&MenuItem::with_id(app, "about", "About Tauri Note", true, None)?)?;
    
    // Add all menus
    menu.append(&file_menu)?;
    menu.append(&edit_menu)?;
    menu.append(&view_menu)?;
    menu.append(&help_menu)?;
    
    Ok(menu)
}

pub fn setup_menu_handlers(app: &tauri::AppHandle) {
    let app_handle = app.clone();
    
    app.on_menu_event(move |_app, event| {
        match event.id().as_ref() {
            "new_note" => {
                app_handle.emit("menu:new-note", ()).unwrap();
            }
            "save_note" => {
                app_handle.emit("menu:save-note", ()).unwrap();
            }
            "delete_note" => {
                app_handle.emit("menu:delete-note", ()).unwrap();
            }
            "quit" => {
                std::process::exit(0);
            }
            "about" => {
                app_handle.emit("menu:about", ()).unwrap();
            }
            "auto_save" => {
                app_handle.emit("menu:toggle-auto-save", ()).unwrap();
            }
            _ => {}
        }
    });
}
```

### Using in lib.rs

```rust
// src-tauri/src/lib.rs

mod menu;

pub fn run() {
    tauri::Builder::default()
        .setup(|app| {
            // Create and set menu
            let menu = menu::create_app_menu(app.handle())?;
            app.set_menu(menu)?;
            
            // Setup event handlers
            menu::setup_menu_handlers(app.handle());
            
            Ok(())
        })
        .invoke_handler(tauri::generate_handler![...])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Frontend Event Handling

Create a Svelte action or use `onMount` to listen for menu events:

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { listen, type UnlistenFn } from '@tauri-apps/api/event';

    let unlisteners: UnlistenFn[] = [];

    onMount(async () => {
        // Listen for menu events
        unlisteners.push(
            await listen('menu:new-note', () => {
                console.log('New note from menu');
                handleNewNote();
            })
        );
        
        unlisteners.push(
            await listen('menu:save-note', () => {
                console.log('Save from menu');
                handleSaveNote();
            })
        );
        
        unlisteners.push(
            await listen('menu:delete-note', () => {
                console.log('Delete from menu');
                handleDeleteNote();
            })
        );
        
        unlisteners.push(
            await listen('menu:toggle-auto-save', () => {
                console.log('Toggle auto-save');
                autoSave = !autoSave;
            })
        );
    });

    onDestroy(() => {
        // Clean up listeners
        unlisteners.forEach(unlisten => unlisten());
    });

    let autoSave = true;

    function handleNewNote() {
        // Create new note logic
    }

    function handleSaveNote() {
        // Save note logic
    }

    function handleDeleteNote() {
        // Delete note logic
    }
</script>
```

You can also create a reusable utility for menu events:

```typescript
// src/lib/utils/menuEvents.ts
import { listen, type UnlistenFn } from '@tauri-apps/api/event';

export type MenuEventHandlers = {
    onNewNote?: () => void;
    onSaveNote?: () => void;
    onDeleteNote?: () => void;
    onAbout?: () => void;
    onToggleAutoSave?: () => void;
};

export async function setupMenuListeners(handlers: MenuEventHandlers): Promise<UnlistenFn[]> {
    const unlisteners: UnlistenFn[] = [];

    if (handlers.onNewNote) {
        unlisteners.push(await listen('menu:new-note', handlers.onNewNote));
    }
    if (handlers.onSaveNote) {
        unlisteners.push(await listen('menu:save-note', handlers.onSaveNote));
    }
    if (handlers.onDeleteNote) {
        unlisteners.push(await listen('menu:delete-note', handlers.onDeleteNote));
    }
    if (handlers.onAbout) {
        unlisteners.push(await listen('menu:about', handlers.onAbout));
    }
    if (handlers.onToggleAutoSave) {
        unlisteners.push(await listen('menu:toggle-auto-save', handlers.onToggleAutoSave));
    }

    return unlisteners;
}

export function cleanupMenuListeners(unlisteners: UnlistenFn[]) {
    unlisteners.forEach(unlisten => unlisten());
}
```

Using the utility in a component:

```svelte
<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { setupMenuListeners, cleanupMenuListeners, type MenuEventHandlers } from '$lib/utils/menuEvents';
    import type { UnlistenFn } from '@tauri-apps/api/event';

    let unlisteners: UnlistenFn[] = [];

    const handlers: MenuEventHandlers = {
        onNewNote: () => console.log('New note'),
        onSaveNote: () => console.log('Save note'),
        onDeleteNote: () => console.log('Delete note'),
    };

    onMount(async () => {
        unlisteners = await setupMenuListeners(handlers);
    });

    onDestroy(() => {
        cleanupMenuListeners(unlisteners);
    });
</script>
```

---

## Hands-On Exercises

### Exercise 1: Create Basic Menu
1. Create a File menu with New, Save, Quit
2. Add keyboard shortcuts
3. Log when each item is clicked

### Exercise 2: Add Edit Menu
1. Add Edit menu with Undo, Redo, Cut, Copy, Paste
2. Use predefined menu items
3. Test the shortcuts work

### Exercise 3: Handle Menu Events
1. Emit events to the frontend
2. Listen for events in Svelte
3. Trigger note operations from menu

### Exercise 4: Add View Menu
1. Create a View menu with zoom options
2. Add a checkbox item for Auto Save
3. Handle the toggle state

### Exercise 5: Platform-Specific Menu
1. Add macOS app menu (About, Quit)
2. Test on different platforms
3. Handle platform differences

---

## Common Pitfalls

### 1. Forgetting to Set Menu
```rust
// ❌ Menu created but not set
let menu = create_menu(app.handle())?;

// ✅ Menu set on app
let menu = create_menu(app.handle())?;
app.set_menu(menu)?;
```

### 2. Missing Event Handler
```rust
// ❌ No handler for menu events
app.set_menu(menu)?;

// ✅ Handler registered
app.set_menu(menu)?;
app.on_menu_event(|app, event| { ... });
```

### 3. Wrong Accelerator Syntax
```rust
// ❌ Wrong syntax
Some("Ctrl-N")

// ✅ Correct syntax
Some("CmdOrCtrl+N")
```

---

## Additional Resources

- [Tauri Menu Documentation](https://v2.tauri.app/learn/window-menu/)
- [Accelerator Reference](https://v2.tauri.app/reference/config/#accelerator)

---

## Summary

In this phase, you learned:
- ✅ How to create native application menus
- ✅ How to add keyboard shortcuts
- ✅ How to handle menu events
- ✅ How to emit events to the frontend
- ✅ Platform-specific menu considerations

Next: [Phase 5 - System Tray Integration](../phase-5-system-tray/01-objectives.md)