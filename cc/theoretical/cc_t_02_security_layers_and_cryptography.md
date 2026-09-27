# CCT02 - Security in the Network Layers and Cryptography

## Topics

1. [Key Concepts](#1-key-concepts)
2. [Terminology](#2-terminology)
3. [Security in the Network Layers](#3-security-in-the-network-layers)
4. [Introduction to Cryptography](#4-introduction-to-cryptography)

## 1. Key Concepts

Network communication is organized in layers (the OSI and TCP/IP models), and security can be added at any of them: at the application layer (e.g. HTTPS), the transport layer (TLS), the internet layer (IPsec, VPNs) or the network access layer (e.g. 802.1X). Each layer protects something different, so in principle all of them should be secured, but which ones are actually needed depends on the use case. Using Instagram only needs HTTPS over TLS, while an organization may also need a VPN so its members can reach internal resources from outside.

Almost all of these solutions are built on cryptography, the science of securing communication against an adversary. Its two basic building blocks are symmetric ciphers, where both sides share the same secret key, and asymmetric (public key) ciphers, where each user has a public key and a private key. Real systems combine both: public key cryptography to agree on a secret key, and that secret key to encrypt the actual data.

Cryptography is old: Caesar's cipher was used over 2000 years ago, and each cipher that got broken taught a lesson that still applies, such as the need for a large key space. The most widely used protocol built on it today is TLS, which secures the connection for most applications on the Internet, including HTTPS.

## 2. Terminology

**Cryptology** is the whole field of secret communication. It has two sides: cryptography and cryptanalysis.

**Cryptography** is the science of securing communication against an adversary.

**Cryptanalysis** is the science (or art) of breaking cryptographic systems.

The **plaintext** (m) is the original, readable message. The **ciphertext** (c) is the encrypted, unreadable version of it.

A **key** (k) is the secret value that controls how a message is encrypted and decrypted. The same algorithm with a different key gives a different ciphertext.

**Encryption** (E) turns plaintext into ciphertext using a key. **Decryption** (D) turns the ciphertext back into plaintext.

A **cipher** is the pair of algorithms used to encrypt and decrypt.

A **symmetric cipher** uses the same secret key to encrypt and decrypt, so both sides need to share it.

An **asymmetric cipher** (or **public key cipher**) uses a pair of keys: a public key that anyone can know and a private key that only its owner knows. What one key encrypts, only the other can decrypt.

The **key space** is the set of all possible keys for a cipher. The bigger it is, the harder it is to try them all.

A **brute-force attack** tries every possible key until one works.

A **message authentication code (MAC)** is a short value computed from a message and a shared secret key. The receiver recomputes it to check that the message wasn't changed and came from someone with the key.

A **digital signature** is the public key equivalent of a MAC: it's created with the signer's private key and checked with their public key, so anyone can verify it, and only the owner of the private key could have made it.

A **protocol**, in cryptography, combines encryption algorithms to perform more complex security functions, e.g. TLS.

**IPsec** (IP Security) is a set of protocols that secures traffic at the internet layer, by authenticating and encrypting IP packets.

A **VPN** (Virtual Private Network) is a secure tunnel over a public network that makes a remote device behave as if it were directly connected to a private network, e.g. accessing DEI's internal resources from home.

## 3. Security in the Network Layers

### OSI and TCP/IP Models

The OSI model splits network communication into 7 layers. The TCP/IP model, which is what the Internet actually uses, merges them into 4:

| OSI layer | Description | TCP/IP layer |
| --------- | ----------- | ------------ |
| 7 - Application | Services, end-user applications | 4 - Application |
| 6 - Presentation | Data formats, encryption, compression | 4 - Application |
| 5 - Session | Handles sessions | 4 - Application |
| 4 - Transport | Reliable data transport | 3 - Transport (TCP, UDP) |
| 3 - Network | Path for data, logical addressing, routing | 2 - Internet (IP) |
| 2 - Data Link | Physical addressing (MAC), error detection | 1 - Network access |
| 1 - Physical | Transmission over the medium (bitstreams) | 1 - Network access |

### Security at Each Layer

Each TCP/IP layer has its own protocols and its own security solutions:

| TCP/IP layer | Protocols | Security solutions |
| ------------ | --------- | ------------------ |
| 4 - Application | HTTP, FTP, SMTP, POP, DNS, DHCP, SSH, SNMP | HTTPS, S/MIME, DNSSEC, ... |
| 3 - Transport | TCP, UDP, SCTP, QUIC | TLS |
| 2 - Internet | IPv4, IPv6, ARP, ICMP | IPsec, VPNs |
| 1 - Network access | Ethernet, Wi-Fi, PPP, 5G NR | Link-level mechanisms, e.g. 802.1X |

Each layer protects something different:

- **Application layer:** protects one specific service (e.g. HTTPS only protects web traffic, S/MIME only email).
- **Transport layer:** TLS gives a common layer of protection that any application can use.
- **Internet layer:** adds an extra layer of protection that also covers the IP information, e.g. hiding the real addresses inside a VPN tunnel.
- **Network access layer:** controls who can connect to the network in the first place.

### Which Layer to Secure?

All of them can be secured, but not every use case needs all of them at once:

- **Instagram:** uses HTTPS, which relies on TLS underneath. The network access layer may also be secured (e.g. a protected Wi-Fi network), but a VPN isn't needed: Instagram is a public service, and TLS already protects the connection to it end to end.
- **An organization:** may use HTTPS over TLS for its services, and also a **VPN** so members can reach internal resources from outside. For example, with the DEI VPN you can access the same resources from home as if you were physically at DEI.

## 4. Introduction to Cryptography

Almost all the security solutions in the previous section (HTTPS, TLS, IPsec, ...) are built on cryptography. There are two basic types of encryption, symmetric and asymmetric, and real systems use both together.

### The Field of Cryptology

Cryptology splits into two sides: building secure systems (cryptography) and breaking them (cryptanalysis). This course focuses on the cryptography side.

```
                               Cryptology
                  ┌────────────────┴────────────────┐
             Cryptography                      Cryptanalysis
      ┌───────────┼───────────┐         ┌───────────┼───────────────┐
  Symmetric   Asymmetric  Protocols  Classical  Implementation   Social
   ciphers     ciphers              cryptanalysis   attacks    engineering
                                   ┌──────┴──────┐
                              Mathematical   Brute-force
                                analysis       attacks
```

- **Symmetric ciphers:** both sides use the same secret key.
- **Asymmetric ciphers:** each user has two keys, a public one and a private one.
- **Protocols:** combine ciphers to do more complex security functions, e.g. TLS.

On the cryptanalysis side, classical cryptanalysis attacks the algorithm itself, either with maths or by trying every key (brute force). Implementation attacks go after how the algorithm is run instead (e.g. measuring timing or power use), and social engineering goes after the people, e.g. tricking someone into giving away the key.

### Notation

The usual example is **Alice** sending a message to **Bob** while **Eve** (an eavesdropper) listens on the channel. Without protection, Eve reads the message just like Bob does, so confidentiality is lost. With encryption, Eve still sees what's sent, but it's ciphertext she can't read without the key.

| Symbol | Meaning |
| ------ | ------- |
| m | Plaintext message |
| Gen | Key generation algorithm |
| k | Key |
| E | Encryption algorithm |
| c | Ciphertext (encrypted message) |
| D | Decryption algorithm |

```
c = E(k, m)          encrypting m with k gives c
m = D(k, c)          decrypting c with k gives m back
D(k, E(k, m)) = m    decryption undoes encryption (correctness)
```

### Symmetric Encryption

Alice and Bob share the same secret key k:

```
                       Eve
                        c
                        ↑
 Alice ─────────────────┴─────────────────→ Bob
 c = E(k, m)            c                   m = D(k, c)
```

Symmetric encryption is fast, but it has a key distribution problem: before they can talk securely, Alice and Bob need to agree on k through some secure channel, and every pair of users needs its own key.

### Asymmetric Encryption

Asymmetric encryption is also called public key encryption. Each user has a key pair:

- A **public key** (P), which anyone can know.
- A **private key** (S), which only its owner knows. The slides call it the secret key, which is where the S comes from.

What's encrypted with one key of the pair can only be decrypted with the other. To send Bob a message, Alice encrypts it with Bob's public key, and only Bob can decrypt it with his private key:

```
 Alice ────────────────── c ──────────────→ Bob
 c = E(P_Bob, m)                            m = D(S_Bob, c)
```

This simplifies key distribution: only the public key has to be shared, and it doesn't need to be kept secret.

The downside is that asymmetric encryption is much slower than symmetric encryption. It's based on mathematical operations on very large numbers (e.g. modular exponentiation with 2048-bit numbers in RSA), while symmetric ciphers use simple, fast operations on small blocks of bits, often with dedicated hardware support (e.g. AES instructions in CPUs).

### Hybrid Encryption

Practical systems combine both types:

1. **Asymmetric encryption** is used to establish and exchange a secret key.
2. That **secret key** is then used to encrypt the actual data with a symmetric cipher.

The slow asymmetric operations only happen once, at the start, and all the data is encrypted with the fast symmetric cipher. TLS works this way.

### Digital Signatures

A digital signature is the asymmetric equivalent of a message authentication code (MAC). Instead of encrypting, Alice signs the message with her private key, and anyone can verify it with her public key:

```
 Alice ──────────────── m, s ──────────────→ Bob
 s = σ(S_Alice, m)                           v(P_Alice, m, s) → valid?
```

- σ is the signing algorithm, which uses Alice's private key (S_Alice).
- v is the verification algorithm, which uses Alice's public key (P_Alice) and says whether s is a valid signature on m.

With a MAC, both sides share the same key, so the receiver could have created the MAC too. With a signature, only Alice has her private key, so a valid signature proves she sent the message. This gives integrity, authentication and non-repudiation, but not confidentiality: the message m is sent in the clear.
