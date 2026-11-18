# Strongbox

A single-file, offline password manager. Everything runs in your browser. There is no server, no account, and no network access.

Your passwords are encrypted with a key derived from your master password. The only thing ever saved or exported is the encrypted vault.

## Contents

- `vault.html`: the entire app (HTML, CSS, and JavaScript in one file)
- `*.vault`: encrypted vault files you export (created by you)

## Quick start

1. Open `vault.html` in a modern browser (Chrome, Edge, Firefox, Safari).
2. Choose a master password of at least 12 characters. A passphrase of 5 or 6 random words is a good choice.
3. Click **Add** to save a login. New entries come with a generated 20-character password, or press **Generate** for another.
4. Click **Export encrypted file** to make your first backup.

## Features

- Add, edit, delete, and search entries (name, username, password, website, notes)
- Cryptographically random password generator
- Copy username or password with one click (clipboard clears after 20 seconds)
- Auto-lock after 3 minutes idle, or 30 seconds after the tab is hidden
- Export and import an encrypted vault file
- Merge changes from another device's vault file
- Change master password
- Light and dark mode, follows your system setting

## Using Strongbox on multiple devices

Strongbox has no cloud sync, by design. You move the encrypted file yourself.

**Set up a new device**
1. Copy `vault.html` to the new device (email, USB drive, cloud drive, or any file transfer).
2. Copy your latest `.vault` file there too. It is safe to store in iCloud, Google Drive, Dropbox, or similar, because it is useless without your master password.
3. Open `vault.html`, choose **Open a vault file from another device**, then unlock with your master password.

**Keep devices in sync**
1. On the device where you made changes, click **Export encrypted file**.
2. Move that file to your other device.
3. On the other device, unlock the vault and click **Merge from file**. Enter the master password of that file.

Merge rules:
- Entries that exist only in the file are added.
- If an entry exists in both, the most recently edited version wins.
- Deletions do not carry over. Delete the entry on each device, or replace the vault with the newer file using **Open a vault file**.

Tip: if you change your master password, re-export and replace old `.vault` files so they all use the new password.

## Security design

| Item | Detail |
|---|---|
| Encryption | AES-256-GCM (authenticated, so tampering is detected) |
| Key derivation | PBKDF2-SHA256, 600,000 iterations, random 16-byte salt |
| Nonce | Fresh random 12-byte IV on every save |
| Randomness | `crypto.getRandomValues` (unbiased selection for generated passwords) |
| Network | Blocked by a Content Security Policy (`default-src 'none'`) |
| Rendering | Entries are inserted as text, never as HTML |
| Stored data | Encrypted blob only, in the browser's `localStorage` and in files you export |

The encryption key and decrypted entries exist only in memory while the vault is unlocked, and are discarded when it locks.

### Vault file format

A `.vault` file is JSON:

```json
{ "v": 1, "kdf": "PBKDF2-SHA256", "iter": 600000, "salt": "...", "iv": "...", "ct": "..." }
```

`salt`, `iv`, and `ct` (ciphertext) are base64. Only `ct` contains your data, and it is encrypted.

## Important limitations

- **There is no password recovery.** If you forget your master password, your data cannot be restored. Keep at least one backup `.vault` file.
- **Each browser keeps its own copy.** Clearing site data, using private browsing, or switching browsers can make the local copy disappear. Your exported file is the real backup.
- **Not audited.** This is a personal tool and has not had an independent security review. For high-value accounts, also use two-factor authentication, or use an audited manager such as Bitwarden or KeePassXC.
- **PBKDF2 is weaker than Argon2** against dedicated cracking hardware. A long passphrase offsets this.
- **No autofill or browser extension.** You copy and paste passwords yourself.
- **The clipboard is shared with other apps.** The 20-second wipe helps, but clipboard managers may keep history.
- **A compromised device defeats any password manager.** Malware or a keylogger on your computer can capture your master password.

## Good habits

- Only run `vault.html` from a copy you trust. The file is the code that handles your passwords, so don't edit it unless you know what you are doing, and don't download copies from unknown sources.
- Don't unlock your vault on shared or public computers.
- Back up your `.vault` file regularly, and keep a copy offline.
- Use a master password you don't use anywhere else.

## Troubleshooting

**"Wrong password, or the vault file is damaged."**
Check caps lock and keyboard layout. If you recently changed your master password, make sure the file you opened was exported after the change.

**My vault is gone after opening in another browser.**
Browsers don't share storage. Open your exported `.vault` file with **Open a vault file from another device**.

**Copy doesn't work.**
Some browsers block clipboard access when a file is opened directly from disk or in an embedded view. Try a different browser, or click the button again after interacting with the page.

**Export did nothing on my phone.**
Some mobile browsers handle downloads differently. Check your Downloads folder, or try another browser.
