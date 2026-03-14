<div align="center">

# 🧱 Project Bricked

**Military-grade AES-256-GCM file encryption for Windows.**  
Built-in password vault · Chrome & Edge autofill · Google Drive sync · One-click installer.

[![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-0078D4?style=flat-square&logo=windows&logoColor=white)](https://github.com/MaRk9O/Project-Bricked/releases/latest)
[![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=flat-square&logo=dotnet&logoColor=white)](https://dotnet.microsoft.com/download)
[![License](https://img.shields.io/badge/license-MIT-ef4444?style=flat-square)](LICENSE)
[![Release](https://img.shields.io/github/v/release/MaRk9O/Project-Bricked?style=flat-square&color=ef4444&label=latest)](https://github.com/MaRk9O/Project-Bricked/releases/latest)

[**Download Installer →**](https://github.com/MaRk9O/Project-Bricked/releases/latest) &nbsp;·&nbsp; [Website](https://mark9o.github.io/Project-Bricked) &nbsp;·&nbsp; [Report a Bug](https://github.com/MaRk9O/Project-Bricked/issues)

</div>

---

## What It Does

Project Bricked is a Windows desktop application that makes strong encryption practical for everyday use. Right-click any file to encrypt it. Manage all your passwords in a secure vault. Autofill login forms in Chrome or Edge directly from that vault. Sync everything to Google Drive — only the ciphertext ever leaves your machine.

No command line. No configuration files. No accounts. Everything runs locally.

---

## Features

| | Feature | Description |
|---|---|---|
| 🔒 | **AES-256-GCM Encryption** | Every file encrypted and individually authenticated. Tamper-proof by design. |
| 🔑 | **Password Vault** | Encrypted credential store protected by your master password. Import from Chrome in one click. |
| 🌐 | **Browser Extension** | Chrome & Edge extension (MV3) that detects login forms and autofills from your vault. |
| ☁️ | **Google Drive Sync** | Sync your encrypted vault to Drive. Only ciphertext leaves the device. |
| 🖱️ | **Explorer Integration** | "Brick This File" and "Unbrick" live directly in Windows Explorer's right-click menu. |
| 🔄 | **Auto-Updater** | Silently checks for new releases and downloads the latest installer automatically. |
| 🔐 | **Safe Password Change** | Changing your master password re-encrypts the vault and every registered file atomically. |
| 🧹 | **Secure Delete** | Original files are overwritten before deletion. Keys are zeroed from memory after use. |

---

## Quick Start

```bash
git clone https://github.com/MaRk9O/Project-Bricked
cd Project-Bricked
dotnet run --project ProjectBricked
```

---

## Prerequisites

| Tool | Version | Link |
|------|---------|------|
| .NET SDK | 8.0+ | https://dotnet.microsoft.com/download |
| Inno Setup | 6.3+ | https://jrsoftware.org/isdl.php |
| Chrome / Edge | Any current | For the browser extension |
| Visual Studio | 2022+ | Optional — `dotnet` CLI is sufficient |

---

## Building

### WPF App — Debug
```bash
dotnet run --project ProjectBricked
```

### WPF App — Release (single-file)
```bash
dotnet publish ProjectBricked -c Release -r win-x64 \
  --self-contained true \
  -p:PublishSingleFile=true \
  -p:IncludeNativeLibrariesForSelfExtract=true
# → ProjectBricked/bin/Release/net8.0-windows/win-x64/publish/ProjectBricked.exe
```

### Native Messaging Host
```bash
dotnet publish NativeMessagingHost -c Release -r win-x64 \
  --self-contained true \
  -p:PublishSingleFile=true
# → NativeMessagingHost/bin/Release/net8.0/win-x64/publish/ProjectBrickedNativeHost.exe
```

### Installer
1. Build both EXEs above.
2. Open `ProjectBricked.iss` in Inno Setup Compiler.
3. Press **F9** → Compile.
4. Output: `Output/ProjectBrickedInstaller.exe`

---

## Project Layout

```
Project-Bricked/
├── ProjectBricked.csproj          Main WPF app (.NET 8, Windows)
├── App.xaml / App.xaml.cs         Application entry, tray icon, IPC pipes, global styles
├── LockWindow.xaml / .cs          First-launch setup + every-launch unlock screen
├── MainWindow.xaml / .cs          Sidebar layout: Files tab, Vault tab, Settings button
├── SettingsWindow.xaml / .cs      All settings: context menu, CSV import, sync, updates
├── ChangePasswordWindow.xaml/.cs  Master password change — re-encrypts vault + all files
├── EntryEditorWindow.xaml / .cs   Add / edit a vault credential
├── IconHelper.cs                  Runtime tray and window icon generation
│
├── Models/
│   └── Models.cs                  AppConfig, BrickedFileRecord, VaultEntry, UpdateInfo
│
├── Services/
│   ├── CoreServices.cs            SessionService, AppDataService, VaultService
│   │                                 • VaultService.ImportCsv()  — validated CSV import
│   │                                 • VaultService.Rekey()      — password-change re-keying
│   ├── CryptoService.cs           AES-256-GCM, PBKDF2, RekeyFile, secure delete
│   ├── UpdateService.cs           GitHub releases check + installer download
│   └── MigrationService.cs        v1 → v2 format upgrade
│
├── Controls/
│   └── UpdatesPanel.xaml / .cs    Migration panel (shown in SettingsWindow)
│
├── NativeMessagingHost/
│   ├── NativeMessagingHost.csproj Console app — Chrome native messaging bridge
│   ├── NativeMessagingHost.cs     stdin/stdout ↔ named pipe relay
│   └── com.projectbricked.vault.json  Manifest template (paths filled by installer)
│
├── BrickedExtension/              Chrome / Edge extension (Manifest V3)
│   ├── manifest.json
│   ├── popup.html                 Vault search + autofill popup UI
│   ├── background.js              Service worker + native host connection
│   └── content.js                 Form detection, save / update prompts
│
└── ProjectBricked.iss             Inno Setup 6 installer script
```

---

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                   WPF Desktop App                         │
│  MainWindow (sidebar)         VaultService               │
│  ├─ 📁 Files panel             ├─ vault.vlt (AES-GCM)   │
│  ├─ 🔑 Vault panel             ├─ ImportCsv()            │
│  └─ ⚙  Settings button        └─ Google Drive sync      │
│  SettingsWindow                                           │
│  ├─ Context menu (registry)   UpdateService              │
│  ├─ CSV import                └─ GitHub releases API     │
│  ├─ Drive sync                                           │
│  └─ Auto-update               CryptoService              │
│                                ├─ EncryptFile / Decrypt  │
│  ChangePasswordWindow          ├─ RekeyFile (all files)  │
│  └─ Re-encrypts vault +        └─ SecureDelete           │
│     all .bricked files                                   │
│  App.xaml.cs                                             │
│  ├─ Tray icon (NotifyIcon)                               │
│  ├─ Named Pipe: ProjectBrickedIPC       (one-way, CLI)   │
│  └─ Named Pipe: ProjectBrickedIPC_query (InOut, vault)   │
└───────────────────────────┬──────────────────────────────┘
                            │ Named pipe · length-prefixed JSON
┌───────────────────────────▼──────────────────────────────┐
│          NativeMessagingHost.exe (console)                │
│  Chrome native messaging wire protocol (stdin/stdout)     │
│  → relays to WPF via ProjectBrickedIPC_query             │
└───────────────────────────┬──────────────────────────────┘
                            │ Chrome Native Messaging
┌───────────────────────────▼──────────────────────────────┐
│         BrickedExtension (Chrome MV3)                     │
│  background.js — service worker, caches credentials       │
│  popup.html    — vault search + autofill UI               │
│  content.js    — form detection, save/update prompts      │
└──────────────────────────────────────────────────────────┘
```

---

## UI Layout

```
┌──────────────────────────────────────────────────────┐
│  🧱 Project Bricked                       ─  □  ✕   │  Title bar (48 px)
├────┬─────────────────────────────────────────────────┤
│ 📁 │  [Toolbar for current tab]                      │
│    │─────────────────────────────────────────────────│
│ 🔑 │                                                 │
│    │   Tab content (Files or Vault)                  │
│    │                                                 │
│ ⚙  │                                                 │  Settings pinned to bottom
├────┴─────────────────────────────────────────────────┤
│  🔓 Unlocked                               v1.0.0   │  Status bar (28 px)
└──────────────────────────────────────────────────────┘
```

- Sidebar: **56 px** wide · `#161B22` background · `#30363D` divider
- Active item: purple left-edge stripe · `#2D1F5E` fill
- Hover: `#1C2130` fill
- ⚙ opens `SettingsWindow` as a modal dialog

---

## Crypto Reference

### File Format v2 — `magic = "BRICKED\x02"`

```
[magic 8][nonce 12][tag 16][filename_len 4 LE][AES-GCM ciphertext]
```

Ciphertext decrypts to `[filename_bytes (filename_len)][file content]`.

```
AAD = "ProjectBricked-v2"
Key = PBKDF2-SHA256(password, PasswordSalt, 800_000 iterations, 32 bytes)
```

### Vault File — `vault.vlt`

```
[magic "BRICKEDVLT1" 11][nonce 12][tag 16][AES-GCM ciphertext of UTF-8 JSON]
```

### Password Verification — `config.json`

| Field | Purpose |
|-------|---------|
| `PasswordSalt` | Derives the **master key** used to encrypt files and the vault |
| `VerifySalt` | Derives a **verify key** — its SHA-256 hash is stored as `VerifyHash` |

---

## Password Change — Re-encryption Flow

When the user changes their master password, every piece of encrypted data is re-keyed atomically:

1. Current password verified against `VerifyHash` in `config.json`
2. Current session master key cloned as `oldMasterKey`
3. New PBKDF2 master key and verify key derived from the new password
4. `VaultService.Rekey(oldKey, newKey)` re-encrypts `vault.vlt` (`.rekeybackup` created first, removed on success)
5. Every `.bricked` file in `config.json` re-encrypted via `CryptoService.RekeyFile()` on a background thread
6. `config.json` updated with new salts and hash
7. `SessionService.Unlock(newMasterKey)` swaps the in-memory key
8. All sensitive byte arrays zeroed

---

## Key API

### `VaultService.ImportCsv(string filePath)`
*(Added in v1.1)*

```csharp
// Validates header: name,url,username,password (case-insensitive)
// Throws InvalidDataException if header does not match.
// Returns number of entries added or updated.
int count = VaultService.Instance.ImportCsv(path);
```

Export source: `chrome://password-manager → Settings → Export passwords`

### `VaultService.Rekey(byte[] oldKey, byte[] newKey)`

Re-encrypts `vault.vlt` in-place.
Called automatically by `ChangePasswordWindow` — **do not call directly** without also calling `CryptoService.RekeyFile()` for every registered `.bricked` file.

---

## Browser Extension

### Loading for Development

1. Open Chrome → `chrome://extensions`
2. Enable **Developer mode** (top-right toggle)
3. Click **Load unpacked** → select `BrickedExtension/`
4. Copy the **Extension ID** (32-character string on the card)
5. Run the installer and paste the ID when prompted

### Manual Native Host Registration

```powershell
# Edit NativeMessagingHost/com.projectbricked.vault.json first:
#   Replace EXTENSION_ID_PLACEHOLDER with your actual Extension ID
#   Update the path to point to ProjectBrickedNativeHost.exe

reg add "HKCU\Software\Google\Chrome\NativeMessagingHosts\com.projectbricked.vault" `
    /ve /d "C:\path\to\com.projectbricked.vault.json" /f

reg add "HKCU\Software\Microsoft\Edge\NativeMessagingHosts\com.projectbricked.vault" `
    /ve /d "C:\path\to\com.projectbricked.vault.json" /f
```

---

## Google Drive Sync

Built-in OAuth credentials are included — click **Settings → Link Drive** to authorize.

### Custom API Credentials (Optional)

Prefer your own Google Cloud project? Create a Desktop app OAuth Client ID, then:

```json
// %LocalAppData%\ProjectBricked\drive_config.json
{
  "client_id": "YOUR_CLIENT_ID",
  "client_secret": "YOUR_CLIENT_SECRET"
}
```

Restart the app and click **Link Drive**.

---

## Context Menu Integration

| Registry Key | Verb |
|---|---|
| `HKCR\*\shell\BrickThisFile\command` | `"ProjectBricked.exe" --brick "%1"` |
| `HKCR\ProjectBricked.BrickedFile\shell\unbrick\command` | `"ProjectBricked.exe" --unbrick "%1"` |

Written by both the installer and **Settings → Context Menu**. The Settings toggle always reads live registry state, so it stays accurate regardless of how the keys were created.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Extension shows **"Not connected"** | Make sure the app is open and unlocked. Check the native host manifest path in the registry. |
| Extension shows **"Locked"** | Unlock Project Bricked first. |
| Right-click menu missing | Re-run the installer, or use **Settings → Context Menu → Install**. |
| `.bricked` files open in Notepad | Re-run the installer or check `HKCR\.bricked` in Registry Editor. |
| CSV import — header error | Chrome export must produce the header: `name,url,username,password` |
| CSV import — 0 entries imported | All entries already exist in the vault with identical passwords. |
| Drive sync fails | Complete the OAuth flow in Settings; verify `drive_config.json` exists if using custom credentials. |
| Auto-update not working | Enable it via **Settings → Updates → Enable auto-update**. |
| Password change — failed files warning | Some `.bricked` files could not be re-keyed; they still open with the old password. Re-encrypt them manually. |

---

<div align="center">

**🧱 Project Bricked** — Built with security and simplicity in mind.

Free and open source · No warranty · Always keep backups

</div>
