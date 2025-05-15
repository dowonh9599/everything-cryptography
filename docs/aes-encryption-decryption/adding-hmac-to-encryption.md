# Adding HMAC to Encryption

Without HMAC:

* Assumes the encrypted content (`IV + ciphertext`) is untampered.
* No integrity checks are performed. If the ciphertext or IV is altered, decryption will proceed, potentially producing corrupted or incorrect plaintext.
* Vulnerable to **bit-flipping attacks** (where an attacker modifies parts of the ciphertext and observes changes in the decrypted data).
* **simpler use cases** where tampering is not a concern

With HMAC:

* Verifies data integrity using HMAC, protecting against tampering or accidental corruption during storage or transmission.
* Uses **key separation**: separate keys for encryption (`encryption_key`) and integrity verification (`hmac_key`).
* If the data is altered (e.g., by an attacker), decryption will fail.
* **more secure encryption and decryption** where data integrity is critical (e.g., production systems, file transmission over networks).

| Criteria                   | Without HMAC                                                | With HMAC                                           |
| -------------------------- | ----------------------------------------------------------- | --------------------------------------------------- |
| Security                   | High: Protects against tampering and corruption.            | Low: No protection against tampering or corruption. |
| **Integrity Verification** | Yes, uses HMAC to verify data integrity.                    | No, assumes data integrity.                         |
| **Key Separation**         | Uses separate keys for encryption and HMAC.                 | Uses a single key for encryption.                   |
| **Complexity**             | More complex (requires HMAC key and verification).          | Simple (no HMAC or integrity checks).               |
| **Performance**            | Slightly slower due to HMAC verification.                   | Faster, as it skips integrity checks.               |
| **Use Case**               | Secure environments with untrusted storage or transmission. | Simple tasks in trusted environments.               |

## Encryption / Decryption Full Example with AES-CBC, PKCS#7 padding, and HMAC

NOTE: adding `salt` is applied for higher security against tampering.

```python
import os
import hmac
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import padding, hashes
from cryptography.hazmat.backends import default_backend
from secrets import token_bytes

class FileEncryptionHandler:
    @staticmethod
    def derive_key(password, salt, length=32, iterations=310000):
        kdf = PBKDF2HMAC(
            algorithm=hashes.SHA256(),
            length=length,
            salt=salt,
            iterations=iterations,
            backend=default_backend(),
        )
        return kdf.derive(password.encode())

    def encrypt_file(file, encryption_key, hmac_key, chunk_size=1024 * 1024):
        salt = token_bytes(16)
        iv = token_bytes(16)

        encryption_key = self.derive_key(encryption_key, salt)
        hmac_key = self.derive_key(hmac_key, salt)

        cipher = Cipher(algorithms.AES(encryption_key), modes.CBC(iv))
        encryptor = cipher.encryptor()

        mac = hmac.new(hmac_key, iv, hashlib.sha512)

        padder = padding.PKCS7(algorithms.AES.block_size).padder()
        ciphertext = b""

        while chunk := file.read(chunk_size):
            padded_chunk = padder.update(chunk)
            encrypted_chunk = encryptor.update(padded_chunk)
            ciphertext += encrypted_chunk
            mac.update(encrypted_chunk)

        final_padded_data = padder.finalize()
        ciphertext += encryptor.update(final_padded_data) + encryptor.finalize()
        mac.update(ciphertext)

        return b'\x01' + salt + iv + ciphertext + mac.digest()

    def decrypt_file(encrypted_content, encryption_key, hmac_key):
        version = encrypted_content[0]
        if version != 1:
            raise ValueError("Unsupported encryption version.")

        salt = encrypted_content[1:17]
        iv = encrypted_content[17:33]
        ciphertext = encrypted_content[33:-64]
        received_mac = encrypted_content[-64:]

        encryption_key = self.derive_key(encryption_key, salt)
        hmac_key = self.derive_key(hmac_key, salt)

        mac = hmac.new(hmac_key, iv + ciphertext, hashlib.sha512)
        if not hmac.compare_digest(received_mac, mac.digest()):
            raise ValueError("Integrity check failed.")

        cipher = Cipher(algorithms.AES(encryption_key), modes.CBC(iv))
        decryptor = cipher.decryptor()
        padded_data = decryptor.update(ciphertext) + decryptor.finalize()

        try:
            unpadder = padding.PKCS7(algorithms.AES.block_size).unpadder()
            return unpadder.update(padded_data) + unpadder.finalize()
        except ValueError:
            raise ValueError("Decryption failed. File may be corrupted.")
```
