# Phase 4: Native Menus - Quiz

Test your understanding of native menus in Tauri.

---

## Multiple Choice Questions

### Question 1
What method is used to create a menu item with a specific ID?

- A) `MenuItem::new()`
- B) `MenuItem::with_id()`
- C) `MenuItem::create()`
- D) `MenuItem::named()`

<details>
<summary>Show Answer</summary>

**B) `MenuItem::with_id()`**

`MenuItem::with_id()` creates a menu item with a specific ID that can be used to identify it in event handlers.
</details>

---

### Question 2
What is the correct accelerator syntax for a cross-platform "Save" shortcut?

- A) `"Ctrl+S"`
- B) `"Cmd+S"`
- C) `"CmdOrCtrl+S"`
- D) `"Control+S"`

<details>
<summary>Show Answer</summary>

**C) `"CmdOrCtrl+S"`**

`CmdOrCtrl` maps to Cmd on macOS and Ctrl on Windows/Linux, providing a consistent cross-platform shortcut.
</details>

---

### Question 3
How do you listen for menu events in Tauri?

- A) `app.on_menu_click()`
- B) `app.on_menu_event()`
- C) `app.menu_handler()`
- D) `app.listen_menu()`

<details>
<summary>Show Answer</summary>

**B) `app.on_menu_event()`**

`app.on_menu_event()` registers a callback that is called whenever a menu item is clicked.
</details>

---

### Question 4
What type is used for toggleable menu items?

- A) `ToggleMenuItem`
- B) `CheckMenuItem`
- C) `SwitchMenuItem`
- D) `BoolMenuItem`

<details>
<summary>Show Answer</summary>

**B) `CheckMenuItem`**

`CheckMenuItem` creates a menu item with a checkbox that can be toggled on/off.
</details>

---

### Question 5
How do you emit an event from a menu handler to the frontend?

- A) `app.send("event-name", data)`
- B) `app.emit("event-name", data)`
- C) `app.dispatch("event-name", data)`
- D) `app.trigger("event-name", data)`

<details>
<summary>Show Answer</summary>

**B) `app.emit("event-name", data)`**

`app.emit()` sends an event that can be listened to in the frontend using `listen()` from `@tauri-apps/api/event`.
</details>

---

### Question 6
What is `PredefinedMenuItem` used for?

- A) Creating custom menu items
- B) Creating platform-specific standard menu items (Copy, Paste, etc.)
- C) Creating menu separators only
- D) Creating disabled menu items

<details>
<summary>Show Answer</summary>

**B) Creating platform-specific standard menu items (Copy, Paste, etc.)**

`PredefinedMenuItem` provides standard menu items like Copy, Paste, Undo, Redo, Quit, etc., that follow platform conventions.
</details>

---

## True or False

### Question 7
True or False: Menu items created without an ID cannot trigger events.

<details>
<summary>Show Answer</summary>

**False**

Menu items without explicit IDs still trigger events, but they use auto-generated IDs which are harder to match in event handlers. Using `with_id()` makes event handling more reliable.
</details>

---

### Question 8
True or False: You must call `app.set_menu()` after creating a menu for it to appear.

<details>
<summary>Show Answer</summary>

**True**

Creating a menu with `Menu::new()` doesn't automatically display it. You must call `app.set_menu(menu)` to attach it to the application.
</details>

---

## Short Answer

### Question 9
What's the difference between `MenuItem::new()` and `MenuItem::with_id()`?

<details>
<summary>Show Answer</summary>

- `MenuItem::new()` creates a menu item with an auto-generated ID
- `MenuItem::with_id()` creates a menu item with a specific, custom ID

Using `with_id()` is preferred when you need to handle menu events, as you can match the specific ID in your event handler:

```rust
// with_id - easy to match
MenuItem::with_id(app, "save", "Save", true, Some("CmdOrCtrl+S"))?

// In handler
match event.id().as_ref() {
    "save" => { /* handle save */ }
}
```
</details>

---

### Question 10
How do you listen for menu events in the Svelte frontend?

<details>
<summary>Show Answer</summary>

Use the `listen` function from `@tauri-apps/api/event`:

```typescript
import { listen } from '@tauri-apps/api/event';

// Setup listener
const unlisten = await listen('menu:save-note', (event) => {
    console.log('Save triggered from menu');
    // Handle save
});

// Cleanup when component unmounts
unlisten();
```

The event name must match what's emitted from Rust with `app.emit()`.
</details>

---

## Practical Exercise

### Question 11
Write the Rust code to create a simple Edit menu with Cut, Copy, and Paste using predefined menu items.

<details>
<summary>Show Answer</summary>

```rust
use tauri::menu::{Menu, Submenu, PredefinedMenuItem};

fn create_edit_menu(app: &tauri::AppHandle) -> Result<Submenu<tauri::Wry>, tauri::Error> {
    let edit_menu = Submenu::new(app, "Edit", true)?;
    
    edit_menu.append(&PredefinedMenuItem::cut(app, Some("Cut"))?)?;
    edit_menu.append(&PredefinedMenuItem::copy(app, Some("Copy"))?)?;
    edit_menu.append(&PredefinedMenuItem::paste(app, Some("Paste"))?)?;
    
    Ok(edit_menu)
}
```

Key points:
- Uses `Submenu::new()` to create the dropdown
- Uses `PredefinedMenuItem` for standard edit operations
- These items automatically have the correct keyboard shortcuts for each platform
</details>

---

### Question 12
Write a menu event handler that emits different events for "new_note", "save_note", and "quit" menu items.

<details>
<summary>Show Answer</summary>

```rust
use tauri::Manager;

fn setup_menu_handlers(app: &tauri::AppHandle) {
    let app_handle = app.clone();
    
    app.on_menu_event(move |_app, event| {
        match event.id().as_ref() {
            "new_note" => {
                if let Err(e) = app_handle.emit("menu:new-note", ()) {
                    eprintln!("Failed to emit new-note event: {}", e);
                }
            }
            "save_note" => {
                if let Err(e) = app_handle.emit("menu:save-note", ()) {
                    eprintln!("Failed to emit save-note event: {}", e);
                }
            }
            "quit" => {
                std::process::exit(0);
            }
            _ => {
                // Unknown menu item, ignore
            }
        }
    });
}
```

Key points:
- Clones `app_handle` to use inside the closure
- Uses `match` to handle different menu IDs
- Emits events to frontend for new/save
- Exits directly for quit
- Handles errors gracefully
</details>

---

## Score Yourself

| Score | Rating |
|-------|--------|
| 10-12 | ⭐⭐⭐ Excellent! Ready for Phase 5 |
| 7-9 | ⭐⭐ Good! Review the materials you missed |
| 4-6 | ⭐ Fair. Re-read the learning materials |
| 0-3 | Need more practice. Start from the beginning |

---

## Next Steps

If you scored well, proceed to [Phase 5: System Tray Integration](../phase-5-system-tray/01-objectives.md)

If you need more practice, review:
- [Phase 4 Learning Materials](./02-materials.md)
- [Tauri Menu Documentation](https://v2.tauri.app/learn/window-menu/)