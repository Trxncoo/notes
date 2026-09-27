# SAIIT02 - DNS

## Topics

1. [Key Concepts](#1-key-concepts)
2. [Terminology](#2-terminology)
3. [Domain Name Space](#3-domain-name-space)
4. [Resource Records](#4-resource-records)
5. [Message Format](#5-message-format)
6. [Zones and Name Servers](#6-zones-and-name-servers)
7. [Resolution](#7-resolution)

## 1. Key Concepts

The Domain Name System (DNS) maps domain names to information about them, most commonly host addresses. It replaced HOSTS.TXT, a single file listing every host name and address, maintained by the NIC and downloaded by every host. That stopped scaling as the Internet grew, and organizations had to wait for the NIC to publish any change to their own names.

DNS is a distributed database. The name space is a tree, each organization manages its own part of it, and servers cache answers to improve performance. All data attached to a name is tagged with a type (e.g. host address, mail server) and a class (the protocol family, e.g. Internet), and a query can ask for a single type.

DNS has three main components: the "domain name space and resource records", a tree of names and the data associated with each name; "name servers", which hold complete information about part of the tree and point to other name servers for the rest; and "resolvers", programs that get information from name servers for client programs, either answering from one server directly or following referrals to others.

## 2. Terminology

The **domain name space** is the tree that holds every name. Each node in the tree can have data attached to it.

A **label** is the name of a single node, from 0 to 63 octets long. Sibling nodes can't share a label. The root's label is empty.

A **domain name** is the list of labels on the path from a node up to the root, written from the most specific to the least specific and separated by dots (e.g. `www.uc.pt.`). A whole domain name is at most 255 octets. Comparisons are case-insensitive, so `UC.PT` and `uc.pt` are the same name.

An **absolute** domain name (also called fully qualified) is a complete name and ends with a dot, the empty root label. A **relative** name is incomplete and is completed by local software using the local domain, e.g. `www` inside `uc.pt`.

A **domain** is the part of the tree at or below a given domain name. A domain is a **subdomain** of another if it's contained in it, i.e. its name ends with the other's name (`dei.uc.pt` is a subdomain of `uc.pt` and `pt`).

A **resource record (RR)** is a piece of data attached to a domain name, such as an address.

A **zone** is a unit of authoritative information: the part of the tree that a name server has complete data for.

A **name server** is a server program that holds information about the domain tree. A name server is an **authority** for the zones it has complete information about.

A **resolver** is a program that gets information from name servers in response to client requests. It usually runs as a system routine that user programs call directly.

## 3. Domain Name Space

The name space is a single tree starting at the root. Below the root are the top-level domains, and each level below that is usually managed by a different organization:

```
                         . (root)
                         |
        +----------------+----------------+
        |                |                |
       pt               com              arpa
        |                |                |
        uc            google           in-addr
        |                |
   +----+----+          www
   |         |
  dei       www
```

`dei.uc.pt.` is read from the node up to the root: `dei`, then `uc`, then `pt`, then the empty root label.

### Structure and Use

DNS itself doesn't force any particular tree structure or label rules, so that it can be used for any kind of application. Different parts of the tree can follow different logic. For example, `in-addr.arpa` is organized by IP address because its job is to map addresses back to names (reverse lookups).

A domain that will later be split into several zones should branch near its top, so it can be split without renaming anything.

To store information about some kind of object in DNS, you need two things: a way to map the object's name to a domain name, and record types to describe it. For hosts, the host name is used directly. For mailboxes, the part before the `@` becomes a single label, so `hostmaster@uc.pt` becomes `hostmaster.uc.pt`.

### Preferred Name Syntax

Names that follow these rules cause fewer problems with applications like mail:

- Labels start with a letter and end with a letter or a digit.
- Inside a label, only letters, digits and hyphens are allowed.
- Labels are at most 63 characters.
- Upper and lower case are allowed but mean the same thing.

For example, `www.uc.pt` and `dei.uc.pt` are valid, and `-uc.pt` and `uc_.pt` are not. (A later update also allowed labels to start with a digit, which is why names like `3com.com` work.)

### Internal Representation

Inside programs and DNS messages, a name is stored as a sequence of labels. Each label is one length octet followed by its characters, and the name ends with a zero-length octet for the root:

```
www.uc.pt.  →  | 3 | w w w | 2 | u c | 2 | p t | 0 |
```

### Relative Names

A relative name is completed either from a single origin (as in zone files) or from a search list of domains set on the host. For example, with `uc.pt` in the search list, typing `www` is tried as `www.uc.pt.`.

## 4. Resource Records

Each node in the tree has a set of resource records (RRs), which can be empty. The order of the records in a set doesn't matter.

### Format

Every RR has the same fields:

| Field    | Size     | Description |
| -------- | -------- | ----------- |
| NAME     | variable | Owner: the domain name the record belongs to |
| TYPE     | 16 bits  | What kind of record it is (A, MX, ...) |
| CLASS    | 16 bits  | Protocol family, almost always IN (Internet) |
| TTL      | 32 bits  | How many seconds the record can be cached before it has to be fetched again. 0 means don't cache |
| RDLENGTH | 16 bits  | Length of RDATA in octets |
| RDATA    | variable | The data itself; its format depends on TYPE and CLASS |

In text form (as in zone files), a record is written on one line as owner, TTL, class, type and data. A line that starts with a blank has the same owner as the line before:

```
uc.pt.        3600  IN  MX     10 mail.uc.pt.
                        MX     20 mail2.uc.pt.
mail.uc.pt.   3600  IN  A      192.0.2.10
web.uc.pt.    3600  IN  CNAME  www.uc.pt.
```

### Common Types

| Type  | Value | RDATA | Use |
| ----- | ----- | ----- | --- |
| A     | 1     | 32-bit IPv4 address | Host address |
| NS    | 2     | Host name | Authoritative name server for the domain |
| CNAME | 5     | Domain name | Marks the owner as an alias of another (canonical) name |
| SOA   | 6     | Several fields (see below) | Start of a zone of authority |
| PTR   | 12    | Domain name | Pointer to another name, used for reverse lookups |
| MX    | 15    | 16-bit preference + host name | Mail server for the domain; lower preference is tried first |
| TXT   | 16    | Text strings | Free text |
| AAAA  | 28    | 128-bit IPv6 address | IPv6 host address |

Queries can also ask for some types that are never stored as records: `AXFR` (252) asks for a whole zone, and `*` (255) asks for all records of a name.

### TTL

The zone's administrator sets the TTL. Short TTLs reduce caching, but for normal hosts the TTL should be around days. If a change is planned, the TTL can be lowered beforehand so old data disappears from caches quickly, and raised again afterwards.

### Aliases (CNAME)

A CNAME says that its owner is an alias and gives the canonical (primary) name. A node with a CNAME can't have any other records, so an alias and its canonical name can never disagree.

When a name server looks for a record and finds a CNAME instead, it adds the CNAME to the answer and restarts the lookup with the canonical name:

```
web.uc.pt.  IN  CNAME  www.uc.pt.
www.uc.pt.  IN  A      192.0.2.20
```

A query for the A record of `web.uc.pt` returns both records. A query for the CNAME itself only returns the CNAME.

Records that point to another name (NS, MX, PTR, ...) should point to the canonical name, not the alias, to avoid an extra lookup. CNAME chains should be followed, and loops reported as errors.

### SOA Fields

| Field   | Description |
| ------- | ----------- |
| MNAME   | The primary name server for the zone |
| RNAME   | Mailbox of the person responsible, written as a domain name (`hostmaster.uc.pt` for `hostmaster@uc.pt`) |
| SERIAL  | Version number of the zone, which secondaries check to see if they need to update |
| REFRESH | Seconds before a secondary checks for a newer version |
| RETRY   | Seconds before retrying a failed refresh |
| EXPIRE  | Seconds after which a secondary that can't refresh stops being authoritative |
| MINIMUM | Minimum TTL for every record sent from the zone |

### Reverse Lookups (in-addr.arpa)

To find the name for an IP address, DNS uses the `in-addr.arpa` domain. The four octets of the address become labels in reverse order, and a PTR record there points back to the host name:

```
10.2.0.192.in-addr.arpa.  IN  PTR  mail.uc.pt.
```

The address is reversed so that the most significant octet is closest to the root. That way a whole network can be delegated as its own zone, e.g. `192.in-addr.arpa`.

## 5. Message Format

### Transport

DNS uses port 53 on both UDP and TCP.

- **UDP** is used for normal queries because it's lighter and faster. A UDP message is at most 512 bytes (not counting IP and UDP headers). If a reply is longer, it's truncated and the TC bit is set, and the client can retry over TCP. UDP packets can be lost or arrive out of order, so the client has to retransmit: it should try other servers first, and wait at least 2 to 5 seconds before retrying the same one.
- **TCP** is required for zone transfers, because they need reliable delivery. Each message is prefixed with a 2-byte length field. The server should support several connections at once and let the client close them, only closing idle connections after about two minutes.

### Layout

Queries and responses use the same message format, split into five sections:

```
+---------------------+
|        Header       |
+---------------------+
|       Question      |   what is being asked
+---------------------+
|        Answer       |   RRs that answer the question
+---------------------+
|      Authority      |   RRs pointing to an authoritative name server
+---------------------+
|      Additional     |   RRs related to the query that aren't strictly answers
+---------------------+
```

The header is always there. The other sections can be empty, and the header says how many entries each one has. Answer, Authority and Additional are all lists of resource records in the same format as in section 4.

### Header

The header is 12 bytes:

```
  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                      ID                       |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|QR|   Opcode  |AA|TC|RD|RA|   Z    |   RCODE   |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    QDCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    ANCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    NSCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                    ARCOUNT                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

| Field   | Bits | Description |
| ------- | ---- | ----------- |
| ID      | 16   | Chosen by whoever sends the query and copied into the reply, to match replies to queries |
| QR      | 1    | 0 = query, 1 = response |
| Opcode  | 4    | Kind of query: 0 = standard query, 1 = inverse query, 2 = server status. Copied into the response |
| AA      | 1    | Authoritative Answer: the responding server is an authority for the name asked |
| TC      | 1    | TrunCation: the message was cut because it was too long for the channel |
| RD      | 1    | Recursion Desired: set in the query to ask the server to resolve the name recursively. Copied into the response |
| RA      | 1    | Recursion Available: set in the response if the server supports recursion |
| Z       | 3    | Reserved, must be 0 |
| RCODE   | 4    | Response code (see below) |
| QDCOUNT | 16   | Number of entries in the Question section |
| ANCOUNT | 16   | Number of RRs in the Answer section |
| NSCOUNT | 16   | Number of RRs in the Authority section |
| ARCOUNT | 16   | Number of RRs in the Additional section |

| RCODE | Name | Meaning |
| ----- | ---- | ------- |
| 0 | No error | |
| 1 | Format error | The server couldn't understand the query |
| 2 | Server failure | The server had a problem processing the query |
| 3 | Name error | The name doesn't exist (only meaningful from an authoritative server) |
| 4 | Not implemented | The server doesn't support this kind of query |
| 5 | Refused | The server won't do it for policy reasons, e.g. a zone transfer to someone it doesn't trust |

### Question

The Question section usually has one entry:

| Field  | Description |
| ------ | ----------- |
| QNAME  | The domain name, in label format (length octet + characters, ending in a zero octet). No padding, so it can be an odd number of octets |
| QTYPE  | The type asked for: any RR type, or a query-only type like `AXFR` or `*` |
| QCLASS | The class, e.g. IN |

### Name Compression

The same name often appears several times in a message (in the question and again in every answer). To save space, a name, or the end of one, can be replaced by a 2-byte pointer to an earlier copy of it in the message:

```
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
| 1  1|                OFFSET                   |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
```

The two leading 1 bits mark it as a pointer. A normal label length always starts with two 0 bits, because labels are at most 63 octets. OFFSET is counted from the start of the message (the first byte of the ID).

So a name in a message can be written as a list of labels ending in a zero octet, as a single pointer, or as some labels followed by a pointer.

For example, the header is 12 bytes, so the question name always starts at offset 12. If the question asks for `www.uc.pt`, an answer record for the same name can just write `C0 0C` (pointer to offset 12) instead of repeating the whole name. A record for `mail.uc.pt` can write the label `mail` and then a pointer to where `uc.pt` starts inside the question name.

Senders don't have to compress names, but every receiver must understand pointers.

## 6. Zones and Name Servers

A name server's main job is to answer queries using the data in its zones. It can always answer from local data alone: the reply either contains the answer or a referral to other name servers that are closer to it.

A name server usually has one or more zones, which makes it an authority for only a small part of the tree. It may also have cached data about other parts, which isn't authoritative. The AA bit in the response tells the requester which kind of data it got.

Every zone has to be available on at least two name servers, so it stays reachable if a host or link fails.

### Dividing the Tree into Zones

The database is split into zones by making "cuts" between nodes in the tree. Each connected part left after the cuts is a zone, and is named after its highest node (the one closest to the root).

Cuts are made where an organization wants to take control of a subtree. Once it controls its zone, it can change the data, add or remove nodes, and delegate subzones of its own. For example, a university could keep everything in one zone, or delegate a subzone to each department.

```
               pt                 zone pt
               |
  - - - - - - -|- - - - - - - -   cut: pt delegates uc.pt
               uc                 zone uc.pt (uc, www)
             /    \
           www     |
  - - - - - - - - -|- - - - - -   cut: uc.pt delegates dei.uc.pt
                  dei             zone dei.uc.pt (dei, www)
                   |
                  www
```

### What's in a Zone

A zone is described completely by RRs. For example, the full zone file for `uc.pt`, which has delegated `dei.uc.pt` to the department, could be:

```
; 1. Top of the zone
uc.pt.         IN  SOA  ns1.uc.pt. hostmaster.uc.pt. (...)
uc.pt.         IN  NS   ns1.uc.pt.
uc.pt.         IN  NS   ns2.uc.pt.

; 2. Authoritative data
www.uc.pt.     IN  A    192.0.2.20
mail.uc.pt.    IN  A    192.0.2.10
ns1.uc.pt.     IN  A    192.0.2.1
ns2.uc.pt.     IN  A    192.0.2.2

; 3. Delegation
dei.uc.pt.     IN  NS   ns.dei.uc.pt.

; 4. Glue
ns.dei.uc.pt.  IN  A    192.0.2.53
```

1. **Top of the zone:** the SOA and NS records at `uc.pt` itself. They say this is a zone, which name servers serve it, and what its management parameters are. Every zone has them.
2. **Authoritative data:** all the names the zone is in charge of, like `www` and `mail`. Answers with these records are authoritative (AA bit set).
3. **Delegation:** `dei.uc.pt` has been handed over, so `uc.pt` doesn't hold its data anymore. It only keeps an NS record saying "for anything under `dei.uc.pt`, ask `ns.dei.uc.pt`". This record isn't authoritative in `uc.pt`, because it belongs to the `dei.uc.pt` zone, which has the same NS record at its top. The two copies should match.
4. **Glue:** the NS record only gives the server's name, not its address. A resolver looking for `www.dei.uc.pt` is told by `uc.pt` to ask `ns.dei.uc.pt`. To reach that server it needs its address, but `ns.dei.uc.pt` is itself inside `dei.uc.pt`, so looking it up leads back to `ns.dei.uc.pt`, which is a loop. The glue A record breaks it: `uc.pt` keeps the server's address and sends it along with the referral. Glue is only needed when the server's name is inside the delegated zone. If DEI used `ns.example.com`, the resolver could look up that address normally.

### Delegating a Zone

To get its own zone, an organization:

1. Agrees with the owners of the parent zone to delegate the subtree to it.
2. Shows it has redundant name servers for the new zone. The servers don't need to be inside the domain, and spreading them out geographically makes the zone more reachable.
3. Gets the delegation NS records and any glue records added to the parent zone. Both sides have to keep them consistent.

### Primary and Secondary Servers

One name server is the **primary** (master) for a zone. Changes are made there, usually by editing the zone's master file and reloading it. The other servers are **secondaries**, which keep copies of the zone and update them from the primary.

Every time the zone changes, the SERIAL in its SOA record is increased. Secondaries compare serials to know when their copy is out of date, using the SOA timers:

```
secondary loads zone
        |
   wait REFRESH
        |
  ask primary for SOA ──── fails ──→ retry every RETRY seconds
        |                              |
  same SERIAL? ── yes → wait REFRESH   no answer for EXPIRE seconds
        |                              → discard the zone copy
        no
        |
  request zone transfer (AXFR)
```

### Zone Transfers

When the serial has changed, the secondary asks for the whole zone with an `AXFR` query. The answer is a sequence of messages: the first and last carry the SOA of the zone, and the ones in between carry every other record. Because the copy has to be exact, zone transfers always use TCP.

Secondaries must be able to transfer from the primary, and can also transfer from other secondaries, e.g. when the primary is down or another secondary is closer.

## 7. Resolution

A resolver runs on the same machine as the program asking for the name (a browser, a mail client, ...), usually as a system call or library function. It may need to ask several name servers, or it may already have the answer cached, so a lookup can take anywhere from milliseconds to a few seconds. Answering from cache is the resolver's most important way to cut down on network delay and server load, and a cache shared by many users or machines works better than one per program.

### Resolver Functions

A resolver typically offers three functions:

- **Name to address:** asks for the A records of a name.
- **Address to name:** reverses the address into an `in-addr.arpa` name and asks for its PTR record (e.g. `1.2.3.4` → `4.3.2.1.in-addr.arpa`).
- **General lookup:** any QNAME, QTYPE and QCLASS, returning all matching records.

The result is one of:

| Result | Meaning |
| ------ | ------- |
| Answer | One or more RRs with the requested data |
| Name error | The name doesn't exist, e.g. it was mistyped |
| Data not found | The name exists but has no records of the requested type, e.g. asking for the address of a name that only has MX records |
| Temporary failure | The resolver couldn't reach the servers. This must never be reported as "name doesn't exist" |

### Recursive and Iterative Queries

A name server can answer a query in two ways:

- **Iterative (non-recursive):** the server only uses its own data and returns the answer, an error, or a **referral** to name servers closer to the answer. The client then has to follow the referral itself. Every name server must support this.
- **Recursive:** the server does all the work for the client, following referrals itself, and returns only the final answer or an error, never a referral. This is optional, and a server can limit which clients may use it.

The client asks for recursion by setting RD (Recursion Desired) in the query. The server sets RA (Recursion Available) in every response if it's willing to recurse. Recursion was used only if both RD and RA are set in the reply. A server should never recurse unless RD is set.

Recursion suits simple clients that can't follow referrals, and networks that want one shared cache instead of one per client.

### Stub Resolvers

Most hosts use a **stub resolver**: a minimal resolver that doesn't follow referrals itself. It just sends a recursive query to a configured recursive name server (e.g. the one the host got from DHCP) and waits for the answer. The recursive server does the real work and keeps the shared cache.

### A Full Resolution

A stub resolver asks for `www.dei.uc.pt`, and no server has anything cached yet:

```
stub resolver --(1) www.dei.uc.pt? (RD)--> recursive server
                                               |
                                               |--(2)--> root server       replies: referral to the pt servers
                                               |--(3)--> pt server         replies: referral to the uc.pt servers
                                               |--(4)--> uc.pt server      replies: referral to ns.dei.uc.pt (+ glue)
                                               |--(5)--> dei.uc.pt server  replies: A 192.0.2.80 (AA)
                                               |
stub resolver <--(6) A 192.0.2.80------------- recursive server
```

1. The stub resolver sends a recursive query (RD set) to its recursive server.
2. The recursive server starts at a root server, whose addresses it has preconfigured. The root doesn't know the answer, so it returns a referral to the `pt` servers.
3. The `pt` server refers it to the `uc.pt` servers.
4. The `uc.pt` server refers it to `ns.dei.uc.pt`, including the glue A record since that server is inside the zone it's delegating.
5. The `dei.uc.pt` server is authoritative and returns the A record with AA set.
6. The recursive server caches everything it learned along the way and returns the answer to the stub.

The next query for anything under `uc.pt` can skip the root and `pt` steps, because the recursive server already has the `uc.pt` servers cached.

### Caching and TTL

Every cached record is kept for its TTL, then thrown away and fetched again. Cached answers aren't authoritative, so a server answering from cache doesn't set AA.

### Negative Caching

A server can also cache the fact that a name doesn't exist, so repeated lookups for a missing name (common with search lists) don't go to the authoritative servers every time. An authoritative server adds the zone's SOA record to a "name doesn't exist" response, and the resolver caches the negative result using the SOA's MINIMUM field as its TTL.
