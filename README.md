# Vault — Personal Organizer

A privacy-first PWA (Progressive Web App) that runs on **iOS, Android, Windows, macOS, Linux** from a single codebase. All data stays on the user's device.

## Features (built and working)

- **Notes** — title + body, edit, delete, optionally store inside the private Vault
- **Shopping list** — quantities, check off, "clear done"
- **Wishlist** — title, link, price, note; running total
- **Reminders with notifications** — date/time, optional vault-locked, foreground notifications + `.ics` export so any calendar (Google/Outlook/Apple) auto-imports them
- **Important documents** — pictures, PDFs, scans; only visible inside the Vault
- **Private Vault** — biometric (Face ID / Touch ID / Windows Hello / fingerprint via WebAuthn) **or** PIN (PBKDF2-hashed). Auto-locks when the app is backgrounded.
- **Backup & restore** — JSON export/import
- **Offline-first** — service worker caches the app shell

## Install on each platform

| Platform | How |
| --- | --- |
| iOS / iPadOS | Open in Safari → Share → **Add to Home Screen** |
| Android | Open in Chrome → menu → **Install app** |
| Windows / macOS / Linux desktop | Open in Chrome / Edge / Brave → install icon in the address bar |

After installing, the app opens in its own window, gets a launcher icon, and works offline.

## Files

- `index.html` — the entire app (HTML + CSS + vanilla JS, no build step)
- `manifest.json` — PWA manifest with icons + app shortcuts
- `sw.js` — service worker (offline cache + notification click handling)
- `icon.png`, `icon-192.png` — app icons

## Honest limitations of the current build

The original feature request asked for two things that **cannot** be delivered by a pure browser PWA:

1. **Reading Gmail / Outlook inbox inside the app**
   This requires:
   - A backend server holding the OAuth client secret
   - Google OAuth verification (weeks-long review for the `gmail.readonly` scope)
   - Microsoft Graph app registration with admin consent for personal accounts
   - Token storage and refresh logic on the server

   The current build links to Gmail and Outlook web instead. To actually pull mail into the app, add a backend (e.g. a small Node/Express service) that exposes `/auth/google`, `/auth/microsoft`, `/messages`, etc., and call it from the front-end.

2. **Auto-writing appointments to the OS calendar**
   Browsers do not have permission to write to the system calendar. The PWA exports `.ics` files instead — every major calendar app (Google, Apple, Outlook) opens them and offers "Add to calendar" with one tap. A native wrapper (see below) can use the OS calendar API directly.

## Going from PWA to App Store / Play Store

The same `index.html` can be wrapped into native binaries:

- **iOS + Android**: [Capacitor](https://capacitorjs.com/) (`npx cap init` → add `ios`/`android` platforms → `npx cap copy`). Requires Apple Developer ($99/yr) and Google Play Developer ($25 one-time) accounts to publish.
- **Windows / macOS / Linux desktop**: [Tauri](https://tauri.app/) for tiny native binaries, or [Electron](https://www.electronjs.org/) for the more common path.

Native wrappers also unlock:
- Real OS-level notifications scheduled while the app is closed
- Native calendar / contacts integration
- Native IMAP if you don't want to go the OAuth route for mail

## Development

There is no build step. Serve the folder over HTTP/HTTPS (HTTPS is required for service workers, WebAuthn, and notifications outside `localhost`).

```sh
# any static server, e.g.
python3 -m http.server 8000
# or
npx serve .
```

Then open `http://localhost:8000`.
