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

```svelte
<!-- src/lib/components/Titlebar.svelte -->
<script lang="ts">
    import { getCurrentWindow } from '@tauri-apps/api/window';

    const appWindow = getCurrentWindow();

    function handleMinimize() {
        appWindow.minimize();
    }

    function handleMaximize() {
        appWindow.toggleMaximize();
    }

    function handleClose() {
        appWindow.close();
    }
</script>

<div 
    data-tauri-drag-region
    class="h-8 bg-gray-800 flex items-center justify-between px-2 select-none"
>
    <!-- App title -->
    <div class="flex items-center gap-2" data-tauri-drag-region>
        <span class="text-white text-sm font-medium">Tauri Note</span>
    </div>

    <!-- Window controls -->
    <div class="flex items-center">
        <button
            on:click={handleMinimize}
            class="w-8 h-8 flex items-center justify-center hover:bg-gray-700 text-white"
        >
            ─
        </button>
        <button
            on:click={handleMaximize}
            class="w-8 h-8 flex items-center justify-center hover:bg-gray-700 text-white"
        >
            □
        </button>
        <button
            on:click={handleClose}
            class="w-8 h-8 flex items-center justify-center hover:bg-red-600 text-white"
        >
            ✕
        </button>
    </div>
</div>
```

### Using in Layout

```svelte
<!-- src/routes/+layout.svelte -->
<script>
    import Titlebar from '$lib/components/Titlebar.svelte';
    import '../app.css';
</script>

<div class="h-screen flex flex-col">
    <Titlebar />
    <main class="flex-1 overflow-auto">
        <slot />
    </main>
</div>
```

---

## Window Dragging

### The `data-tauri-drag-region` Attribute

Add this attribute to any element that should allow window dragging:

```svelte
<div data-tauri-drag-region class="titlebar">
    <!-- Content -->
</div>
```

### Important Notes

1. **Child elements**: Buttons and interactive elements inside the drag region should NOT have the attribute
2. **CSS**: Add `user-select: none` to prevent text selection while dragging
3. **Nested regions**: The attribute works on nested elements too

```svelte
<!-- ✅ Correct - buttons don't have drag region -->
<div data-tauri-drag-region class="titlebar">
    <span data-tauri-drag-region>App Title</span>
    <button on:click={handleClose}>✕</button>  <!-- No drag region -->
</div>

<!-- ❌ Wrong - button has drag region, won't be clickable -->
<div data-tauri-drag-region class="titlebar">
    <button data-tauri-drag-region on:click={handleClose}>✕</button>
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

```svelte
<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { getCurrentWindow } from '@tauri-apps/api/window';

    let isMaximized = false;
    const appWindow = getCurrentWindow();
    let unlisten: (() => void) | null = null;

    onMount(async () => {
        // Check initial state
        isMaximized = await appWindow.isMaximized();

        // Listen for changes
        unlisten = await appWindow.onResized(async () => {
            isMaximized = await appWindow.isMaximized();
        });
    });

    onDestroy(() => {
        if (unlisten) unlisten();
    });

    function toggleMaximize() {
        appWindow.toggleMaximize();
    }
</script>

<button on:click={toggleMaximize}>
    {isMaximized ? '❐' : '□'}
</button>
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

```svelte
<!-- src/lib/components/Titlebar.svelte -->
<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { getCurrentWindow } from '@tauri-apps/api/window';

    let isMaximized = false;
    const appWindow = getCurrentWindow();
    let unlisten: (() => void) | null = null;

    onMount(async () => {
        isMaximized = await appWindow.isMaximized();
        
        unlisten = await appWindow.onResized(async () => {
            isMaximized = await appWindow.isMaximized();
        });
    });

    onDestroy(() => {
        if (unlisten) unlisten();
    });

    function minimize() {
        appWindow.minimize();
    }

    function toggleMaximize() {
        appWindow.toggleMaximize();
    }

    function close() {
        appWindow.close();
    }
</script>

<header 
    data-tauri-drag-region
    class="h-10 bg-slate-900 flex items-center justify-between px-4 select-none"
>
    <!-- Left: App icon and title -->
    <div class="flex items-center gap-3" data-tauri-drag-region>
        <img src="/favicon.png" alt="" class="w-5 h-5" />
        <span class="text-slate-200 text-sm font-medium">
            Tauri Note
        </span>
    </div>

    <!-- Right: Window controls -->
    <div class="flex -mr-2">
        <!-- Minimize -->
        <button
            on:click={minimize}
            class="w-10 h-10 flex items-center justify-center 
                   text-slate-400 hover:text-white hover:bg-slate-700
                   transition-colors"
            aria-label="Minimize"
        >
            <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                <path d="M4 8h8v1H4z"/>
            </svg>
        </button>

        <!-- Maximize/Restore -->
        <button
            on:click={toggleMaximize}
            class="w-10 h-10 flex items-center justify-center 
                   text-slate-400 hover:text-white hover:bg-slate-700
                   transition-colors"
            aria-label={isMaximized ? 'Restore' : 'Maximize'}
        >
            {#if isMaximized}
                <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                    <path d="M3 5v8h8V5H3zm7 7H4V6h6v6z"/>
                    <path d="M5 3h8v8h-1V4H5V3z"/>
                </svg>
            {:else}
                <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                    <path d="M3 3v10h10V3H3zm9 9H4V4h8v8z"/>
                </svg>
            {/if}
        </button>

        <!-- Close -->
        <button
            on:click={close}
            class="w-10 h-10 flex items-center justify-center 
                   text-slate-400 hover:text-white hover:bg-red-600
                   transition-colors"
            aria-label="Close"
        >
            <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 16 16">
                <path d="M4.646 4.646a.5.5 0 0 1 .708 0L8 7.293l2.646-2.647a.5.5 0 0 1 .708.708L8.707 8l2.647 2.646a.5.5 0 0 1-.708.708L8 8.707l-2.646 2.647a.5.5 0 0 1-.708-.708L7.293 8 4.646 5.354a.5.5 0 0 1 0-.708z"/>
            </svg>
        </button>
    </div>
</header>
```

### App Layout

```svelte
<!-- src/routes/+layout.svelte -->
<script>
    import Titlebar from '$lib/components/Titlebar.svelte';
    import '../app.css';
</script>

<div class="h-screen flex flex-col bg-slate-800">
    <Titlebar />
    <main class="flex-1 overflow-hidden">
        <slot />
    </main>
</div>
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
```svelte
<!-- ❌ Button inside drag region with attribute -->
<div data-tauri-drag-region>
    <button data-tauri-drag-region>Click me</button>
</div>

<!-- ✅ Button without drag region attribute -->
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
```svelte
<!-- ❌ Static icon -->
<button>□</button>

<!-- ✅ Dynamic icon based on state -->
<button>{isMaximized ? '❐' : '□'}</button>
```

### 4. Not Cleaning Up Event Listeners
```svelte
<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    
    let unlisten: (() => void) | null = null;

    onMount(async () => {
        unlisten = await appWindow.onResized(() => { /* ... */ });
    });

    // ✅ Always clean up
    onDestroy(() => {
        if (unlisten) unlisten();
    });
</script>
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