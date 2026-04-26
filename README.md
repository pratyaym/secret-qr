# secure-qr

Encrypt and decrypt text offline in your browser.

Encryption:
- Password → NFKC normalize → PBKDF2-HMAC-SHA256 (salt=16 random bytes, iterations=600000) → 256-bit AES-GCM key
- Plaintext → AES-GCM encrypt (IV=12 random bytes) → ciphertext || 128-bit tag
- Output: `<base64(salt)>.<base64(iv)>.<base64(ct+tag)>`

Decryption:
Splits the payload into salt, IV, and ciphertext. Re-derives the key via PBKDF2 (600000 iterations) with the stored salt, then decrypts with AES-GCM using the stored IV. All via Web Crypto SubtleCrypto, client-side only.
