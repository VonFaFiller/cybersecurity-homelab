# UDP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

UDP is common in PCAP analysis because many protocols use it for fast, lightweight communication.

It is frequently seen with DNS, DHCP, NTP, QUIC, VoIP, streaming traffic, discovery protocols, and some malware or C2 communication.

UDP is useful when investigating short request-response exchanges, repeated callbacks, scanning behavior, service discovery, and traffic where no TCP handshake exists.

> [!NOTE]
> UDP does not establish a connection before sending data. This means there is no SYN, SYN/ACK, ACK sequence to confirm that a session was established.

## Core Concept

UDP is connectionless.

A host can send a UDP packet to another host without first establishing a connection.

The receiving host may reply, but UDP itself does not guarantee delivery, ordering, retransmission, or session state.

Example:

```text
Client -> Server: UDP packet
Server -> Client: UDP reply
```

In this case:

```text
Client packet: request or outbound message
Server reply: possible response
```

Unlike TCP, UDP does not prove that communication was accepted through a handshake.

The protocol running over UDP must be inspected to understand what happened.

Example with DNS:

```text
Client -> DNS Server: DNS query over UDP
DNS Server -> Client: DNS response over UDP
```

In this case:

```text
UDP: transport layer
DNS: application protocol that explains the actual action
```

## Common UDP-Based Protocols

| Protocol | Practical use |
|---|---|
| DNS | Domain resolution |
| DHCP | IP address assignment |
| NTP | Time synchronization |
| QUIC | Encrypted web traffic over UDP |
| TFTP | Simple file transfer |
| SNMP | Network device monitoring |
| mDNS | Local name resolution |
| LLMNR | Windows local name resolution |
| RTP | Voice or media transport |

## Common UDP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| UDP request followed by UDP reply | Application-layer exchange likely occurred |
| UDP packet with no reply | No response, blocked service, one-way traffic, or missing capture data |
| Repeated UDP packets to same destination | Retry behavior, beaconing, streaming, or polling |
| Many UDP packets to many ports | Possible UDP scan |
| Many UDP packets to many hosts | Possible discovery or scanning activity |
| UDP followed by ICMP Port Unreachable | Destination port is likely closed |
| Large UDP payloads | Possible file transfer, tunneling, media, or unusual application data |
| Regular UDP traffic at fixed intervals | Possible keepalive, beaconing, polling, or normal background service |
| UDP/443 traffic | Possible QUIC |
| UDP/53 traffic | Usually DNS |
| UDP/67 and UDP/68 traffic | Usually DHCP |
| UDP/123 traffic | Usually NTP |

## Main UDP Fields

| Field | Practical use |
|---|---|
| Source port | Identifies the sending UDP port |
| Destination port | Identifies the receiving UDP port |
| Length | Shows UDP datagram size |
| Checksum | Used to detect corruption |
| Payload | Contains the application-layer data carried by UDP |

## Wireshark Filters

```text
udp.length
```

Shows UDP packets with length information.

Useful to inspect datagram size.

```text
udp.length > 1000
```

Shows large UDP packets.

Useful when looking for unusually large UDP payloads, file transfer, tunneling, media traffic, or abnormal communication.

```text
udp.length < 20
```

Shows very small UDP packets.

Useful when checking for minimal probes, keepalives, or unusual short datagrams.

```text
udp.checksum
```

Shows UDP packets containing checksum information.

Useful when checking packet integrity fields.

```text
udp.checksum.status
```

Shows UDP checksum status information.

Useful when checking whether Wireshark marks UDP checksums as valid, invalid, or unverified.

```text
udp.payload
```

Shows UDP packets containing payload data.

Useful when checking whether the UDP datagram carries visible application-layer content.

```text
udp.payload contains "example"
```

Shows UDP payloads containing a specific string.

Useful when searching for cleartext indicators inside UDP traffic.