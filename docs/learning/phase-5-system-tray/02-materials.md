# Phase 5: System Tray Integration - Learning Materials

## Table of Contents
1. [Introduction to System Tray](#introduction-to-system-tray)
2. [Setting Up the Tray Plugin](#setting-up-the-tray-plugin)
3. [Creating a Tray Icon](#creating-a-tray-icon)
4. [Tray Menus](#tray-menus)
5. [Window Management](#window-management)
6. [Complete Example](#complete-example)
7. [Hands-On Exercises](#hands-on-exercises)

---

## Introduction to System Tray

The system tray (notification area) allows your app to remain accessible even when the main window is closed. Common uses include:
- Background applications
- Quick access menus
- Status indicators
- Minimize-to-tray functionality

---

## Setting Up the Tray Plugin

### Installation

In Tauri v2, system tray functionality is built into the core but requires configuration.

**1. Enable tray in Cargo.toml:**

```toml
[dependencies]
tauri = { version = "2", features = ["tray-icon"] }
```

**2. Add tray icon to your project:**

Place your icon in `src-tauri/icons/`:
- `icon.png` (32x32 or 64x64 recommended)
- For Windows, also include `icon.ico`

**3. Configure in tauri.conf.json:**

```json
{
  "app": {
    "trayIcon": {
      "iconPath": "icons/icon.png",
      "iconAsTemplate": true
    }
  }
}
```

---

## Creating a Tray Icon

### Basic Tray Setup

```rust
use tauri::{
    tray::{TrayIcon, TrayIconBuilder},
    Manager,
};

pub fn create_tray(app: &tauri::AppHandle) -> Result<TrayIcon, tauri::Error> {
    let tray = TrayIconBuilder::new()
        .icon(app.default_window_icon().unwrap().clone())
        .tooltip("Tauri Note")
        .build(app)?;
    
    Ok(tray)
}
```

### With Menu

```rust
use tauri::{
    tray::{TrayIcon, TrayIconBuilder},
    menu::{Menu, MenuItem},
    Manager,
};

pub fn create_tray(app: &tauri::AppHandle) -> Result<TrayIcon, tauri::Error> {
    // Create tray menu
    let menu = Menu::new(app)?;
    menu.append(&MenuItem::with_id(app, "show", "Show Window", true, None)?)?;
    menu.append(&MenuItem::with_id(app, "hide", "Hide Window", true, None)?)?;
    menu.append(&MenuItem::with_id(app, "quit", "Quit", true, None)?)?;
    
    let tray = TrayIconBuilder::new()
        .icon(app.default_window_icon().unwrap().clone())
        .tooltip("Tauri Note")
        .menu(&menu)
        .build(app)?;
    
    Ok(tray)
}
```

---

## Tray Menus

### Creating a Tray Menu

```rust
use tauri::menu::{Menu, MenuItem, PredefinedMenuItem};

fn create_tray_menu(app: &tauri::AppHandle) -> Result<Menu<tauri::Wry>, tauri::Error> {
    let menu = Menu::new(app)?;
    
    menu.append(&MenuItem::with_id(app, "show", "Show Window", true, None)?)?;
    menu.append(&MenuItem::with_id(app, "hide", "Hide Window", true, None)?)?;
    menu.append(&PredefinedMenuItem::separator(app)?)?;
    menu.append(&MenuItem::with_id(app, "new_note", "New Note", true, None)?)?;
    menu.append(&PredefinedMenuItem::separator(app)?)?;
    menu.append(&MenuItem::with_id(app, "quit", "Quit", true, None)?)?;
    
    Ok(menu)
}
```

### Handling Tray Menu Events

```rust
use tauri::Manager;

pub fn setup_tray(app: &tauri::AppHandle) -> Result<(), tauri::Error> {
    let menu = create_tray_menu(app)?;
    
    let tray = TrayIconBuilder::new()
        .icon(app.default_window_icon().unwrap().clone())
        .tooltip("Tauri Note")
        .menu(&menu)
        .on_menu_event(|app, event| {
            match event.id().as_ref() {
                "show" => {
                    if let Some(window) = app.get_webview_window("main") {
                        window.show().unwrap();
                        window.set_focus().unwrap();
                    }
                }
                "hide" => {
                    if let Some(window) = app.get_webview_window("main") {
                        window.hide().unwrap();
                    }
                }
                "new_note" => {
                    app.emit("tray:new-note", ()).unwrap();
                }
                "quit" => {
                    std::process::exit(0);
                }
                _ => {}
            }
        })
        .build(app)?;
    
    Ok(())
}
```

### Handling Tray Icon Clicks

```rust
TrayIconBuilder::new()
    .on_tray_icon_event(|tray, event| {
        match event {
            tauri::tray::TrayIconEvent::Click {
                button,
                button_state,
                ..
            } => {
                if button == tauri::tray::MouseButton::Left 
                    && button_state == tauri::tray::MouseButtonState::Up 
                {
                    // Left click - show/toggle window
                    let app = tray.app_handle();
                    if let Some(window) = app.get_webview_window("main") {
                        if window.is_visible().unwrap() {
                            window.hide().unwrap();
                        } else {
                            window.show().unwrap();
                            window.set_focus().unwrap();
                        }
                    }
                }
            }
            tauri::tray::TrayIconEvent::DoubleClick { button, .. } => {
                if button == tauri::tray::MouseButton::Left {
                    // Double click - show and focus window
                    let app = tray.app_handle();
                    if let Some(window) = app.get_webview_window("main") {
                        window.show().unwrap();
                        window.set_focus().unwrap();
                    }
                }
            }
            _ => {}
        }
    })
    .build(app)?;
```

---

## Window Management

### Minimize to Tray on Close

To minimize to tray instead of closing:

```rust
// In setup
app.get_webview_window("main")
    .unwrap()
    .on_window_event(|event| {
        if let tauri::WindowEvent::CloseRequested { api, .. } = event {
            // Prevent the window from closing
            api.prevent_close();
            // Hide instead
            event.window().hide().unwrap();
        }
    });
```

### Window Visibility Commands

```rust
#[tauri::command]
fn show_window(app: tauri::AppHandle) -> Result<(), String> {
    if let Some(window) = app.get_webview_window("main") {
        window.show().map_err(|e| e.to_string())?;
        window.set_focus().map_err(|e| e.to_string())?;
    }
    Ok(())
}

#[tauri::command]
fn hide_window(app: tauri::AppHandle) -> Result<(), String> {
    if let Some(window) = app.get_webview_window("main") {
        window.hide().map_err(|e| e.to_string())?;
    }
    Ok(())
}

#[tauri::command]
fn toggle_window(app: tauri::AppHandle) -> Result<(), String> {
    if let Some(window) = app.get_webview_window("main") {
        if window.is_visible().map_err(|e| e.to_string())? {
            window.hide().map_err(|e| e.to_string())?;
        } else {
            window.show().map_err(|e| e.to_string())?;
            window.set_focus().map_err(|e| e.to_string())?;
        }
    }
    Ok(())
}
```

---

## Complete Example

### Full Tray Implementation

```rust
// src-tauri/src/tray.rs

use tauri::{
    tray::{TrayIcon, TrayIconBuilder},
    menu::{Menu, MenuItem, PredefinedMenuItem},
    Manager,
};

pub fn setup_tray(app: &tauri::AppHandle) -> Result<TrayIcon, tauri::Error> {
    // Create tray menu
    let menu = Menu::new(app)?;
    
    menu.append(&MenuItem::with_id(app, "show", "Show Tauri Note", true, None)?)?;
    menu.append(&MenuItem::with_id(app, "hide", "Hide", true, None)?)?;
    menu.append(&PredefinedMenuItem::separator(app)?)?;
    menu.append(&MenuItem::with_id(app, "new_note", "Quick Note", true, None)?)?;
    menu.append(&PredefinedMenuItem::separator(app)?)?;
    menu.append(&MenuItem::with_id(app, "quit", "Quit", true, None)?)?;
    
    // Build tray icon
    let tray = TrayIconBuilder::new()
        .icon(app.default_window_icon().unwrap().clone())
        .tooltip("Tauri Note - Click to show")
        .menu(&menu)
        .menu_on_left_click(false)  // Show menu only on right-click
        .on_menu_event(|app, event| {
            handle_tray_menu_event(app, event.id().as_ref());
        })
        .on_tray_icon_event(|tray, event| {
            handle_tray_icon_event(tray, event);
        })
        .build(app)?;
    
    Ok(tray)
}

fn handle_tray_menu_event(app: &tauri::AppHandle, id: &str) {
    match id {
        "show" => {
            if let Some(window) = app.get_webview_window("main") {
                let _ = window.show();
                let _ = window.set_focus();
            }
        }
        "hide" => {
            if let Some(window) = app.get_webview_window("main") {
                let _ = window.hide();
            }
        }
        "new_note" => {
            // Show window and emit event
            if let Some(window) = app.get_webview_window("main") {
                let _ = window.show();
                let _ = window.set_focus();
            }
            let _ = app.emit("tray:new-note", ());
        }
        "quit" => {
            std::process::exit(0);
        }
        _ => {}
    }
}

fn handle_tray_icon_event(tray: &TrayIcon, event: tauri::tray::TrayIconEvent) {
    use tauri::tray::{TrayIconEvent, MouseButton, MouseButtonState};
    
    if let TrayIconEvent::Click { button, button_state, .. } = event {
        if button == MouseButton::Left && button_state == MouseButtonState::Up {
            let app = tray.app_handle();
            if let Some(window) = app.get_webview_window("main") {
                if window.is_visible().unwrap_or(false) {
                    let _ = window.hide();
                } else {
                    let _ = window.show();
                    let _ = window.set_focus();
                }
            }
        }
    }
}

pub fn setup_close_to_tray(app: &tauri::AppHandle) {
    if let Some(window) = app.get_webview_window("main") {
        window.on_window_event(|event| {
            if let tauri::WindowEvent::CloseRequested { api, .. } = event {
                api.prevent_close();
                let _ = event.window().hide();
            }
        });
    }
}
```

### Using in lib.rs

```rust
// src-tauri/src/lib.rs

mod tray;

pub fn run() {
    tauri::Builder::default()
        .setup(|app| {
            // Setup tray
            tray::setup_tray(app.handle())?;
            
            // Setup close-to-tray behavior
            tray::setup_close_to_tray(app.handle());
            
            Ok(())
        })
        .invoke_handler(tauri::generate_handler![...])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

---

## Hands-On Exercises

### Exercise 1: Basic Tray Icon
1. Add tray icon feature to Cargo.toml
2. Create a simple tray icon
3. Verify it appears in the system tray

### Exercise 2: Tray Menu
1. Create a menu with Show, Hide, Quit
2. Handle menu item clicks
3. Test showing/hiding the window

### Exercise 3: Click Events
1. Handle left-click to toggle window
2. Handle double-click to show window
3. Keep right-click for menu

### Exercise 4: Close to Tray
1. Intercept window close event
2. Hide window instead of closing
3. Add "Quit" option to actually exit

### Exercise 5: Dynamic Menu
1. Update menu based on window state
2. Show "Show" when hidden, "Hide" when visible
3. Add note count to tooltip

---

## Common Pitfalls

### 1. Missing Tray Feature
```toml
# ❌ Missing feature
tauri = { version = "2" }

# ✅ With tray feature
tauri = { version = "2", features = ["tray-icon"] }
```

### 2. Window Not Found
```rust
// ❌ Assuming window exists
let window = app.get_webview_window("main").unwrap();

// ✅ Handle missing window
if let Some(window) = app.get_webview_window("main") {
    window.show().unwrap();
}
```

### 3. Forgetting to Prevent Close
```rust
// ❌ Window closes normally
window.on_window_event(|event| {
    if let tauri::WindowEvent::CloseRequested { .. } = event {
        event.window().hide().unwrap();  // Window still closes!
    }
});

// ✅ Prevent close first
window.on_window_event(|event| {
    if let tauri::WindowEvent::CloseRequested { api, .. } = event {
        api.prevent_close();  // Prevent closing
        event.window().hide().unwrap();  // Then hide
    }
});
```

---

## Summary

In this phase, you learned:
- ✅ How to create a system tray icon
- ✅ How to build tray menus
- ✅ How to handle tray events
- ✅ How to implement minimize-to-tray
- ✅ How to manage window visibility

Next: [Phase 6 - Custom Window Controls](../phase-6-custom-window/01-objectives.md)