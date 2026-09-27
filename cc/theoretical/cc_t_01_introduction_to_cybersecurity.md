# CCT01 - Introduction to Cybersecurity

## Topics

1. [Key Concepts](#1-key-concepts)
2. [Terminology](#2-terminology)
3. [CIA Triad](#3-cia-triad)
4. [AAA and Access Control](#4-aaa-and-access-control)
5. [Security of Communications](#5-security-of-communications)
6. [Types of Attacks](#6-types-of-attacks)

## 1. Key Concepts

Communication is the exchange of information between a sender and a receiver. Cybersecurity is about protecting that exchange and the systems behind it.

NIST defines cybersecurity as the prevention of damage to, protection of, and restoration of computers, electronic communication systems and services, and the information they contain, to ensure their availability, integrity, authentication, confidentiality and non-repudiation. More broadly, it covers all the activities, people and technology meant to avoid security incidents, data breaches and the loss of critical systems.

It isn't possible to prevent all damage or protect everything. The main security properties pull against each other (e.g. making data harder to access for attackers also makes it harder to access for legitimate users), so cybersecurity is about trade-offs, and the right balance depends on the organization and the system.

## 2. Terminology

An **asset** is anything of value to an organization that needs protecting, such as data, a service or a system.

An **authorized party** is a person, process or program that is allowed to access an asset in a particular way (e.g. read it but not change it).

**Confidentiality** is the absence of unauthorized disclosure: an asset is only seen by authorized parties.

**Integrity** is protection against illicit or undetected modification: an asset is only changed by authorized parties, in authorized ways.

**Availability** is protection against denial of service: an asset can be used by authorized parties when they need it.

**Privacy** is the right of individuals to control what information about them is collected and stored, by whom, and to whom it is disclosed.

**Authentication** is verifying the identity of a user or system.

**Authorization** is granting or denying an authenticated user or system the right to use a specific resource.

**Accounting** is recording what users and systems do, so their actions can be tracked afterwards.

**Non-repudiation** is the assurance that someone can't convincingly deny having performed an action, e.g. sending a message.

An **attack** is any malicious activity that tries to collect, disrupt, deny, degrade or destroy information system resources or the information itself.

The **attack surface** is the set of points on the boundary of a system where an attacker can try to get in, cause an effect, or extract data.

A **passive attack** doesn't alter systems or data. The attacker only observes, e.g. by listening to traffic.

An **active attack** involves the attacker sending or changing data, e.g. impersonating someone or placing themselves between two parties (man-in-the-middle).

## 3. CIA Triad

Cybersecurity tries to prevent unauthorized viewing (confidentiality) and unauthorized modification (integrity) while keeping access for those who are allowed (availability).

| Property        | An asset is...                         | Protects against |
| --------------- | -------------------------------------- | ---------------- |
| Confidentiality | only viewed by authorized parties      | Unauthorized disclosure |
| Integrity       | only modified by authorized parties    | Illicit or undetected modification |
| Availability    | usable by authorized parties when needed | Denial of service |

### Confidentiality

Data is only disclosed as the organization intends. Whether someone is authorized depends on who they are and what they want to do: a person, process or program may be allowed to access a data item in one way but not another. Different data needs different levels of protection, from military secrets and product plans down to student grades.

It has two sides:

- **Data confidentiality:** private or confidential information isn't made available or disclosed to unauthorized individuals.
- **Privacy:** individuals control what information about them is collected and stored, by whom, and who it's disclosed to.

The hard questions are about who decides and who owns:

- Who decides which people or systems are authorized, and at what level (a single bit, or the whole collection)?
- Can someone who is authorized pass the data on to others?
- How much does protecting it limit what the system can do?
- Who owns the data? When you visit a web page, does the fact that you clicked a link belong to you, the page's owner, your ISP, or all of them?

### Integrity

Assets are only changed by authorized parties, and never against the organization's wishes. Integrity problems are easy to understand, but many are hard to detect.

It has two sides:

- **Data integrity:** information and programs are only changed in a specified, authorized way.
- **System integrity:** a system performs its intended function without being impaired, free from deliberate or accidental unauthorized manipulation.

It's harder to pin down than confidentiality, because it means different things in different contexts: precise, unmodified, modified only in acceptable ways, modified only by authorized people, consistent, and so on. On top of that, a clever attacker changes data in ways that are hard to notice, and even authorized parties can change data incorrectly by mistake.

### Availability

Assets are accessible to authorized parties when they need them. It applies to both data and services, and its exact meaning depends on the system and what users expect: the asset is there in a usable form, has enough capacity, and responds in an acceptable time.

It has two sides:

- **Data availability:** information and programs can be used in their intended way.
- **System availability:** the system is ready to provide correct service to authorized users.

The challenges are mostly practical:

- Capacity and performance
- Concurrency: simultaneous access, deadlocks, exclusive access
- Fair allocation of resources between users
- Fault tolerance
- Usability
- The authentication service itself has to be available, or nobody can log in

### Trade-offs

The three properties can't all be maximized at once. Confidentiality and availability in particular pull in opposite directions: the harder data is to reach, the harder it is for legitimate users too. Security is about choosing a compromise, and the right one depends on the system:

- **Business-critical systems** (e.g. a bank's customer data) usually prioritize confidentiality.
- **Safety-critical systems** (e.g. a hospital's patient monitoring) usually prioritize availability.

## 4. AAA and Access Control

### AAA

Authentication, authorization and accounting work as a sequence every time someone uses a system:

```
 user ──→ Authentication ──→ Authorization ──→ Accounting
           who are you?       what can you do?   what did you do?
```

1. **Authentication** checks the identity of the user or system, e.g. with a password, a certificate or a fingerprint.
2. **Authorization** decides what that identity is allowed to do, e.g. a student can read their own grades but not change them.
3. **Accounting** records what was done, e.g. in logs. Those records are what make non-repudiation possible: a user can't convincingly deny an action when there's a trustworthy record of it.

Each step depends on the one before it. Authorization is meaningless if the identity wasn't checked, and accounting is useless if it can't tell users apart.

### Access Control

Access control is the mechanism that puts authentication and authorization into practice: it sits between users and assets and decides who can access what, and how. It's one of the basic paradigms of computer security.

A small, centralized access control component is the usual way to protect confidentiality and integrity, because every request goes through one place that's easy to check and enforce.

For availability it's the opposite. A single central component is a single point of failure: if it goes down or gets overloaded, nobody can access anything, even though the assets themselves are fine. That's why availability is still one of the hardest properties to guarantee.

## 5. Security of Communications

Securing a real application takes several protocols working together, and each one only protects part of the communication. Using Instagram as an example:

### Web Traffic: HTTPS and QUIC

When you open Instagram, the connection uses:

- **HTTPS** (HTTP over TLS), used by almost every web application and API. It encrypts the traffic (confidentiality), detects changes to it (integrity), and uses the server's **certificate** to prove you're really talking to Instagram (authentication).
- **QUIC** as the transport protocol. It runs over UDP and has TLS built in, so the connection is encrypted from the first packet. The encryption algorithms can include quantum-safe ones, which protect the traffic even against future quantum computers.
- **Secure resources only**: all third-party content on the page (images, scripts, ...) is also loaded over HTTPS. One resource loaded over plain HTTP would be a weak point an attacker could tamper with.

This protects the connection to Instagram's servers, but it's not enough on its own.

### DNS and DNSSEC

Before connecting, the browser has to find Instagram's IP address using DNS (see the [SAII DNS notes](../../saii/theoretical/saii_t_02_dns.md)). DNS is essential to the Internet, but it's an insecure protocol: answers aren't authenticated. If an attacker gets the client to accept a forged answer with a different IP address, the browser connects to the attacker's server, which can then pretend to be the real site.

**DNSSEC** (DNS Security Extensions) fixes this. It adds new resource record types and changes to the DNS protocol, and uses **digital signatures** on the records to provide:

- **Data origin authentication:** the data really comes from the zone that owns it.
- **Data integrity:** the record hasn't been modified on the way.

DNSSEC doesn't encrypt anything. Anyone watching can still see which names are being looked up.

### Email

Even with HTTPS and DNSSEC, users can still be tricked by fake emails. Instagram uses email to recover accounts and send updates, so a forged email pretending to be from Instagram is an easy way to steal an account.

There are two groups of protections. The first protects the message while it travels:

| Protocol   | What it protects | Provides |
| ---------- | ---------------- | -------- |
| STARTTLS   | The SMTP connection between two mail servers, by upgrading it to TLS | Confidentiality, integrity and server authentication for that hop |
| S/MIME, PGP | The message body, end to end | Authentication, integrity, non-repudiation (signatures) and confidentiality (encryption) |

STARTTLS only protects each hop between servers, so a message can still be read on the servers along the way. S/MIME and PGP protect the message itself from sender to recipient.

The second group uses DNS to check that a message really comes from the domain it claims to be from:

| Protocol | Standard | How it works |
| -------- | -------- | ------------ |
| **SPF** (Sender Policy Framework) | RFC 7208 | The domain owner publishes a DNS record listing the IP addresses allowed to send mail for the domain. Receivers check the sender's IP against it |
| **DKIM** (DomainKeys Identified Mail) | RFC 6376 | The sending mail server signs selected headers and the body. Receivers check the signature with a public key published in DNS, which proves the source domain and the body's integrity |
| **DMARC** (Domain-based Message Authentication, Reporting, and Conformance) | RFC 7489 | The domain owner publishes a DNS policy saying what receivers should do when SPF or DKIM checks fail (none, quarantine or reject), and receivers send back reports on how the checks went |

## 6. Types of Attacks

Attacks can be classified in two independent ways: by where the attacker is, and by whether they change anything.

|              | Passive (only observes) | Active (sends or changes data) |
| ------------ | ----------------------- | ------------------------------ |
| **Internal** (from inside the organization) | e.g. an employee reading traffic they shouldn't | e.g. an employee leaking or deleting data |
| **External** (from outside) | e.g. someone sniffing a public Wi-Fi network | e.g. a DDoS against the company's website |

### Passive Attacks

Passive attacks don't alter anything, so they're hard to detect. They mostly threaten confidentiality.

- **Sniffing:** listening to or capturing communications without authorization. A current example is **Harvest Now, Decrypt Later (HNDL)**: attackers store encrypted traffic today, hoping to decrypt it once quantum computers can break today's encryption. So traffic that has to stay secret for years needs quantum-safe encryption already.
- **Traffic analysis:** even when traffic is encrypted, the attacker studies its patterns: who talks to whom, how often, when, and how much. That can reveal a lot without reading any content.

### Active Attacks

| Attack | What it does | Mainly hits |
| ------ | ------------ | ----------- |
| **Denial of Service (DoS)** | Floods a service with requests until it can't serve legitimate users. Can use many protocols: DNS flood, SYN flood, UDP flood, RST flood, ICMP flood, TCP flood, ... | Availability |
| **Distributed DoS (DDoS)** | A DoS launched from hundreds to millions of devices at once, usually a botnet | Availability |
| **Botnet** | Takes full control of many devices (servers, PCs, IoT) to use them for other attacks, like DDoS | All three |
| **Data exfiltration** | Accesses data without authorization and copies it to other servers or publishes it | Confidentiality |
| **Masquerade** | The attacker pretends to be someone else to get access to resources | Confidentiality, integrity |
| **Replay** | Captured traffic is sent again later to repeat its effect, e.g. duplicating a payment | Integrity |
| **Man-in-the-Middle (MitM)** | The attacker places themselves between two parties, and can read or change what they send each other | Confidentiality, integrity |
| **Injection** | Malicious commands are inserted into an application's input to access data illegitimately (e.g. SQL injection) | Confidentiality, integrity |
| **Ransomware** | Encrypts the victim's data and demands a ransom to restore access | Availability |
| **Session hijacking** | Steals session information (e.g. a session ID cookie) to take over a user's logged-in session | Confidentiality, integrity |
| **Advanced Persistent Threat (APT)** | A long-term, targeted attack where the attacker gets into a network and stays hidden to steal sensitive information. Usually well funded, sometimes backed by governments | Confidentiality |

### Example: Instagram

Several of these apply to the Instagram example from section 5:

- **DDoS** against Instagram's servers would make the service unavailable.
- **Forged DNS answers** (if DNSSEC isn't used) lead to a **masquerade** or **MitM**, where a fake server pretends to be Instagram.
- **Fake emails** pretending to be from Instagram (if SPF, DKIM and DMARC aren't enforced) can trick users into giving away their account, which is also a masquerade.
- **Session hijacking** would let an attacker use someone's account without knowing their password.
- **Sniffing:** HTTPS traffic captured today could be decrypted later if it doesn't use quantum-safe encryption (HNDL).
