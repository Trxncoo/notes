# SAIIT01 - DHCP

## Topics

1. [Key Concepts](#1-key-concepts)
2. [Terminology](#2-terminology)
3. [Message Format](#3-message-format)
4. [Message Types](#4-message-types)
5. [Client-Server Interaction](#5-client-server-interaction)

## 1. Key Concepts

The Dynamic Host Configuration Protocol (DHCP) provides configuration parameters to network hosts. It consists of two components: a protocol for delivering host-specific configuration parameters from a DHCP server to a host, and a mechanism for allocation of network addresses to hosts.

DHCP is built on a client-server model, where designated DHCP servers allocate network addresses and deliver configuration parameters to dynamically configured hosts.

DHCP supports three mechanisms for IP address allocation: in "automatic allocation", it assigns a permanent IP address to a client; in "dynamic allocation", it assigns an IP address to a client for a limited period of time (or until the client explicitly releases it); in "manual allocation", a client's IP address is assigned by the network administrator, and DHCP is used simply to convey the assigned address to the client.

## 2. Terminology

A **DHCP client** is a network host using DHCP to obtain configuration parameters such as a network address.

A **DHCP server** is a network host that returns configuration parameters to DHCP clients.

A **DHCP relay agent** is a network host or router that passes DHCP messages between DHCP clients and DHCP servers.

A **binding** is a collection of configuration parameters, including at least an IP address, associated with or "bound to" a DHCP client. Bindings are managed by DHCP servers.

## 3. Message Format

DHCP runs over UDP. Clients send to the server port (67) and servers send to the client port (68).

Every DHCP message has the same layout (sizes in octets):

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+---------------+---------------+---------------+---------------+
|     op (1)    |   htype (1)   |   hlen (1)    |   hops (1)    |
+---------------+---------------+---------------+---------------+
|                            xid (4)                            |
+-------------------------------+-------------------------------+
|           secs (2)            |           flags (2)           |
+-------------------------------+-------------------------------+
|                          ciaddr  (4)                          |
+---------------------------------------------------------------+
|                          yiaddr  (4)                          |
+---------------------------------------------------------------+
|                          siaddr  (4)                          |
+---------------------------------------------------------------+
|                          giaddr  (4)                          |
+---------------------------------------------------------------+
|                          chaddr  (16)                         |
+---------------------------------------------------------------+
|                          sname   (64)                         |
+---------------------------------------------------------------+
|                          file    (128)                        |
+---------------------------------------------------------------+
|                          options (variable)                   |
+---------------------------------------------------------------+
```

| Field   | Octets | Description                                                                                                                |
| ------- | ------ | -------------------------------------------------------------------------------------------------------------------------- |
| op      | 1      | Message op code: 1 = BOOTREQUEST (client to server), 2 = BOOTREPLY (server to client)                                      |
| htype   | 1      | Hardware address type, e.g. 1 = Ethernet                                                                                   |
| hlen    | 1      | Hardware address length, e.g. 6 for Ethernet                                                                               |
| hops    | 1      | Set to 0 by the client; relay agents may use it                                                                            |
| xid     | 4      | Transaction ID, a random number chosen by the client to match requests with replies                                        |
| secs    | 2      | Seconds since the client started acquiring or renewing an address                                                          |
| flags   | 2      | Flags (see below)                                                                                                          |
| ciaddr  | 4      | Client IP address, filled in only if the client already has one (BOUND, RENEWING or REBINDING) and can answer ARP requests |
| yiaddr  | 4      | "Your" IP address: the address the server is giving the client                                                             |
| siaddr  | 4      | IP address of the next server to use in the boot process, sent in OFFER and ACK                                            |
| giaddr  | 4      | Relay agent IP address, when the message goes through a relay agent                                                        |
| chaddr  | 16     | Client hardware address                                                                                                    |
| sname   | 64     | Optional server host name                                                                                                  |
| file    | 128    | Boot file name                                                                                                             |
| options | var    | Optional parameters                                                                                                        |

### Flags

Only the leftmost bit is used: the broadcast (B) flag. A client that can't receive unicast packets before its IP is configured sets it to ask for replies by broadcast. The other 15 bits are reserved and must be zero.

```
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|B|             MBZ             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Options

The options field starts with a four-octet magic cookie, 99.130.83.99, followed by the options themselves. The last option is always `end`.

A client must accept an options field of at least 312 octets, which means a whole message of up to 576 octets. Larger messages can be negotiated with the "maximum DHCP message size" option, and options can also overflow into the `sname` and `file` fields.

## 4. Message Types

The `op` field only says whether a message is a request or a reply. The actual DHCP message type goes in option 53 ("DHCP message type").

| Type         | Value | Direction                   | Use                                                                                                                                     |
| ------------ | ----- | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| DHCPDISCOVER | 1     | Client → server (broadcast) | Find available servers                                                                                                                  |
| DHCPOFFER    | 2     | Server → client             | Answer a DISCOVER with an offer of configuration parameters                                                                             |
| DHCPREQUEST  | 3     | Client → server             | Accept one server's offer (and implicitly decline the others), confirm a previously allocated address after a reboot, or extend a lease |
| DHCPDECLINE  | 4     | Client → server             | Report that the offered address is already in use                                                                                       |
| DHCPACK      | 5     | Server → client             | Confirm, with the configuration parameters and the committed address                                                                    |
| DHCPNAK      | 6     | Server → client             | Refuse: the client's address is wrong (e.g. it moved to another subnet) or its lease has expired                                        |
| DHCPRELEASE  | 7     | Client → server             | Give the address back and cancel the rest of the lease                                                                                  |
| DHCPINFORM   | 8     | Client → server             | Ask only for configuration parameters, because the client already has an address configured some other way                              |

## 5. Client-Server Interaction

### Allocating a New Address

A client with no address gets one through four messages, known as DORA (Discover, Offer, Request, Ack).

```
  Server A          Client          Server B
(not chosen)                        (chosen)
     |                |                |
     |<-- DISCOVER ---|--- DISCOVER -->|   broadcast
     |                |                |
     |---- OFFER ---->|<---- OFFER ----|
     |                |                |
     |<-- REQUEST ----|--- REQUEST --->|   broadcast, names Server B
     |                |                |
     |                |<----- ACK -----|   Server B commits the binding
     |                |                |
     |                |--- RELEASE --->|   optional, on shutdown
```

1. **Discover:** the client broadcasts a DHCPDISCOVER on its local subnet. It can suggest an address and a lease duration. Relay agents forward it to servers on other subnets.
2. **Offer:** each server may answer with a DHCPOFFER, with a free address in `yiaddr` and other parameters in the options. Before offering a new address, the server should check it isn't already in use, e.g. by pinging it (ICMP Echo Request). The server doesn't have to reserve the offered address, but things work better if it does.
3. **Request:** the client may wait for several offers, picks one, and broadcasts a DHCPREQUEST. The `server identifier` option names the chosen server, and the `requested IP address` option is set to the offered `yiaddr`. Because the request is broadcast, the other servers learn their offers were declined. If no offer arrives, the client times out and resends the DISCOVER.
4. **Ack:** the chosen server saves the binding and replies with a DHCPACK containing the parameters. If it can't satisfy the request (e.g. the address was given to someone else in the meantime), it answers with a DHCPNAK. The lease is identified by the `client identifier` (or `chaddr`) together with the assigned address.

After the ACK, the client should do a final check on the address, e.g. with ARP. If the address is already in use, it sends a DHCPDECLINE and starts over after waiting at least 10 seconds, to avoid looping. On a DHCPNAK it also starts over. If it gets no reply at all, it retransmits the REQUEST (e.g. four times over 60 seconds) and then goes back to the start.

The client can give the address back at any time with a DHCPRELEASE.

### Reusing a Previous Address

A client that remembers its previous address (after a reboot, for example) can skip DISCOVER and OFFER.

```
  Server A          Client          Server B
     |                |                |
     |<-- REQUEST ----|--- REQUEST --->|   broadcast
     |                |                |
     |----- ACK ----->|<----- ACK -----|   first ACK is used, later ones ignored
```

1. The client broadcasts a DHCPREQUEST with its old address in the `requested IP address` option. It leaves `ciaddr` empty, because it doesn't have the address yet.
2. Servers that know the client's configuration answer with a DHCPACK. If the request is invalid (e.g. the client moved to another subnet), they answer with a DHCPNAK. A server that isn't sure its information is accurate shouldn't answer at all.
3. The client does the same final check as above. On a conflict it sends a DHCPDECLINE and asks for a new address. On a DHCPNAK it can't reuse the old address and runs the full DORA exchange instead. If no reply comes after retransmitting, it may keep using the old address until the lease runs out.

A client reusing its address usually doesn't send a DHCPRELEASE on shutdown. It only does so when it has to give the address up, e.g. because it's being moved to another subnet.

### Renewing a Lease

A lease lasts a fixed number of seconds, and `0xffffffff` means infinite. Times are sent as relative values (seconds from now) and read against the client's own clock, so client and server don't need synchronized clocks. To allow for clock drift, the server may give the client a shorter lease than the one it records for itself.

To keep its address, the client tries to extend the lease at two points, T1 and T2:

```
lease start           T1 (50%)          T2 (87.5%)      expiry
    |--------------------|-------------------|--------------|
          BOUND              RENEWING           REBINDING
                         unicast REQUEST    broadcast REQUEST
                         to its server      to any server
```

- **T1 (default: 0.5 × lease):** the client enters RENEWING and unicasts a DHCPREQUEST to the server that gave it the address. It puts its current address in `ciaddr` and leaves out the `server identifier` option.
- **T2 (default: 0.875 × lease):** if no ACK has arrived by T2, the client enters REBINDING and broadcasts the same DHCPREQUEST, so any server can answer.

When a DHCPACK arrives, the new expiry time is the time the REQUEST was sent plus the lease duration in the ACK, and the client goes back to BOUND. ACKs with a different `xid` are ignored.

If there's no answer, the client resends after half the remaining time (until T2 in RENEWING, until expiry in REBINDING), but never waits less than 60 seconds.

The server sets T1 and T2 through options and should add a bit of randomness to them, so that clients don't all renew at the same moment. A client can also renew before T1.

If the lease expires without an ACK, the client stops using the address immediately and starts over from DORA. If it gets its old address back, it carries on. If it gets a different address, it must not use the old one again.

### Configuration Only (DHCPINFORM)

A client that already has an IP address (e.g. set manually) can use DHCPINFORM to get only the other configuration parameters, like DNS servers or the default gateway.

```
  Client                    Server
     |                         |
     |------- DHCPINFORM ----->|   ciaddr = client's address
     |                         |
     |<------- DHCPACK --------|   unicast to ciaddr, no address, no lease
```

1. The client puts its own address in `ciaddr` and sends a DHCPINFORM. It can list the parameters it wants in the "parameter request list" option. It unicasts the message if it knows the server's address, and broadcasts it otherwise.
2. The server replies with a DHCPACK sent directly to `ciaddr`, containing only the configuration parameters. It doesn't allocate an address, fill in `yiaddr`, check for an existing lease, or include a lease time.
3. The first ACK with a matching `xid`, from any server, completes the configuration. If none arrives within about 60 seconds (4 tries), the client tells the user and carries on with default values.
