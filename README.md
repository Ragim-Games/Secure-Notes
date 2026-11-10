# EncNote

A tiny encrypted notepad in **one HTML file**. No install, no server, no dependencies. Double-click `encnote.html`, type, save. Your notes are stored as an encrypted `.encnote` file.

The UI is intentionally plain and WinRAR-like: big labeled buttons, and hovering any button explains what it does in the status bar.

## Quick start

1. Open `encnote.html` in a modern browser (Chrome, Edge, Firefox, Safari).
2. Click **New**, write your note, click **Save**, choose a password.
3. Later, click **Open**, pick your `.encnote` file, enter the password.

| Button | What it does |
|---|---|
| New | Start an empty note |
| Open | Decrypt an `.encnote` file |
| Save | Encrypt and download the note (`Ctrl+S`) |
| Lock | Wipe text and key from memory (`Ctrl+L`) |
| Keyfile | Use any file as a second secret |
| About | Show crypto settings |

## Security design

- **Cipher:** AES-256-GCM (authenticated; any modification is detected). New random 96-bit IV on every save.
- **Key derivation:** PBKDF2-HMAC-SHA512, 1,000,000 iterations, random 128-bit salt.
- **Keyfile (optional):** SHA-256 of the file is mixed into the key material. Without the exact file, the note cannot be decrypted.
- **Header binding:** the KDF settings are authenticated as AAD, so they cannot be downgraded by editing the file.
- **Length hiding:** plaintext is padded to 1 KiB blocks.
- **Primitives:** only the browser's built-in Web Crypto API. No custom cryptography.
- **No network:** a strict Content-Security-Policy blocks all requests, so the page cannot send data anywhere.
- **Auto-lock:** wipes the text and key after idle time (default 5 min). Unsaved changes are downloaded encrypted first.
- **Wrong password:** fails after a 1.5 s delay with a generic message.

### File format

JSON with `app`, `v`, `kdf`, `iter`, `salt`, `iv`, `ct` (all binary fields Base64). Nothing in the file reveals the note content.

## Threat model

**Protects against**
- Someone who steals or copies your `.encnote` file (cloud leak, stolen laptop, backup).
- Offline password guessing, as far as your password allows (see below).
- Tampering with the file.

**Does NOT protect against**
- **A weak password.** Offline attacks cannot be rate-limited by software. Use a random passphrase of 5-6+ words (or 16+ random characters), and/or a keyfile.
- **Malware or a keylogger on your computer.** Once the note is decrypted, the plaintext is on screen and in memory.
- Browser extensions that can read page content. Use a clean browser profile.
- Residue outside the app's control: clipboard contents, OS swap files, screenshots.
- Forgetting the password or losing the keyfile: **there is no recovery.**

## Known limitations

- PBKDF2 is GPU-friendly. Argon2id is stronger against GPU attacks but needs a vendored WASM library, which would break the single-file, zero-dependency goal. PBKDF2 at 1M iterations plus a strong passphrase is still solid; Argon2id is on the roadmap.
- JavaScript cannot reliably zero memory. Locking drops all references, but the garbage collector decides when the bytes are actually gone.
- The session key is kept (non-extractable) in memory while unlocked so saving is fast.

## Roadmap

- Argon2id (vendored WASM) as an option
- Multiple notes per file
- Built-in random password generator

## Contributing

Issues and pull requests are welcome. Please do not add network access or third-party scripts. Security reports: open a private advisory on GitHub.

## License

Pick a license before publishing (MIT is a common choice) and add a `LICENSE` file.
