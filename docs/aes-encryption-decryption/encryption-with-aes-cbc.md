# Encryption with AES-CBC

This note explains step-by-step how to decrypt a file encrypted with AES in CBC mode, using a prepended Initialization Vector (IV) and PKCS#7 padding. It includes code examples, explanations, and key takeaways.

## Full Encryption Example

```python
import os
import hashlib
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import padding

def encrypt_file(file, key):
    # Step 1: Generate IV
    iv = os.urandom(16)

    # Step 2: Derive Key
    key = hashlib.sha256(key.encode()).digest()

    # Step 3: Create AES Cipher
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv))
    encryptor = cipher.encryptor()

    # Step 4: Read and Pad Plaintext
    plaintext = file.read()
    padder = padding.PKCS7(algorithms.AES.block_size).padder()
    padded_data = padder.update(plaintext) + padder.finalize()

    # Step 5: Encrypt Padded Data
    ciphertext = encryptor.update(padded_data) + encryptor.finalize()

    # Step 6: Combine IV and Ciphertext
    return iv + ciphertext
```

## 1. Generate a Random Initialization Vector (IV)

```python
iv = os.urandom(16)
```

* **Purpose**:
  * IV ensures that encrypting the same plaintext with the same key produces different ciphertexts.
  * It adds randomness and prevents patterns in the ciphertext.
* **Example Output**:

```
iv = b'\xa0\xc1\xc8\x93\x8e\x12\xd4\x01\xad\x7d\x91\x12\x8c\x8b\x6f\xf3'
```

***

## 2. Derive the Encryption Key

```python
key = hashlib.sha256(key.encode()).digest()
```

* **Purpose**:
  * AES requires a key of specific lengths (16, 24, or 32 bytes). Hashing the key ensures it is the correct length and uniformly random.

**Example**:

* Input Key: `"mypassword"`
* Derived Key (SHA-256):

```
b'\xe7\xc3\xa1... (32 bytes)'
```

***

## **3. Create AES Cipher in CBC Mode**

*   **Code**:

    ```python
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv))
    encryptor = cipher.encryptor()
    ```
* **Purpose**:
  * CBC (Cipher Block Chaining) mode requires an IV to make encryption non-deterministic.
  * The `encryptor` object is used to encrypt data.

***

## **4. Read Plaintext and Apply PKCS#7 Padding**

*   **Code**:pythonRunCopy

    ```
    plaintext = file.read()
    padder = padding.PKCS7(algorithms.AES.block_size).padder()
    padded_data = padder.update(plaintext) + padder.finalize()
    ```
* **Purpose**:
  * AES requires the input length to be a multiple of the block size (16 bytes).
  * PKCS#7 padding adds extra bytes to the plaintext to meet this requirement.
* **Example**:
  * Plaintext: `"Hello, World!"` (13 bytes)
  *   Padded Plaintext:

      ```
      b'Hello, World!\x03\x03\x03'  # Adds 3 padding bytes
      ```

***

## **5. Encrypt the Padded Data**

*   **Code**:

    ```
    ciphertext = encryptor.update(padded_data) + encryptor.finalize()
    ```
* **Purpose**:
  * The padded plaintext is processed by the AES encryptor to produce the ciphertext.
*   **Example Output** (example, as it depends on the IV and key):Copy

    ```
    b'\x8f\xdb\x92\x1c... (encrypted bytes)'
    ```

***

## **6. Combine IV and Ciphertext**

*   **Code**:pythonRunCopy

    ```
    return iv + ciphertext
    ```
* **Purpose**:
  * The IV is prepended to the ciphertext. This is necessary because the IV is required for decryption but doesn’t need to be kept secret.
*   **Output Format**:Copy

    ```
    [16 bytes IV][Ciphertext]
    ```
*   **Example Output**:taggerscriptCopy

    ```
    b'\xa0\xc1\xc8\x93\x8e\x12\xd4\x01\xad\x7d\x91\x12\x8c\x8b\x6f\xf3\x8f\xdb\x92\x1c...'
    ```

***

### **Key Takeaways**

1. **Initialization Vector (IV)**:
   * The IV must be **random** and **unique** for every encryption.
   * The IV does not need to be secret but must be stored with the ciphertext.
2. **PKCS#7 Padding**:
   * Ensures the plaintext length is a multiple of the AES block size (16 bytes).
   * Padding must be removed after decryption.
3. **AES-CBC Mode**:
   * AES in CBC mode needs an IV to prevent deterministic encryption.
   * The same key and IV must be used for both encryption and decryption.
4. **Output Structure**:
   * The encrypted file is stored as `[IV][Ciphertext]`.

***

### **Example Input and Output**

#### **Input**

* Plaintext: `"Hello, World!"`
* Key: `"mypassword"`

#### **Output**

* IV (16 bytes): `b'\xa0\xc1\xc8\x93\x8e\x12\xd4\x01\xad\x7d\x91\x12\x8c\x8b\x6f\xf3'`
* Ciphertext: `b'\x8f\xdb\x92\x1c...'`
*   Combined Output:taggerscriptCopy

    ```
    b'\xa0\xc1\xc8\x93\x8e\x12\xd4\x01\xad\x7d\x91\x12\x8c\x8b\x6f\xf3\x8f\xdb\x92\x1c...'
    ```

***

### **Additional Notes**

* Always use a **secure random generator** (e.g., `os.urandom()`) to generate the IV.
* Never reuse the same IV with the same key for different plaintexts.
* Use a secure key management system (KMS) for storing and retrieving keys in production environments.
