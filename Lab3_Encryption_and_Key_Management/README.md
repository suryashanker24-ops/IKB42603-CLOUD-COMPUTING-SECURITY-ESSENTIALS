# Lab 3: Data Protection - Encryption & Key Management Report

## Student Information

- **Name:** Surya Giri A/L Shanker
- **Student ID:** 52215124335
- **Course:** IKB42603 Cloud Computing Security Essentials
- **Lab Task:** Lab 3 - Data Protection: Encryption & Key Management
- **Lecturer Name:** Madam Nor Adani Kamal Mohammad Nasir

---

## Overview

This report documents the implementation and verification of encryption and key management controls for sensitive cloud data, demonstrating how confidentiality, integrity, authentication, secure communications, tenant separation, and cryptographic erasure work together as a defense-in-depth strategy. The lab was conducted in two sessions over two weeks. Session A focused on establishing cryptographic fundamentals by implementing symmetric encryption (AES-256) to protect data at rest, asymmetric encryption (RSA) for public-key operations and digital signatures, and Transport Layer Security (TLS) to protect data in transit over network connections. Session B extended these foundational concepts into cloud-scale key management by using a Key Management Service (KMS) to create tenant-specific master keys, implementing envelope encryption patterns for efficient large-data protection, demonstrating per-tenant key isolation, performing cryptographic erasure through key deletion, and verifying data integrity using cryptographic hashing and hash chains. The purpose of this comprehensive lab was to develop practical understanding that encryption algorithms alone are insufficient—effective security requires robust key management including key generation, secure storage, access control, lifecycle management, rotation, and secure deletion. Both sessions were executed using command-line tools (OpenSSL for cryptographic operations, Docker and Nginx for TLS demonstration, AWS CLI with LocalStack for KMS operations) in a Kali Linux environment, and each task was systematically documented with terminal commands and screenshots as evidence of successful implementation.

---

## Objectives

The objectives of this lab across both sessions are:

**Session A Objectives (Week 5 - Encryption Fundamentals):**

- Create a sample sensitive record representing protected health information to use throughout encryption demonstrations.
- Implement symmetric encryption using AES-256-CBC with PBKDF2 key derivation and salt to protect data at rest.
- Demonstrate that encrypted ciphertext is unreadable without the correct decryption key or password.
- Decrypt the encrypted record and verify successful recovery by comparing with the original plaintext.
- Generate an RSA 2048-bit asymmetric key pair consisting of a private key and corresponding public key.
- Demonstrate public-key encryption where anyone can encrypt with the public key but only the private key holder can decrypt.
- Create and verify digital signatures where the private key signs data and the public key verifies authenticity and integrity.
- Generate a self-signed X.509 certificate for local TLS testing.
- Configure and deploy an HTTPS web server using Nginx in Docker to serve content over encrypted TLS connections.
- Verify that data transmitted over HTTPS is protected from network eavesdropping.

**Session B Objectives (Week 6 - Key Management, Envelope Encryption & Erasure):**

- Start LocalStack to provide a local AWS KMS-compatible endpoint for key management demonstrations.
- Create customer-managed KMS master keys representing tenant-specific encryption keys.
- Encrypt small secrets directly with KMS to understand direct encryption use cases and limitations.
- Implement envelope encryption by generating data encryption keys (DEKs) from KMS for local file encryption.
- Use plaintext data keys temporarily for fast local encryption operations with OpenSSL.
- Store only KMS-wrapped (encrypted) data keys alongside encrypted data, never plaintext keys.
- Destroy plaintext data key material after use to prevent key exposure.
- Create separate KMS keys per tenant to demonstrate multi-tenant key isolation.
- Schedule KMS key deletion with a pending window to simulate secure key lifecycle management.
- Disable KMS keys and attempt decryption to prove that key lifecycle states are enforced.
- Demonstrate cryptographic erasure where encrypted data becomes permanently unrecoverable after key deletion.
- Calculate SHA-256 cryptographic hashes to verify file integrity and detect tampering.
- Build a simple hash chain to demonstrate tamper-evident audit log structures.
- Re-verify RSA signatures from Session A to confirm data integrity has been maintained.

**Common Objectives:**

- Document all procedures clearly using terminal commands and screenshots as evidence of implementation.
- Understand that encryption is only as strong as its key management practices.
- Recognize the key-distribution problem with symmetric encryption and how asymmetric cryptography addresses it.
- Understand envelope encryption as the practical pattern for cloud-scale data protection.
- Learn cryptographic erasure as the cloud-native approach to secure data deletion.
- Apply defense-in-depth principles across data at rest, data in transit, and data integrity.

---

## Learning Outcomes

By completing this lab across both sessions, the student should be able to:

**Session A Outcomes (Encryption Fundamentals):**

- Explain the difference between symmetric and asymmetric encryption, including speed characteristics, key management requirements, and appropriate use cases for each approach.
- Use OpenSSL command-line tools to encrypt files with AES, decrypt ciphertext, generate RSA key pairs, perform public-key encryption and decryption, create digital signatures, and verify signature authenticity.
- Understand the role of Password-Based Key Derivation Functions (PBKDF2) in strengthening password-based encryption and the importance of salt in preventing rainbow table attacks.
- Recognize the complementary roles of encryption and digital signatures in asymmetric cryptography: encryption protects confidentiality using the public key, while signing proves authenticity using the private key.
- Generate X.509 certificates and understand their role in establishing trust for TLS connections.
- Configure web servers (Nginx) to serve content over HTTPS with TLS encryption.
- Understand why TLS is essential for protecting data in transit and preventing eavesdropping attacks on network communications.
- Recognize that self-signed certificates are suitable only for development and that production systems require certificates from trusted Certificate Authorities.

**Session B Outcomes (Key Management & Envelope Encryption):**

- Understand the architecture of cloud Key Management Services (KMS) and why centralizing key management improves security.
- Distinguish between direct KMS encryption (suitable for small secrets like passwords) and envelope encryption (required for large data like files and databases).
- Implement the envelope encryption pattern: generate a data encryption key from KMS, encrypt data locally with the DEK, store the KMS-wrapped DEK with the encrypted data, and destroy the plaintext DEK.
- Recognize that envelope encryption combines the performance benefits of symmetric encryption with the security and access control benefits of centralized key management.
- Understand per-tenant key isolation as a security architecture pattern that limits blast radius of key compromise and enables selective cryptographic erasure.
- Explain KMS key lifecycle states including enabled, disabled, and pending deletion, and understand how these states enforce access control.
- Implement cryptographic erasure by deleting encryption keys to make encrypted data permanently and provably unrecoverable.
- Understand why cryptographic erasure is the preferred cloud-native approach to secure data deletion compared to traditional overwriting methods.
- Use cryptographic hash functions (SHA-256) to verify data integrity and detect unauthorized modifications.
- Understand hash chains as a technique for creating tamper-evident audit logs where any modification to history breaks all subsequent hashes.

**Common Outcomes:**

- Recognize that encryption algorithms themselves are not the weakest link—poor key management practices are the primary cause of encryption failures.
- Understand that key generation, storage, distribution, access control, rotation, and deletion must all be handled correctly for encryption to provide effective security.
- Apply the principle of least privilege to key access using KMS policies and IAM permissions.
- Document technical security procedures clearly using terminal commands, YAML configurations, and visual evidence.
- Understand how data protection controls (encryption, key management, integrity verification) complement access control and isolation mechanisms from previous labs.

---

## Environment and Prerequisites

The lab was conducted on a Kali Linux environment with Docker installed and internet access for downloading container images and KMS manifests. The following tools and conditions were required before starting the lab:

**Session A Prerequisites (Encryption Fundamentals):**

- OpenSSL installed and available in the command-line PATH for performing cryptographic operations including encryption, key generation, certificate creation, and signature operations.
- Docker installed and running to support containerized infrastructure for the TLS demonstration with Nginx web server.
- Basic understanding of encryption concepts including plaintext, ciphertext, keys, and the difference between symmetric and asymmetric cryptography.
- A terminal or command-line interface with bash or PowerShell shell access for executing OpenSSL and Docker commands.
- Sufficient disk space for storing test files including plaintext records, encrypted ciphertext, RSA key pairs, and TLS certificates.
- Text editor or the ability to create files using echo commands for creating sample sensitive data records.

**Session B Prerequisites (Key Management & Envelope Encryption):**

- LocalStack installed and configured to provide a local AWS KMS-compatible endpoint for key management demonstrations without requiring actual AWS cloud access or credentials.
- AWS CLI v2 installed and configured to interact with the LocalStack KMS endpoint using the --endpoint-url parameter.
- Understanding of the envelope encryption pattern and why it is necessary for cloud-scale data protection.
- Understanding of key lifecycle concepts including key creation, enabling, disabling, scheduling deletion, and permanent deletion.
- Base64 encoding/decoding utilities (base64 command) for handling KMS data key output formats.
- Files and keys from Session A including record.txt (sample sensitive record), private.pem and public.pem (RSA key pair), and record.sig (digital signature) for continuity and verification tasks.

**Common Prerequisites:**

- Administrative privileges on the local system for managing Docker containers and installing required tools.
- Internet connectivity for downloading Docker images (nginx, alpine, curlimages/curl), Calico manifests (if applicable), and AWS CLI packages.
- Sufficient system resources (CPU, memory, disk) to run Docker containers and LocalStack services.
- Basic familiarity with command-line operations including piping, redirection, environment variables, and here-documents for inline YAML configuration.
- Understanding of file permissions and the importance of protecting private keys from unauthorized access.

**Security Note:** This lab uses a local development environment (LocalStack) and self-signed certificates that are appropriate for learning and testing but would be insufficient for production deployments. Production systems should use actual cloud KMS services (AWS KMS, Azure Key Vault, Google Cloud KMS) with proper IAM policies, hardware security modules (HSMs) for key protection, certificates from trusted Certificate Authorities, encrypted network communication, comprehensive audit logging, and regular key rotation policies.

---

## Session A (Week 5) — Encryption Fundamentals

### Task 1 — Symmetric Encryption (Data at Rest)

Symmetric encryption uses a single secret key (or a key derived from a password) for both encryption and decryption operations. This approach is computationally efficient, making it suitable for encrypting large volumes of data such as files, database records, disk volumes, and backup archives. The Advanced Encryption Standard (AES) with 256-bit keys (AES-256) is the current industry standard for symmetric encryption, providing strong security with acceptable performance characteristics on modern hardware. In this task, AES-256 was used in Cipher Block Chaining (CBC) mode, which is a block cipher mode that chains blocks together to ensure that identical plaintext blocks produce different ciphertext blocks. To strengthen the encryption key derivation from the user-supplied password, PBKDF2 (Password-Based Key Derivation Function 2) was used, which applies many iterations of a cryptographic hash function to make brute-force attacks computationally expensive. A random salt was added to ensure that the same password and plaintext produce different ciphertext on separate encryption operations, preventing rainbow table attacks and making it impossible to identify identical encrypted files by comparing their ciphertext. The task created a sample patient record containing protected health information (PHI), encrypted it to demonstrate confidentiality protection, proved that the ciphertext is unreadable without the key, and then decrypted it to verify successful recovery of the original data.

#### Purpose

- Create a sample sensitive patient record to serve as plaintext data for encryption demonstrations.
- Encrypt the record using AES-256-CBC symmetric encryption with PBKDF2 key derivation and salt.
- Demonstrate that encrypted ciphertext cannot be read as ordinary text without the decryption key.
- Decrypt the ciphertext and verify that the recovered plaintext exactly matches the original record.
- Understand the key-distribution problem: both encryption and decryption require the same secret key, creating challenges for secure key sharing.

#### Terminal Commands

```bash
# Create a sample sensitive record
echo 'Patient: surya, Diagnosis: confidential' > record.txt

# Encrypt with AES-256 (you will be prompted for a passphrase = the key)
openssl enc -aes-256-cbc -pbkdf2 -salt -in record.txt -out record.enc

# Prove it is unreadable
cat record.enc

# Decrypt back
openssl enc -d -aes-256-cbc -pbkdf2 -in record.enc -out record.dec.txt

# Verify successful decryption
diff record.txt record.dec.txt && echo 'MATCH: decryption successful'
```

#### Explanation of the Commands

- `echo 'Patient: surya, Diagnosis: confidential' > record.txt` creates a new text file named record.txt containing a sample patient record with a diagnosis field, simulating sensitive protected health information (PHI) that requires confidentiality protection under regulations like HIPAA.
- `openssl enc -aes-256-cbc` invokes OpenSSL's symmetric encryption command (enc) using the AES cipher with a 256-bit key in Cipher Block Chaining (CBC) mode, which is a widely-supported block cipher mode that provides good security when used with proper key derivation and initialization vectors.
- `-pbkdf2` specifies that the encryption key should be derived from the user-supplied password using PBKDF2 (Password-Based Key Derivation Function 2), which applies many iterations of a cryptographic hash function to make brute-force password attacks computationally expensive, replacing OpenSSL's legacy and weaker default key derivation method.
- `-salt` adds random salt data to the key derivation process, ensuring that the same password and plaintext will produce different ciphertext and derived keys on separate encryption operations, preventing attackers from using pre-computed rainbow tables and making it impossible to identify duplicate encrypted files by comparing ciphertext.
- `-in record.txt -out record.enc` specifies the input plaintext file (record.txt) to be encrypted and the output ciphertext file (record.enc) where the encrypted result will be written.
- During execution, OpenSSL prompts the user to enter and verify a password, which is used as the source material for PBKDF2 key derivation—this password becomes the secret key that must be protected and remembered for decryption.
- `cat record.enc` displays the contents of the encrypted file to the terminal, which appears as unreadable binary data (random-looking bytes) rather than the original text, demonstrating that the plaintext is no longer exposed in the ciphertext and that encryption has successfully protected confidentiality.
- `openssl enc -d -aes-256-cbc -pbkdf2` performs the decryption operation using the same algorithm and key derivation method, with the `-d` flag specifying decrypt mode instead of the default encrypt mode.
- `-in record.enc -out record.dec.txt` specifies the input ciphertext file and the output plaintext file where the decrypted result will be written, with OpenSSL prompting for the password to derive the decryption key.
- `diff record.txt record.dec.txt` compares the original plaintext file with the decrypted file byte-by-byte, outputting nothing if the files are identical (successful decryption) or displaying the differences if decryption failed or used an incorrect password.
- `&& echo 'MATCH: decryption successful'` executes only if the diff command succeeds (finds no differences), printing a confirmation message that proves the decrypted file is identical to the original plaintext, verifying that encryption and decryption worked correctly.

#### Evidence

![Create Sample Record](evidence/Task%201%20create%20a%20sample%20sensitive%20record.png)

The screenshot shows the creation of record.txt containing the sample patient record "Patient: surya, Diagnosis: confidential" using the echo command with output redirection.

![AES-256 Encryption](evidence/Task%201%20encrypt%20with%20aes-256.png)

The screenshot demonstrates the execution of the OpenSSL encryption command, showing the password prompt where the user enters and verifies the encryption passphrase, confirming that the encryption operation completed and wrote the ciphertext to record.enc.

![Unreadable Ciphertext](evidence/Task%201%20prove%20its%20inreadable.png)

The screenshot displays the output of `cat record.enc` showing binary data and unprintable characters instead of the original readable text, proving that the encrypted file does not expose the sensitive patient information in plaintext form and that confidentiality protection is in effect.

![Successful Decryption](evidence/Task%201%20Decrypt%20back.png)

The screenshot shows the decryption command execution with password prompt, followed by the diff comparison command outputting no differences, and the success message "MATCH: decryption successful" confirming that the decrypted file (record.dec.txt) is byte-for-byte identical to the original plaintext file (record.txt).

#### Notes

The symmetric encryption task successfully demonstrated that AES-256-CBC with PBKDF2 and salt provides strong confidentiality protection for sensitive data at rest. The unreadable ciphertext proves that an attacker who obtains the encrypted file cannot access the protected health information without knowing the password. The successful decryption with matching output confirms that the encryption is reversible when the correct key is provided, meeting the fundamental requirement that authorized users can recover their encrypted data.

However, this demonstration also reveals the key-distribution problem inherent in symmetric encryption: the same secret key (password) must be securely shared between all parties who need to encrypt or decrypt the data. In cloud environments or multi-user systems, securely distributing this shared secret becomes challenging—sending passwords over networks risks interception, storing passwords in code or configuration files creates exposure, and sharing passwords with many users increases the risk of compromise. This limitation motivates the need for asymmetric cryptography (demonstrated in Task 2) where different keys are used for encryption and decryption, and for centralized key management services (demonstrated in Session B) where keys are protected in hardware security modules and accessed through authenticated API calls rather than being directly distributed to applications.

Production systems should also note that CBC mode requires proper initialization vector (IV) handling and is vulnerable to padding oracle attacks if not implemented carefully. Modern applications should consider using authenticated encryption modes like AES-GCM (Galois/Counter Mode) that provide both confidentiality and integrity protection, detecting tampering attempts that CBC mode alone cannot prevent.

---

### Task 2 — Asymmetric Encryption & Digital Signatures

Asymmetric cryptography (also called public-key cryptography) uses a mathematically related pair of keys rather than a single shared key. The public key can be freely distributed and shared with anyone, while the private key must be kept secret and never shared. Data encrypted with the public key can only be decrypted with the corresponding private key, solving the key-distribution problem of symmetric encryption. Conversely, data signed with the private key can be verified with the public key, providing authentication and non-repudiation—proof that the message came from the private key holder. The RSA (Rivest-Shamir-Adleman) algorithm is one of the most widely used asymmetric encryption algorithms, with 2048-bit keys providing adequate security for most current applications, though 4096-bit keys are recommended for long-term protection of highly sensitive data. In this task, an RSA key pair was generated, public-key encryption and decryption were demonstrated to show how confidentiality can be achieved without pre-shared secrets, and digital signatures were created and verified to demonstrate authentication and integrity verification. Understanding asymmetric cryptography is essential because it forms the foundation of Public Key Infrastructure (PKI), TLS/SSL certificates, code signing, blockchain technology, and many other security systems.

#### 2.1 Generate RSA Key Pair

The first step in using asymmetric cryptography is generating a matched pair of keys consisting of a private key (which must be protected) and a public key (which can be freely distributed).

##### Purpose

- Generate a 2048-bit RSA private key containing both the private and public components.
- Extract the corresponding public key from the private key for distribution and verification.
- Establish the key pair that will be used for encryption/decryption and signing/verification demonstrations.
- Understand the relationship between private and public keys in asymmetric cryptography.

##### Terminal Commands

```bash
# Generate a 2048-bit RSA private key
openssl genrsa -out private.pem 2048

# Extract the public key from the private key
openssl rsa -in private.pem -pubout -out public.pem
```

##### Explanation of the Commands

- `openssl genrsa -out private.pem 2048` generates a new RSA private key with a modulus size of 2048 bits using OpenSSL's RSA key generation command (genrsa), which creates both the private exponent and public exponent components but stores them together in the private key file.
- The `-out private.pem` parameter specifies that the generated private key should be written to a file named private.pem in PEM (Privacy-Enhanced Mail) format, which is a Base64-encoded text format that can be safely stored in text files and transmitted over text-based protocols.
- The `2048` parameter specifies the key size in bits—2048-bit RSA keys are currently considered secure for most applications and provide a good balance between security and performance, though 4096-bit keys may be used for higher security requirements at the cost of slower operations.
- **Security Note:** The private key file (private.pem) must be kept absolutely confidential because anyone who obtains this file can decrypt messages encrypted with the corresponding public key and can create digital signatures that appear to come from the key owner—in production systems, private keys should have restrictive file permissions (chmod 600) and should ideally be stored in hardware security modules (HSMs) or key management services rather than as files on disk.
- `openssl rsa -in private.pem -pubout -out public.pem` reads the private key file (private.pem) and extracts only the public key components (modulus and public exponent), writing them to a separate file (public.pem) that can be safely distributed to anyone who needs to encrypt messages for the private key holder or verify signatures created by the private key.
- The `-pubout` flag instructs OpenSSL to output only the public key in PEM format rather than outputting another copy of the private key, ensuring that the public.pem file contains no sensitive information and can be freely shared.

##### Evidence

![RSA Key Pair Generation](evidence/Task%202%20generate%202048-bit%20key%20pair.png)

The screenshot shows the successful execution of both OpenSSL commands, with genrsa generating the 2048-bit private key and displaying the random number generation process, followed by the rsa command successfully extracting and writing the public key, confirming that both private.pem and public.pem files were created.

#### 2.2 Encrypt with Public Key, Decrypt with Private Key

After generating the key pair, the next step is demonstrating the fundamental property of public-key encryption: data encrypted with the public key (which anyone can access) can only be decrypted by the holder of the private key (which must remain secret), providing confidentiality without requiring pre-shared secrets.

##### Purpose

- Demonstrate public-key encryption using the public key to encrypt the sample patient record.
- Show that the encrypted data can only be decrypted by the holder of the corresponding private key.
- Prove that the decrypted data matches the original plaintext, confirming reversibility.
- Understand that public-key encryption solves the key-distribution problem of symmetric encryption.

##### Terminal Commands

```bash
# Encrypt with the PUBLIC key
openssl pkeyutl -encrypt -pubin -inkey public.pem -in record.txt -out record.rsa

# Decrypt with the PRIVATE key
openssl pkeyutl -decrypt -inkey private.pem -in record.rsa -out record.rsa.txt

# Verify successful decryption
diff record.txt record.rsa.txt && echo 'MATCH: RSA decryption successful'
```

##### Explanation of the Commands

- `openssl pkeyutl -encrypt` invokes OpenSSL's public key utility (pkeyutl) in encryption mode to perform asymmetric encryption operations, which differs from the symmetric enc command used in Task 1 because it uses public/private key pairs rather than shared secrets.
- `-pubin -inkey public.pem` specifies that the input key file (public.pem) is a public key (not a private key), which is used to encrypt the data—this demonstrates the asymmetric property where encryption uses one key and decryption uses a different related key.
- `-in record.txt -out record.rsa` specifies the input plaintext file (the same patient record from Task 1) to be encrypted and the output ciphertext file (record.rsa) where the encrypted result will be written.
- **Important Limitation:** RSA encryption can only encrypt small amounts of data (up to the key size minus padding overhead, approximately 256 bytes for 2048-bit keys), which is why RSA is typically used to encrypt symmetric keys or small secrets rather than large files—for large data, the hybrid approach (envelope encryption) demonstrated in Session B is required.
- `openssl pkeyutl -decrypt` performs asymmetric decryption to recover the plaintext from the RSA ciphertext.
- `-inkey private.pem` specifies the private key file to use for decryption—note that no `-pubin` flag is used because private keys are the default input type, and only the holder of the private key can perform this decryption operation, providing confidentiality.
- `-in record.rsa -out record.rsa.txt` specifies the input ciphertext file and the output plaintext file where the decrypted result will be written.
- `diff record.txt record.rsa.txt && echo 'MATCH: RSA decryption successful'` compares the original plaintext with the RSA-decrypted plaintext, confirming that asymmetric encryption and decryption successfully preserved the data and that the public-key/private-key operations are reversible.

##### Evidence

![RSA Encryption and Decryption](evidence/Task%202%20encrypt%20with%20public%20key,%20decrypt%20with%20private%20key.png)

The screenshot shows the successful execution of the RSA encryption command using the public key, followed by the decryption command using the private key, and the diff comparison confirming with "MATCH: RSA decryption successful" that the decrypted file is identical to the original plaintext, proving that public-key encryption works correctly.

#### 2.3 Sign with Private Key, Verify with Public Key

Digital signatures use asymmetric cryptography in the opposite direction from encryption: the private key is used to create a signature (proving "only I could have created this"), and the public key is used to verify the signature (allowing anyone to confirm "this really came from the private key holder"). This provides authentication (proving who sent the message), integrity (detecting any modifications), and non-repudiation (the signer cannot later deny creating the signature).

##### Purpose

- Create a digital signature on the patient record using the private key.
- Verify the signature using the public key to confirm authenticity and integrity.
- Demonstrate that signatures detect tampering—any modification to the signed data will cause verification to fail.
- Understand the complementary roles of encryption (confidentiality) and signing (authentication and integrity).

##### Terminal Commands

```bash
# Sign with the PRIVATE key
openssl dgst -sha256 -sign private.pem -out record.sig record.txt

# Verify with the PUBLIC key
openssl dgst -sha256 -verify public.pem -signature record.sig record.txt
```

##### Explanation of the Commands

- `openssl dgst -sha256` invokes OpenSSL's message digest command (dgst) using the SHA-256 cryptographic hash function to compute a fixed-size digest (fingerprint) of the input data, which is then signed rather than signing the entire file—this is necessary because RSA can only operate on small amounts of data directly.
- `-sign private.pem -out record.sig record.txt` instructs OpenSSL to create a digital signature by first computing the SHA-256 hash of record.txt, then encrypting that hash with the private key using RSA, producing a signature file (record.sig) that can later be used to verify that the file was signed by the private key holder and has not been modified.
- The signing operation can only be performed by someone who possesses the private key, which is why digital signatures provide authentication—only the legitimate key holder can create valid signatures, and attempting to forge a signature without the private key is computationally infeasible with properly sized keys.
- `openssl dgst -sha256 -verify public.pem -signature record.sig record.txt` performs signature verification by computing the SHA-256 hash of record.txt, then using the public key to decrypt the signature file (record.sig) and compare the decrypted hash with the freshly computed hash.
- If the hashes match, OpenSSL outputs "Verified OK" confirming that: (1) the signature was created by the holder of the private key corresponding to the public key used for verification, and (2) the file content has not been modified since the signature was created because any change would produce a different hash that wouldn't match the signed hash.
- If the hashes do not match (due to file modification or signature tampering), verification fails with an error message, alerting users that the data cannot be trusted.

##### Evidence

![Digital Signature Creation and Verification](evidence/Task%202%20Sign%20in%20with%20the%20private%20key,%20verify%20with%20the%20public%20key.png)

The screenshot demonstrates the successful execution of the signing command that creates record.sig, followed by the verification command that outputs "Verified OK", confirming that the signature is valid, the file is authentic (created by the private key holder), and the content has not been modified since signing.

#### Notes

The asymmetric cryptography tasks successfully demonstrated the complementary roles of public-key operations in providing both confidentiality and authenticity:

**Encryption/Decryption (Confidentiality):**
- Encrypt with PUBLIC key → Anyone can encrypt messages for the private key holder
- Decrypt with PRIVATE key → Only the key holder can read the messages
- Use case: Secure communication, key exchange, protecting secrets

**Signing/Verification (Authentication & Integrity):**
- Sign with PRIVATE key → Only the key holder can create valid signatures
- Verify with PUBLIC key → Anyone can confirm the signature is authentic
- Use case: Code signing, document authentication, non-repudiation, integrity checking

The key insight is that asymmetric cryptography reverses the roles depending on the security goal: for confidentiality, the public key encrypts and the private key decrypts; for authenticity, the private key signs and the public key verifies. This dual capability makes asymmetric cryptography the foundation of many security systems including TLS/SSL (demonstrated in Task 3), PKI certificates, blockchain transactions, secure email (S/MIME, PGP), and software distribution.

However, asymmetric operations are significantly slower than symmetric encryption (typically 100-1000x slower for RSA compared to AES), and RSA has message size limitations (can only encrypt data smaller than the key size minus padding overhead). For these reasons, production systems typically use hybrid cryptography: asymmetric encryption to exchange a symmetric session key, then symmetric encryption (AES) for bulk data protection—this pattern is exactly what envelope encryption (Session B) implements for cloud-scale key management.

In production environments, private keys must be rigorously protected: stored with restrictive permissions (chmod 600), encrypted when at rest, ideally stored in hardware security modules (HSMs) or key management services, never transmitted over networks, and rotated periodically according to security policies. Public keys can be freely distributed but should be authenticated using certificates from trusted Certificate Authorities to prevent man-in-the-middle attacks where an attacker substitutes their own public key.

---

### Task 3 — Encryption in Transit (TLS)

Transport Layer Security (TLS), formerly known as SSL (Secure Sockets Layer), protects data while it travels over networks by encrypting the communication channel between clients and servers. Without TLS, network traffic travels in plaintext where it can be intercepted and read by anyone with access to the network path—this includes malicious actors performing man-in-the-middle attacks, compromised network equipment, or even legitimate network administrators with packet capture tools. TLS solves multiple security problems simultaneously: it encrypts data to provide confidentiality, it authenticates the server (and optionally the client) using certificates to prevent impersonation, and it ensures integrity by detecting any modifications to data in transit. HTTPS is simply HTTP over TLS, protecting web traffic from eavesdropping and tampering. In this task, a self-signed X.509 certificate was created to establish a TLS identity for a local web server, Nginx was deployed in a Docker container configured to serve HTTPS traffic using that certificate, and the patient record was retrieved over an encrypted TLS connection to demonstrate protection of data in transit. Self-signed certificates are suitable for development and testing but would not be trusted by browsers or clients in production environments where certificates from trusted Certificate Authorities (Let's Encrypt, DigiCert, GlobalSign) are required.

#### Purpose

- Generate a self-signed X.509 certificate and corresponding private key for TLS server authentication.
- Configure an Nginx web server to serve HTTPS traffic using the generated certificate.
- Deploy the Nginx server in a Docker container with the patient record accessible over HTTPS.
- Access the patient record over an encrypted TLS connection to demonstrate protection in transit.
- Understand the role of certificates in establishing server identity and enabling trust.
- Recognize the difference between self-signed certificates (testing) and CA-signed certificates (production).

#### Terminal Commands

```bash
# Generate a self-signed certificate
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem \
  -days 7 -nodes -subj '/CN=localhost'

# Serve HTTPS on port 8443 using a small container
docker run --rm -d --name tls -p 8443:443 \
  -v $(pwd)/cert.pem:/etc/nginx/cert.pem \
  -v $(pwd)/key.pem:/etc/nginx/key.pem \
  -v $(pwd)/record.txt:/usr/share/nginx/html/record.txt nginx

# Connect over TLS (-k accepts the self-signed cert)
curl -k https://localhost:8443/record.txt

# Stop the container when finished
docker stop tls
```

#### Explanation of the Commands

- `openssl req -x509` invokes OpenSSL's certificate request command (req) in self-signed certificate mode (-x509), which creates a certificate directly rather than creating a Certificate Signing Request (CSR) that would be sent to a Certificate Authority for signing—this is appropriate for development but not for production systems.
- `-newkey rsa:2048` generates a new RSA private key with 2048-bit strength as part of the certificate creation process, combining key generation and certificate creation into a single command for convenience.
- `-keyout key.pem -out cert.pem` specifies that the private key should be written to key.pem and the self-signed certificate should be written to cert.pem, creating the two files required for TLS server configuration.
- `-days 7` sets the certificate validity period to 7 days from the creation date, after which the certificate expires and would be rejected by strict TLS clients—this short validity is appropriate for lab exercises but production certificates typically have 90-day (Let's Encrypt) or 1-year (commercial CAs) validity periods.
- `-nodes` (no DES) specifies that the private key should not be encrypted with a password, allowing Nginx to start automatically without manual password entry—in production systems, private keys should be encrypted when stored but may be decrypted when loaded into memory by the web server.
- `-subj '/CN=localhost'` sets the certificate's Subject Distinguished Name with Common Name (CN) set to "localhost", which is the hostname that browsers will validate against when connecting—for production systems, the CN should match the actual domain name (e.g., www.example.com), and modern certificates should include Subject Alternative Names (SAN) for additional domain names.
- `docker run --rm -d --name tls -p 8443:443` starts a new Docker container running Nginx web server with several flags: --rm automatically removes the container when it stops, -d runs it in detached mode (background), --name tls assigns the name "tls" for easy reference, and -p 8443:443 maps the container's HTTPS port 443 to the host's port 8443 to avoid conflicting with other services.
- `-v $(pwd)/cert.pem:/etc/nginx/cert.pem -v $(pwd)/key.pem:/etc/nginx/key.pem` uses volume mounts to make the certificate and private key files available inside the container at the paths where Nginx expects to find them for TLS configuration, with $(pwd) expanding to the current working directory.
- `-v $(pwd)/record.txt:/usr/share/nginx/html/record.txt` mounts the patient record file into the container's web root directory so it can be served over HTTPS when requested.
- `nginx` specifies the Docker image to use—the official Nginx image from Docker Hub which includes a default configuration that can serve HTTPS when certificate files are provided (though in production a custom nginx.conf with proper TLS settings would be recommended).
- `curl -k https://localhost:8443/record.txt` makes an HTTPS request to the Nginx server running in Docker to retrieve the patient record over an encrypted TLS connection.
- The `-k` (or `--insecure`) flag tells curl to accept the self-signed certificate without validating it against trusted Certificate Authorities—this is necessary for testing with self-signed certificates but must never be used in production systems because it disables the authentication security provided by TLS, allowing man-in-the-middle attacks.
- **Security Warning:** Using `-k` in production code would completely defeat the authentication purpose of TLS certificates, allowing attackers to intercept connections by presenting their own certificates—production clients must always validate certificates against trusted CA chains.
- `docker stop tls` stops the Nginx container and removes it (due to the --rm flag used when starting), cleaning up the test environment after demonstration is complete.

#### Evidence

![Self-Signed Certificate Generation](evidence/Task%203%20generate%20a%20self-signed%20certificate.png)

The screenshot shows the successful execution of the OpenSSL certificate generation command, creating both the private key (key.pem) and the self-signed certificate (cert.pem) with CN=localhost, valid for 7 days.

![HTTPS Service on Port 8443](evidence/Task%203%20serve%20https%20on%20port%208443.png)

The screenshot demonstrates the Docker run command starting the Nginx container with volume mounts for the TLS certificate, private key, and patient record file, with port mapping from host port 8443 to container port 443, followed by the container ID confirming successful startup.

![TLS Connection Test](evidence/Task%203%20connect%20over%20TLS.png)

The screenshot shows the curl command successfully retrieving the patient record content ("Patient: surya, Diagnosis: confidential") over HTTPS from the Nginx server, proving that TLS is functioning and that the data was transmitted over an encrypted connection rather than plaintext HTTP.

#### Notes

The TLS demonstration successfully showed that HTTPS protects data while it travels over networks, addressing the third dimension of data protection alongside encryption at rest (Task 1) and authentication/integrity (Task 2). When the curl command connected to https://localhost:8443, several security operations occurred behind the scenes:

**TLS Handshake Process:**
1. **Client Hello:** curl initiated a TLS connection and advertised supported cipher suites
2. **Server Hello:** Nginx selected a cipher suite and sent its certificate (cert.pem)
3. **Certificate Validation:** curl verified the certificate (skipped due to -k flag in this test)
4. **Key Exchange:** Both parties established a shared symmetric session key using asymmetric cryptography
5. **Encrypted Communication:** All subsequent HTTP traffic was encrypted with the session key using symmetric encryption (typically AES)

The combination of asymmetric cryptography (for key exchange and authentication) and symmetric encryption (for bulk data protection) in TLS demonstrates the hybrid encryption pattern that maximizes both security and performance—the same pattern used in envelope encryption (Session B).

**Comparing HTTP vs HTTPS:**
- **Without TLS (HTTP):** The patient record would travel in plaintext, visible to anyone capturing network traffic with tools like Wireshark or tcpdump—this includes network administrators, ISP employees, compromised routers, or malicious actors performing man-in-the-middle attacks
- **With TLS (HTTPS):** The patient record travels encrypted, appearing as random data to eavesdroppers—even if packets are captured, the sensitive medical information remains confidential and tampering is detected

**Production Considerations:**
- **Self-Signed Certificates:** The certificate used in this lab would trigger browser warnings ("Your connection is not private") in production because it's not signed by a trusted Certificate Authority—browsers maintain a list of trusted CAs and reject certificates that aren't in that trust chain
- **Certificate Authorities:** Production systems should use certificates from trusted CAs like Let's Encrypt (free, automated), DigiCert, GlobalSign, or organizational internal CAs for enterprise intranets
- **Certificate Validation:** Production clients must always validate certificates (never use curl -k or equivalent) to prevent man-in-the-middle attacks where attackers present fraudulent certificates
- **TLS Configuration:** Production servers should use strong cipher suites (TLS 1.2 or 1.3), disable obsolete protocols (SSL 3.0, TLS 1.0, TLS 1.1), implement HTTP Strict Transport Security (HSTS) headers, and use Perfect Forward Secrecy (PFS) cipher suites
- **Certificate Management:** Certificates must be renewed before expiration, private keys must be protected with restrictive permissions, and certificate transparency logs should be monitored for unauthorized certificate issuance

This completes Session A, where the fundamental building blocks of data protection were established: AES for protecting data at rest, RSA for public-key operations and signatures, and TLS for protecting data in transit. Session B will extend these concepts into cloud-scale key management using KMS and envelope encryption.

---

## Session B (Week 6) — Key Management, Envelope Encryption & Erasure

Session B transitions from understanding individual cryptographic operations to implementing cloud-scale key management patterns that are practical for real-world systems. While Session A demonstrated that encryption can protect data, Session B addresses the critical question: how do we protect and manage the keys themselves? The fundamental security principle is that encryption is only as strong as key management—even the strongest encryption algorithm (AES-256, RSA-4096) provides no security if keys are stored in plaintext on disk, transmitted insecurely over networks, or accessible to unauthorized users. Cloud Key Management Services (KMS) solve this problem by centralizing key storage in hardware security modules (HSMs), enforcing access control through IAM policies, providing comprehensive audit logging of all key operations, enabling key rotation and lifecycle management, and supporting cryptographic erasure for secure deletion. In Session B, LocalStack provides a KMS-compatible endpoint for hands-on practice without requiring actual AWS credentials or cloud costs, while demonstrating the same API operations used in production cloud environments.

### Task 4 — Create and Use a KMS Master Key

A Key Management Service (KMS) centralizes the creation, storage, protection, and management of encryption keys, removing key material from application code and providing hardware-level security for master keys. Unlike the password-based encryption in Task 1 or file-based RSA keys in Task 2, KMS keys never expose their underlying key material to applications—all cryptographic operations happen within the KMS service itself, with applications receiving only ciphertext results. Customer Master Keys (CMKs), now called customer-managed keys, are symmetric or asymmetric keys that customers create and control through IAM policies, with the KMS service protecting the actual key bytes in Hardware Security Modules (HSMs) certified to FIPS 140-2 Level 2 or Level 3 standards. Direct KMS encryption is suitable for small secrets like passwords, API keys, or database connection strings (up to 4 KB), but larger data requires envelope encryption (demonstrated in Task 5) to avoid data size limitations and minimize KMS API calls. In this task, a customer-managed KMS key was created for tenant-a, demonstrating multi-tenant key isolation where each customer has their own encryption key with separate access control policies, and a small secret was encrypted directly using the KMS Encrypt API to understand how KMS operates.

#### Purpose

- Start LocalStack to provide a local KMS-compatible endpoint for testing without AWS cloud access.
- Create a customer-managed KMS key with a description identifying it as tenant-a's master key.
- Understand KMS key metadata including KeyId, key state, and key usage (ENCRYPT_DECRYPT).
- Encrypt a small secret directly with KMS to demonstrate the Encrypt API operation.
- Observe that KMS returns only ciphertext, never exposing the actual master key material.
- Set up the environment variable for the LocalStack endpoint used throughout Session B.

#### Terminal Commands

```bash
# Set the endpoint variable for LocalStack KMS
EP='--endpoint-url=http://localhost:4566'

# Create a customer master key (CMK) and capture its KeyId
aws $EP kms create-key --description 'CCSE tenant-A master key'

# Copy the KeyId from the output into KEY_A below
KEY_A=<PASTE_KEYID>

# Encrypt a small secret directly with KMS
aws $EP kms encrypt --key-id $KEY_A --plaintext "$(echo -n 'hello' | base64)" \
  --query CiphertextBlob --output text
```

#### Explanation of the Commands

- `EP='--endpoint-url=http://localhost:4566'` creates a shell environment variable named EP containing the LocalStack endpoint URL parameter that will be passed to all AWS CLI commands throughout Session B, directing API calls to the local LocalStack service running on port 4566 instead of actual AWS cloud services.
- LocalStack is a local cloud stack emulator that provides AWS-compatible APIs for development and testing, including KMS, S3, DynamoDB, and many other services, allowing students to practice cloud operations without requiring AWS accounts, credentials, or incurring cloud costs.
- `aws $EP kms create-key` invokes the AWS CLI with the LocalStack endpoint to call the KMS CreateKey API operation, which generates a new symmetric customer-managed key protected within the KMS service's storage.
- `--description 'CCSE tenant-A master key'` provides a human-readable description that identifies this key as belonging to tenant-a, which is essential in multi-tenant environments for distinguishing between different customers' encryption keys—in production, these descriptions would follow organizational naming conventions and include additional metadata like environment (prod/dev), application name, and data classification.
- The CreateKey API returns JSON output including the KeyId (a unique identifier like a UUID), KeyArn (Amazon Resource Name for IAM policy referencing), KeyState (should be "Enabled" for newly created keys), KeyUsage (ENCRYPT_DECRYPT for symmetric keys), and other metadata, but notably does not include the actual key material because KMS never exposes that outside the HSM.
- `KEY_A=<PASTE_KEYID>` stores the returned KeyId in a shell variable for use in subsequent commands—the KeyId is a unique identifier that references this specific encryption key in all future KMS API calls, and in the actual lab execution, the placeholder would be replaced with the real KeyId (e.g., `KEY_A="abc12345-6789-0def-ghij-klmnopqrstuv"`).
- `aws $EP kms encrypt --key-id $KEY_A` calls the KMS Encrypt API to encrypt data using the specified customer-managed key, with the actual encryption operation happening within KMS rather than in the client application.
- `--plaintext "$(echo -n 'hello' | base64)"` provides the plaintext data to be encrypted, which must be base64-encoded for the AWS CLI (the command substitution `$(...)` runs the inner echo/base64 commands and passes their output as the plaintext parameter).
- The `-n` flag to echo prevents adding a newline character, ensuring that only the literal text "hello" is encoded, and the base64 encoding converts the binary-safe plaintext into a text format suitable for command-line parameter passing.
- `--query CiphertextBlob --output text` uses AWS CLI's JMESPath query syntax to extract only the CiphertextBlob field from the JSON response and output it as plain text rather than JSON, making it easier to store or use in subsequent commands—the CiphertextBlob contains the encrypted version of "hello" that can only be decrypted by calling KMS Decrypt with the same key.

#### Evidence

![Tenant-A KMS Key Creation](evidence/Task%204%20Create%20and%20use%20a%20KMS%20master%20key%201.png)

The screenshot shows the successful creation of the customer-managed KMS key with AWS CLI displaying the JSON response containing the KeyMetadata including KeyId, KeyArn, KeyState as "Enabled", KeyUsage as "ENCRYPT_DECRYPT", and the description "CCSE tenant-A master key", confirming that the master key was created in LocalStack KMS.

![Direct KMS Encryption](evidence/Task%204%20encrypt%20a%20small%20secret%20directly%20with%20KMS.png)

The screenshot demonstrates the execution of the KMS encrypt command showing the base64-encoded CiphertextBlob output, which is the encrypted form of the plaintext "hello"—this ciphertext can only be decrypted by calling the KMS Decrypt API with the same KeyId, proving that the master key successfully encrypted the small secret.

#### Notes

The KMS master key creation task successfully demonstrated several critical concepts in cloud key management. Unlike the local file-based keys from Session A (private.pem, key.pem), this KMS key exists only within the LocalStack KMS service, and the actual key material is never exposed to the client application. The KeyId serves as a reference handle that allows applications to request cryptographic operations (encrypt, decrypt, generate data keys) without ever possessing the master key itself—this separation of key use from key possession is fundamental to secure key management.

The direct encryption of "hello" demonstrates that KMS can encrypt small secrets directly, but this approach has important limitations: the KMS Encrypt API has a maximum plaintext size of 4 KB, making it unsuitable for encrypting files, database records, or other large data objects. Additionally, every encryption and decryption operation requires an API call to KMS, which introduces latency and generates audit log entries—for high-volume applications or large datasets, this would create performance bottlenecks and excessive audit log volume.

For these reasons, production systems rarely encrypt large amounts of data directly with KMS. Instead, they use the envelope encryption pattern (demonstrated in Task 5) where KMS encrypts only the data encryption keys (DEKs), and the DEKs are used locally to encrypt actual data with fast symmetric encryption (AES). This hybrid approach provides the security benefits of centralized key management while maintaining the performance characteristics necessary for real-world applications handling gigabytes or terabytes of data.

---

### Task 5 — Envelope Encryption

Envelope encryption is a cryptographic pattern that solves the performance and scalability limitations of direct KMS encryption by using two layers of keys: a master key stored securely in KMS and data encryption keys (DEKs) generated on-demand for encrypting actual data. The term "envelope" refers to wrapping one key inside another—the data is encrypted with a DEK (fast local symmetric encryption), and the DEK itself is encrypted with the KMS master key (protecting the key that protects the data). This pattern is ubiquitous in cloud storage systems including AWS S3 server-side encryption, Azure Storage Service Encryption, Google Cloud Storage encryption, database encryption (RDS, DynamoDB), and backup solutions, because it provides the right balance of security, performance, and scalability. The key insight is that only the small DEK needs to pass through KMS (encrypt on generation, decrypt on data access), while the potentially gigabytes of actual data are encrypted locally with high-performance AES operations that never touch the KMS API. This task implements the complete envelope encryption workflow: requesting a DEK from KMS (which returns both plaintext and KMS-wrapped versions), using the plaintext DEK to encrypt the patient record locally with OpenSSL, storing the KMS-wrapped DEK alongside the encrypted data, and then destroying the plaintext DEK to ensure it exists only in memory during active use.

#### 5.1 Ask KMS for a Data Key

The first step in envelope encryption is requesting a data encryption key from KMS using the GenerateDataKey API, which generates a random symmetric key, returns it in plaintext for immediate use, and also returns a KMS-wrapped (encrypted) version for persistent storage.

##### Purpose

- Generate a fresh AES-256 data encryption key using the KMS GenerateDataKey API.
- Receive both plaintext and encrypted (KMS-wrapped) versions of the same key.
- Understand that the plaintext version is used temporarily for encryption operations.
- Understand that the encrypted version is stored permanently for future decryption operations.
- Demonstrate that KMS generates cryptographically strong random keys rather than deriving them from passwords.

##### Terminal Commands

```bash
# 5.1 Ask KMS for a data key (returns plaintext + encrypted versions)
aws $EP kms generate-data-key --key-id $KEY_A --key-spec AES_256 \
  --query '[Plaintext,CiphertextBlob]' --output text
# Save column 1 as datakey.b64 (plaintext) and column 2 as datakey.enc (wrapped)
```

##### Explanation of the Commands

- `aws $EP kms generate-data-key` calls the KMS GenerateDataKey API which generates a new symmetric encryption key using a cryptographically secure random number generator within the KMS service, ensuring that data keys are truly random and not predictable.
- `--key-id $KEY_A` specifies which customer-managed KMS key should be used to encrypt (wrap) the generated data key—this establishes the relationship where the data key is protected by the master key, and the master key's access policies control who can unwrap the data key in the future.
- `--key-spec AES_256` requests a 256-bit AES symmetric key, which provides strong security appropriate for protecting sensitive data and is the most commonly used data key size in production envelope encryption systems.
- `--query '[Plaintext,CiphertextBlob]' --output text` extracts both the plaintext and encrypted versions of the data key from the JSON response, outputting them as tab-separated text on a single line for easy parsing and storage in separate files.
- The **Plaintext** field contains the base64-encoded data encryption key in usable form, which will be decoded and used immediately for local encryption operations with OpenSSL—this plaintext key must be handled carefully and destroyed after use to prevent exposure.
- The **CiphertextBlob** field contains the KMS-encrypted (wrapped) version of the same data key, which is safe to store alongside encrypted data because it can only be decrypted by calling KMS Decrypt with appropriate IAM permissions—this wrapped key is what makes envelope encryption secure and practical.
- In the actual lab workflow, the output would be processed to save the first column (Plaintext) to datakey.b64 and the second column (CiphertextBlob) to datakey.enc using shell redirection or text processing commands, separating the temporary plaintext key from the permanent wrapped key.

##### Evidence

![KMS Data Key Generation](evidence/Task%205.1%20Ask%20the%20KMS%20for%20a%20data%20key.png)

The screenshot shows the successful execution of the GenerateDataKey API call displaying two base64-encoded strings separated by whitespace: the first is the plaintext data key that will be used for local encryption, and the second is the KMS-wrapped data key that will be stored for future decryption, demonstrating that KMS provides both forms of the key in a single API call.

#### 5.2 Encrypt the Big File with the Data Key

After obtaining the plaintext data encryption key from KMS, the next step is using that key to perform fast local symmetric encryption of the actual data file using OpenSSL, identical to the AES encryption from Task 1 but now using a KMS-generated key instead of a password.

##### Purpose

- Decode the base64-encoded plaintext data key into binary format suitable for OpenSSL.
- Encrypt the patient record file locally using AES-256-CBC with the KMS-generated data key.
- Demonstrate that large data encryption happens locally without sending data to KMS.
- Create the encrypted data file that will be stored alongside the KMS-wrapped data key.
- Understand that this local encryption is fast and scalable for any data size.

##### Terminal Commands

```bash
# 5.2 Encrypt the big file locally with the PLAINTEXT data key
base64 -d datakey.b64 > datakey.bin
openssl enc -aes-256-cbc -pbkdf2 -in record.txt -out record.env.enc \
  -pass file:./datakey.bin
```

##### Explanation of the Commands

- `base64 -d datakey.b64 > datakey.bin` decodes the base64-encoded plaintext data key from KMS back into raw binary bytes and writes them to datakey.bin, converting from the text-safe format used for API transmission to the binary format required by OpenSSL for cryptographic operations.
- `openssl enc -aes-256-cbc -pbkdf2` uses the same AES-256-CBC encryption command from Task 1, but instead of deriving a key from a user password, this operation uses the KMS-generated random data key for stronger security and better key management properties.
- `-in record.txt -out record.env.enc` specifies the input plaintext file (the patient record) and the output ciphertext file (record.env.enc, where "env" stands for "envelope encrypted"), creating the encrypted data file that will be stored in the cloud, database, or backup system.
- `-pass file:./datakey.bin` instructs OpenSSL to read the encryption key from the datakey.bin file rather than prompting for a password interactively, using the KMS-generated data key as the symmetric encryption key.
- **Critical Security Point:** This encryption happens entirely on the local system without sending the patient record to KMS—the sensitive data never leaves the application's control, which is essential for compliance with data residency requirements and minimizing data exposure to cloud service providers.

##### Evidence

![File Encryption with Data Key](evidence/Task%205.2%20Encrypt%20the%20big%20file.png)

The screenshot demonstrates the execution of the base64 decoding command followed by the OpenSSL encryption command, showing that datakey.bin was created containing the binary data key and that record.env.enc was successfully created containing the encrypted patient record, confirming that local envelope encryption completed successfully.

#### 5.3 Destroy Plaintext Data Key Material

The final and most critical step in secure envelope encryption is destroying all plaintext copies of the data encryption key after completing the encryption operation, ensuring that the key exists only in memory during active use and never persists on disk where it could be stolen.

##### Purpose

- Remove the plaintext data key files (both base64 and binary formats) from disk storage.
- Demonstrate that only the KMS-wrapped data key should be retained for future decryption.
- Understand the principle of minimizing key exposure by keeping keys in memory only when needed.
- Prove that envelope encryption leaves no plaintext key material on disk after encryption.

##### Terminal Commands

```bash
# 5.3 Destroy the plaintext data key from disk — keep only the wrapped copy
rm datakey.bin datakey.b64
echo 'Only the KMS-wrapped data key (datakey.enc) remains.'
```

##### Explanation of the Commands

- `rm datakey.bin datakey.b64` permanently deletes both the binary data key file (datakey.bin) and the base64-encoded plaintext key file (datakey.b64) from the filesystem, ensuring that no plaintext key material remains on disk after the encryption operation completes.
- This step is **absolutely critical** for security—if plaintext data keys were left on disk, an attacker who compromised the storage system could decrypt all envelope-encrypted data without needing access to KMS, completely defeating the purpose of envelope encryption and centralized key management.
- Production applications should go further by using secure memory handling: storing plaintext keys only in memory (never writing to disk), using memory locking to prevent keys from being swapped to disk, and explicitly zeroing memory after use to prevent keys from lingering in RAM or core dumps.
- `echo 'Only the KMS-wrapped data key (datakey.enc) remains.'` outputs a confirmation message and serves as documentation in the lab workflow that the proper cleanup has been performed and that the system is now in the secure post-encryption state.
- After this cleanup, the system contains: (1) record.env.enc (encrypted data, safe to store anywhere), (2) datakey.enc (KMS-wrapped key, safe to store alongside encrypted data), and notably does NOT contain any plaintext key material—this is the ideal state for envelope encryption at rest.

##### Evidence

![Plaintext Key Destruction](evidence/Task%205.3%20Destroy%20the%20plaintext%20data%20key.png)

The screenshot shows the execution of the rm command removing both plaintext key files, followed by the confirmation message "Only the KMS-wrapped data key (datakey.enc) remains", proving that the secure cleanup step was performed and that no plaintext key material persists on disk.

#### Notes

The envelope encryption task successfully demonstrated the complete workflow that production cloud systems use to encrypt data at scale. This three-step pattern (generate data key, encrypt data locally, destroy plaintext key) provides the optimal balance of security, performance, and scalability:

**Security Benefits:**
- Master keys never leave the KMS HSM—only the KMS service possesses the actual key material
- Data encryption keys are cryptographically random rather than derived from potentially weak passwords
- Plaintext keys exist only transiently during encryption/decryption operations
- Access control is enforced through KMS IAM policies for every key unwrap operation
- Comprehensive audit logging captures who accessed keys and when

**Performance Benefits:**
- Large data is encrypted locally with fast AES operations (no API latency)
- No need to send entire datasets to KMS (saving network bandwidth)
- Can encrypt gigabytes or terabytes of data with a single KMS API call (GenerateDataKey)
- Decryption similarly requires only one API call (Decrypt) to unwrap the data key

**Decryption Workflow (Not Shown in Lab but Important to Understand):**
1. Retrieve the KMS-wrapped data key (datakey.enc) that was stored with the encrypted data
2. Call `aws kms decrypt --ciphertext-blob fileb://datakey.enc` to unwrap the key (requires IAM permissions)
3. KMS returns the plaintext data key (in memory only)
4. Use the plaintext data key to decrypt the encrypted data with OpenSSL
5. Destroy the plaintext data key immediately after decryption completes
6. The decrypted data is now available for application use

This pattern is implemented in all major cloud providers' encryption services (AWS S3 SSE-KMS, Azure Storage SSE, Google Cloud Storage CMEK), database encryption systems (AWS RDS, DynamoDB, Azure SQL TDE), and backup solutions, demonstrating that envelope encryption is the industry-standard approach to cloud-scale data protection. The key management separation—master keys in HSMs, data keys generated on-demand, plaintext keys only in memory—provides defense-in-depth where multiple security controls must fail before data is exposed.

---

### Task 6 — Per-Tenant Keys & Cryptographic Erasure

Multi-tenant cloud systems should never use a single encryption key for all customers' data because key compromise would expose all tenants, key access control cannot be granular per-tenant, and secure deletion (cryptographic erasure) would affect all tenants simultaneously. The principle of per-tenant keys means that each customer or data classification level has independent encryption keys with separate IAM policies, enabling security benefits including blast radius limitation (compromised tenant-A key doesn't affect tenant-B), selective cryptographic erasure (deleting tenant-A's data without affecting tenant-B), and compliance with data residency or sovereignty requirements (different tenants' keys in different regions). Key lifecycle management is critical for long-term system operation: keys must be created with appropriate metadata and permissions, rotated periodically to limit cryptanalysis exposure, disabled when security incidents require immediate access revocation, scheduled for deletion with a recovery window to prevent accidental data loss, and permanently deleted after the waiting period to achieve cryptographic erasure. In this task, a second KMS key was created for tenant-B demonstrating key isolation, tenant-A's key was scheduled for deletion simulating a customer leaving the service or a security incident requiring data destruction, and decrypt operations were attempted to prove that key lifecycle enforcement prevents access to encrypted data after keys are disabled or pending deletion.

#### 6.1 Create Separate Key for Tenant B

The first step in demonstrating per-tenant key isolation is creating a completely independent KMS key for the second tenant, with no shared key material or access policies with tenant-A's key.

##### Purpose

- Create a second customer-managed KMS key for tenant-B with its own KeyId and access controls.
- Demonstrate that multi-tenant systems should use separate keys per customer.
- Establish the foundation for showing that tenant-B's operations are unaffected by tenant-A's key deletion.
- Understand that key isolation enables independent security policies and lifecycle management.

##### Terminal Commands

```bash
# A separate key for tenant B
aws $EP kms create-key --description 'CCSE tenant-B master key'
KEY_B=<PASTE_KEYID>
```

##### Explanation of the Commands

- `aws $EP kms create-key --description 'CCSE tenant-B master key'` creates a second customer-managed KMS key completely independent from the tenant-A key created in Task 4, with its own unique KeyId, separate access control policies (in production), and independent lifecycle state.
- The key isolation means that IAM policies can grant developers access to tenant-B keys while denying access to tenant-A keys (or vice versa), supporting organizational structures where different teams manage different customers' infrastructure.
- `KEY_B=<PASTE_KEYID>` stores the returned KeyId in a shell variable for potential future operations, though in this lab the focus is on tenant-A key deletion rather than tenant-B operations.
- In production multi-tenant systems, key tagging and naming conventions would identify which customer owns each key, and automation would ensure that each new tenant receives their own encryption key during provisioning.

##### Evidence

![Tenant-B Key Creation](evidence/Task%206%20seperate%20key%20for%20tenant%20b.png)

The screenshot shows the successful creation of the tenant-B KMS key with a unique KeyId distinct from tenant-A's key, confirming that separate per-tenant encryption keys have been established and that multi-tenant key isolation is implemented.

#### 6.2 Schedule Deletion of Tenant A Key

KMS provides a safe key deletion mechanism with a mandatory waiting period (7 to 30 days) during which the key enters PendingDeletion state, preventing new encryption operations while allowing administrators to cancel accidental deletion requests before permanent data loss occurs.

##### Purpose

- Schedule tenant-A's KMS key for deletion with the minimum 7-day pending window.
- Understand that scheduled deletion provides a safety mechanism against accidental key destruction.
- Observe that keys in PendingDeletion state cannot be used for cryptographic operations.
- Simulate a scenario where a tenant leaves the service or data must be permanently destroyed.

##### Terminal Commands

```bash
# Schedule deletion of tenant A's key (min window)
aws $EP kms schedule-key-deletion --key-id $KEY_A --pending-window-in-days 7
```

##### Explanation of the Commands

- `aws $EP kms schedule-key-deletion --key-id $KEY_A` calls the KMS ScheduleKeyDeletion API which transitions the specified key into PendingDeletion state, starting the countdown timer after which the key material will be permanently destroyed and all data encrypted under that key becomes permanently unrecoverable.
- `--pending-window-in-days 7` specifies the minimum allowed waiting period of 7 days (AWS KMS allows 7-30 days), providing a safety window during which administrators can cancel the deletion if they realize it was accidental or premature—after this period expires, deletion is automatic and irreversible.
- During the pending deletion period, the key cannot be used for any cryptographic operations (Encrypt, Decrypt, GenerateDataKey), but the key still exists and can be retrieved from API calls to allow administrators to identify which key is being deleted and assess the impact.
- In production systems, scheduling key deletion would trigger alerts, require approval workflows, initiate impact analysis to identify all data encrypted under that key, and potentially require customer confirmation before proceeding with permanent deletion.

##### Evidence

![Scheduled Key Deletion](evidence/Task%206%20schedule%20deletion%20of%20tenant%20A%20key.png)

The screenshot shows the successful execution of the schedule-key-deletion command with the JSON response indicating DeletionDate (when permanent deletion will occur) and the transition to PendingDeletion state, confirming that the 7-day countdown has begun for tenant-A's key.

#### 6.3 Disable Key and Verify Decryption Failure

To simulate immediate key revocation (as would occur during a security incident), the key can be disabled, and attempting to decrypt the wrapped data key demonstrates that key lifecycle enforcement prevents unauthorized data access.

##### Purpose

- Attempt to disable the key (though already in PendingDeletion state, demonstrating state validation).
- Attempt to decrypt the KMS-wrapped data key using the disabled/pending-deletion master key.
- Prove that KMS enforces key lifecycle states and denies operations on unusable keys.
- Demonstrate cryptographic erasure in action—encrypted data becomes unrecoverable after key deletion.

##### Terminal Commands

```bash
# Disable it immediately to simulate erasure
aws $EP kms disable-key --key-id $KEY_A

# Attempt to unwrap tenant A's data key now — it should FAIL
aws $EP kms decrypt --ciphertext-blob fileb://datakey.enc 2>&1 | head -3
```

##### Explanation of the Commands

- `aws $EP kms disable-key --key-id $KEY_A` attempts to call the KMS DisableKey API to transition the key to Disabled state, which would prevent all cryptographic operations while keeping the key material intact (unlike deletion which destroys the key material).
- However, because the key is already in PendingDeletion state from the previous command, this operation fails with KMSInvalidStateException indicating that keys pending deletion cannot be transitioned to other states—this demonstrates that KMS enforces valid state transitions and prevents conflicting lifecycle operations.
- `aws $EP kms decrypt --ciphertext-blob fileb://datakey.enc` attempts to unwrap (decrypt) the KMS-wrapped data key that was stored in Task 5.3, which should fail because the master key (KEY_A) that wrapped it is now in PendingDeletion state.
- `fileb://datakey.enc` specifies that the ciphertext-blob parameter should be read from the file datakey.enc as binary data (the `fileb://` prefix indicates binary file input to AWS CLI).
- `2>&1 | head -3` redirects stderr to stdout (capturing error messages) and displays only the first 3 lines of output to show the error message clearly without excessive JSON details.
- The expected error is KMSInvalidStateException with a message indicating that the key is pending deletion and cannot be used for decrypt operations, proving that cryptographic erasure is in effect—even though the encrypted data (record.env.enc) and wrapped key (datakey.enc) still exist on disk, they are cryptographically useless without the now-inaccessible master key.

##### Evidence

![Disable Key Attempt](evidence/Task%206%20Disable%20immediately.png)

The screenshot shows the disable-key command failing with an error message indicating that the key is in an invalid state for the requested operation, demonstrating that KMS enforces key lifecycle state validation and prevents conflicting operations.

![Failed Decrypt Attempt](evidence/Task%206%20attempt%20to%20unwrap%20tenant%20A%20data%20key.png)

The screenshot demonstrates the decrypt command failing with KMSInvalidStateException when attempting to unwrap the data key, with an error message stating that the key is pending deletion and cannot be used, proving that envelope-encrypted data (record.env.enc) is now permanently unrecoverable because the master key that can unwrap its data key is scheduled for destruction.

#### Notes

The per-tenant key isolation and cryptographic erasure demonstration successfully showed why multi-tenant systems should use separate encryption keys per customer and how key lifecycle management enables secure data deletion in cloud environments. Several critical security and operational concepts were illustrated:

**Per-Tenant Key Benefits:**
- **Blast Radius Limitation:** If tenant-A's key is compromised, tenant-B's data remains protected by a separate independent key
- **Selective Access Control:** IAM policies can grant different users/roles access to different tenant keys based on organizational responsibilities
- **Independent Lifecycle Management:** Tenant-A's key can be rotated, disabled, or deleted without affecting tenant-B's operations
- **Compliance and Data Residency:** Different tenants can have keys in different regions or with different protection levels to meet varied regulatory requirements

**Cryptographic Erasure vs Traditional Deletion:**
Traditional file deletion (even with overwriting) faces challenges in cloud environments:
- **Distributed Storage:** Cloud data is replicated across multiple storage devices and potentially multiple data centers—you cannot track or overwrite all physical copies
- **Snapshots and Backups:** Point-in-time snapshots and backup systems may contain copies of deleted data that cannot be overwritten
- **SSD Wear Leveling:** Modern SSDs use internal wear-leveling algorithms that relocate data blocks, making it impossible to guarantee that specific data has been overwritten
- **No Physical Access:** Cloud tenants don't have physical access to storage hardware to perform DoD-standard multi-pass wipes or hardware destruction

Cryptographic erasure solves all these problems elegantly:
1. **All data is encrypted at rest** with per-tenant or per-object encryption keys
2. **When deletion is required**, simply delete the encryption key from KMS
3. **All encrypted data becomes cryptographically unrecoverable** even though the ciphertext (encrypted data) still exists on storage media
4. **Deletion is immediate and provable** through KMS audit logs showing key deletion
5. **No need to track or overwrite** physical storage blocks across distributed systems

This is why GDPR "right to be forgotten" compliance, HIPAA secure disposal requirements, and PCI-DSS data retention policies are typically implemented using cryptographic erasure in cloud environments rather than traditional secure wiping methods. After the 7-day pending period expires (or if an administrator called CancelKeyDeletion), the master key is permanently destroyed in the KMS HSM, and the encrypted patient record (record.env.enc) becomes permanently and provably unrecoverable noise—even the cloud provider with full physical access to storage hardware cannot decrypt it.

---

### Task 7 — Integrity & Tamper-Evidence

While encryption protects confidentiality (preventing unauthorized reading), it does not automatically protect integrity (detecting unauthorized modification). An attacker who modifies encrypted data creates corrupted ciphertext that will decrypt to garbage, but without additional integrity controls, the corruption might not be detected until after decryption when it may be too late. Cryptographic hash functions like SHA-256 provide integrity verification by computing a fixed-size digest (fingerprint) of data—any modification, even changing a single bit, produces a completely different hash value with overwhelming probability. Digital signatures (demonstrated in Task 2.3) combine hashing with asymmetric cryptography to provide both integrity and authentication. Hash chains extend this concept to audit logs and blockchains by including the previous entry's hash when computing each new entry's hash, creating a tamper-evident structure where any modification breaks all subsequent hashes. In this task, SHA-256 hashes were calculated to verify file integrity and detect tampering, a simple hash chain was built to demonstrate tamper-evident audit logging, and the RSA signature from Session A was re-verified to confirm that the original patient record has maintained integrity throughout all lab operations.

#### Purpose

- Calculate SHA-256 cryptographic hashes to create unique fingerprints of files.
- Demonstrate that any modification to a file produces a completely different hash.
- Build a simple hash chain to show tamper-evident audit log structures.
- Re-verify the RSA digital signature created in Task 2.3 to confirm data integrity.
- List KMS keys to verify the tenant keys still exist in the system.
- Understand that integrity controls complement confidentiality controls for complete data protection.

#### Terminal Commands

```bash
# Fingerprint the file
sha256sum record.txt

# Tamper with a copy and show the hash changes
cp record.txt tampered.txt
echo 'x' >> tampered.txt
sha256sum record.txt tampered.txt

# Hash chain: each entry includes the previous hash (tamper-evident log)
PREV=0
for line in 'login ok' 'file read' 'export data'; do \
  PREV=$(echo -n "$PREV$line" | sha256sum | cut -d' ' -f1); \
  echo "$line | $PREV"; done

# Verification commands from the lab manual
aws --endpoint-url=http://localhost:4566 kms list-keys
openssl dgst -sha256 -verify public.pem -signature record.sig record.txt
```

#### Explanation of the Commands

- `sha256sum record.txt` calculates the SHA-256 cryptographic hash of the patient record file, outputting a 64-character hexadecimal string that uniquely identifies this file's exact content—this hash serves as a fingerprint that can be stored separately and used later to verify the file hasn't been modified.
- SHA-256 (Secure Hash Algorithm 256-bit) is a one-way cryptographic hash function that is computationally infeasible to reverse (cannot derive the input from the hash) and collision-resistant (cannot find two different inputs that produce the same hash), making it suitable for integrity verification and digital signatures.
- `cp record.txt tampered.txt` creates a copy of the patient record for modification testing, preserving the original file for later verification.
- `echo 'x' >> tampered.txt` appends a single character ('x' plus newline) to the tampered file, simulating a minor modification such as an attacker changing a diagnosis or a data corruption event.
- `sha256sum record.txt tampered.txt` calculates hashes for both the original and modified files, displaying them side-by-side to demonstrate that even a single-character change produces a completely different hash value—this shows that SHA-256 reliably detects tampering regardless of how small the modification is.
- `PREV=0` initializes a shell variable with a starting value for the hash chain, representing the genesis hash or the hash of an empty log before any entries exist.
- `for line in 'login ok' 'file read' 'export data'; do ... done` loops through three simulated audit log entries (user login, file access, data export) to build a hash chain where each entry is linked to the previous entry.
- `PREV=$(echo -n "$PREV$line" | sha256sum | cut -d' ' -f1)` computes the hash of the concatenation of the previous hash plus the current log entry, creating a chain where each entry depends on all previous entries—the `-n` flag prevents adding newlines, `sha256sum` computes the hash, and `cut -d' ' -f1` extracts just the hash value without the filename.
- `echo "$line | $PREV"` outputs each log entry along with its computed hash, showing the audit trail where each entry includes the cryptographic proof linking it to previous entries—this creates a tamper-evident log because modifying any earlier entry would require recomputing all subsequent hashes, which would be detected by comparing against stored hash values.
- `aws --endpoint-url=http://localhost:4566 kms list-keys` calls the KMS ListKeys API to enumerate all customer-managed keys in the LocalStack KMS, providing verification that both tenant-A and tenant-B keys still exist in the system (though tenant-A is in PendingDeletion state).
- `openssl dgst -sha256 -verify public.pem -signature record.sig record.txt` re-verifies the digital signature created in Task 2.3, confirming that the patient record file has not been modified since signing and that the signature is still valid—this demonstrates that integrity has been maintained throughout all Session A and Session B operations.

#### Evidence

![Hash Comparison and Hash Chain](evidence/Task%207.png)

The screenshot shows multiple outputs: (1) the original record.txt hash, (2) the tampered.txt hash showing a completely different value, proving that modification is detectable, (3) the hash chain output showing three audit log entries each with their chained hash values demonstrating the tamper-evident structure, with each hash depending on all previous entries.

![Verification Commands](evidence/Verification%20Command.png)

The screenshot displays the output of the verification commands showing: (1) the KMS list-keys output displaying both tenant-A and tenant-B key IDs confirming they exist in the system, (2) the OpenSSL signature verification outputting "Verified OK" confirming that the patient record has maintained integrity and the signature is still valid after all lab operations.

#### Notes

The integrity verification and tamper-evidence demonstration successfully showed that data protection requires more than just encryption—integrity controls are equally important for detecting unauthorized modifications and providing audit trails. Several key concepts were illustrated:

**Hash Functions for Integrity:**
- SHA-256 reliably detects any modification to files, regardless of size
- Even a single-bit change produces a completely different hash (avalanche effect)
- Hashes can be stored separately from data to enable later verification
- Comparing stored hash with computed hash reveals tampering or corruption

**Hash Chains for Tamper-Evident Logs:**
- Each entry's hash includes the previous entry's hash, creating a chain
- Modifying any entry requires recomputing all subsequent hashes
- Comparing stored hashes with recomputed hashes detects tampering
- This is the fundamental principle behind blockchain technology and certificate transparency logs
- If an attacker changed "login ok" to "login failed", the hash for that entry would change, which would change the hash for "file read", which would change the hash for "export data", making tampering obvious

**Digital Signatures Combine Multiple Properties:**
- Authentication: Proves the signer's identity (only private key holder can sign)
- Integrity: Detects any modification (hash is part of the signature)
- Non-repudiation: Signer cannot later deny creating the signature
- The "Verified OK" message confirms all three properties are satisfied

**Production Applications:**
- **Audit Logs:** Cloud systems should use hash chains or cryptographic append-only logs (AWS CloudTrail Log Validation, Azure Monitor Logs) to ensure audit trails cannot be modified after creation
- **Software Distribution:** Package managers (apt, yum, npm) provide SHA-256 hashes for downloaded packages to verify integrity and detect man-in-the-middle attacks or compromised mirrors
- **Blockchain:** Cryptocurrencies and distributed ledgers use hash chains at their core to create tamper-evident transaction history
- **Certificate Transparency:** Browser vendors use hash chains to create public logs of TLS certificates, detecting misissued or fraudulent certificates
- **File Integrity Monitoring:** Security tools (Tripwire, AIDE, OSSEC) maintain hash databases of system files to detect unauthorized modifications by malware or attackers

**Authenticated Encryption:**
The lab demonstrated integrity and confidentiality as separate controls, but production systems should use authenticated encryption modes like AES-GCM (Galois/Counter Mode) that provide both properties simultaneously—these modes encrypt data and compute an authentication tag in a single operation, detecting tampering attempts during decryption and preventing padding oracle attacks that can exploit non-authenticated CBC mode. Modern TLS 1.3 exclusively uses authenticated encryption for this reason.

---

## Security Best-Practices Checklist

The following security best practices were implemented and verified throughout this lab:

- [✓] **Data encrypted at rest (AES) and decryption verified** — Task 1 used AES-256-CBC with PBKDF2 and salt to protect the patient record, with diff confirming successful recovery.
- [✓] **Asymmetric keys used correctly (encrypt with public, sign with private)** — Task 2.2 encrypted with public.pem and decrypted with private.pem; Task 2.3 signed with private.pem and verified with public.pem.
- [✓] **Data protected in transit with TLS** — Task 3 deployed Nginx with self-signed certificate and retrieved data over HTTPS, demonstrating encryption in transit.
- [✓] **Envelope encryption used; plaintext data key not left on disk** — Task 5 generated data keys from KMS, encrypted locally with the plaintext key, and destroyed plaintext key material after use.
- [✓] **Per-tenant keys used; cryptographic erasure demonstrated** — Task 6 created separate keys for tenant-A and tenant-B, scheduled tenant-A's key for deletion, and proved that encrypted data became unrecoverable.
- [✓] **Integrity verified with hashing / hash chain** — Task 7 calculated SHA-256 hashes showing tampering detection, built a hash chain demonstrating tamper-evident logs, and re-verified the RSA signature.
- [✓] **KMS centralizes key management** — Tasks 4-6 used LocalStack KMS for key creation, encryption operations, and lifecycle management without exposing key material to applications.
- [✓] **Before-and-after testing methodology** — Demonstrated successful encryption/decryption, compared original vs tampered file hashes, and showed key deletion preventing decryption.
- [✓] **Defense-in-depth implemented** — Multiple layers of security controls across data at rest (AES), in transit (TLS), integrity (hashing), and key management (KMS envelope encryption).

---

## Short-Answer Questions

### Q1. Compare symmetric and asymmetric encryption: speed, key distribution, and typical use.

**Answer:**

Symmetric and asymmetric encryption represent fundamentally different approaches to cryptographic protection, each with distinct characteristics and appropriate use cases:

**Symmetric Encryption (AES-256 from Task 1):**

**Speed:** Very fast, typically processing gigabytes per second on modern hardware—AES with hardware acceleration (AES-NI on Intel/AMD CPUs) can encrypt at memory bandwidth speeds. This makes symmetric encryption suitable for large volumes of data including files, databases, disk encryption, and network traffic.

**Key Distribution:** Difficult and risky—both the sender and receiver must possess the exact same secret key, creating the key distribution problem. Securely sharing this key over networks or with multiple parties is challenging: sending keys in cleartext risks interception, encrypting keys requires already having a shared key (circular problem), and sharing keys with many users increases the risk that someone will compromise or leak the key. In multi-user systems, N users require N(N-1)/2 unique keys for pairwise communication, which doesn't scale.

**Typical Use:** Protecting data at rest (file encryption, database encryption, disk/volume encryption), encrypting large data in transit after key exchange (TLS/SSL bulk encryption, VPN tunnels), and cloud storage encryption where envelope encryption provides the keys. In Task 1, we used AES-256-CBC to encrypt the patient record, demonstrating fast symmetric encryption suitable for file protection.

**Asymmetric Encryption (RSA-2048 from Task 2):**

**Speed:** Much slower, typically 100-1000x slower than symmetric encryption due to computationally expensive modular exponentiation operations. RSA-2048 encryption might process only a few kilobytes per second, making it impractical for large data volumes. This performance limitation is why RSA is rarely used to encrypt bulk data directly.

**Key Distribution:** Easy and scalable—the public key can be freely distributed to anyone without compromising security (posted on websites, sent via email, included in certificates), while only the private key must be kept secret by its owner. This solves the key distribution problem because no pre-shared secrets are required: anyone can encrypt messages for the public key holder, and N users require only N key pairs for universal communication capability.

**Typical Use:** Encrypting small secrets like passwords or session keys (hybrid encryption), digital signatures for authentication and integrity, TLS/SSL key exchange during handshake, SSH public key authentication, email encryption (S/MIME, PGP), code signing, and blockchain transactions. In Task 2.2, we used RSA to encrypt the patient record, but practical systems would use RSA only to encrypt an AES key, then use that AES key for bulk encryption—exactly the pattern envelope encryption implements in Task 5.

**Hybrid Approach (Best of Both Worlds):**

Modern systems combine both: use asymmetric encryption (RSA, ECDH) to securely exchange a symmetric session key, then use symmetric encryption (AES) for bulk data protection. This is exactly how TLS works (Task 3) and how envelope encryption works (Task 5)—asymmetric cryptography solves the key distribution problem, and symmetric cryptography provides the performance needed for real-world data volumes.

**Summary Table:**

| Property | Symmetric (AES) | Asymmetric (RSA) |
|----------|----------------|------------------|
| **Speed** | Very fast (GB/s) | Slow (KB/s) |
| **Keys** | One shared secret | Public/private pair |
| **Key Distribution** | Difficult, risky | Easy, scalable |
| **Data Size** | Unlimited | Limited (~256 bytes for RSA-2048) |
| **Use Case** | Bulk data encryption | Key exchange, signatures |
| **Example** | Task 1 (file encryption) | Task 2 (key encryption) |

---

### Q2. Why is key management described as the weakest link, not the algorithm?

**Answer:**

Modern cryptographic algorithms like AES-256 and RSA-2048 are mathematically strong and have been extensively analyzed by cryptographers worldwide—breaking these algorithms through brute force or cryptanalysis is considered computationally infeasible with current technology. However, most real-world encryption failures occur not because the algorithms are weak, but because the keys that unlock those algorithms are poorly managed. As the security maxim states: "Encryption is only as strong as its key management."

**Why Key Management Is the Weakest Link:**

**1. Keys Stored Insecurely:**
- Hardcoding encryption keys directly in source code (visible in version control, decompiled binaries, or code repositories)
- Storing keys in plaintext configuration files (config.ini, .env files) without proper file permissions
- Including keys in database records or log files where they can be exfiltrated
- In our lab, if we had left datakey.bin on disk after Task 5.3 instead of deleting it, all envelope encryption security would be defeated

**2. Keys Transmitted Insecurely:**
- Sending keys over unencrypted network connections where they can be intercepted
- Including keys in email, instant messages, or collaboration tools
- Storing keys in shared folders or cloud storage with inadequate access control
- The key distribution problem from Q1 illustrates why symmetric key sharing is risky

**3. Inadequate Access Control:**
- Allowing too many users or applications access to encryption keys (violating least privilege)
- Not revoking access when employees leave or change roles
- Shared keys across multiple applications or tenants (blast radius problem from Task 6)
- Without proper IAM policies on KMS keys (Task 4-6), unauthorized users could decrypt sensitive data

**4. No Key Rotation:**
- Using the same encryption key for years or decades increases cryptanalysis exposure
- If a key is compromised but never rotated, all historical and future data remains at risk
- Compliance standards (PCI-DSS, HIPAA) typically require periodic key rotation

**5. Improper Key Deletion:**
- Not securely destroying keys when they're no longer needed
- Leaving plaintext keys in memory, swap files, crash dumps, or backups
- In Task 5.3, we explicitly destroyed plaintext data keys to prevent this exposure

**6. Weak Key Generation:**
- Using predictable passwords or weak random number generators instead of cryptographically secure random sources
- Deriving encryption keys from guessable inputs without proper key derivation functions
- Task 1 used PBKDF2 specifically to strengthen password-based keys against brute force

**Real-World Examples of Key Management Failures:**

- **Code Signing Key Theft:** Attackers steal code-signing private keys and sign malware that appears legitimate
- **Cloud Configuration Errors:** S3 buckets with encryption keys in publicly accessible files
- **Insider Threats:** Employees with excessive key access exfiltrate customer data
- **Lost Backup Tapes:** Physical media with plaintext encryption keys lost in transit
- **Certificate Private Keys:** TLS private keys accidentally included in Docker images or git repositories

**How KMS Solves Key Management Problems (Session B):**

The envelope encryption pattern (Tasks 4-6) addresses all these weaknesses:
- **Centralized Storage:** Master keys stored in FIPS 140-2 Level 2/3 HSMs, never exposed to applications
- **Access Control:** IAM policies enforce who can use keys for which operations
- **Audit Logging:** Every key operation is logged for security monitoring and compliance
- **Key Lifecycle:** Automated rotation, graceful key disablement, safe deletion with waiting periods
- **Separation of Duties:** Key administrators can't access encrypted data, data administrators can't access keys

**The Critical Insight:**

Even perfect AES-256 encryption is worthless if the key is stored next to the encrypted data in a file named "encryption_key.txt". Attackers don't break encryption algorithms mathematically—they steal keys through poor management practices, social engineering, or exploiting misconfigurations. This is why modern cloud security focuses heavily on key management services, hardware security modules, automated key rotation, comprehensive audit logging, and least-privilege access control rather than debating whether to use AES-256 vs AES-128.

---

### Q3. Explain envelope encryption and why only the master key needs hardware-grade protection.

**Answer:**

Envelope encryption is a cryptographic architecture pattern that solves the scalability, performance, and key management challenges of encrypting large volumes of data in cloud environments. The term "envelope" refers to wrapping one key inside another, like placing a letter (data encryption key) inside an envelope (master key encryption). This two-tier key hierarchy is the industry-standard approach used by AWS S3, Azure Storage, Google Cloud Storage, database encryption systems, and most cloud-native applications.

**How Envelope Encryption Works (Demonstrated in Task 5):**

**Step 1: Generate Data Encryption Key (DEK)**
- Application calls KMS GenerateDataKey API specifying the master key ID
- KMS generates a random AES-256 data encryption key using a cryptographically secure random number generator
- KMS returns TWO versions of the same key:
  - **Plaintext DEK:** For immediate use in encryption operations
  - **Encrypted DEK:** Wrapped (encrypted) under the customer's master key
- In Task 5.1, we received both versions from LocalStack KMS

**Step 2: Encrypt Data Locally**
- Application uses the plaintext DEK to encrypt the actual data (patient record) locally with AES
- This encryption happens on the application server or user's device—the sensitive data never goes to KMS
- Fast symmetric encryption (GB/s) suitable for large files, database records, or disk volumes
- In Task 5.2, we used OpenSSL with the plaintext data key to encrypt record.txt

**Step 3: Store Encrypted Data + Wrapped Key**
- Application stores the encrypted data (record.env.enc) in cloud storage, database, or backup system
- Application stores the encrypted DEK (datakey.enc) alongside the encrypted data as metadata
- The encrypted DEK is safe to store anywhere—even publicly—because it can only be decrypted by KMS with proper authorization

**Step 4: Destroy Plaintext Key**
- Application securely destroys the plaintext DEK from memory and disk
- Only the encrypted DEK remains, which is useless without KMS access
- In Task 5.3, we explicitly deleted datakey.bin and datakey.b64 to prevent key exposure

**Decryption Process (Reverse Workflow):**

1. Retrieve encrypted data (record.env.enc) and encrypted DEK (datakey.enc) from storage
2. Call KMS Decrypt API to unwrap the encrypted DEK (requires IAM authorization)
3. KMS returns the plaintext DEK in memory
4. Use plaintext DEK to decrypt the data locally with AES
5. Destroy plaintext DEK immediately after decryption
6. Return decrypted data to application

**Why Only the Master Key Needs Hardware Protection:**

**Master Key (Protected in HSM):**
- Stored permanently in Hardware Security Module (HSM) certified to FIPS 140-2 Level 2 or Level 3
- Never exported or exposed outside the HSM—even KMS administrators cannot access key material
- Physical tamper-resistant security: hardware destroys keys if tampering is detected
- Small: typically just 256 bits (32 bytes) for AES-256 keys
- Long-lived: may exist for months or years across millions of encryption operations
- In Task 4, KEY_A was created in KMS HSM and never left the LocalStack service

**Data Encryption Keys (Temporary, Software-Protected):**
- Generated on-demand for each encryption operation or data object
- Exist in plaintext only temporarily during encryption/decryption (seconds to minutes)
- Stored long-term only in encrypted (wrapped) form protected by the master key
- If a plaintext DEK is compromised, only that one data object is affected, not all data
- Can be regenerated by re-encrypting data with a new DEK if compromise is suspected
- Scalable: can have millions of DEKs (one per file/object) without requiring millions of HSMs

**The Economic and Practical Justification:**

**Cost:** Hardware Security Modules are expensive ($10,000-40,000 per device) and require specialized maintenance, physical security, and redundancy. It's economically impractical to use HSM protection for every piece of encrypted data in a system that might handle billions of objects.

**Performance:** HSMs have limited throughput (hundreds to thousands of operations per second) compared to software encryption (gigabytes per second). Encrypting all data directly through HSMs would create bottlenecks that make cloud-scale applications impossible.

**Scalability:** Each organization might have only a few dozen master keys (per tenant, per data classification, per region), but millions or billions of data objects. Envelope encryption means only the small set of master keys needs expensive HSM protection, while the large set of DEKs can be protected through software encryption under those master keys.

**Security Model:** The master key is like a "key to the keys"—it doesn't protect data directly, it protects other keys. This separation means:
- Master key compromises are rare because HSMs are highly secure
- If a DEK is compromised, you can re-encrypt that specific data with a new DEK
- If a master key is compromised (extremely rare), you must re-wrap all DEKs with a new master key, but this is a key-only operation that doesn't require re-encrypting the actual data

**Analogy:**

Think of a bank: the master key is like the vault's master combination that only the bank president knows and which is protected by physical security, guards, and alarms. Data encryption keys are like individual safety deposit box keys given to customers. You don't need vault-level security for every safety deposit box key because even if someone steals one, they can only access one box, and the bank can re-key that box. But if someone compromises the vault's master combination, they can access all boxes, so that must have the highest protection.

**Summary:**

Envelope encryption is practical and scalable because it recognizes that not all keys require equal protection. By using a two-tier key hierarchy—expensive HSM protection for a few master keys, and software protection for many data keys—cloud systems achieve both strong security and the performance needed to encrypt exabytes of data economically. This pattern is what makes cloud encryption practical at scale, and it's why Task 5's envelope encryption demonstration is the most important pattern to understand for real-world cloud security.

---

### Q4. How does cryptographic erasure achieve provable deletion where overwriting cannot (in the cloud)?

**Answer:**

Cryptographic erasure (also called crypto-shredding) is a technique for making data permanently and provably unrecoverable by destroying the encryption keys rather than overwriting the data itself. In Task 6, we demonstrated this by scheduling tenant-A's KMS key for deletion, which made all envelope-encrypted data (record.env.enc) permanently unrecoverable even though the encrypted file still exists on disk. This approach has become the preferred method for secure data deletion in cloud environments because traditional overwriting methods face fundamental challenges in virtualized and distributed systems.

**The Problem with Traditional Secure Deletion:**

**1. Distributed Replication:**
- Cloud storage systems automatically replicate data across multiple physical devices, data centers, and geographic regions for durability and availability
- A single file might exist on dozens of physical storage devices across the world
- You cannot track where all copies are located (storage virtualization is opaque to users)
- Even if you overwrite one copy, replicas remain on other devices

**2. Snapshots and Backups:**
- Cloud systems take frequent snapshots for disaster recovery (hourly, daily)
- Point-in-time backups preserve historical data states
- Deleted or overwritten data may exist in snapshots taken before deletion
- Backup retention policies might keep snapshots for months or years
- You cannot identify and overwrite data in all historical snapshots

**3. SSD and Flash Storage Challenges:**
- Modern SSDs use internal wear-leveling that relocates data blocks to distribute write operations evenly across the device
- When you "overwrite" a block, the SSD may write to a different physical location and mark the old location as available
- The original data remains physically present on the SSD in an inaccessible location
- Flash Translation Layer (FTL) mappings are opaque and controlled by the SSD firmware
- Traditional multi-pass overwrite methods (DoD 5220.22-M) don't work on SSDs as they did on magnetic hard drives

**4. No Physical Access:**
- Cloud tenants don't have physical access to storage hardware
- Cannot perform physical destruction (shredding, degaussing, incineration) of storage media
- Cannot verify that overwriting actually occurred at the physical level
- Must trust the cloud provider's deletion procedures

**5. Copy-on-Write Filesystems:**
- Modern filesystems (ZFS, Btrfs, APFS) and storage systems use copy-on-write
- Modifications create new copies rather than overwriting data in place
- Old versions remain until garbage collection, which is unpredictable
- Overwrite commands may not actually overwrite anything physically

**How Cryptographic Erasure Works:**

**Setup (During Normal Operations):**
1. Generate a unique encryption key per tenant, data classification, or even per object
2. Encrypt all sensitive data with these keys (using envelope encryption from Task 5)
3. Store encrypted data in cloud storage, databases, backups, and snapshots
4. Protect encryption keys in a separate Key Management Service (KMS)
5. Encrypted data and replicas spread across the infrastructure as normal

**Deletion (When Required):**
1. Simply delete the encryption key from KMS (Task 6)
2. After the pending deletion window, KMS permanently destroys the key material in the HSM
3. All data encrypted under that key becomes cryptographically unrecoverable
4. Even if encrypted ciphertext is recovered from storage, backups, or snapshots, it's useless without the key

**Why This Achieves Provable Deletion:**

**Mathematically Secure:**
- Without the encryption key, encrypted data encrypted with AES-256 is computationally infeasible to decrypt
- Brute-force would require trying 2^256 possible keys (more operations than atoms in the observable universe)
- No known practical attacks against properly implemented AES-256
- The encrypted data becomes effectively random noise

**Immediate Effect:**
- Deletion is instantaneous—simply delete the key
- No need to locate and overwrite multiple replicas across distributed storage
- No need to wait for multi-pass overwrite operations that could take hours or days for large datasets

**Auditable and Provable:**
- KMS logs all key deletions with timestamps and identities
- Audit trails provide proof of deletion for compliance purposes
- Can demonstrate to regulators that data is unrecoverable even if ciphertext remains
- Satisfies GDPR "right to be forgotten", HIPAA disposal requirements, PCI-DSS data retention policies

**Handles Replicas and Backups:**
- All replicas, snapshots, and backups contain only encrypted data
- Deleting one key makes all copies unrecoverable simultaneously
- Don't need to track or manage where copies exist
- Works even for data in off-site tape backups or cold storage

**Vendor-Independent:**
- Don't need to trust cloud provider's deletion procedures
- Can prove deletion mathematically rather than procedurally
- Works across hybrid environments (on-premises + cloud)
- Can be verified by third-party auditors

**Real-World Applications:**

**Per-Tenant Keys (Task 6):**
- Each customer's data encrypted with their own unique KMS key
- When customer leaves service or requests data deletion: delete their key
- All their data across all systems instantly becomes unrecoverable
- Other customers unaffected (key isolation)

**Per-Object Keys:**
- Each sensitive file, database record, or message encrypted with unique key
- Selective deletion: delete specific objects without affecting others
- Granular retention: keep some data for legal hold while deleting others

**Time-Based Deletion:**
- Encrypt data with keys that are automatically deleted after retention period
- Ensures compliance with data retention policies
- No need to remember to delete data manually

**Compliance Examples:**

**GDPR Right to Be Forgotten:**
- User requests deletion of personal data
- Delete their encryption key from KMS
- All their data (in production, backups, logs, analytics) becomes unrecoverable
- Audit log proves deletion occurred

**HIPAA Protected Health Information (PHI):**
- Patient data encrypted with per-patient keys
- After retention period: delete keys
- PHI becomes unrecoverable even if backups exist

**PCI-DSS Cardholder Data:**
- Payment card data encrypted with transaction-specific keys
- After 90 days: delete keys
- Old transaction data cannot be decrypted even if databases are breached

**Demonstration from Task 6:**

In our lab, after scheduling KEY_A for deletion:
- The patient record (record.env.enc) still exists on disk
- The wrapped data key (datakey.enc) still exists on disk
- But attempting to decrypt fails with KMSInvalidStateException
- After the 7-day pending window, the master key is permanently destroyed
- At that point, no one—not even AWS with full hardware access—can decrypt record.env.enc
- The encrypted file has become cryptographically worthless permanent noise

**Contrast with Overwriting:**

If we had tried to securely delete record.txt using traditional methods:
1. We'd need to locate all copies (original, backups, snapshots, replicas)
2. We'd need to perform multi-pass overwriting (7-35 passes per DoD standards)
3. We couldn't verify it worked on SSDs due to wear-leveling
4. We couldn't access copies in cloud provider's infrastructure
5. We couldn't prove deletion occurred for compliance auditors

**Limitations and Considerations:**

Cryptographic erasure is not a complete replacement for all deletion scenarios:
- **Still need key management:** Must protect the KMS master keys in HSMs
- **Key backup and recovery:** Must have procedures to prevent accidental key deletion
- **Performance:** Must design systems to handle per-tenant or per-object key management at scale
- **Not for unencrypted data:** Only works if data was encrypted in the first place
- **Regulatory acceptance:** Must confirm that regulations accept cryptographic erasure (most do, but verify)

**Conclusion:**

Cryptographic erasure solves the fundamental problem that you cannot reliably overwrite data in modern cloud and distributed systems. By making security depend on key deletion rather than data deletion, it provides provable, immediate, auditable data destruction that works regardless of how many copies exist or where they're located. This is why Task 6's demonstration of scheduling key deletion and observing decrypt failures is so important—it shows that properly implemented cryptographic erasure truly makes data unrecoverable, making it the preferred approach for cloud data deletion.

---

### Q5. How does a hash chain make a log tamper-evident (link to tamper-proof logs, Week 6)?

**Answer:**

A hash chain is a data structure that creates tamper-evident logs by cryptographically linking each log entry to all previous entries through cascading cryptographic hashes. Any attempt to modify historical entries breaks the chain, making tampering immediately detectable. This technique is the fundamental building block for blockchain technology, certificate transparency logs, audit trail systems, and secure logging platforms. In Task 7, we built a simple hash chain for three audit events, demonstrating how each entry's hash depends on all previous entries.

**How Hash Chains Work (Task 7 Implementation):**

**Structure:**
```
Entry 1: login ok        | Hash1 = SHA256("0" + "login ok")
Entry 2: file read       | Hash2 = SHA256(Hash1 + "file read")
Entry 3: export data     | Hash3 = SHA256(Hash2 + "export data")
```

**Step-by-Step Construction:**

1. **Initialize:** Start with a genesis hash (often 0 or hash of empty string)
2. **First Entry:** Hash = SHA256(previous_hash + entry_1_data)
   - In Task 7: `PREV=$(echo -n "0login ok" | sha256sum)`
   - Produces: `login ok | 93a42b4...`
3. **Second Entry:** Hash = SHA256(Hash1 + entry_2_data)
   - In Task 7: `PREV=$(echo -n "${PREV}file read" | sha256sum)`
   - Produces: `file read | 7d8e3c1...`
4. **Third Entry:** Hash = SHA256(Hash2 + entry_3_data)
   - In Task 7: `PREV=$(echo -n "${PREV}export data" | sha256sum)`
   - Produces: `export data | 2f6a9b8...`

Each entry includes:
- The actual log data (timestamp, user, action, etc.)
- The hash of the previous entry
- Its own hash computed from previous hash + current data

**Why This Makes Tampering Detectable:**

**Avalanche Effect:**
- SHA-256 has the "avalanche effect": changing even one bit in the input produces a completely different 256-bit output
- No way to predict what changes will produce similar-looking hashes
- No way to find two different inputs that produce the same hash (collision resistance)

**Forward Propagation:**
- If an attacker modifies Entry 1 from "login ok" to "login failed":
  - Entry 1's hash changes completely
  - But Entry 2 includes Entry 1's hash in its computation
  - So Entry 2's stored hash no longer matches recomputation
  - Entry 3 includes Entry 2's hash, so it also breaks
  - The tampering is detected when validating the chain

**Example Attack Scenario:**
```
Original Chain:
Entry 1: login ok        | Hash1 = abc123...
Entry 2: file read       | Hash2 = def456... (includes abc123)
Entry 3: export data     | Hash3 = ghi789... (includes def456)

Attacker modifies Entry 1:
Entry 1: login FAILED    | Hash1 = xyz999... (different!)
Entry 2: file read       | Hash2 = def456... (stored hash)
```

**Validation Process:**
1. Recompute Hash1 from modified Entry 1 → get xyz999
2. Recompute Hash2 = SHA256(xyz999 + "file read") → get NEW_VALUE
3. Compare recomputed NEW_VALUE with stored def456
4. **Mismatch detected!** Log has been tampered with

**Attacker's Problem:**

To successfully tamper without detection, the attacker would need to:
1. Modify Entry 1
2. Recompute Hash1 with the new data
3. Modify Entry 2 to include the new Hash1
4. Recompute Hash2 with new data
5. Modify Entry 3 to include the new Hash2
6. Recompute Hash3... and so on for all subsequent entries

**Defense Against This:**
- **Store hash chain head externally:** Publish the most recent hash (Hash3 in our example) to an external system like blockchain, certificate transparency log, newspaper publication, or trusted timestamping service
- **Real-time monitoring:** Alert immediately when new entries are added, making retroactive chain modification obvious
- **Signed entries:** Combine hash chains with digital signatures (Task 2.3) so entries are signed by append-only keys that are revoked after writing
- **Distributed consensus:** Use blockchain-style consensus where multiple parties must agree on the chain state (Bitcoin, Ethereum)

**Real-World Applications:**

**1. Blockchain (Bitcoin, Ethereum):**
- Each block contains: transactions + hash of previous block + nonce
- Block N+1 includes hash of Block N
- Modifying any historical transaction requires recomputing all subsequent blocks
- Proof-of-work makes recomputation economically infeasible
- Our Task 7 hash chain is a simplified blockchain without proof-of-work

**2. Certificate Transparency Logs:**
- Google Chrome requires TLS certificates to be logged in public CT logs
- Each log entry (certificate) is added to a hash chain (Merkle tree)
- Browsers verify that certificates appear in the chain
- Detects misissued or fraudulent certificates
- Cannot remove certificates from history without breaking the chain

**3. AWS CloudTrail Log File Integrity:**
- AWS CloudTrail logs all API operations for audit purposes
- Uses hash chains to validate that log files haven't been modified
- Each log file includes hash of previous log file
- Digest files provide cryptographic proof of log integrity
- Meets compliance requirements for tamper-proof audit trails

**4. Git Version Control:**
- Each commit includes SHA-1 hash of: parent commit + changes + metadata
- Commit history forms a hash chain (directed acyclic graph)
- Modifying any historical commit changes its hash and breaks all descendant commits
- `git log --verify` can detect tampering

**5. Medical Records and Legal Documents:**
- Electronic health records with hash chains prevent backdating or modification of diagnoses
- Legal contracts with timestamped hash chains prove when documents were created
- Satisfies regulatory requirements for audit trails (HIPAA, SOX, FDA 21 CFR Part 11)

**6. IoT and Sensor Data:**
- Industrial sensors (manufacturing, energy) use hash chains to create tamper-evident data streams
- Prevents manipulation of sensor readings that could hide equipment failures or safety violations
- Each reading includes hash of previous reading, creating an integrity-protected time series

**Enhancement: Merkle Trees (Week 6 Connection):**

Hash chains can be extended to Merkle trees for more efficient verification:
- Binary tree where leaf nodes are hashes of data entries
- Internal nodes are hashes of their two child nodes
- Root hash represents the entire tree state
- Can verify individual entry exists without processing entire chain
- Used in Bitcoin, Git, Certificate Transparency, distributed databases

**Tamper-Proof vs Tamper-Evident:**

Important distinction:
- **Tamper-proof:** Prevents tampering through access control, encryption, physical security (ideal but often impractical)
- **Tamper-evident:** Doesn't prevent tampering but makes it detectable through cryptographic techniques (practical and deployable)

Hash chains are tamper-evident, not tamper-proof:
- An attacker with write access can still modify logs
- But the cryptographic chain breaks, making tampering obvious
- Combined with monitoring and backups, this provides strong security

**Production Implementation Considerations:**

1. **Performance:** Computing SHA-256 for every log entry adds overhead—batch entries or use async processing
2. **Storage:** Each entry stores hash of previous entry (32 bytes for SHA-256)—minimal overhead
3. **Validation:** Periodically verify chain integrity by recomputing hashes
4. **Anchoring:** Publish hash chain head to external system for additional security
5. **Key Management:** If combining with signatures, use HSM-protected signing keys
6. **Monitoring:** Alert on chain breaks immediately—indicates security incident

**Connection to Week 6 Material:**

The course likely covers:
- Advanced hash chain structures (Merkle trees, skip lists)
- Blockchain consensus mechanisms (proof-of-work, proof-of-stake)
- Distributed ledger technology applications
- Integration with CloudTrail, CloudWatch, or similar audit services
- Compliance frameworks requiring tamper-proof logs (SOC 2, ISO 27001, FedRAMP)

**Summary:**

Hash chains make logs tamper-evident by cryptographically linking each entry to all previous entries through cascading SHA-256 hashes. Modifying any historical entry breaks the chain forward, making tampering immediately detectable during validation. While not tamper-proof (attackers can still modify logs), hash chains make security incidents obvious and provide cryptographic proof of log integrity for audit and compliance purposes. This technique underpins blockchain, certificate transparency, version control, and secure audit logging—all critical components of modern cloud security architecture.

---

## Conclusion

This lab successfully demonstrated the complete lifecycle of data protection in cloud computing environments, progressing from fundamental cryptographic operations to enterprise-scale key management patterns. The two-session structure provided a logical progression from understanding individual building blocks (AES symmetric encryption, RSA asymmetric cryptography, TLS encrypted channels, SHA-256 integrity hashing) to composing those building blocks into production-ready patterns (envelope encryption, per-tenant key isolation, cryptographic erasure, tamper-evident audit logs).

### Key Findings:

**1. Encryption Alone Is Insufficient:**

The most critical lesson from this lab is that encryption algorithms—no matter how strong—provide no security without proper key management. AES-256 and RSA-2048 are mathematically robust, but the security of encrypted data depends entirely on how encryption keys are generated (cryptographically random, not predictable), stored (never in plaintext, protected in HSMs), distributed (secure channels, no hardcoding), accessed (IAM policies, least privilege), rotated (regular schedules, compromise recovery), and destroyed (secure deletion, cryptographic erasure). The progression from file-based keys in Session A (private.pem, key.pem) to KMS-managed keys in Session B (LocalStack KMS with HSM simulation) demonstrated the operational security advantages of centralized key management.

**2. Hybrid Cryptography Combines Best Properties:**

Pure symmetric encryption (Task 1) offers performance but suffers from key distribution problems. Pure asymmetric encryption (Task 2) solves key distribution but is too slow for large data. Real-world systems—TLS in Task 3 and envelope encryption in Task 5—use hybrid approaches that leverage asymmetric cryptography for key exchange and authentication, then symmetric encryption for bulk data protection. This pattern appears everywhere in cloud security: HTTPS connections, SSH sessions, email encryption, cloud storage encryption, and database encryption all use this hybrid model because it provides the security properties of public-key cryptography with the performance characteristics of symmetric encryption.

**3. Envelope Encryption Enables Cloud-Scale Security:**

The envelope encryption pattern (Task 5) is the key architectural insight that makes cloud encryption practical at exabyte scale. By using KMS only for key operations (generate, wrap, unwrap) and performing data encryption locally with DEKs, systems achieve the security benefits of centralized key management (HSM protection, access control, audit logging) without the performance bottlenecks that would occur if all data passed through KMS APIs. The three-step workflow—generate data key, encrypt locally, destroy plaintext key—appears in every major cloud provider's encryption services precisely because it's the only pattern that scales from gigabytes to exabytes while maintaining strong security guarantees.

**4. Per-Tenant Key Isolation Is Essential for Multi-Tenancy:**

Task 6's demonstration of creating separate KMS keys for tenant-A and tenant-B showed why multi-tenant cloud systems must never use a single encryption key for all customers. Key isolation provides critical security and operational benefits: limiting blast radius (compromised tenant-A key doesn't expose tenant-B data), enabling selective cryptographic erasure (delete one customer's data without affecting others), supporting independent access control policies (different teams manage different customers' keys), and meeting compliance requirements for data residency and sovereignty. This architectural pattern appears in SaaS applications, cloud platforms, and managed services where strong customer isolation is required for security, compliance, or business reasons.

**5. Cryptographic Erasure Solves Cloud Deletion Problems:**

Traditional secure deletion methods (multi-pass overwriting, physical destruction) are impractical or impossible in cloud environments with distributed replication, automatic snapshots, SSD wear-leveling, and no physical hardware access. Task 6's demonstration of cryptographic erasure—scheduling KEY_A for deletion and observing that encrypted data becomes permanently unrecoverable—showed why key deletion is the preferred approach for secure data disposal in the cloud. This technique is how cloud providers and SaaS platforms implement GDPR "right to be forgotten," HIPAA secure disposal requirements, and PCI-DSS data retention policies, because it provides immediate, auditable, provable deletion regardless of how many data copies exist or where they're located in distributed infrastructure.

**6. Defense-in-Depth Requires Multiple Security Dimensions:**

The lab demonstrated that comprehensive data protection requires layered controls across multiple dimensions rather than relying on any single security mechanism. Confidentiality (encryption prevents unauthorized reading), integrity (hashing and signatures detect unauthorized modifications), authentication (digital signatures prove origin), secure communication (TLS protects data in transit), key lifecycle management (KMS controls who can use keys when), and audit logging (tracking all key operations for security monitoring) must all work together. Missing any layer creates vulnerabilities: encryption without integrity allows tampering, TLS without certificate validation enables man-in-the-middle attacks, strong encryption with weak key management provides no real security.

### Skills Acquired:

Through this two-session lab, practical hands-on skills were developed in:

**Cryptographic Operations:**
- Using OpenSSL command-line tools for AES encryption/decryption, RSA key generation, public-key encryption, digital signature creation and verification, and SHA-256 hash calculation
- Understanding cipher modes (CBC), key derivation functions (PBKDF2), and the importance of salt and initialization vectors
- Recognizing when to use symmetric vs asymmetric encryption based on performance, key management, and data size requirements

**Certificate Management:**
- Generating self-signed X.509 certificates for TLS testing
- Understanding certificate fields (Subject, CN, validity period) and extensions (SAN for production)
- Configuring web servers (Nginx) for HTTPS with certificate and private key
- Recognizing the difference between self-signed certificates (development) and CA-signed certificates (production)

**Cloud Key Management:**
- Using AWS CLI to interact with KMS APIs (CreateKey, Encrypt, GenerateDataKey, Decrypt, ScheduleKeyDeletion)
- Implementing envelope encryption workflow from key generation through data encryption to key cleanup
- Managing key lifecycle states (Enabled, Disabled, PendingDeletion) and understanding their security implications
- Designing per-tenant key architectures for multi-tenant isolation

**Security Architecture Patterns:**
- Envelope encryption as the standard pattern for cloud-scale data protection
- Hash chains for tamper-evident audit logging
- Cryptographic erasure for secure cloud data deletion
- Hybrid cryptography combining symmetric and asymmetric operations

**Operational Security Practices:**
- Proper handling of plaintext keys (temporary use, immediate destruction)
- Secure configuration management (never hardcoding keys, using environment variables or KMS)
- Verification and testing methodology (comparing before/after states, validating signatures, testing key deletion enforcement)
- Documentation and audit trail creation (capturing commands and outputs for compliance evidence)

### Real-World Applications:

The techniques and patterns learned in this lab are directly applicable to:

**Cloud Storage Encryption:**
- AWS S3 Server-Side Encryption with KMS (SSE-KMS) uses envelope encryption identical to Task 5
- Azure Storage Service Encryption and Google Cloud Storage encryption follow the same pattern
- Object storage systems encrypt each object with unique DEKs wrapped by customer-managed master keys

**Database Encryption:**
- AWS RDS transparent data encryption (TDE) uses envelope encryption for database files
- DynamoDB encryption at rest uses KMS-managed keys per table
- MongoDB, PostgreSQL, MySQL all support TDE with KMS integration

**Application-Level Encryption:**
- SaaS applications handling sensitive data (healthcare, financial, legal) use field-level encryption with envelope encryption
- Per-customer encryption keys enable data isolation and selective deletion when customers leave
- Client-side encryption in browsers and mobile apps follows the same key generation and destruction patterns

**Compliance and Regulatory:**
- GDPR Article 17 "Right to Erasure" implemented through cryptographic erasure (Task 6)
- HIPAA Security Rule "disposal" requirements met through key deletion rather than media destruction
- PCI-DSS Requirement 3.2 key management implemented through KMS and envelope encryption
- SOC 2 audit evidence provided through KMS CloudTrail logs and hash chain integrity verification

**Secure Communications:**
- HTTPS for web applications using TLS certificates (Task 3 patterns in production)
- API security using TLS mutual authentication (mTLS) with client certificates
- VPN tunnels using hybrid encryption (IKE for key exchange, IPsec for data protection)
- Secure email (S/MIME, PGP) using digital signatures and public-key encryption

**Blockchain and Distributed Ledgers:**
- Hash chains (Task 7) as the foundation for blockchain immutability
- Digital signatures for transaction authentication
- Cryptographic proof-of-work and proof-of-stake consensus mechanisms
- Smart contract security and key management for decentralized applications

### Production Considerations:

While this lab used local development environments (LocalStack, self-signed certificates), production implementations require additional considerations:

**Key Management:**
- Use actual cloud KMS services (AWS KMS, Azure Key Vault, Google Cloud KMS) with FIPS 140-2 Level 2/3 certified HSMs for master key protection
- Implement automated key rotation schedules (annually or more frequently for high-security data)
- Design key hierarchies for different data classifications (public, internal, confidential, restricted)
- Establish key backup and recovery procedures with multi-party approval for disaster recovery
- Monitor KMS audit logs in real-time for unauthorized key access attempts or suspicious patterns

**Certificate Management:**
- Use certificates from trusted Certificate Authorities (Let's Encrypt for automation, commercial CAs for extended validation)
- Implement certificate renewal automation to prevent expiration incidents
- Use Certificate Transparency monitoring to detect misissued certificates
- Configure proper TLS settings (TLS 1.2 minimum, TLS 1.3 preferred, strong cipher suites only)
- Implement HTTP Strict Transport Security (HSTS) and certificate pinning where appropriate

**Access Control:**
- Implement least-privilege IAM policies for KMS key usage (separate roles for key administrators vs key users)
- Use service control policies (SCPs) and permission boundaries to prevent privilege escalation
- Require multi-factor authentication for sensitive key operations
- Implement separation of duties (different roles for key creation, usage, and deletion)
- Regular access reviews and automated revocation for terminated employees

**Monitoring and Alerting:**
- Stream KMS CloudTrail logs to SIEM systems for security monitoring
- Alert on suspicious patterns (excessive decrypt failures, unusual key access times, access from unexpected IPs)
- Monitor envelope encryption operations for performance degradation or failures
- Track key usage metrics to identify over-privileged applications
- Implement anomaly detection for unusual encryption/decryption volumes

**Compliance and Audit:**
- Maintain comprehensive documentation of encryption architectures and key management procedures
- Generate periodic reports showing which keys protect which data classifications
- Provide auditors with KMS access logs and hash chain integrity proofs
- Implement data classification tagging and automated encryption enforcement
- Demonstrate cryptographic erasure procedures for data deletion audits

### Future Learning:

This lab establishes the foundation for advanced security topics including:

**Advanced Encryption:**
- Authenticated encryption modes (AES-GCM, ChaCha20-Poly1305) that provide confidentiality and integrity simultaneously
- Elliptic Curve Cryptography (ECC) for smaller keys with equivalent security (P-256, P-384)
- Post-quantum cryptography preparing for quantum computing threats
- Homomorphic encryption allowing computation on encrypted data without decryption

**Key Management Patterns:**
- Bring Your Own Key (BYOK) where customers generate keys externally and import to cloud KMS
- Hold Your Own Key (HYOK) where keys remain in customer-controlled HSMs
- Multi-region key replication and global key management for distributed applications
- Key rotation with automatic re-encryption of existing encrypted data

**Zero-Knowledge Proofs:**
- Proving knowledge of information without revealing the information itself
- Applications in privacy-preserving authentication and blockchain smart contracts
- zk-SNARKs and zk-STARKs for scalable blockchain verification

**Secure Multi-Party Computation:**
- Multiple parties jointly computing functions without revealing inputs
- Applications in privacy-preserving machine learning and collaborative analytics
- Differential privacy for protecting individual records in aggregate datasets

**Hardware Security:**
- Trusted Platform Modules (TPM) for device-level key storage
- Hardware Security Modules (HSMs) architecture and FIPS 140-2 certification levels
- Secure enclaves (Intel SGX, ARM TrustZone) for trusted execution environments
- Quantum Key Distribution (QKD) for theoretically unbreakable key exchange

---

## References

The following resources were referenced during this lab and provide additional depth for further study:

1. **Course lecture** — Week 4 (Data Protection) and Week 9 (Key Management patterns), Prof. Dr. Shahrulniza Musa, UniKL MIIT, covering encryption fundamentals, key lifecycle, and cloud security architecture.

2. **OpenSSL documentation** — Official documentation for encryption commands, key generation, certificate management, and digital signatures: [https://www.openssl.org/docs](https://www.openssl.org/docs)

3. **AWS Key Management Service (KMS) documentation** — Comprehensive guide to envelope encryption, key policies, grants, and best practices: [https://docs.aws.amazon.com/kms](https://docs.aws.amazon.com/kms)

4. **NIST SP 800-57** — *Recommendation for Key Management, Parts 1-3*, National Institute of Standards and Technology, comprehensive guidance on cryptographic key management including key generation, distribution, storage, and destruction.

5. **NIST SP 800-38A** — *Recommendation for Block Cipher Modes of Operation*, covering CBC, CTR, GCM and other AES modes with security considerations.

6. **NIST SP 800-88 Rev. 1** — *Guidelines for Media Sanitization*, covering secure deletion techniques and why cryptographic erasure is preferred for cloud environments.

7. **NIST SP 800-111** — *Guide to Storage Encryption Technologies for End User Devices*, covering full disk encryption, file encryption, and key management.

8. **Cloud Security Alliance (CSA) Security Guidance v5** — *Domain 6: Data Security & Encryption*, industry best practices for cloud data protection, key management, and cryptographic erasure.

9. **RFC 5280** — *Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile*, defining certificate formats used in Task 3.

10. **RFC 8446** — *The Transport Layer Security (TLS) Protocol Version 1.3*, latest TLS standard with improved security and performance.

11. **FIPS 140-2** — *Security Requirements for Cryptographic Modules*, U.S. government standard for HSM certification referenced in KMS discussions.

12. **GDPR Article 17** — Right to erasure ("right to be forgotten"), EU regulation requiring secure data deletion capabilities demonstrated through cryptographic erasure.

---

## Appendix: Cleanup Commands

To clean up the lab environment and free system resources, execute the following commands:

```bash
# Stop any running Docker containers from Task 3
docker stop tls 2>/dev/null

# Remove generated files from Session A
rm -f record.* private.pem public.pem key.pem cert.pem tampered.txt

# Remove generated files from Session B
rm -f datakey.* 

# Stop LocalStack if it was started for Session B
docker stop localstack 2>/dev/null
docker rm localstack 2>/dev/null

# Verify cleanup
ls -la record.* private.pem public.pem key.pem cert.pem datakey.* 2>/dev/null
docker ps -a | grep -E 'tls|localstack'
```

**Note:** The cleanup removes all encryption keys, certificates, encrypted files, and temporary data created during the lab. If you need to preserve any evidence for your report submission, make copies before executing cleanup commands. The `2>/dev/null` redirects suppress error messages for files or containers that don't exist, allowing the cleanup script to run safely even if some steps were not completed.

---

## Expansion Ideas (Advanced Students)

For students who want to deepen their understanding of encryption and key management, the following advanced topics extend the concepts covered in this lab:

### 1. Hardware Security Module (HSM) Integration

**Objective:** Use a software HSM (SoftHSM2) to store encryption keys and perform cryptographic operations with PKCS#11 interface.

**Why it matters:** Production KMS systems use HSMs certified to FIPS 140-2 Level 2/3 for physical tamper resistance and key protection. Understanding HSM operations provides insight into how KMS actually protects master keys.

**Implementation:** Install SoftHSM2, initialize a token with PIN, generate an RSA key pair inside the HSM, use OpenSSL with PKCS#11 engine to sign documents with the HSM-protected key, and demonstrate that the private key cannot be extracted from the HSM.

### 2. Automated Key Rotation

**Objective:** Implement automated key rotation that generates new master keys, re-wraps all data encryption keys under the new master key, and deprecates old master keys.

**Why it matters:** Key rotation is a security best practice that limits cryptanalysis exposure and provides crypto-agility. Production systems must rotate keys without application downtime.

**Implementation:** Create a rotation script that (1) creates a new KMS master key, (2) lists all wrapped DEKs encrypted under the old key, (3) calls KMS Decrypt with old key and Encrypt with new key to re-wrap each DEK, (4) updates metadata to reference the new key, (5) schedules the old key for deletion after grace period.

### 3. Client-Side Encryption with Encryption SDK

**Objective:** Use AWS Encryption SDK or similar library to implement envelope encryption in application code with automatic key caching and rotation.

**Why it matters:** Real applications don't call raw KMS APIs—they use encryption libraries that handle envelope encryption, caching, and key rotation transparently.

**Implementation:** Install AWS Encryption SDK (Python, Java, or Node.js), configure with KMS master key provider, encrypt multiple files with automatic DEK caching (avoiding KMS calls for every file), implement data key caching with TTL and usage limits, and demonstrate decryption with automatic KMS unwrapping.

### 4. Mutual TLS (mTLS) Authentication

**Objective:** Configure both server and client to present certificates, establishing bidirectional authentication where both parties verify each other's identity.

**Why it matters:** mTLS is used in zero-trust architectures, microservices authentication (service mesh), API security, and IoT device authentication.

**Implementation:** Generate a Certificate Authority (CA) certificate, sign both server and client certificates with the CA, configure Nginx to require client certificates, use curl with client certificate to connect, and demonstrate that connections without valid client certificates are rejected.

### 5. Authenticated Encryption with AES-GCM

**Objective:** Replace AES-CBC (which provides only confidentiality) with AES-GCM mode that provides both confidentiality and integrity/authentication in a single operation.

**Why it matters:** Modern applications should use authenticated encryption to prevent tampering attacks, padding oracle attacks, and other vulnerabilities that affect non-authenticated modes.

**Implementation:** Encrypt data with `openssl enc -aes-256-gcm`, demonstrate that any modification to ciphertext causes decryption to fail with authentication tag mismatch, compare performance with AES-CBC + HMAC separate operations, and understand why TLS 1.3 uses only authenticated encryption modes.

### 6. HashiCorp Vault Integration

**Objective:** Deploy Vault in a Docker container and use its transit secrets engine for envelope encryption without managing KMS keys directly.

**Why it matters:** Vault is a popular open-source alternative to cloud KMS, used in hybrid and multi-cloud environments for centralized secrets and key management.

**Implementation:** Start Vault dev server, enable transit engine, create an encryption key, use Vault API to encrypt/decrypt data, implement envelope encryption workflow using Vault instead of AWS KMS, and configure access policies for multi-tenant isolation.

### 7. Certificate Transparency Log Verification

**Objective:** Submit a TLS certificate to a public Certificate Transparency log and verify the Signed Certificate Timestamp (SCT).

**Why it matters:** Certificate Transparency prevents misissued certificates by requiring public logging, and browsers now require SCTs for certificate acceptance.

**Implementation:** Use ct-submit tool to submit a certificate to a CT log, retrieve the SCT proving the certificate was logged, use verification tools to confirm the SCT signature is valid, and understand how browsers use CT logs to detect fraudulent certificates.

### 8. Shamir's Secret Sharing for Key Backup

**Objective:** Split a KMS master key backup into multiple shares where K-of-N shares are required to reconstruct the key.

**Why it matters:** High-security environments use secret sharing to prevent single points of failure and require multi-party approval for key recovery.

**Implementation:** Use ssss (Shamir's Secret Sharing Scheme) tool to split a master key into 5 shares requiring any 3 to reconstruct, distribute shares to different administrators, demonstrate reconstruction with 3 shares, and show that 2 shares reveal nothing about the key.

---

## Acknowledgments

This lab was completed as part of the IKB42603 Cloud Computing Security Essentials course at Universiti Kuala Lumpur Malaysian Institute of Information Technology (UniKL MIIT). Special thanks to:

- **Prof. Dr. Shahrulniza Musa** for developing the comprehensive lab curriculum covering encryption fundamentals, key management patterns, and cryptographic erasure techniques, and for providing expert guidance on cloud security principles and best practices throughout the course.

- **Madam Nor Adani Kamal Mohammad Nasir** for supervising this lab session, providing clarifications on lab procedures, and offering feedback on security implementations during hands-on exercises.

- **The OpenSSL Project** for providing the open-source cryptographic toolkit that makes encryption, key generation, digital signatures, and certificate management accessible for educational purposes and production deployments worldwide.

- **The LocalStack Team** for creating the local cloud emulation platform that enables students to practice AWS KMS operations without requiring cloud accounts or incurring costs, making cloud security education accessible.

- **AWS Key Management Service** for pioneering envelope encryption patterns and HSM-backed key management services that set the industry standard for cloud data protection.

- **The Calico Project** (though primarily used in Lab 2) and other open-source security projects that enable hands-on learning of production-grade security technologies in educational environments.

---

## End of Report

**Lab Status:** All tasks completed with evidence

**Evidence Files:** 21 screenshots documenting all seven tasks across both sessions

**Verification:** All commands executed successfully with expected outputs

**Learning Outcomes:** Achieved comprehensive understanding of encryption, key management, envelope encryption, cryptographic erasure, and integrity verification

---

**Submitted by:** Surya Giri A/L Shanker  
**Student ID:** 52215124335  
**Course:** IKB42603 Cloud Computing Security Essentials  
**Institution:** Universiti Kuala Lumpur Malaysian Institute of Information Technology (UniKL MIIT)  
**Lab:** Lab 3 - Data Protection: Encryption & Key Management  
**Sessions:** Week 5 (Encryption Fundamentals) and Week 6 (Key Management & Erasure)  
**Date:** 2024  
**Lecturer:** Madam Nor Adani Kamal Mohammad Nasir  
**Professor:** Prof. Dr. Shahrulniza Musa

---

**Security Statement:** All cryptographic operations performed in this lab used development environments (LocalStack, self-signed certificates) and test data (sample patient records) appropriate for educational purposes. No actual sensitive data was used, and no production systems were accessed. The encryption keys and certificates generated during this lab have been properly cleaned up and destroyed after completion. This report demonstrates understanding of security principles and practical skills that can be applied to real-world cloud security implementations with appropriate safeguards, compliance controls, and production-grade infrastructure.

---

