# SeguraPass
A free-to-use, secure, zero-knowledge, end-to-end encrypted online password manager built in Java

## Table of Contents
- [Introduction](#introduction)
- [Why SeguraPass](#why-segurapass)

## Introduction
SeguraPass is an online desktop-based password manager built with Java, JavaFX and PostgreSQL, designed with one primary goal:
Your passwords should belong only to you, and not the server, not the cloud and not the application developer.

Unlike traditional password managers, SeguraPass follows a zero-knowledge security model:

- The server never sees any of your passwords
- The server never stores a hash of any of your passwords
- The server never stores a master password derivative
- The server never receives unencrypted credentials
- All sensitive data is encrypted before leaving your device
- All sensitive data is decrypted only on your device

Authentication is handled using Secure Remote Password (SRP-6a), a cryptographic protocol that ensures:

- The master password is never transmitted
- The server cannot derive the master password in any way
- The client and server mutually authenticate
- The login flow resists MITM attacks
- Even if the database is compromised, login secrets remain unusable

Credential storage is protected with AES-256-GCM using keys derived from your master password and a per-user salt. Only the encrypted blobs ever reach the backend.

SeguraPass therefore includes:

- Strong cryptography
- Zero-knowledge architecture
- End-to-end encryption
- No cloud trust required
- No telemetry or data collection

## Why SeguraPass
Most password managers currently out fall into one of two categories:

### 1. Cloud-first password managers
- They receive your data unencrypted, encrypting it on the server instead of the client
- They control how your key is derived
- They often send telemetry
- They often rely on storing hashes of master passwords for user authentication
- Many use PBKDF2 or other outdated KDFs
- They rely heavily on security-by-policy rather than security-by-design

These systems can technically function, but users ultimately have to trust:

- The company’s servers
- The implementation of its crypto
- That no one has added or will add a backdoor
- That breaches won’t expose sensitive data
- That the master password hash is adequately protected

Any system that features trust as a necessary component is a fundamentally insecure one, which history has repeatedly proven.

### 2. Local password managers

These avoid cloud trust but introduce other problems:

- No syncing
- Local-only backups
- Manual export/import
- High risk of data loss
- No multi-device access
- Easy to corrupt or lose vaults

When choosing the type of password manager to use, clients are constantly forced to compromise between the convenience and insecurity of typical cloud managers, and the security but lack of reliability of local ones. 

SeguraPass is built to provide the best of both worlds without compromising on security. It was built with a very specific goal in mind: 

**Even in the event of compromise, backdooring, malicious activity or leaking of data on the server, the client must remain secure.**

