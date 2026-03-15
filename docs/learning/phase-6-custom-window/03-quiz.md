# Phase 6: Custom Window Controls - Quiz

Test your understanding of custom window controls in Tauri.

---

## Multiple Choice Questions

### Question 1
What configuration removes the native window titlebar?

- A) `"titlebar": false`
- B) `"decorations": false`
- C) `"frame": false`
- D) `"chrome": false`

<details>
<summary>Show Answer</summary>

**B) `"decorations": false`**

Set `decorations: false` in the window configuration in `tauri.conf.json`.
</details>

---

### Question 2
What attribute makes an element draggable for window movement?

- A) `draggable="true"`
- B) `data-drag-region`
- C) `data-tauri-drag-region`
- D) `tauri-draggable`

<details>
<summary>Show Answer</summary>

**C) `data-tauri-drag-region`**

Add `data-tauri-drag-region` to any element that should allow window dragging.
</details>

---

### Question 3
How do you get the current window instance in the frontend?

- A) `getWindow()`
- B) `window.tauri`
- C) `getCurrentWindow()`
- D) `appWindow()`

<details>
<summary>Show Answer</summary>

**C) `getCurrentWindow()`**

Import from `@tauri-apps/api/window`: `import { getCurrentWindow } from '@tauri-apps/api/window';`
</details>

---

### Question 4
What method toggles between maximized and restored states?

- A) `appWindow.maximize()`
- B) `appWindow.toggleMaximize()`
- C) `appWindow.switchMaximize()`
- D) `appWindow.setMaximized(!isMaximized)`

<details>
<summary>Show Answer</summary>

**B) `appWindow.toggleMaximize()`**

`toggleMaximize()` automatically switches between maximized and restored states.
</details>

---

## True or False

### Question 5
True or False: Buttons inside a `data-tauri-drag-region` element should also have the attribute.

<details>
<summary>Show Answer</summary>

**False**

Buttons and interactive elements should NOT have `data-tauri-drag-region` or they won't be clickable. Only the container should have it.
</details>

---

### Question 6
True or False: You need to add `user-select: none` to prevent text selection while dragging.

<details>
<summary>Show Answer</summary>

**True**

Adding `user-select: none` (or the Tailwind class `select-none`) prevents accidental text selection while dragging the window.
</details>

---

## Short Answer

### Question 7
Write the code to check if the window is maximized and update state accordingly.

<details>
<summary>Show Answer</summary>

```typescript
import { getCurrentWindow } from '@tauri-apps/api/window';

const appWindow = getCurrentWindow();
const isMaximized = await appWindow.isMaximized();
setIsMaximized(isMaximized);
```
</details>

---

### Question 8
How do you listen for window resize events to track maximize state changes?

<details>
<summary>Show Answer</summary>

```typescript
const appWindow = getCurrentWindow();

const unlisten = await appWindow.onResized(() => {
    appWindow.isMaximized().then(setIsMaximized);
});

// Cleanup
return () => {
    unlisten();
};
```
</details>

---

## Score Yourself

| Score | Rating |
|-------|--------|
| 7-8 | ⭐⭐⭐ Excellent! Ready for Phase 7 |
| 5-6 | ⭐⭐ Good! Review the materials you missed |
| 3-4 | ⭐ Fair. Re-read the learning materials |
| 0-2 | Need more practice |

---

## Next Steps

Proceed to [Phase 7: Auto-Updates](../phase-7-auto-updates/01-objectives.md)