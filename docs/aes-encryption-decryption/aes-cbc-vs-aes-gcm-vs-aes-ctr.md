# AES-CBC vs AES-GCM vs AES-CTR

AES offers multiple modes of operation, each tailored to specific security and performance needs. Among the most widely used modes are AES-GCM (Galois/Counter Mode), AES-CBC (Cipher Block Chaining), and AES-CTR (Counter Mode).

## **1. CBC (Cipher Block Chaining)**

**Overview**:

* **Cipher Block Chaining (CBC)** is one of the oldest and most widely used AES modes. It encrypts data in fixed-size **blocks** (16 bytes for AES) and uses a **chaining mechanism**, where the output of one block encryption is used as input for the next block.
* It requires an **Initialization Vector (IV)** for the first block to ensure randomness and prevent identical plaintext blocks from producing identical ciphertexts.

**How It Works**:

1. The plaintext is divided into fixed-size blocks.
2. Each block is XORed with the ciphertext of the previous block before encryption.
3. The first block is XORed with the IV.

**Key Characteristics**:

* **Integrity**: Does not provide built-in integrity verification. A separate **HMAC** is required to ensure data integrity.
* **Chaining**: Errors in one block propagate to subsequent blocks, making it less resilient to tampering.
* **Padding**: Requires padding for plaintext that is not a multiple of the block size.

**Common Use Cases**:

* Encryption of fixed-size data (e.g., files or database records) in **legacy systems**.
* Applications where built-in integrity is not required.

***

## **2. CTR (Counter Mode)**

**Overview**:

* **Counter Mode (CTR)** transforms AES into a **stream cipher**, making it suitable for encrypting arbitrary-length data. Instead of chaining blocks, CTR generates a **keystream** by encrypting a counter value combined with a nonce (unique number).
* Each block is processed independently, making CTR highly parallelizable and efficient.

**How It Works**:

1. A unique **nonce** and a **counter** are combined for each block.
2. The combined value is encrypted to produce a keystream block.
3. The plaintext is XORed with the keystream to produce ciphertext.

**Key Characteristics**:

* **No Padding**: Works like a stream cipher, so padding is unnecessary.
* **Parallelism**: Each block can be encrypted or decrypted independently, making it highly efficient.
* **No Integrity**: No built-in integrity verification; requires a separate **HMAC** for tamper detection.

**Common Use Cases**:

* **Streaming encryption** (e.g., video or audio).
* Encrypting **large files** or data streams due to its efficiency.
* **Real-time applications** where performance is critical.

***

## **3. GCM (Galois/Counter Mode)**

**Overview**:

* **Galois/Counter Mode (GCM)** is a modern mode of AES that combines the performance of **CTR** mode with built-in **integrity verification** using a **Galois field**. It is an **authenticated encryption mode** that ensures both **confidentiality** and **integrity** of the data.
* GCM generates an **authentication tag** during encryption, which can be used during decryption to verify the integrity of the ciphertext.

**How It Works**:

1. GCM operates like CTR mode for encryption, using a nonce and counter to generate a keystream.
2. Simultaneously, it computes an **authentication tag** using a Galois field multiplication of the ciphertext and additional authenticated data (if any).
3. During decryption, the tag is verified to ensure data integrity.

**Key Characteristics**:

* **Authenticated Encryption**: Provides both encryption and integrity verification in a single step.
* **No Padding**: Like CTR, GCM does not require padding.
* **Parallelism**: Encryption and integrity computations are highly parallelizable, offering excellent performance.

**Common Use Cases**:

* **Secure communication protocols** (e.g., TLS, HTTPS, SSH).
* **Data transmission** where both confidentiality and integrity are critical.
* **Modern systems** where performance, security, and simplicity are priorities.

{% hint style="info" %}
While AES-CBC or AES-CTR with HMAC is secure when implemented correctly (as in your code), switching to **AES-GCM** (Galois/Counter Mode) is often recommended for simplicity and modern cryptographic practices. AES-GCM provides **authenticated encryption** (encryption + integrity verification) in one step, reducing the complexity and potential for implementation errors.
{% endhint %}

| **Encryption/Decryption**  | Authenticated encryption (encryption + integrity verification). | Pure encryption, no built-in integrity.               | Pure encryption, no built-in integrity.     |
| -------------------------- | --------------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------- |
| **Integrity Verification** | Built-in (authentication tag).                                  | Requires separate HMAC for integrity.                 | Requires separate HMAC for integrity.       |
| **Performance**            | Fast due to combined encryption and integrity.                  | Slower than AES-CTR because of padding overhead.      | Fast; no padding required.                  |
| **Padding**                | Not required.                                                   | Requires padding for non-block aligned data.          | Not required (stream-like mode).            |
| **Parallelization**        | Easily parallelizable for high-performance.                     | Not parallelizable; operates in blocks.               | Easily parallelizable for high-performance. |
| **Best Use Case**          | Secure transmission of data (e.g., network communication).      | Legacy systems or compatibility with older standards. | Large files or streaming encryption.        |
| **Cryptographic Strength** | Strong due to combined encryption + authentication.             | Strong but must use HMAC for integrity.               | Strong but must use HMAC for integrity.     |
| **Complexity**             | Simpler (combines encryption and authentication).               | More complex (requires separate HMAC).                | More complex (requires separate HMAC).      |
