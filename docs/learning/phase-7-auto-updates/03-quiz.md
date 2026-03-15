# Phase 7: Auto-Updates - Quiz

Test your understanding of auto-updates in Tauri.

---

## Multiple Choice Questions

### Question 1
What plugin is used for auto-updates in Tauri v2?

- A) `tauri-plugin-update`
- B) `tauri-plugin-updater`
- C) `@tauri-apps/updater`
- D) `tauri-auto-update`

<details>
<summary>Show Answer</summary>

**B) `tauri-plugin-updater`**

The updater plugin is `tauri-plugin-updater` for Rust and `@tauri-apps/plugin-updater` for JavaScript.
</details>

---

### Question 2
What command generates signing keys for updates?

- A) `pnpm tauri key generate`
- B) `pnpm tauri signer generate`
- C) `pnpm tauri sign create`
- D) `pnpm tauri update-key`

<details>
<summary>Show Answer</summary>

**B) `pnpm tauri signer generate`**

Use `pnpm tauri signer generate -w ~/.tauri/myapp.key` to generate signing keys.
</details>

---

### Question 3
Where should the private signing key be stored for CI/CD?

- A) In `tauri.conf.json`
- B) In the repository root
- C) As a GitHub secret
- D) In `Cargo.toml`

<details>
<summary>Show Answer</summary>

**C) As a GitHub secret**

The private key should be stored as `TAURI_SIGNING_PRIVATE_KEY` in GitHub secrets, never committed to the repository.
</details>

---

### Question 4
What file format does the update manifest use?

- A) XML
- B) YAML
- C) JSON
- D) TOML

<details>
<summary>Show Answer</summary>

**C) JSON**

The update manifest (`latest.json`) is a JSON file containing version info, signatures, and download URLs.
</details>

---

## True or False

### Question 5
True or False: Updates must be signed to be installed.

<details>
<summary>Show Answer</summary>

**True**

Tauri requires updates to be cryptographically signed. The app verifies the signature using the public key before installing.
</details>

---

### Question 6
True or False: The public key should be kept secret.

<details>
<summary>Show Answer</summary>

**False**

The public key is safe to include in `tauri.conf.json` and your repository. Only the private key must be kept secret.
</details>

---

## Short Answer

### Question 7
What function is used to check for updates in the frontend?

<details>
<summary>Show Answer</summary>

```typescript
import { check } from '@tauri-apps/plugin-updater';

const update = await check();
if (update) {
    console.log(`Update available: ${update.version}`);
}
```
</details>

---

### Question 8
How do you restart the app after installing an update?

<details>
<summary>Show Answer</summary>

```typescript
import { relaunch } from '@tauri-apps/plugin-process';

await update.downloadAndInstall();
await relaunch();
```

Use the `relaunch()` function from `@tauri-apps/plugin-process`.
</details>

---

## Score Yourself

| Score | Rating |
|-------|--------|
| 7-8 | ⭐⭐⭐ Excellent! You've mastered Tauri! |
| 5-6 | ⭐⭐ Good! Review the materials you missed |
| 3-4 | ⭐ Fair. Re-read the learning materials |
| 0-2 | Need more practice |

---

## 🎉 Congratulations!

You have completed all 7 phases of the Tauri Note App learning journey!

### Skills Acquired

| Phase | Skill |
|-------|-------|
| 1 | Project Setup & Configuration |
| 2 | IPC & Tauri Commands |
| 3 | File System & Persistence |
| 4 | Native Menus |
| 5 | System Tray |
| 6 | Custom Window Controls |
| 7 | Auto-Updates |

### What's Next?

1. **Build your app**: `pnpm tauri build`
2. **Distribute**: Create releases on GitHub
3. **Explore mobile**: Tauri v2 supports iOS and Android
4. **Create plugins**: Build your own Tauri plugins
5. **Contribute**: Join the Tauri community

### Resources

- [Tauri Documentation](https://v2.tauri.app/)
- [Tauri GitHub](https://github.com/tauri-apps/tauri)
- [Tauri Discord](https://discord.com/invite/tauri)