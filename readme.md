# File Encryption & Decryption using OpenSSL

A hands-on cryptography experiment demonstrating how to securely generate keys, encrypt sensitive files, and restore them using OpenSSL in a Linux/WSL environment. This project showcases practical implementation of symmetric key cryptography concepts and data integrity verification.

## 🛠️ Prerequisites

To run this experiment, you need to have **OpenSSL** installed on your Linux or Windows Subsystem for Linux (WSL) terminal. 

Check if OpenSSL is installed:
```bash
openssl version
```

---

## 🚀 Step-by-Step Implementation

### Step 1: Generate a Secure Key
Before encrypting a file, we need to generate a strong random key file that will act as our password.
```bash
openssl rand -out secret.key 56
```
* **Explanation:** This command generates 56 bytes of cryptographically secure random data using `openssl rand` and saves it into a key file named `secret.key`.

### Step 2: Encrypt the Target File
Now, we encrypt the original plain text file using the newly generated key with the Triple DES (`3DES`) algorithm.
```bash
openssl enc -des3 -in confidential.txt -out encrypted_data.enc -pass file:secret.key
```
* **Explanation:** 
  * `enc`: Calls the OpenSSL symmetric cipher routines.
  * `-des3`: Specifies the Triple DES encryption algorithm.
  * `-in confidential.txt`: The input file containing sensitive plain text.
  * `-out encrypted_data.enc`: The resulting locked/encrypted binary file.
  * `-pass file:secret.key`: Instructs OpenSSL to read the password directly from our secret key file.

> ⚠️ **Security Note:** Once encryption is successful, the original file (`confidential.txt`) should be manually removed (`rm confidential.txt`) to keep the data secure on the local environment.

### Step 3: Decrypt and Restore the File
To read the contents again, we can unlock the file by reversing the encryption process.
```bash
openssl enc -des3 -d -in encrypted_data.enc -out restored_data.txt -pass file:secret.key
```
* **Explanation:** The additional `-d` flag signals OpenSSL to **decrypt** the file instead of encrypting it, restoring the authentic text into `restored_data.txt`.

---

## 🛡️ Advanced Experiment: Data Tampering & Integrity Verification

To demonstrate how cyber security controls detect unauthorized modifications (Tampering), we can use a Cryptographic Hash Function (SHA-256) to verify data integrity.

### Step 1: Generate Original File Hash
Before sharing or leaving the encrypted file, generate its digital fingerprint (Hash):
```bash
sha256sum encrypted_data.enc > hash.txt
```

### Step 2: Simulate Tampering (The Attack)
We append a single extra byte to the end of `encrypted_data.enc` to corrupt it:
```bash
echo "tamper" >> encrypted_data.enc
```

### Step 3: Detect Tampering (The Defense)
Now run the integrity check using OpenSSL/Linux built-in tool:
```bash
sha256sum -c hash.txt
```

#### 🚫 Expected Output:
```text
encrypted_data.enc: FAILED
sha256sum: WARNING: 1 computed checksum did not match
```
* **Conclusion:** Even though the file is encrypted, the system successfully **detected the tampering attempt** because the file's dynamic footprint changed.
* **Decryption Failure:** If you try to decrypt this tampered file using your key, OpenSSL will throw a `bad decrypt` error, proving that corrupted ciphertexts cannot be safely restored.

---

## 🔒 Best Practices Followed
* **Encrypted Output:** The `.enc` extension is adopted for easy file type identification, although extensions are optional in Linux.
* **Separation of Concerns:** Kept key files and data files separate to match standard cryptographic designs.
* **Credential Protection:** Hardcoded password strings were strictly avoided by utilizing the `-pass file:` parameter.

## 🛑 Warning
Never upload your actual plain text files (`confidential.txt`, `restored_data.txt`) or your secret key files (`secret.key`) to a public repository like GitHub. Doing so completely compromises the security of your encrypted data.

---

## 🔍 Detailed Code & Command Explanation

Here is the technical breakdown of how each command works under the hood:

### 1. Key Generation Command
```bash
openssl rand -out secret.key 56
```
* **`openssl`**: The core command-line toolkit used for cryptographic operations.
* **`rand`**: A specific sub-command within OpenSSL tasked with generating cryptographically secure pseudo-random bytes.
* **`-out secret.key`**: Redirects the generated random bytes away from the terminal screen and saves them into a file named `secret.key`.
* **`56`**: The number of bytes to generate. Since 1 byte equals 8 bits, this creates a key size of 448 bits.

### 2. Encryption Command
```bash
openssl enc -des3 -in confidential.txt -out encrypted_data.enc -pass file:secret.key
```
* **`enc`**: The symmetric cipher routine used to handle encryption and decryption.
* **`-des3`**: Instructs the system to use the Triple DES (3DES) block cipher algorithm. It applies the DES cipher three times to each data block for enhanced security over standard DES.
* **`-in confidential.txt`**: Specifies the target plain-text input file that needs protection.
* **`-out encrypted_data.enc`**: Defines the name of the output ciphertext file.
* **`-pass file:secret.key`**: Instead of typing a visible password string in the terminal, this flag securely feeds the raw binary content of the `secret.key` file as the encryption passphrase.

### 3. Decryption Command
```bash
openssl enc -des3 -d -in encrypted_data.enc -out restored_data.txt -pass file:secret.key
```
* **`-d`**: This is the critical flag for **Decryption**. It signals OpenSSL to reverse the algorithm workflow. It takes the scrambled ciphertext and uses the exact same key to reconstruct the original plaintext structure.

### 4. Integrity Check & Tampering Detection
```bash
sha256sum encrypted_data.enc > hash.txt
```
* **`sha256sum`**: A utility that passes the entire file through the SHA-256 (Secure Hash Algorithm) hashing function. It outputs a fixed-length 64-character hexadecimal string representing the exact mathematical state of the file.
* **`>`**: The bash redirection operator that pipes the hash string directly into a new reference file called `hash.txt`.

```bash
sha256sum -c hash.txt
```
* **`-c`**: The **Check** flag. It reads the filenames and hash records inside `hash.txt`, recalculates the SHA-256 hash of those physical files in real-time, and compares them. If a single bit has been altered, the hashes conflict, forcing a `FAILED` integrity warning.
