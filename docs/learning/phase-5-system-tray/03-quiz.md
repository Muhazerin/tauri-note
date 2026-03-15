# Phase 5: System Tray Integration - Quiz

Test your understanding of system tray integration in Tauri.

---

## Multiple Choice Questions

### Question 1
What Cargo feature is required for system tray functionality?

- A) `tray`
- B) `system-tray`
- C) `tray-icon`
- D) `notification-area`

<details>
<summary>Show Answer</summary>

**C) `tray-icon`**

Add `features = ["tray-icon"]` to your tauri dependency in Cargo.toml.
</details>

---

### Question 2
How do you prevent a window from closing when the user clicks the close button?

- A) `event.cancel()`
- B) `api.prevent_close()`
- C) `window.prevent_close()`
- D) `event.stop_propagation()`

<details>
<summary>Show Answer</summary>

**B) `api.prevent_close()`**

In the `CloseRequested` event handler, call `api.prevent_close()` to prevent the window from closing.
</details>

---

### Question 3
What method is used to get a window by its label?

- A) `app.get_window("main")`
- B) `app.get_webview_window("main")`
- C) `app.window("main")`
- D) `app.find_window("main")`

<details>
<summary>Show Answer</summary>

**B) `app.get_webview_window("main")`**

In Tauri v2, use `app.get_webview_window("label")` to get a window by its label.
</details>

---

### Question 4
What does `menu_on_left_click(false)` do?

- A) Disables the menu entirely
- B) Shows menu only on right-click
- C) Shows menu only on left-click
- D) Disables left-click on tray icon

<details>
<summary>Show Answer</summary>

**B) Shows menu only on right-click**

Setting `menu_on_left_click(false)` means the menu will only appear on right-click, allowing you to use left-click for other actions like toggling the window.
</details>

---

## True or False

### Question 5
True or False: You must call `api.prevent_close()` before hiding the window in a close event handler.

<details>
<summary>Show Answer</summary>

**True**

If you don't call `api.prevent_close()`, the window will close regardless of any other actions you take in the handler.
</details>

---

### Question 6
True or False: The tray icon automatically disappears when the main window is hidden.

<details>
<summary>Show Answer</summary>

**False**

The tray icon remains visible regardless of the window state. This is the intended behavior - the tray icon allows users to access the app even when the window is hidden.
</details>

---

## Short Answer

### Question 7
Write the code to toggle window visibility (show if hidden, hide if visible).

<details>
<summary>Show Answer</summary>

```rust
if let Some(window) = app.get_webview_window("main") {
    if window.is_visible().unwrap_or(false) {
        window.hide().unwrap();
    } else {
        window.show().unwrap();
        window.set_focus().unwrap();
    }
}
```
</details>

---

### Question 8
How do you handle a left-click on the tray icon?

<details>
<summary>Show Answer</summary>

```rust
.on_tray_icon_event(|tray, event| {
    use tauri::tray::{TrayIconEvent, MouseButton, MouseButtonState};
    
    if let TrayIconEvent::Click { button, button_state, .. } = event {
        if button == MouseButton::Left && button_state == MouseButtonState::Up {
            // Handle left click
            let app = tray.app_handle();
            // ... your logic here
        }
    }
})
```
</details>

---

## Score Yourself

| Score | Rating |
|-------|--------|
| 7-8 | ⭐⭐⭐ Excellent! Ready for Phase 6 |
| 5-6 | ⭐⭐ Good! Review the materials you missed |
| 3-4 | ⭐ Fair. Re-read the learning materials |
| 0-2 | Need more practice |

---

## Next Steps

Proceed to [Phase 6: Custom Window Controls](../phase-6-custom-window/01-objectives.md)