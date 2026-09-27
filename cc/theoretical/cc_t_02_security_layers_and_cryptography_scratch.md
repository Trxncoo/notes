# CC T02 notes: plan (scratch)

Based on the slides `CC-02-Security-OSI-Encryption-history (1).pdf` (Topic 2 – Security in OSI Layers & Historical perspective of cryptography, 21/09/2026).

## Planned structure

Same layout as T01: Key Concepts and Terminology first, then one section per part of the lecture.

1. **Key Concepts** (slides 4 to 10, 13 to 14)
   - Security can be added at every network layer, and which layers to secure depends on the use case
   - Cryptography is the main tool behind that security, and TLS is the protocol most applications rely on
2. **Terminology** (slides 13 to 21)
   - Cryptology, cryptography, cryptanalysis
   - Plaintext, ciphertext, key, cipher, encryption and decryption
   - Symmetric and asymmetric (public key) ciphers, key space, brute-force attack
   - Message authentication code (MAC), digital signature
   - VPN, IPsec
3. **Security in the Network Layers** (slides 4 to 10, 30)
   - OSI vs TCP/IP models (table of both, side by side)
   - Protocols and security solution per TCP/IP layer: HTTPS, TLS, IPsec/VPN, 802.1X
   - Which layer to secure: all of them can be, but it depends on the use case
   - Instagram (HTTPS over TLS) vs an organization (adds a VPN, e.g. the DEI VPN), and whether a VPN makes sense for Instagram
   - What each layer protects: application = one service, transport = all applications, network = also hides IP information
4. **Introduction to Cryptography** (slides 13 to 21)
   - Cryptology tree: cryptography (symmetric, asymmetric, protocols) vs cryptanalysis (classical, implementation attacks, social engineering)
   - Notation: m, k, Gen, E, D, c, and `D(k, E(k, m)) = m`
   - Symmetric encryption (Alice and Bob sharing a key)
   - Public key encryption: key pair, simpler key distribution, but much slower
   - Hybrid encryption: public key to exchange a secret key, secret key for the data
   - Digital signatures as the public key equivalent of MACs
5. **History of Cryptography** (slides 24 to 26)
   - Caesar cipher (shift by 3, no key)
   - Shift cipher (key = shift, only 26 keys, brute force) → key space must be large
   - Monoalphabetic substitution (26! keys, but broken by letter frequencies)
   - Enigma (rotors, daily key, broken by the Allies with Turing's help)
6. **TLS** (slides 29 to 41)
   - Where TLS sits and why it's a common layer of protection for all applications
   - Architecture: Record Protocol + Handshake, Change Cipher Spec and Alert protocols
   - Connection vs session
   - Session state and connection state (tables)
   - Record Protocol services: confidentiality and message integrity
   - TLS 1.3 handshake (ClientHello, ServerHello, authentication) and how it differs from older versions
   - Versions: 1.3 recommended, 1.2 acceptable, 1.1 and older deprecated
   - Example: `openssl s_client` connection to instagram.com

## Extra sources (only if needed)

- TLS 1.3: RFC 8446
- TLS 1.2: RFC 5246 (the session/connection state and Change Cipher Spec protocol in the slides come from here)
- Deprecation of TLS 1.0 and 1.1: RFC 8996
- IPsec architecture: RFC 4301

## Things in the slides to double check

- **Asymmetric ciphers** (slide 14): "based on the same public key" is wrong. They use two different keys, a public one and a private one (which the same bullet then says).
- **Shift cipher** (slide 24): the key range "0<k<25" should be 0 to 25 (26 keys, which the slide then says). "> 270" and "288" are the PDF losing superscripts: they mean 2^70 and 2^88 (26! ≈ 2^88).
- **802.11X** (slide 5) should be **802.1X** (port-based network access control, used for WPA-Enterprise and wired networks).
- **Enigma** (slide 26): the Allies used electromechanical machines (the Bombes), not electronic computers. The electronic Colossus was built to break a different German cipher (Lorenz).
- **TLS versions mixed** (slides 31 to 35 vs 37 to 39): the architecture, session state (compression, 48-byte master secret) and connection state (MAC secrets, CBC IVs) describe TLS 1.2. TLS 1.3 removed compression, CBC mode, separate MAC keys and the real Change Cipher Spec protocol (it's only sent for compatibility, which is why it shows up in the openssl output). The handshake slides are TLS 1.3.
- **TLS 1.0** is also deprecated, not just 1.1 (both since 2021).
- **Enigma** (slide 26): it wasn't only for U-boats. All branches of the German military used it, and it started as a commercial machine sold in the 1920s. It was also electromechanical (electrical wiring plus moving rotors), not electronic.

Add `6. [TLS](#6-tls)` to the Topics list.

Terminology candidates for section 2, since this section uses them:

- **AEAD** (Authenticated Encryption with Associated Data) is a type of cipher that encrypts and protects integrity in a single operation, so no separate MAC is needed. TLS 1.3 only allows AEAD ciphers.
- **Forward secrecy** means that if a long-term private key is stolen later, past sessions still can't be decrypted, because each session's keys came from temporary (ephemeral) Diffie-Hellman keys that were thrown away.
- A **cipher suite** is the set of algorithms two sides agree to use in a TLS connection.
- A **pre-shared key (PSK)** is a secret key both sides already have before the handshake, either configured by hand or saved from a previous connection.

## 6. TLS

Transport Layer Security (TLS) gives two applications a secure channel over a reliable, in-order transport such as TCP. It's the protocol under HTTPS, and QUIC has it built in. Its goals are:

- **Authentication:** the server is always authenticated, and the client optionally. This is done with asymmetric cryptography (certificates and signatures) or with a pre-shared key.
- **Confidentiality:** after the handshake, only the two endpoints can see the data. TLS doesn't hide how long the data is, but records can be padded to make that harder to see.
- **Integrity:** after the handshake, attackers can't change the data without it being detected.

These should hold even against an attacker who controls the whole network. The current version is TLS 1.3, defined in RFC 8446. The slides mix TLS 1.2 and 1.3, so where the two differ, these notes say which one they're describing.

### Architecture

TLS isn't a single protocol, but two layers of protocols on top of TCP:

```
+-----------+---------------+-------+------+-----------+
| Handshake | Change Cipher | Alert | HTTP | Heartbeat |
| Protocol  | Spec Protocol | Prot. |      | Protocol  |
+-----------+---------------+-------+------+-----------+
|                   Record Protocol                    |
+------------------------------------------------------+
|                         TCP                          |
+------------------------------------------------------+
|                          IP                          |
+------------------------------------------------------+
```

- **Record Protocol:** the lower layer. It gives basic security services (confidentiality and integrity) to everything above it, in particular HTTP.
- **Handshake Protocol:** authenticates the two sides, negotiates the algorithms and establishes the shared keys.
- **Change Cipher Spec Protocol:** in TLS 1.2, a single message that tells the other side to start using the keys that were just negotiated. TLS 1.3 doesn't need it anymore and only sends a dummy one so old middleboxes (firewalls, proxies) don't break the connection.
- **Alert Protocol:** reports errors and closes connections.
- **Heartbeat Protocol:** an optional extension (RFC 6520) that checks the other side is still alive.

The application data itself (e.g. HTTP) also goes directly over the Record Protocol.

### Connections and Sessions

TLS separates the connection (the actual transport between two peers) from the session (the security parameters they agreed on):

|              | TLS connection                                                                       | TLS session                                      |
| ------------ | ------------------------------------------------------------------------------------ | ------------------------------------------------ |
| What it is   | A transport that provides a suitable type of service, as a peer-to-peer relationship | An association between a client and a server     |
| Lifetime     | Transient                                                                            | Can outlive a connection                         |
| Created by   | The transport (e.g. TCP)                                                             | The Handshake Protocol                           |
| Relationship | Every connection is associated with one session                                      | One session can be shared by several connections |

A session defines a set of cryptographic security parameters. Reusing it for new connections avoids repeating the expensive negotiation of new parameters every time.

In TLS 1.3, resuming a session works differently: after the handshake, the server sends a `NewSessionTicket`, and the client uses it as a pre-shared key in its next handshake (see [Resumption and 0-RTT](#resumption-and-0-rtt)). The session IDs used by TLS 1.2 are obsolete.

### Session and Connection State (TLS 1.2)

Each side keeps two sets of state. These tables describe TLS 1.2. Several fields no longer exist in TLS 1.3 (see the notes after each table).

**Session state:**

| Field              | Description                                                                                                       |
| ------------------ | ----------------------------------------------------------------------------------------------------------------- |
| Session identifier | An arbitrary byte sequence chosen by the server to identify an active or resumable session                        |
| Peer certificate   | An X.509v3 certificate of the peer. May be null                                                                   |
| Compression method | The algorithm used to compress data before encryption                                                             |
| Cipher spec        | The bulk data encryption algorithm and the hash algorithm used for the MAC, plus attributes such as the hash size |
| Master secret      | A 48-byte secret shared between client and server                                                                 |
| Is resumable       | A flag saying whether the session can be used to start new connections                                            |

TLS 1.3 removed compression (it made attacks such as CRIME possible), and replaced the master secret with a key schedule that derives separate secrets for each stage of the connection.

**Connection state:**

| Field                    | Description                                                                                                                                                               |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Server and client random | Byte sequences chosen by the server and the client for each connection                                                                                                    |
| Server write MAC secret  | The secret key used in MAC operations on data sent by the server                                                                                                          |
| Client write MAC secret  | The secret key used in MAC operations on data sent by the client                                                                                                          |
| Server write key         | The symmetric key for data encrypted by the server and decrypted by the client                                                                                            |
| Client write key         | The symmetric key for data encrypted by the client and decrypted by the server                                                                                            |
| Initialization vectors   | When a block cipher in CBC mode is used, one IV per key. The handshake sets the first one, and after that the last ciphertext block of each record is the IV for the next |
| Sequence numbers         | Each side keeps separate sequence numbers for sent and received messages. They reset to zero on a Change Cipher Spec and can't go above 2^64 - 1                          |

TLS 1.3 keeps the randoms, the write keys and the 64-bit sequence numbers. The MAC secrets are gone, because AEAD ciphers handle integrity themselves, and CBC mode is gone too. Instead of a chained IV, each record's nonce is made from a per-connection write IV and the sequence number.

### Record Protocol

The Record Protocol gives TLS connections two services, both using keys set up by the Handshake Protocol:

- **Confidentiality:** a shared secret key is used for symmetric encryption of the payloads.
- **Message integrity:** a shared secret key is used to form a message authentication code (MAC). In TLS 1.3, the AEAD cipher does this as part of encrypting.

To send data, it splits it into fragments, protects each one and adds a header. In TLS 1.2 (as in the slides):

```
Application data  ──→  Fragment  ──→  Compress  ──→  Add MAC  ──→  Encrypt  ──→  Add record header
```

In TLS 1.3, compression and the separate MAC are gone:

```
Application data  ──→  Fragment  ──→  Add content type + padding  ──→  AEAD encrypt  ──→  Add record header
```

The receiver does the reverse: it checks and decrypts each record, puts the fragments back together and passes the data up.

### Record Format

Every record starts with a 5-byte header (sizes in bytes):

```
+----------+------------------------------+--------------+
| type (1) |  legacy_record_version (2)   |  length (2)  |
+----------+------------------------------+--------------+
|                fragment (length bytes)                 |
+--------------------------------------------------------+
```

| Field                 | Bytes | Description                                                                                                      |
| --------------------- | ----- | ---------------------------------------------------------------------------------------------------------------- |
| type                  | 1     | Which protocol the fragment belongs to (see the table below)                                                     |
| legacy_record_version | 2     | Ignored in TLS 1.3. Set to `0x0303` (TLS 1.2), or `0x0301` (TLS 1.0) in the first ClientHello, for compatibility |
| length                | 2     | Length of the fragment. At most 2^14 bytes (16 KB), or 2^14 + 256 for an encrypted record                        |
| fragment              | var   | The data being carried                                                                                           |

| Content type       | Value | Carries                                                            |
| ------------------ | ----- | ------------------------------------------------------------------ |
| change_cipher_spec | 20    | The Change Cipher Spec message (only for compatibility in TLS 1.3) |
| alert              | 21    | Alert messages                                                     |
| handshake          | 22    | Handshake messages                                                 |
| application_data   | 23    | Application data, and in TLS 1.3, every encrypted record           |

Once encryption starts in TLS 1.3, the header's type is always 23 (application_data), whatever the record really carries. The real content type is encrypted with the data, followed by any zero-byte padding:

```
+------------------------+----------+------------------+
|  content (var)         | type (1) | zeros (padding)  |   ← this whole part is AEAD encrypted
+------------------------+----------+------------------+
```

That way an observer can't even tell whether a record is a handshake message, an alert or application data. The header is still used as additional data in the AEAD, so changing it makes decryption fail. A record that fails to decrypt ends the connection with a `bad_record_mac` alert.

### Handshake Protocol

The Handshake Protocol negotiates the security parameters of a connection:

- **Key exchange:** find out what both sides support and establish the shared secrets.
- **Negotiation of extensions.**
- **Server parameters:** share the rest of the server's settings.
- **Authentication:** prove who the server (and optionally the client) is.

Each handshake message has a small header (sizes in bytes), and it's carried inside records of type 22:

```
+--------------+-------------------------------+
| msg_type (1) |          length (3)           |
+--------------+-------------------------------+
|          message body (length bytes)         |
+----------------------------------------------+
```

### Message Types

| Type                | Value | Sent by | Use                                                                                              |
| ------------------- | ----- | ------- | ------------------------------------------------------------------------------------------------ |
| ClientHello         | 1     | Client  | Starts the handshake: supported versions, cipher suites, key shares and extensions               |
| ServerHello         | 2     | Server  | The chosen version and cipher suite, and the server's key share                                  |
| NewSessionTicket    | 4     | Server  | After the handshake, gives the client a ticket to resume the session later                       |
| EndOfEarlyData      | 5     | Client  | Marks the end of the 0-RTT data                                                                  |
| EncryptedExtensions | 8     | Server  | Answers to the client's extensions that aren't needed to set up the keys                         |
| Certificate         | 11    | Both    | The sender's certificate chain                                                                   |
| CertificateRequest  | 13    | Server  | Asks the client to authenticate with a certificate                                               |
| CertificateVerify   | 15    | Both    | A signature over the whole handshake so far, made with the private key of the certificate        |
| Finished            | 20    | Both    | A MAC over the whole handshake, confirming both sides have the same keys and nothing was changed |
| KeyUpdate           | 24    | Both    | Switches to new traffic keys in the middle of a connection                                       |

The HelloRetryRequest (see [Full Handshake](#full-handshake-tls-13)) isn't a separate type: it's a ServerHello with a special value in its random field. Messages must arrive in the right order, or the handshake is aborted with an `unexpected_message` alert.

### Full Handshake (TLS 1.3)

TLS 1.3 does the handshake in one round trip (1-RTT), in three phases:

```
  Client                                           Server
     |                                                |
     |--- ClientHello + key_share ------------------->|   1. Key exchange
     |                                                |
     |<------------------- ServerHello + key_share ---|   1. Key exchange
     |<------------------- {EncryptedExtensions} -----|   2. Server parameters
     |<------------------- {CertificateRequest*} -----|
     |<------------------- {Certificate*} ------------|   3. Authentication
     |<------------------- {CertificateVerify*} ------|
     |<------------------- {Finished} ----------------|
     |                                                |
     |--- {Certificate*} ---------------------------->|   3. Authentication
     |--- {CertificateVerify*} ---------------------->|
     |--- {Finished} -------------------------------->|
     |                                                |
     |<========== [Application Data] ================>|

 *  optional or depends on the situation
 {} encrypted with the handshake keys
 [] encrypted with the application keys
```

1. **Key exchange:** the client sends a ClientHello with a random nonce, the versions and cipher suites it supports, and one or more Diffie-Hellman key shares (in the `key_share` extension). It guesses which group the server will pick, so it can send its share right away. The server picks the parameters and answers with a ServerHello containing its own key share. With both shares, each side can now compute the same shared secret, and **everything after the ServerHello is encrypted**.
2. **Server parameters:** the server sends EncryptedExtensions, with the answers to the client's other extensions (e.g. which application protocol to use). If it wants the client to authenticate too, it also sends a CertificateRequest.
3. **Authentication:** the server sends its Certificate, a CertificateVerify (a signature over the handshake with the certificate's private key, which proves it owns the certificate) and a Finished (a MAC over the whole handshake). The client checks all three, then sends its own Finished, plus a Certificate and CertificateVerify if they were requested. Only after the Finished messages is the application data sent with the final keys.

If the client didn't send a key share in a group the server accepts, the server answers with a HelloRetryRequest naming the group it wants, and the client sends a new ClientHello with the right share. That costs an extra round trip. If no common parameters exist at all, the server aborts with an alert.

### Resumption and 0-RTT

After a handshake, the server can send a NewSessionTicket. In the next connection, the client offers it as a pre-shared key (in the `pre_shared_key` extension), and the handshake skips the Certificate and CertificateVerify messages, since the PSK already authenticates the server. The client should also send a key share, so the new connection still gets fresh keys (forward secrecy).

With a PSK, the client can even send application data in its very first message, before the handshake finishes. This is **0-RTT** or early data. It's faster, but weaker: the early data isn't forward secret, and an attacker can replay it to the server. It should only be used for requests that are safe to repeat.

### Handshake in Older Versions (TLS 1.2)

The TLS 1.2 handshake has four phases and takes two round trips:

```
  Client                                           Server
     |                                                |
     |--- ClientHello ------------------------------->|   1. Security capabilities
     |<------------------------------- ServerHello ---|
     |                                                |
     |<------------------------------- Certificate* --|   2. Server authentication
     |<------------------------- ServerKeyExchange* --|      and key exchange
     |<------------------------ CertificateRequest* --|
     |<--------------------------- ServerHelloDone ---|
     |                                                |
     |--- Certificate* ------------------------------>|   3. Client authentication
     |--- ClientKeyExchange ------------------------->|      and key exchange
     |--- CertificateVerify* ------------------------>|
     |                                                |
     |--- ChangeCipherSpec -------------------------->|   4. Finish
     |--- Finished ---------------------------------->|
     |<-------------------------- ChangeCipherSpec ---|
     |<---------------------------------- Finished ---|
```

The main differences from TLS 1.3:

|                                     | TLS 1.2                                                                                                   | TLS 1.3                                        |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| Round trips before application data | 2                                                                                                         | 1 (0 with early data)                          |
| Handshake encryption                | Everything up to Finished is sent in the clear, including the certificates                                | Everything after ServerHello is encrypted      |
| Algorithms                          | Includes legacy ones, e.g. static RSA key exchange, CBC mode, RC4, and even suites without authentication | Only AEAD ciphers and ephemeral Diffie-Hellman |
| Forward secrecy                     | Optional (not with static RSA)                                                                            | Always, except for 0-RTT data                  |
| ChangeCipherSpec                    | A real protocol message                                                                                   | Only a dummy, for middlebox compatibility      |

### Cipher Suites

In TLS 1.3, a cipher suite only names the AEAD cipher for the records and the hash used to derive the keys, in the format `TLS_AEAD_HASH`:

| Cipher suite                 | Value  |
| ---------------------------- | ------ |
| TLS_AES_128_GCM_SHA256       | 0x1301 |
| TLS_AES_256_GCM_SHA384       | 0x1302 |
| TLS_CHACHA20_POLY1305_SHA256 | 0x1303 |
| TLS_AES_128_CCM_SHA256       | 0x1304 |
| TLS_AES_128_CCM_8_SHA256     | 0x1305 |

The key exchange group (e.g. X25519) and the signature algorithm (e.g. ECDSA, RSA-PSS) are negotiated separately, through extensions. In TLS 1.2, one long cipher suite name covered all of them, e.g. `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`. TLS 1.2 and TLS 1.3 suites can't be mixed.

For example, the browser's security panel for Instagram shows a connection using **QUIC**, **X25519MLKEM768** and **AES_128_GCM**. The key exchange, X25519MLKEM768, combines classic elliptic curve Diffie-Hellman (X25519) with ML-KEM, a quantum-safe algorithm. The records are then encrypted with AES-128 in GCM mode.

### Alert Protocol

Alerts are sent as records of type 21 and, like everything else after the handshake, are encrypted. Each alert has two bytes:

```
+-----------+-----------------+
| level (1) | description (1) |
+-----------+-----------------+
```

The level was `warning` (1) or `fatal` (2). In TLS 1.3 it's ignored, because the description already says what kind of alert it is. There are two classes:

- **Closure alerts:** `close_notify` says the sender won't send anything else on this connection. Each side must send it before closing its side of the connection. Otherwise the receiver can't tell a normal close from an attacker cutting the connection early (a truncation attack).
- **Error alerts:** all of them are fatal. Both sides close the connection immediately and forget the keys.

Some common ones:

| Alert               | Value | Meaning                                                    |
| ------------------- | ----- | ---------------------------------------------------------- |
| close_notify        | 0     | Normal end of the connection                               |
| unexpected_message  | 10    | A message arrived in the wrong place                       |
| bad_record_mac      | 20    | A record failed to decrypt or verify                       |
| handshake_failure   | 40    | No acceptable set of parameters could be negotiated        |
| bad_certificate     | 42    | The certificate was corrupt or its signature didn't verify |
| certificate_expired | 45    | The certificate has expired                                |
| unknown_ca          | 48    | The certificate chain couldn't be traced to a trusted CA   |
| decrypt_error       | 51    | A signature or Finished message didn't verify              |
| protocol_version    | 70    | The peer's version isn't supported                         |

### TLS Versions

TLS started as SSL, made by Netscape. The version numbers on the wire still continue SSL's: SSL 3.0 is `0x0300`, and TLS 1.0 is `0x0301`.

| Version      | Year       | RFC                | Wire value     | Status                                                               |
| ------------ | ---------- | ------------------ | -------------- | -------------------------------------------------------------------- |
| SSL 2.0, 3.0 | 1995, 1996 | (RFC 6101 for 3.0) | 0x0002, 0x0300 | Prohibited                                                           |
| TLS 1.0      | 1999       | RFC 2246           | 0x0301         | Deprecated (RFC 8996)                                                |
| TLS 1.1      | 2006       | RFC 4346           | 0x0302         | Deprecated (RFC 8996), uses algorithms that aren't safe              |
| TLS 1.2      | 2008       | RFC 5246           | 0x0303         | Acceptable, but moving to 1.3 is better for performance and security |
| TLS 1.3      | 2018       | RFC 8446           | 0x0304         | Current, the one that should be used                                 |

Many middleboxes broke when they saw an unknown version number, so TLS 1.3 pretends to be TLS 1.2 in the old version fields (`0x0303`) and puts the real versions in the `supported_versions` extension.

### Example: Instagram with OpenSSL

`openssl s_client` opens a TLS connection and `-msg` prints every message:

```bash
openssl s_client -connect instagram.com:443 -msg
```

The output (trimmed, `>>>` is sent by the client and `<<<` by Instagram):

```
>>> TLS 1.0, RecordHeader [length 0005]
>>> TLS 1.3, Handshake [length 0605], ClientHello
<<< TLS 1.2, RecordHeader [length 0005]
<<< TLS 1.3, Handshake [length 04ba], ServerHello
<<< TLS 1.2, RecordHeader [length 0005]
<<< TLS 1.3, ChangeCipherSpec [length 0001]
<<< TLS 1.2, RecordHeader [length 0005]
<<< TLS 1.3, InnerContent [length 0001]
<<< TLS 1.3, Handshake [length 0006], EncryptedExtensions
<<< TLS 1.3, Handshake [length 0b48], Certificate
<<< TLS 1.3, Handshake [length 004e], CertificateVerify
<<< TLS 1.3, Handshake [length 0024], Finished
>>> TLS 1.2, RecordHeader [length 0005]
>>> TLS 1.3, ChangeCipherSpec [length 0001]
>>> TLS 1.2, RecordHeader [length 0005]
>>> TLS 1.2, InnerContent [length 0001]
>>> TLS 1.3, Handshake [length 0024], Finished
<<< TLS 1.2, RecordHeader [length 0005]
<<< TLS 1.3, InnerContent [length 0001]
<<< TLS 1.3, Handshake [length 009c], NewSessionTicket
```

Everything from the sections above shows up here:

- **RecordHeader [length 0005]:** the 5-byte record header in front of each record. The first one says "TLS 1.0" because the first ClientHello uses `0x0301` for compatibility, and every later one says "TLS 1.2" (`0x0303`), even though the connection is TLS 1.3.
- **ClientHello → ServerHello:** the key exchange phase, in the clear.
- **ChangeCipherSpec [length 0001]:** the dummy message TLS 1.3 sends for middlebox compatibility, one byte long.
- **InnerContent [length 0001]:** from here on the records are encrypted. After decrypting, openssl prints the 1-byte real content type it found inside.
- **EncryptedExtensions, Certificate, CertificateVerify, Finished:** the server parameters and authentication phases, all encrypted. The Certificate is the biggest message (`0x0b48` = 2888 bytes), since it carries the certificate chain.
- **Finished [length 0024]:** 36 bytes, which is a 4-byte handshake header plus a 32-byte MAC (SHA-256).
- **NewSessionTicket:** sent after the handshake, so the client can resume the session later.
