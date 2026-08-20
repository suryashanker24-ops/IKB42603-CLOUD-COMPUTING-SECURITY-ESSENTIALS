# Lab 3: Encryption and Key Management Report

## Student Information

- **Name:** Surya Giri A/L Shanker
- **Student ID:** 52215124335
- **Course:** IKB42603 Cloud Computing Security Essentials
- **Lab Task:** Lab 3 - Encryption and Key Management
- **Lecturer:** Madam Nor Adani Kamal Mohammad Nasir

## Overview

This report documents the implementation and verification of encryption and key-management controls for sensitive cloud data. The lab demonstrates how confidentiality, integrity, authentication, secure communications, tenant separation, and cryptographic erasure work together as a defence-in-depth strategy. Practical tasks were performed in a Kali Linux terminal using OpenSSL, Docker with Nginx, and the AWS CLI connected to a local KMS-compatible endpoint.

The lab begins with AES-256 symmetric encryption to protect a sensitive record at rest. It then uses RSA public-key cryptography to demonstrate encryption/decryption and digital signatures. A self-signed certificate and Nginx TLS service demonstrate protection of data in transit. The final tasks use a Key Management Service (KMS) to create tenant-specific master keys, encrypt a small secret directly, generate data keys for envelope encryption, remove plaintext key material, schedule a key for deletion, and verify integrity with SHA-256 and audit-chain hashes.

The core security lesson is that encryption is only as effective as its key management. Strong encryption protects data only while keys are protected, access to keys is controlled, and key lifecycle actions such as rotation, disabling, and deletion are handled safely.

## Objectives

The objectives of this lab are to:

1. Encrypt and decrypt a sensitive record using AES-256-CBC with PBKDF2 and a salt.
2. Demonstrate that encrypted ciphertext is unreadable without the correct key or password.
3. Generate an RSA 2,048-bit key pair and use it for public-key encryption and private-key decryption.
4. Create and verify a SHA-256 digital signature to demonstrate integrity and authentication.
5. Generate a self-signed TLS certificate and serve data securely over HTTPS.
6. Create and use KMS customer-managed keys for tenant-specific encryption.
7. Implement envelope encryption by using a KMS-generated data key to encrypt file data locally.
8. Demonstrate secure key handling by destroying temporary plaintext data-key material.
9. Apply key lifecycle controls, including scheduled deletion, and observe the resulting denial of decrypt operations.
10. Verify data and audit-record integrity using SHA-256 hashes and a simple hash chain.

## Learning Outcomes

After completing this lab, the student should be able to:

- Explain the difference between symmetric and asymmetric encryption and select an appropriate use case for each.
- Use OpenSSL to encrypt, decrypt, sign, verify, generate keys, and generate certificates.
- Explain why TLS is essential for data in transit and why clients should validate certificates in production.
- Distinguish direct KMS encryption of small secrets from envelope encryption of large data.
- Describe how KMS separates key control from application data and supports tenant isolation.
- Explain cryptographic erasure: encrypted data becomes unusable when its encryption key is permanently destroyed.
- Recognise the importance of data integrity checks, signatures, and audit logs in security investigations and compliance.

## Environment and Prerequisites

The lab was conducted in a Kali Linux environment. The following tools and conditions were used:

- **OpenSSL** for AES encryption, RSA keys, digital signatures, certificates, and SHA-256 hashing.
- **Docker** to run an Nginx HTTPS container locally.
- **Nginx** as the web server used to demonstrate TLS.
- **AWS CLI** for KMS commands.
- **Local KMS-compatible endpoint** accessed through the following shell variable:

  ```sh
  EP="--endpoint-url=http://localhost:4566"
  ```

- A terminal with Bash-compatible commands, a writable working directory, and the supplied sample record.

> Note: The local endpoint is appropriate for laboratory practice. A production deployment should use an organisation's actual cloud KMS, IAM permissions, key policies, monitoring, backup strategy, and approved certificate authority.

---

## Task 1 - Symmetric Encryption with AES-256

Symmetric encryption uses the same secret key (or password-derived key) for both encryption and decryption. It is efficient and therefore suitable for protecting files, databases, backups, and other large volumes of data. The record was encrypted with AES-256-CBC. PBKDF2 was used to derive a strong key from the supplied password, while a salt ensures the same password produces different ciphertext on separate encryptions.

### Purpose

- Create a sample sensitive patient record.
- Encrypt the record using AES-256-CBC.
- Show that the ciphertext cannot be read as ordinary text.
- Decrypt the record and prove that it matches the original.

### Terminal Commands

```sh
echo 'Patient: surya, Diagnosis: confidential' > record.txt

openssl enc -aes-256-cbc -pbkdf2 -salt -in record.txt -out record.enc

cat record.enc

openssl enc -d -aes-256-cbc -pbkdf2 -in record.enc -out record.dec.txt
diff record.txt record.dec.txt && echo 'MATCH: decryption successful'
```

### Explanation of the Commands

- `echo ... > record.txt` creates the plaintext record used as the sensitive data sample.
- `openssl enc -aes-256-cbc` encrypts the file with the AES cipher using a 256-bit key in CBC mode.
- `-pbkdf2` uses Password-Based Key Derivation Function 2 to derive the encryption key from the password more securely than OpenSSL's legacy derivation method.
- `-salt` adds random salt data, preventing identical plaintext and passwords from generating identical ciphertext.
- `-in record.txt -out record.enc` specifies the input plaintext and output encrypted file.
- `cat record.enc` displays the encrypted bytes; its unreadable output demonstrates that the plaintext is not exposed.
- `openssl enc -d` performs decryption using the same algorithm and password.
- `diff` compares the original and decrypted files. No difference followed by the displayed success message confirms correct recovery.

### Evidence and Result

- [Sample sensitive record](<evidence/Task 1 create a sample sensitive record.png>) shows the patient record before encryption.
- [AES-256 encryption](<evidence/Task 1 encrypt with aes-256.png>) shows the encryption command and password prompt.
- [Unreadable ciphertext](<evidence/Task 1 prove its inreadable.png>) shows that `record.enc` contains non-readable encrypted data.
- [Successful decryption](<evidence/Task 1 Decrypt back.png>) shows that the decrypted file matched `record.txt`.

### Notes

AES provides confidentiality, but not proof of the sender or automatic tamper detection when used in this form. A production system should use authenticated encryption, such as AES-GCM, or combine encryption with a separate integrity control. Passwords must be long, unique, and protected by a password-management or key-management process.

---

## Task 2 - RSA Public-Key Encryption and Digital Signatures

Asymmetric cryptography uses a pair of mathematically related keys. The public key may be shared, while the private key must remain secret. Data encrypted with the public key can only be decrypted with the private key. A digital signature works in the opposite trust direction: the owner signs with the private key, and anyone with the public key can verify the signature.

### 2.1 Generate an RSA Key Pair

#### Purpose

- Generate a 2,048-bit RSA private key.
- Derive the corresponding public key for distribution and verification.

#### Terminal Commands

```sh
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem
```

#### Explanation of the Commands

- `openssl genrsa -out private.pem 2048` generates a 2,048-bit RSA private key and stores it in `private.pem`. The private key must never be shared.
- `openssl rsa -in private.pem -pubout -out public.pem` extracts the matching public component and saves it as `public.pem`. This key can be distributed safely for encryption and signature verification.

#### Evidence

[RSA key-pair generation](<evidence/Task 2 generate 2048-bit key pair.png>) shows successful private- and public-key creation.

### 2.2 Encrypt with the Public Key and Decrypt with the Private Key

#### Purpose

- Demonstrate confidentiality using asymmetric encryption.
- Show that the private key is required to recover the encrypted record.

#### Terminal Commands

```sh
openssl pkeyutl -encrypt -pubin -inkey public.pem -in record.txt -out record.rsa
openssl pkeyutl -decrypt -inkey private.pem -in record.rsa -out record.rsa.txt
diff record.txt record.rsa.txt && echo 'MATCH: RSA decryption successful'
```

#### Explanation of the Commands

- `openssl pkeyutl -encrypt` performs a public-key encryption operation.
- `-pubin -inkey public.pem` tells OpenSSL that `public.pem` is a public key and uses it to encrypt the record.
- `-decrypt -inkey private.pem` uses the private key to recover the plaintext from `record.rsa`.
- `diff` verifies that the recovered RSA plaintext is identical to the source record.

#### Evidence and Result

[RSA encryption and decryption](<evidence/Task 2 encrypt with public key, decrypt with private key.png>) shows successful recovery and the message `MATCH: RSA decryption successful`.

### 2.3 Sign and Verify the Record

#### Purpose

- Demonstrate digital-signature creation using the private key.
- Verify record integrity and signer authenticity using the public key.

#### Terminal Commands

```sh
openssl dgst -sha256 -sign private.pem -out record.sig record.txt
openssl dgst -sha256 -verify public.pem -signature record.sig record.txt
```

#### Explanation of the Commands

- `openssl dgst -sha256` calculates a SHA-256 digest of `record.txt`.
- `-sign private.pem` signs that digest with the private key and writes the signature to `record.sig`.
- `-verify public.pem -signature record.sig` checks the signature using the public key. Verification succeeds only when the record is unchanged and the signature was created by the corresponding private key.

#### Evidence and Result

[Digital-signature verification](<evidence/Task 2 Sign in with the private key, verify with the public key.png>) shows OpenSSL returning `Verified OK`.

### Notes

RSA encryption is not normally used to encrypt large files because it is slower and has message-size limitations. It is commonly used to protect small secrets or exchange a symmetric data key. This principle leads directly to envelope encryption in Task 5.

---

## Task 3 - TLS Protection for Data in Transit

TLS encrypts communications between a client and server, protecting data from network eavesdropping and modification. A self-signed certificate was created for a local Nginx server and the patient record was retrieved over HTTPS.

### Purpose

- Generate a local self-signed TLS certificate.
- Serve `record.txt` through HTTPS on port 8443.
- Confirm that a client can establish a TLS connection and retrieve the protected file.

### Terminal Commands

```sh
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem \
  -days 7 -nodes -subj '/CN=localhost'

docker run --rm -d --name tls -p 8443:443 \
  -v "$(pwd)/cert.pem:/etc/nginx/cert.pem:ro" \
  -v "$(pwd)/key.pem:/etc/nginx/key.pem:ro" \
  -v "$(pwd)/record.txt:/usr/share/nginx/html/record.txt:ro" \
  -v "$(pwd)/nginx-tls.conf:/etc/nginx/nginx.conf:ro" nginx:alpine

curl -k https://localhost:8443/record.txt
docker stop tls
```

### Explanation of the Commands

- `openssl req -x509` creates a self-signed X.509 certificate rather than a certificate-signing request.
- `-newkey rsa:2048` generates a new 2,048-bit RSA private key while creating the certificate.
- `-keyout key.pem -out cert.pem` writes the private key and certificate to separate files.
- `-days 7` makes the development certificate valid for seven days; `-nodes` leaves the private key unencrypted so Nginx can start non-interactively.
- `-subj '/CN=localhost'` sets the certificate's Common Name to the local hostname.
- `docker run ... -p 8443:443` maps local TCP port 8443 to HTTPS port 443 in the Nginx container.
- The read-only volume mounts (`:ro`) provide Nginx with the certificate, key, configuration, and content without letting the container modify them.
- `curl -k` connects to the HTTPS endpoint. `-k` bypasses normal certificate validation because the certificate is self-signed; this must not be used in production.
- `docker stop tls` stops the temporary TLS service after verification.

### Evidence and Result

- [Self-signed certificate generation](<evidence/Task 3 generate a self-signed certificate.png>) shows the certificate and key creation.
- [HTTPS service on port 8443](<evidence/Task 3 serve https on port 8443.png>) shows the Nginx container being started with the TLS materials mounted read-only.
- [TLS connection test](<evidence/Task 3 connect over TLS.png>) shows that the HTTPS request returned the sensitive record successfully.

### Notes

Self-signed certificates are useful for controlled local testing but do not establish public trust. In production, certificates should be issued by a trusted certificate authority, include the correct Subject Alternative Names, be renewed before expiration, and be validated by clients without using `-k`.

---

## Task 4 - KMS Master Key and Direct Encryption

KMS centralises the creation, protection, permission control, and lifecycle management of encryption keys. A customer-managed symmetric KMS key was created for tenant A, then used to encrypt a small value directly. Direct KMS encryption is suitable for small secrets, while larger data should use envelope encryption.

### Purpose

- Create a tenant-A customer-managed KMS key.
- Use the key to encrypt a small plaintext value.
- Observe key metadata and the KMS ciphertext result.

### Terminal Commands

```sh
aws $EP kms create-key --description 'CCSE tenant-A master key'
KEY_A=<returned-key-id>

aws $EP kms encrypt --key-id $KEY_A \
  --plaintext "$(echo -n 'hello' | base64)" \
  --query CiphertextBlob --output text
```

### Explanation of the Commands

- `aws $EP kms create-key` creates a customer-managed KMS key at the configured local endpoint.
- `--description` assigns a meaningful label identifying the key's tenant and purpose.
- `KEY_A=<returned-key-id>` stores the returned key identifier in a shell variable for later commands.
- `aws $EP kms encrypt --key-id $KEY_A` requests that KMS encrypt the supplied plaintext under tenant A's key.
- `echo -n 'hello' | base64` prepares the small plaintext in the base64 form expected by this CLI workflow.
- `--query CiphertextBlob --output text` returns only the encrypted KMS ciphertext blob rather than the full JSON response.

### Evidence and Result

- [Tenant-A KMS key creation](<evidence/Task 4 Create and use a KMS master key 1.png>) shows an enabled `ENCRYPT_DECRYPT` symmetric customer-managed key.
- [Direct KMS encryption](<evidence/Task 4 encrypt a small secret directly with KMS.png>) shows the ciphertext blob returned for the value `hello`.

### Notes

The KMS key does not expose its underlying key material to the application. KMS performs the encryption operation and returns ciphertext. This centralises key protection and supports auditing and access control. However, direct KMS encryption is designed for small payloads; file data should be encrypted locally with a data key as shown in Task 5.

---

## Task 5 - Envelope Encryption

Envelope encryption uses two levels of keys. A KMS master key encrypts (wraps) a short-lived data key, and the data key encrypts the actual file locally using a fast symmetric algorithm. Only the wrapped data key is retained with the ciphertext. When decryption is needed, KMS unwraps the data key after authorising the request.

### 5.1 Generate a Data Key

#### Purpose

- Request a fresh AES-256 data key from KMS.
- Retain the KMS-wrapped key for future recovery.
- Use the plaintext data key only temporarily for local encryption.

#### Terminal Commands

```sh
aws $EP kms generate-data-key --key-id $KEY_A --key-spec AES_256 \
  --query '[Plaintext,CiphertextBlob]' --output text > datakey-output.txt
```

#### Explanation of the Commands

- `generate-data-key` asks KMS to create a new random data key protected by `KEY_A`.
- `--key-spec AES_256` requests 256-bit AES key material.
- The output includes two forms of the same key: `Plaintext`, used briefly for encryption, and `CiphertextBlob`, which is KMS-wrapped and can be stored safely as `datakey.enc`.
- `--query '[Plaintext,CiphertextBlob]' --output text` extracts those two values from the KMS response for the lab workflow.

#### Evidence

[KMS data-key generation](<evidence/Task 5.1 Ask the KMS for a data key.png>) shows KMS returning a plaintext data key and its encrypted KMS-wrapped form.

### 5.2 Encrypt the File with the Data Key

#### Purpose

- Decode the temporary data key for OpenSSL.
- Encrypt the file locally using AES-256.
- Avoid sending the entire file to KMS.

#### Terminal Commands

```sh
base64 -d datakey.b64 > datakey.bin
openssl enc -aes-256-cbc -pbkdf2 -in record.txt -out record.env.enc \
  -pass file:./datakey.bin
```

#### Explanation of the Commands

- `base64 -d datakey.b64 > datakey.bin` decodes the plaintext data key into a temporary binary key file.
- `openssl enc -aes-256-cbc -pbkdf2` encrypts the record locally using AES-256-CBC.
- `-pass file:./datakey.bin` reads the temporary data key from the file rather than requesting a password interactively.
- `record.env.enc` is the encrypted data file. The accompanying KMS-wrapped data key is required later to decrypt it.

#### Evidence

[File encryption with the data key](<evidence/Task 5.2 Encrypt the big file.png>) shows the local encryption of the record using the generated data key.

### 5.3 Destroy Plaintext Data-Key Material

#### Purpose

- Remove plaintext copies of the data key after use.
- Retain only the KMS-wrapped data key needed for future authorised decryption.

#### Terminal Commands

```sh
rm datakey.bin datakey.b64
```

#### Explanation of the Command

`rm datakey.bin datakey.b64` removes the temporary binary and base64 plaintext data-key files. Only the KMS-wrapped key (`datakey.enc`) remains. In a production system, plaintext keys should be kept only in memory where possible, never logged, and cleared promptly after use.

#### Evidence and Result

[Plaintext data-key removal](<evidence/Task 5.3 Destroy the plaintext data key.png>) confirms that only the KMS-wrapped data key remains.

### Notes

Envelope encryption combines the scalability of local symmetric encryption with the centralised controls of KMS. This pattern is commonly used by cloud object storage, database encryption systems, backup platforms, and applications that handle large quantities of sensitive data.

---

## Task 6 - Tenant Key Separation and Key Lifecycle

Multi-tenant cloud systems should avoid sharing one encryption key across all customers. Using independent keys per tenant limits the impact of compromise, supports tenant-specific access policies, and makes key rotation or cryptographic erasure possible without affecting other tenants.

### 6.1 Create a Separate Key for Tenant B

#### Purpose

- Create a KMS key independent from tenant A's key.
- Demonstrate key separation between tenants.

#### Terminal Commands

```sh
aws $EP kms create-key --description 'CCSE tenant-B master key'
KEY_B=<returned-key-id>
```

#### Explanation of the Commands

The command creates a second symmetric customer-managed key. Assigning it to `KEY_B` allows tenant-B data to be encrypted independently. The key material for tenant A is not reused for tenant B.

#### Evidence

[Tenant-B KMS key](<evidence/Task 6 seperate key for tenant b.png>) shows the separate enabled KMS key created for tenant B.

### 6.2 Schedule Deletion of Tenant A's Key

#### Purpose

- Demonstrate a controlled key-retirement process.
- Place the tenant-A key into a pending-deletion state with a recovery window.

#### Terminal Commands

```sh
aws $EP kms schedule-key-deletion --key-id $KEY_A --pending-window-in-days 7
```

#### Explanation of the Command

`schedule-key-deletion` starts a seven-day waiting period before permanent deletion. During this period, KMS reports the key state as `PendingDeletion`. The delay is an important safety control because it gives administrators time to cancel an accidental deletion before the underlying key material is permanently removed.

#### Evidence and Result

[Scheduled key deletion](<evidence/Task 6 schedule deletion of tenant A key.png>) shows `KeyState: PendingDeletion`, a seven-day window, and a scheduled deletion date.

### 6.3 Observe the Effect of the Key State

#### Terminal Commands

```sh
aws $EP kms disable-key --key-id $KEY_A
aws $EP kms decrypt --ciphertext-blob fileb://datakey.enc 2>&1 | head -3
```

#### Explanation and Result

- The `disable-key` attempt returned `KMSInvalidStateException` because the key had already entered `PendingDeletion`. A key already pending deletion cannot be processed as an enabled key.
- The `decrypt` attempt also returned `KMSInvalidStateException`. This demonstrates that the KMS-wrapped data key cannot be unwrapped while its protecting KMS key is pending deletion.

#### Evidence

- [Disable attempt after deletion scheduling](<evidence/Task 6 Disable immediately.png>) records the expected invalid-state error.
- [Blocked unwrap/decrypt attempt](<evidence/Task 6 attempt to unwrap tenant A data key.png>) records the invalid-state error when attempting to decrypt with the pending-deletion key.

### Notes

If a key is permanently deleted after the waiting period, any ciphertext and wrapped data keys protected solely by that key become cryptographically unrecoverable. This is cryptographic erasure. It is especially useful in cloud environments, where securely overwriting every physical replica of a file may be impractical.

---

## Task 7 - Integrity Verification and Audit Evidence

Confidentiality does not show whether a file was changed. Integrity controls detect unauthorised modification. SHA-256 creates a fixed-length digest for a file; even one changed character produces a very different digest. A hash chain extends this principle to audit records by including the previous entry's hash in the calculation of the next entry.

### Purpose

- Calculate a baseline SHA-256 hash for the original record.
- Demonstrate that a modified copy produces a different hash.
- Create a simple tamper-evident audit chain.
- Re-verify the RSA signature and inspect KMS key availability.

### Terminal Commands

```sh
sha256sum record.txt
cp record.txt tampered.txt
echo 'x' >> tampered.txt
sha256sum record.txt tampered.txt

PREV=0
for line in 'login ok' 'file read' 'export data'; do
  PREV=$(echo -n "$PREV$line" | sha256sum | cut -d' ' -f1)
  echo "$line | $PREV"
done

aws $EP kms list-keys
openssl dgst -sha256 -verify public.pem -signature record.sig record.txt
```

### Explanation of the Commands

- `sha256sum record.txt` calculates and displays the original record's SHA-256 digest.
- `cp` creates a test copy, and `echo 'x' >> tampered.txt` deliberately modifies it.
- Hashing both files shows different digest values, proving that the tampered copy is not identical to the original.
- `PREV=0` starts the audit hash chain with a known initial value.
- For each audit event, `echo -n "$PREV$line" | sha256sum` hashes the prior chain value together with the current event. A change to an earlier entry changes all later expected hashes.
- `aws $EP kms list-keys` lists the KMS keys available at the local endpoint.
- The final `openssl dgst ... -verify` command checks the digital signature created in Task 2. `Verified OK` confirms the original record and signature still match.

### Evidence and Result

- [Hash comparison and audit chain](<evidence/Task 7.png>) shows distinct SHA-256 values for `record.txt` and the tampered copy, followed by chained hashes for the audit events.
- [Final verification](<evidence/Verification Command.png>) shows the available KMS keys and OpenSSL returning `Verified OK` for the original record's signature.

### Notes

A hash by itself detects a difference but does not identify the authorised source of a file. A digital signature adds authenticity because only the private-key holder can create a signature that validates under the corresponding public key. In production, audit logs should be centrally collected, access-controlled, time-synchronised, retained according to policy, and protected against alteration.

---

## Security Best-Practices Checklist

- [x] Sensitive data was encrypted using AES-256 with PBKDF2 and salt.
- [x] Encryption was verified by decrypting and comparing the recovered file with the original.
- [x] RSA private and public keys were generated for asymmetric cryptography.
- [x] RSA encryption/decryption and SHA-256 digital-signature verification were demonstrated.
- [x] A TLS service protected network transport of the sample record.
- [x] Separate KMS keys were created for tenant A and tenant B.
- [x] Envelope encryption was used for file data instead of direct KMS encryption of the whole file.
- [x] Plaintext data-key files were removed after use.
- [x] A key was placed in `PendingDeletion`, preventing decrypt operations and demonstrating key lifecycle enforcement.
- [x] SHA-256 and a hash-chain process were used to demonstrate data and audit integrity.
- [x] Evidence screenshots were retained for every completed lab task.

## Key Findings

1. **Encryption at rest requires safe key handling.** AES encryption made the patient record unreadable, but the protection depends on keeping the password or encryption key confidential. Leaving plaintext key material on disk would undermine the control.

2. **Asymmetric cryptography solves different problems from AES.** RSA allows a public key to encrypt data for a private-key holder and enables verifiable signatures. It is best used for keys and signatures, while AES is better for file encryption.

3. **TLS protects the communication channel, not only the stored file.** The HTTPS test showed the record could be delivered through an encrypted connection. Certificate validation is essential; bypassing validation is acceptable only for this self-signed local lab certificate.

4. **Envelope encryption is the practical cloud pattern.** KMS protects the data key while fast local AES handles the actual file. This reduces KMS data-size limitations and centralises authorisation for key recovery.

5. **Per-tenant KMS keys limit blast radius.** Separate keys help ensure that lifecycle actions, permissions, and incidents affecting one tenant do not automatically affect another tenant.

6. **Key deletion enables cryptographic erasure.** Once a key is permanently deleted, data encrypted only under that key cannot be recovered. This makes secure disposal practical even for distributed cloud storage.

7. **Integrity needs explicit verification.** Different SHA-256 hashes exposed the changed file, while `Verified OK` confirmed the signed original had not changed.

## Conclusion

This lab successfully demonstrated the complete lifecycle of protecting sensitive cloud data. AES-256 was used to protect the record at rest, RSA was used for asymmetric encryption and signatures, and TLS protected the record while in transit. KMS then provided centralised master-key management, tenant-specific key separation, direct encryption of a small secret, and envelope encryption for file data.

The most important lesson is that cryptography must be supported by disciplined key management. Key generation, access control, temporary plaintext-key handling, auditing, rotation, and deletion are as important as the encryption algorithm itself. The scheduled deletion of tenant A's key and failed unwrap operation demonstrated how KMS lifecycle state actively enforces access restrictions. The integrity test and signature verification complemented confidentiality controls by showing that unauthorised changes can be detected.

These techniques are directly applicable to cloud databases, object storage, backups, SaaS applications, healthcare data, financial records, and any environment where sensitive information must remain confidential, authentic, and verifiably unchanged.

## References

- IKB42603 Lab 3: Encryption and Key Management lab guide.
- OpenSSL documentation: encryption, RSA, X.509 certificates, and message digests.
- AWS Key Management Service (KMS) documentation: customer-managed keys, data keys, envelope encryption, and key deletion.
- NIST SP 800-57, *Recommendation for Key Management*.
- NIST SP 800-38A, *Recommendation for Block Cipher Modes of Operation*.
- NIST SP 800-111, *Guide to Storage Encryption Technologies for End User Devices*.

## End of Report

- **Lab status:** All evidenced tasks completed.
- **Submitted by:** Surya Giri A/L Shanker
- **Course:** IKB42603 Cloud Computing Security Essentials
- **Institution:** UniKL MIIT




