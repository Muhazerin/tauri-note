# Phase 6: Custom Window Controls - Learning Objectives

## Overview
In this phase, you will learn how to create a custom titlebar with window controls, implement draggable regions, and use the Window API to control window behavior.

---

## Learning Objectives

After completing this phase, you should be able to:

### 1. Customize Window Appearance
- [ ] Remove default window decorations
- [ ] Create a custom titlebar with HTML/CSS
- [ ] Style window controls to match your app design

### 2. Implement Window Controls
- [ ] Create minimize, maximize, and close buttons
- [ ] Handle window state changes
- [ ] Implement window dragging on custom titlebar

### 3. Use the Window API
- [ ] Access window properties (size, position, state)
- [ ] Programmatically control window behavior
- [ ] Handle window events

### 4. Create Platform-Consistent UI
- [ ] Handle platform differences in window controls
- [ ] Implement proper hit-testing for draggable regions
- [ ] Ensure accessibility of custom controls

---

## Key Concepts to Master

| Concept | Description |
|---------|-------------|
| `decorations: false` | Removes native titlebar |
| `data-tauri-drag-region` | Makes element draggable |
| Window API | Control window programmatically |
| `WebviewWindow` | Window instance in frontend |

---

## Success Criteria

You have successfully completed this phase when you can:

1. ✅ Display a custom titlebar without native decorations
2. ✅ Drag the window by the custom titlebar
3. ✅ Minimize, maximize, and close using custom buttons
4. ✅ Handle window state changes (maximized/restored)
5. ✅ Style the titlebar to match your app design