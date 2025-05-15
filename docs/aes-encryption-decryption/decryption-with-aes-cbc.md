# Decryption with AES-CBC

This note explains step-by-step how to decrypt a file encrypted with AES in CBC mode, using a prepended Initialization Vector (IV) and PKCS#7 padding. It includes code examples, explanations, and key takeaways for decryption.

## **Full Decryption Example**

Here’s the complete decryption function:

pythonRunCopy

```python
import hashlib
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.primitives import paddingdding

def decrypt_file(encrypted_content, key):
    """
    Decrypt a file encrypted with AES-CBC and PKCS#7 padding.
    Args:
        encrypted_content (bytes): The encrypted content (IV + Ciphertext).
        key (str): The encryption key (string).
    Returns:
        bytes: The decrypted plaintext.
    """

    # Step 1: Extract IV and Ciphertext
    iv = encrypted_content[:16]
    ciphertext = encrypted_content[16:]

    # Step 2: Derive the Encryption Key
    key = hashlib.sha256(key.encode()).digest()

    # Step 3: Create AES Cipher
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv))
    decryptor = cipher.decryptor()

    # Step 4: Decrypt Ciphertext
    padded_data = decryptor.update(ciphertext) + decryptor.finalize()

    # Step 5: Remove PKCS#7 Padding
    unpadder = padding.PKCS7(algorithms.AES.block_size).unpadder()
    plaintext = unpadder.update(padded_data) + unpadder.finalize()

    return plaintext
```

## **1. Extract the IV and Ciphertext**

*   **Code**:

    ```python
    iv = encrypted_content[:16]
    ciphertext = encrypted_content[16:]
    ```
* **Purpose**:
  * Extract the **first 16 bytes** as the IV from the encrypted file.
  * Use the remaining bytes as the ciphertext.
* **Why**:
  * The IV is prepended to the ciphertext during encryption because it is required for decryption.
* **Example**:
  *   Encrypted File (IV + Ciphertext):

      ```
      b'\xa0\xc1\xc8\x93\x8e\x12\xd4\x01\xad\x7d\x91\x12\x8c\x8b\x6f\xf3\x8f\xdb\x92\x1c...'
      ```
  *   Extracted IV:

      ```
      b'\xa0\xc1\xc8\x93\x8e\x12\xd4\x01\xad\x7d\x91\x12\x8c\x8b\x6f\xf3'
      ```
  *   Ciphertext:

      ```
      b'\x8f\xdb\x92\x1c...'
      ```

***

## **2. Derive the Encryption Key**

*   **Code:**

    ```python
    key = hashlib.sha256(key.encode()).digest()
    ```
* **Purpose**:
  * Convert the provided key (string) into a **32-byte key** using SHA-256.
* **Why**:
  * AES requires a fixed-length key (16, 24, or 32 bytes). Hashing ensures the key is the correct length and uniformly random.
* **Example**:
  * Input Key: `"mypassword"`
  *   Derived Key:

      ```
      b'\xe7\xc3\xa1... (32 bytes)'
      ```

***

## **3. Create the AES Cipher**

*   **Code**:

    ```python
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv))
    decryptor = cipher.decryptor()
    ```
* **Purpose**:
  * Create an AES cipher in **CBC mode** using the extracted IV and the derived key.
  * Initialize a **decryptor** object to reverse the encryption process.
* **Why**:
  * CBC mode requires the same IV and key used during encryption to decrypt the ciphertext.
* **Example**:
  * IV: `b'\xa0\xc1\xc8\x93\x8e\x12\xd4\x01\xad\x7d\x91\x12\x8c\x8b\x6f\xf3'`
  * Key: `b'\xe7\xc3\xa1... (32 bytes)'`

***

## **4. Decrypt the Ciphertext**

*   **Code**:

    ```python
    padded_data = decryptor.update(ciphertext) + decryptor.finalize()
    ```
* **Purpose**:
  * Use the decryptor to reverse the encryption process and obtain the **padded plaintext**.
* **Why**:
  * The ciphertext is transformed back into the padded plaintext using the AES algorithm.
* **Example**:
  *   Ciphertext:

      ```
      b'\x8f\xdb\x92\x1c...'
      ```
  *   Padded Plaintext (example):

      ```
      b'Hello, World!\x03\x03\x03'
      ```

***

## **5. Remove PKCS#7 Padding**

*   **Code**:

    ```python
    unpadder = padding.PKCS7(algorithms.AES.block_size).unpadder()
    plaintext = unpadder.update(padded_data) + unpadder.finalize()
    ```
* **Purpose**:
  * Remove the extra bytes added during encryption to return to the original plaintext.
* **Why**:
  * PKCS#7 padding ensures the plaintext length is a multiple of the AES block size (16 bytes). After decryption, the padding must be removed.
* **Example**:
  *   Padded Plaintext:

      ```
      b'Hello, World!\x03\x03\x03'
      ```
  *   Plaintext:

      ```
      b'Hello, World!'
      ```

***

## **Key Takeaways**

1. **Initialization Vector (IV)**:
   * The IV is required for decryption and must match the IV used during encryption.
   * The IV is typically prepended to the encrypted file for easy retrieval.
2. **AES-CBC Mode**:
   * AES in CBC mode requires the same key and IV for both encryption and decryption.
   * CBC mode ensures that ciphertext blocks depend on all previous plaintext blocks.
3. **PKCS#7 Padding**:
   * Ensures that the plaintext length is compatible with AES's block size (16 bytes).
   * Padding must be removed after decryption to restore the original plaintext.
4. **Output Structure**:
   * Encrypted files are stored as `[IV][Ciphertext]`.
   * During decryption, the IV is extracted first, and the ciphertext is decrypted using the AES algorithm.
5. **Key Security**:
   * The encryption key must be securely managed and never hard-coded in production environments.
   * Consider using a Key Management System (KMS) for secure key storage and retrieval.

***

## **Example Input and Output**

#### **Input**

*   Encrypted File:taggerscriptCopy

    ```
    b'\xa0\xc1\xc8\x93\x8e\x12\xd4\x01\xad\x7d\x91\x12\x8c\x8b\x6f\xf3\x8f\xdb\x92\x1c...'
    ```
* Key: `"mypassword"`

#### **Decryption Steps**

1.  Extract IV:taggerscriptCopy

    ```
    b'\xa0\xc1\xc8\x93\x8e\x12\xd4\x01\xad\x7d\x91\x12\x8c\x8b\x6f\xf3'
    ```
2.  Extract Ciphertext:Copy

    ```
    b'\x8f\xdb\x92\x1c...'
    ```
3.  Decrypt Ciphertext:Copy

    ```
    b'Hello, World!\x03\x03\x03'
    ```
4.  Remove Padding:Copy

    ```
    b'Hello, World!'
    ```

#### **Output**

*   Decrypted Plaintext:Copy

    ```
    "Hello, World!"
    ```

***

## **Security Best Practices**

1. **Key Management**:
   * Use a secure method to store and retrieve encryption keys (e.g., AWS KMS, Secrets Manager).
2. **IV Handling**:
   * Always generate a new random IV for encryption and store it with the ciphertext.
3. **Data Integrity**:
   * Consider using HMAC (Hash-Based Message Authentication Code) to ensure the integrity of the encrypted file.
4. **Environment**:
   * Avoid hardcoding keys in your codebase.
