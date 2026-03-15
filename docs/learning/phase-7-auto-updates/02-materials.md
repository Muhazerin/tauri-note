# Phase 7: Auto-Updates - Learning Materials

## Table of Contents
1. [Introduction to Auto-Updates](#introduction-to-auto-updates)
2. [Setting Up the Updater Plugin](#setting-up-the-updater-plugin)
3. [Generating Signing Keys](#generating-signing-keys)
4. [Update Manifest](#update-manifest)
5. [Implementing Update Logic](#implementing-update-logic)
6. [GitHub Releases Setup](#github-releases-setup)
7. [Complete Example](#complete-example)

---

## Introduction to Auto-Updates

Auto-updates allow your app to automatically download and install new versions. This improves user experience and ensures users have the latest features and security fixes.

### How It Works

1. App checks update endpoint for new version
2. If update available, downloads the update
3. Verifies signature for security
4. Installs update and restarts app

---

## Setting Up the Updater Plugin

### Installation

**1. Add Rust dependency:**

```bash
cd src-tauri
cargo add tauri-plugin-updater
```

**2. Add JavaScript package:**

```bash
pnpm add @tauri-apps/plugin-updater
```

**3. Register plugin:**

```rust
// src-tauri/src/lib.rs
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_updater::Builder::new().build())
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Configuration

In `tauri.conf.json`:

```json
{
  "plugins": {
    "updater": {
      "endpoints": [
        "https://github.com/YOUR_USERNAME/YOUR_REPO/releases/latest/download/latest.json"
      ],
      "pubkey": "YOUR_PUBLIC_KEY_HERE"
    }
  }
}
```

---

## Generating Signing Keys

Updates must be signed to prevent malicious updates.

### Generate Keys

```bash
pnpm tauri signer generate -w ~/.tauri/myapp.key
```

This creates:
- `~/.tauri/myapp.key` - Private key (keep secret!)
- Outputs public key to console

### Store Keys Securely

**Private key**: Store as GitHub secret `TAURI_SIGNING_PRIVATE_KEY`

**Public key**: Add to `tauri.conf.json`:

```json
{
  "plugins": {
    "updater": {
      "pubkey": "dW50cnVzdGVkIGNvbW1lbnQ6IG1pbmlzaWduIHB1YmxpYyBrZXk..."
    }
  }
}
```

### Environment Variable for Signing

Set when building:

```bash
export TAURI_SIGNING_PRIVATE_KEY="content of private key"
export TAURI_SIGNING_PRIVATE_KEY_PASSWORD="your password"
pnpm tauri build
```

---

## Update Manifest

The update manifest (`latest.json`) tells the app about available updates.

### Format

```json
{
  "version": "1.0.1",
  "notes": "Bug fixes and improvements",
  "pub_date": "2024-01-15T12:00:00Z",
  "platforms": {
    "linux-x86_64": {
      "signature": "SIGNATURE_HERE",
      "url": "https://github.com/user/repo/releases/download/v1.0.1/app_1.0.1_amd64.AppImage.tar.gz"
    },
    "windows-x86_64": {
      "signature": "SIGNATURE_HERE",
      "url": "https://github.com/user/repo/releases/download/v1.0.1/app_1.0.1_x64-setup.nsis.zip"
    },
    "darwin-x86_64": {
      "signature": "SIGNATURE_HERE",
      "url": "https://github.com/user/repo/releases/download/v1.0.1/app_1.0.1_x64.app.tar.gz"
    },
    "darwin-aarch64": {
      "signature": "SIGNATURE_HERE",
      "url": "https://github.com/user/repo/releases/download/v1.0.1/app_1.0.1_aarch64.app.tar.gz"
    }
  }
}
```

### Platform Identifiers

| Platform | Identifier |
|----------|------------|
| Windows x64 | `windows-x86_64` |
| macOS Intel | `darwin-x86_64` |
| macOS Apple Silicon | `darwin-aarch64` |
| Linux x64 | `linux-x86_64` |

---

## Implementing Update Logic

### Check for Updates

```typescript
import { check } from '@tauri-apps/plugin-updater';

async function checkForUpdates() {
    try {
        const update = await check();
        
        if (update) {
            console.log(`Update available: ${update.version}`);
            console.log(`Release notes: ${update.body}`);
            return update;
        } else {
            console.log('No update available');
            return null;
        }
    } catch (error) {
        console.error('Failed to check for updates:', error);
        return null;
    }
}
```

### Download and Install

```typescript
import { check } from '@tauri-apps/plugin-updater';
import { relaunch } from '@tauri-apps/plugin-process';

async function downloadAndInstall() {
    const update = await check();
    
    if (update) {
        // Download with progress
        await update.downloadAndInstall((event) => {
            switch (event.event) {
                case 'Started':
                    console.log(`Download started, size: ${event.data.contentLength}`);
                    break;
                case 'Progress':
                    console.log(`Downloaded ${event.data.chunkLength} bytes`);
                    break;
                case 'Finished':
                    console.log('Download finished');
                    break;
            }
        });
        
        // Restart the app
        await relaunch();
    }
}
```

### React Hook for Updates

```typescript
// src/hooks/useUpdater.ts
import { useState, useEffect } from 'react';
import { check, Update } from '@tauri-apps/plugin-updater';
import { relaunch } from '@tauri-apps/plugin-process';

export function useUpdater() {
    const [update, setUpdate] = useState<Update | null>(null);
    const [checking, setChecking] = useState(false);
    const [downloading, setDownloading] = useState(false);
    const [progress, setProgress] = useState(0);

    const checkForUpdate = async () => {
        setChecking(true);
        try {
            const available = await check();
            setUpdate(available);
        } catch (error) {
            console.error('Update check failed:', error);
        } finally {
            setChecking(false);
        }
    };

    const installUpdate = async () => {
        if (!update) return;
        
        setDownloading(true);
        try {
            let totalSize = 0;
            let downloaded = 0;
            
            await update.downloadAndInstall((event) => {
                if (event.event === 'Started') {
                    totalSize = event.data.contentLength || 0;
                } else if (event.event === 'Progress') {
                    downloaded += event.data.chunkLength;
                    setProgress(totalSize ? (downloaded / totalSize) * 100 : 0);
                }
            });
            
            await relaunch();
        } catch (error) {
            console.error('Update failed:', error);
        } finally {
            setDownloading(false);
        }
    };

    return {
        update,
        checking,
        downloading,
        progress,
        checkForUpdate,
        installUpdate,
    };
}
```

---

## GitHub Releases Setup

### GitHub Actions Workflow

Create `.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    permissions:
      contents: write
    strategy:
      fail-fast: false
      matrix:
        platform: [macos-latest, ubuntu-22.04, windows-latest]

    runs-on: ${{ matrix.platform }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Rust
        uses: dtolnay/rust-toolchain@stable

      - name: Install dependencies (Ubuntu)
        if: matrix.platform == 'ubuntu-22.04'
        run: |
          sudo apt-get update
          sudo apt-get install -y libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf

      - name: Install pnpm
        run: npm install -g pnpm

      - name: Install dependencies
        run: pnpm install

      - name: Build Tauri
        uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          TAURI_SIGNING_PRIVATE_KEY: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY }}
          TAURI_SIGNING_PRIVATE_KEY_PASSWORD: ${{ secrets.TAURI_SIGNING_PRIVATE_KEY_PASSWORD }}
        with:
          tagName: v__VERSION__
          releaseName: 'App v__VERSION__'
          releaseBody: 'See the assets to download this version and install.'
          releaseDraft: true
          prerelease: false
          includeUpdaterJson: true
```

### Setting Up Secrets

1. Go to GitHub repo → Settings → Secrets → Actions
2. Add `TAURI_SIGNING_PRIVATE_KEY` (content of your private key)
3. Add `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` (your key password)

---

## Complete Example

### Update Component

```tsx
// src/components/UpdateNotification.tsx
import { useUpdater } from '../hooks/useUpdater';

export function UpdateNotification() {
    const { update, checking, downloading, progress, checkForUpdate, installUpdate } = useUpdater();

    if (checking) {
        return <div className="p-4">Checking for updates...</div>;
    }

    if (!update) {
        return (
            <button onClick={checkForUpdate} className="btn">
                Check for Updates
            </button>
        );
    }

    if (downloading) {
        return (
            <div className="p-4">
                <p>Downloading update...</p>
                <div className="w-full bg-gray-200 rounded">
                    <div 
                        className="bg-blue-600 h-2 rounded" 
                        style={{ width: `${progress}%` }}
                    />
                </div>
                <p>{progress.toFixed(0)}%</p>
            </div>
        );
    }

    return (
        <div className="p-4 bg-blue-100 rounded">
            <h3 className="font-bold">Update Available!</h3>
            <p>Version {update.version} is available.</p>
            {update.body && <p className="text-sm">{update.body}</p>}
            <button onClick={installUpdate} className="btn btn-primary mt-2">
                Install Update
            </button>
        </div>
    );
}
```

---

## Hands-On Exercises

### Exercise 1: Setup Updater Plugin
1. Install the updater plugin
2. Generate signing keys
3. Configure tauri.conf.json

### Exercise 2: Create Update Check
1. Implement checkForUpdate function
2. Display update availability
3. Show version and release notes

### Exercise 3: Implement Download
1. Add download with progress
2. Show progress bar
3. Handle errors

### Exercise 4: GitHub Actions
1. Create release workflow
2. Add secrets to repository
3. Test with a release

---

## Common Pitfalls

### 1. Missing Signing Key
```
Error: Signing key not found
```
Ensure `TAURI_SIGNING_PRIVATE_KEY` is set when building.

### 2. Wrong Endpoint URL
```json
// ❌ Wrong - points to release page
"endpoints": ["https://github.com/user/repo/releases/latest"]

// ✅ Correct - points to latest.json file
"endpoints": ["https://github.com/user/repo/releases/latest/download/latest.json"]
```

### 3. Version Mismatch
Ensure `version` in `tauri.conf.json` matches your release tag.

---

## Summary

In this phase, you learned:
- ✅ How to configure the updater plugin
- ✅ How to generate and manage signing keys
- ✅ How to create update manifests
- ✅ How to implement update UI
- ✅ How to set up GitHub Actions for releases

---

## Congratulations! 🎉

You have completed all phases of the Tauri Note App learning journey!

### What You've Learned

| Phase | Topic |
|-------|-------|
| 1 | Project Setup & Configuration |
| 2 | IPC & Tauri Commands |
| 3 | File System & Persistence |
| 4 | Native Menus |
| 5 | System Tray |
| 6 | Custom Window Controls |
| 7 | Auto-Updates |

### Next Steps

- Build and distribute your app
- Explore Tauri mobile support
- Create your own Tauri plugins
- Contribute to the Tauri community