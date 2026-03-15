# Phase 6: Custom Window Controls - Learning Materials

## Table of Contents
1. [Removing Native Decorations](#removing-native-decorations)
2. [Creating a Custom Titlebar](#creating-a-custom-titlebar)
3. [Window Dragging](#window-dragging)
4. [Window Control Buttons](#window-control-buttons)
5. [Window API](#window-api)
6. [Complete Example](#complete-example)

---

## Removing Native Decorations

### Configuration

In `tauri.conf.json`, set `decorations` to `false`:

```json
{
  "app": {
    "windows": [
      {
        "title": "Tauri Note",
        "width": 800,
        "height": 600,
        "decorations": false,
        "transparent": false
      }
    ]
  }
}
```

This removes the native titlebar, giving you full control over the window appearance.

---

## Creating a Custom Titlebar

### Basic Titlebar Component

```tsx
// src/components/Titlebar.tsx
import { getCurrentWindow } from '@tauri-apps/api/window';

export function Titlebar() {
    const appWindow = getCurrentWindow();

    const handleMinimize = () => appWindow.minimize();
    const handleMaximize = () => appWindow.toggleMaximize();
    const handleClose = () => appWindow.close();

    return (
        <div 
            data-tauri-drag-region
            className="h-8 bg-gray-800 flex items-center justify-between px-2 select-none"
        >
            {/* App title */}
            <div className="flex items-center gap-2" data-tauri-drag-region>
                <span className="text-white text-sm font-medium">Tauri Note</span>
            </div>

            {/* Window controls */}
            <div className="flex items-center">
                <button
                    onClick={handleMinimize}
                    className="w-8 h-8 flex items-center justify-center hover:bg-gray-700 text-white"
                >
                    ─
                </button>
                <button
                    onClick={handleMaximize}
                    className="w-8 h-8 flex items-center justify-center hover:bg-gray-700 text-white"
                >
                    □
                </button>
                <button
                    onClick={handleClose}
                    className="w-8 h-8 flex items-center justify-center hover:bg-red-600 text-white"
                >
                    ✕
                </button>
            </div>
        </div>
    );
}
```

### Using in App

```tsx
// src/App.tsx
import { Titlebar } from './components/Titlebar';

function App() {
    return (
        <div className="h-screen flex flex-col">
            <Titlebar />
            <main className="flex-1 overflow-auto">
                {/* Your app content */}
            </main>
        </div>
    );
}
```

---

## Window Dragging

### The `data-tauri-drag-region` Attribute

Add this attribute to any element that should allow window dragging:

```tsx
<div data-tauri-drag-region className="titlebar">
    {/* Content */}
</div>
```

### Important Notes

1. **Child elements**: Buttons and interactive elements inside the drag region should NOT have the attribute
2. **CSS**: Add `user-select: none` to prevent text selection while dragging
3. **Nested regions**: The attribute works on nested elements too

```tsx
// ✅ Correct - buttons don't have drag region
<div data-tauri-drag-region className="titlebar">
    <span data-tauri-drag-region>App Title</span>
    <button onClick={handleClose}>✕</button>  {/* No drag region */}
</div>

// ❌ Wrong - button has drag region, won't be clickable
<div data-tauri-drag-region className="titlebar">
    <button data-tauri-drag-region onClick={handleClose}>✕</button>
</div>
```

---

## Window Control Buttons

### Window API Methods

```typescript
import { getCurrentWindow } from '@tauri-apps/api/window';

const appWindow = getCurrentWindow();

// Minimize
await appWindow.minimize();

// Maximize / Restore
await appWindow.toggleMaximize();
await appWindow.maximize();
await appWindow.unmaximize();

// Close
await appWindow.close();

// Check state
const isMaximized = await appWindow.isMaximized();
const isMinimized = await appWindow.isMinimized();
const isFullscreen = await appWindow.isFullscreen();
```

### Handling Maximize State

```tsx
import { useState, useEffect } from 'react';
import { getCurrentWindow } from '@tauri-apps/api/window';

function Titlebar() {
    const [isMaximized, setIsMaximized] = useState(false);
    const appWindow = getCurrentWindow();

    useEffect(() => {
        // Check initial state
        appWindow.isMaximized().then(setIsMaximized);

        // Listen for changes
        const unlisten = appWindow.onResized(() => {
            appWindow.isMaximized().then(setIsMaximized);
        });

        return () => {
            unlisten.then(fn => fn());
        };
    }, []);

    return (
        <button onClick={() => appWindow.toggleMaximize()}>
            {isMaximized ? '❐' : '□'}
        </button>
    );
}
```

---

## Window API

### Common Operations

```typescript
import { getCurrentWindow } from '@tauri-apps/api/window';

const appWindow = getCurrentWindow();

// Size
await appWindow.setSize({ width: 800, height: 600 });
const size = await appWindow.innerSize();

// Position
await appWindow.setPosition({ x: 100, y: 100 });
const position = await appWindow.outerPosition();

// Title
await appWindow.setTitle('New Title');

// Visibility
await appWindow.show();
await appWindow.hide();

// Focus
await appWindow.setFocus();

// Always on top
await appWindow.setAlwaysOnTop(true);

// Fullscreen
await appWindow.setFullscreen(true);
```

### Window Events

```typescript
import { getCurrentWindow } from '@tauri-apps/api/window';

const appWindow = getCurrentWindow();

// Resize event
const unlisten = await appWindow.onResized(({ payload: size }) => {
    console.log('Window resized:', size.width, size.height);
});

// Move event
const unlisten2 = await appWindow.onMoved(({ payload: position }) => {
    console.log('Window moved:', position.x, position.y);
});

// Focus events
const unlisten3 = await appWindow.onFocusChanged(({ payload: focused }) => {
    console.log('Window focused:', focused);
});

// Close requested
const unlisten4 = await appWindow.onCloseRequested((event) => {
    // Prevent close
    event.preventDefault();
    // Do something else
});

// Cleanup
unlisten();
```

---

## Complete Example

### Titlebar Component with Icons

```tsx
// src/components/Titlebar.tsx
import { useState, useEffect } from 'react';
import { getCurrentWindow } from '@tauri-apps/api/window';

export function Titlebar() {
    const [isMaximized, setIsMaximized] = useState(false);
    const appWindow = getCurrentWindow();

    useEffect(() => {
        appWindow.isMaximized().then(setIsMaximized);
        
        const setupListener = async () => {
            return appWindow.onResized(() => {
                appWindow.isMaximized().then(setIsMaximized);
            });
        };
        
        const unlisten = setupListener();
        return () => {
            unlisten.then(fn => fn());
        };
    }, []);

    return (
        <header 
            data-tauri-drag-region
            className="h-10 bg-slate-900 flex items-center justify-between px-4 select-none"
        >
            {/* Left: App icon and title */}
            <div className="flex items-center gap-3" data-tauri-drag-region>
                <img src="/icon.png" alt="" className="w-5 h-5" />
                <span className="text-slate-200 text-sm font-medium">
                    Tauri Note
                </span>
            </div>

            {/* Right: Window controls */}
            <div className="flex -mr-2">
                {/* Minimize */}
                <button
                    onClick={() => appWindow.minimize()}
                    className="w-10 h-10 flex items-center justify-center 
                               text-slate-400 hover:text-white hover:bg-slate-700
                               transition-colors"
                    aria-label="Minimize"
                >
                    <svg className="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                        <path d="M4 8h8v1H4z"/>
                    </svg>
                </button>

                {/* Maximize/Restore */}
                <button
                    onClick={() => appWindow.toggleMaximize()}
                    className="w-10 h-10 flex items-center justify-center 
                               text-slate-400 hover:text-white hover:bg-slate-700
                               transition-colors"
                    aria-label={isMaximized ? 'Restore' : 'Maximize'}
                >
                    {isMaximized ? (
                        <svg className="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                            <path d="M3 5v8h8V5H3zm7 7H4V6h6v6z"/>
                            <path d="M5 3h8v8h-1V4H5V3z"/>
                        </svg>
                    ) : (
                        <svg className="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                            <path d="M3 3v10h10V3H3zm9 9H4V4h8v8z"/>
                        </svg>
                    )}
                </button>

                {/* Close */}
                <button
                    onClick={() => appWindow.close()}
                    className="w-10 h-10 flex items-center justify-center 
                               text-slate-400 hover:text-white hover:bg-red-600
                               transition-colors"
                    aria-label="Close"
                >
                    <svg className="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                        <path d="M4.646 4.646a.5.5 0 0 1 .708 0L8 7.293l2.646-2.647a.5.5 0 0 1 .708.708L8.707 8l2.647 2.646a.5.5 0 0 1-.708.708L8 8.707l-2.646 2.647a.5.5 0 0 1-.708-.708L7.293 8 4.646 5.354a.5.5 0 0 1 0-.708z"/>
                    </svg>
                </button>
            </div>
        </header>
    );
}
```

### App Layout

```tsx
// src/App.tsx
import { Titlebar } from './components/Titlebar';

function App() {
    return (
        <div className="h-screen flex flex-col bg-slate-800">
            <Titlebar />
            <main className="flex-1 overflow-hidden">
                {/* Your app content */}
            </main>
        </div>
    );
}
```

---

## Hands-On Exercises

### Exercise 1: Remove Decorations
1. Set `decorations: false` in tauri.conf.json
2. Run the app and observe the frameless window

### Exercise 2: Create Basic Titlebar
1. Create a Titlebar component
2. Add `data-tauri-drag-region`
3. Test window dragging

### Exercise 3: Add Window Controls
1. Add minimize, maximize, close buttons
2. Implement click handlers
3. Test all controls work

### Exercise 4: Track Maximize State
1. Listen for window resize events
2. Update maximize button icon
3. Show correct icon based on state

### Exercise 5: Style the Titlebar
1. Match your app's design
2. Add hover effects
3. Ensure accessibility

---

## Common Pitfalls

### 1. Buttons Not Clickable
```tsx
// ❌ Button inside drag region with attribute
<div data-tauri-drag-region>
    <button data-tauri-drag-region>Click me</button>
</div>

// ✅ Button without drag region attribute
<div data-tauri-drag-region>
    <button>Click me</button>
</div>
```

### 2. Text Selection While Dragging
```css
/* Add to titlebar */
.titlebar {
    user-select: none;
    -webkit-user-select: none;
}
```

### 3. Forgetting to Handle Maximize State
```tsx
// ❌ Static icon
<button>□</button>

// ✅ Dynamic icon based on state
<button>{isMaximized ? '❐' : '□'}</button>
```

---

## Summary

In this phase, you learned:
- ✅ How to remove native window decorations
- ✅ How to create a custom titlebar
- ✅ How to implement window dragging
- ✅ How to create window control buttons
- ✅ How to use the Window API

Next: [Phase 7 - Auto-Updates](../phase-7-auto-updates/01-objectives.md)