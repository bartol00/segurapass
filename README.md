# SeguraPass
A free-to-use, secure, zero-knowledge, end-to-end encrypted online password manager built in Java

## Table of Contents
- [Introduction](#introduction)
- [Why SeguraPass](#why-segurapass)
- [Architecture](#architecture)

## Introduction
SeguraPass is an online desktop-based password manager built with Java, JavaFX and PostgreSQL, designed with one primary goal:

**Your passwords should belong only to you, and not the server, not the cloud and not the application developer.**

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

This is due to two fundamental properties of SeguraPass:
  
  **1. Zero-knowledge authentication.** The master password never leaves the client device and only a verifier from which the password cannot be reverse-engineered is stored on the server.
  
  **2. Client-side end-to-end encryption.** All the credentials are encrypted before leaving the client device, and they are only decrypted after coming back to the device.

## Architecture

A sophisticated application architecture was devised to satisfy the secure properties of SeguraPass. The base operations of the application that need to be implemented securely are:

- Registering as a new user
- Logging in as an existing user
- Reading credentials
- Writing credentials

The base operations are described below. Note: **N** and **g** are fixed primes used during the SRP process, derived from the **rfc5054_3072** standard

### 1. Registering as a new user

![registration_flow](diagrams/register.png)

For both registration and login, SeguraPass uses SRP-6, a proven password-authenticated key exchange protocol that allows secure registration/login without ever sending the password to the server. 

The registration flow is the following:

1. The user specifies an email address and a master password
2. The client generates the following:
   1. A random **authorization salt** that will be used in future authentication flows
   2. Another random **key salt** that will be used with the master password to make the credential encryption key
   3. A verifier **v = g^x mod N**, where **x = H(saltAuth | masterPassword)**, from which the master password cannot be reverse-engineered. 
   4. A randomly generated **device ID** (UUID) that will identify the particular device to the server, for multi-device access purposes
3. The client sends only **authorization salt, key salt, v** and **device ID** to the server, making master password recovery by third parties impossible
4. The user has now successfully registered to the SeguraPass service

The master password **never** leaves the device, as the server only stores the email address, salts, device ID and verifier. This is sufficient for the user later logging in successfully, while attackers cannot reverse engineer master credentials from the verifier or perform offline attacks, as the verifier itself never leaves the server.

Before the user can use the service, they must verify their account by clicking on a randomly generated verification link that will be sent to their email address.

### 2. Logging in as an existing user

![login_flow](diagrams/login.png)

During login, the master password again never leaves the device. The authentication of the user is performed using proof messages and public key parameters, in a way that sensitive data on both the client and server side are never exposed to the public. 

The login flow is the following:

1. The client randomly generates a private integer **a** and a public integer **A**, where **A = g^a mod N**. The public component **A** along with the user's **email** and **device ID** are sent to the server
2. The server randomly generates a private integer **b** and a public integer **B**, where **B = ( k * v  +  g^b mod N ) mod N** and **k = H( pad(N) || pad(g) )**. The server ephemerally stores these values tied to the user's account for a short period and returns the public value **B** to the client as a response
3. The client uses its own private value **a** with their master password and the public values **A** and **B** to generate a proof message **M1** that will be sent to the server
4. The server uses its own private value **b** with the user's stored verifier and the public values **A** and **B** to generate its own proof message **M1** that must be the same as the one generated and sent by the client. If the two are the same, the server generates a short-lived JWT, a refresh token and a new proof message **M2** so the client can also be sure the server is legitimate. These are then returned to the client. If the proof messages **M1** were not the same, the SRP verification has failed and the user has not been authenticated
5. After the client receives the tokens and the server's proof message **M2**, it generates its own **M2** that must be the same as the one received for the client to be sure about the server's identity. If the two match, the client accepts the tokens and uses those as authorization methods when communicating with the server

