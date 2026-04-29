# 🔐 Secret QR

Encrypt text with a passkey or password, share it as a QR code. Decrypt by scanning.  
Runs entirely in the browser — no server, no accounts, no data leaves your device.

---

## How it works

Type a message, choose a passkey or password, press **Encrypt**. You get a ciphertext and a scannable QR code. Hand the QR to someone (or scan it yourself on another device), press **Decrypt**, and the original message appears — provided they have the right passkey or know the password.

Everything happens client-side in a single HTML file.

---

## Encryption modes

### 🔑 Passkey (recommended)

Uses **WebAuthn PRF** — a standard browser API that asks your platform authenticator (Face ID, Windows Hello, Google Password Manager, a hardware security key) to produce a deterministic secret. That secret is passed through **HKDF-SHA-256** to derive an **AES-GCM-256** key.

- No password to remember or forget
- The key never leaves your authenticator
- Any device where you have the passkey synced can decrypt
- First use: the app prompts you to create a passkey, then press Encrypt again

### 🔒 Password

Derives an **AES-GCM-256** key from your password using **PBKDF2-SHA256** at 600,000 iterations with a random 16-byte salt. The salt is embedded in the payload so decryption only needs the password.

---

## Payload format

Both modes produce a compact dot-separated base64url string:

```
<param>.<iv>.<ciphertext>
```

- **Passkey mode** — `param` is the credential ID (ties the payload to the passkey that encrypted it)
- **Password mode** — `param` is the 16-byte PBKDF2 salt

This string is what gets encoded into the QR code and can be safely stored or shared as plain text.

---

## Features

- **QR generation & scanning** — encrypted output renders as a scannable QR; scan from camera or upload an image
- **Auto-clear** — decrypted plaintext is wiped from the screen after 60 seconds
- **Password strength meter** — live feedback on length, character variety, and entropy
- **Offline-capable** — ships with a service worker for full offline use
- **Dark mode** — follows system preference
- **Installable PWA** — add to home screen on mobile

---

## Security notes

- The app never makes network requests during encrypt/decrypt. The only outbound requests are CDN loads for QR libraries on first visit; subsequent visits work offline via the service worker.
- Each passkey registration uses a unique random `user.id` so new passkeys never overwrite existing ones — old encrypted payloads remain decryptable.
- PRF support is verified at passkey creation time. If your authenticator doesn't support PRF, you'll be told immediately rather than at encrypt time.
- Ciphertext integrity is provided by AES-GCM's authentication tag — tampering is detected on decrypt.

---

## Browser support

Passkey mode requires **WebAuthn PRF**, which is supported in:

| Browser | Support |
|---|---|
| Chrome / Edge 116+ | ✅ |
| Safari 18+ | ✅ |
| Firefox | ❌ Not yet |

Password mode works in any modern browser.

---

## Running locally

No build step. Just open `index.html`.

For passkey support the page must be served over HTTPS or `localhost` — the WebAuthn API requires a secure context.

```bash
# Python
python3 -m http.server 8080

# Node
npx serve .
```

Then open `http://localhost:8080`.

---

## Deployment

Drop `index.html`, `manifest.json`, `sw.js`, and the icon files onto any static host (GitHub Pages, Cloudflare Pages, Netlify, etc.). No backend required.

---

## Cryptography

| Primitive | Usage |
|---|---|
| WebAuthn PRF | Passkey key material derivation |
| HKDF-SHA-256 | Key extraction from PRF output |
| AES-GCM-256 | Authenticated encryption |
| PBKDF2-SHA-256 (600k) | Password-based key derivation |

All cryptographic operations use the browser's native **Web Crypto API** (`crypto.subtle`). No third-party crypto libraries.

---

## Acknowledgements

QR rendering by [qr-code-styling](https://github.com/kozakdenys/qr-code-styling). QR scanning by [jsQR](https://github.com/cozmo/jsQR).
