# ARP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

ARP is useful in PCAP analysis when investigating local network communication, IP-to-MAC mapping, host discovery, duplicate IP addresses, ARP scans, gateway identification, and possible ARP spoofing.

ARP helps identify which MAC address is associated with which IPv4 address inside the local network.

> [!NOTE]
> ARP is only used for local IPv4 address resolution. It does not resolve remote Internet hosts directly.

## Core Concept

ARP resolves an IPv4 address to a MAC address.

A host sends an ARP request asking who owns a specific IP address.

The host using that IP replies with its MAC address.

Example:

```text
Host A -> Broadcast: Who has 192.168.1.10?
Host B -> Host A: 192.168.1.10 is at 00:11:22:33:44:55
```

In this case:

```text
Requested IP: 192.168.1.10
Resolved MAC: 00:11:22:33:44:55
```

This allows the sender to build or update its local IP-to-MAC mapping.

## Main ARP Operations

| Opcode | Operation | Practical meaning |
|---:|---|---|
| 1 | Request | A host is asking who owns an IPv4 address |
| 2 | Reply | A host is answering with a MAC address |

## Common ARP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| ARP request followed by ARP reply | IP-to-MAC resolution succeeded |
| ARP request with no reply | Host may be offline, unreachable, filtered, or absent from the local network |
| Many ARP requests to many IPs | Possible ARP scan or local host discovery |
| Repeated ARP requests for gateway IP | Host is trying to reach the default gateway |
| Multiple MAC addresses claiming the same IP | Possible duplicate IP or ARP spoofing |
| One MAC claiming many IP addresses | Possible proxy ARP, misconfiguration, or spoofing |
| Gratuitous ARP | Host announcing or updating its own IP-to-MAC mapping |
| ARP replies without prior visible requests | Possible gratuitous ARP, cached behavior, missing packets, or suspicious ARP activity |
| ARP traffic before TCP/UDP communication | Host is resolving the local next-hop MAC address before communication |

## Main ARP Fields

| Field | Practical use |
|---|---|
| Opcode | Shows whether the packet is an ARP request or reply |
| Sender MAC address | MAC address of the host sending the ARP message |
| Sender IP address | IPv4 address claimed by the sender |
| Target MAC address | MAC address being requested or answered |
| Target IP address | IPv4 address being resolved |
| Hardware type | Usually Ethernet |
| Protocol type | Usually IPv4 |

## Wireshark Filters

```text
arp
```

Shows ARP traffic.

Useful to inspect local IPv4-to-MAC resolution activity.

```text
arp.opcode == 1
```

Shows ARP requests.

Useful to identify hosts asking for MAC addresses.

```text
arp.opcode == 2
```

Shows ARP replies.

Useful to identify IP-to-MAC answers.

```text
arp.src.proto_ipv4
```

Shows ARP packets containing sender IPv4 addresses.

Useful to inspect which IP address a host is claiming.

```text
arp.dst.proto_ipv4
```

Shows ARP packets containing target IPv4 addresses.

Useful to inspect which IP address is being requested.

```text
arp.src.hw_mac
```

Shows ARP packets containing sender MAC addresses.

Useful to map MAC addresses to IPv4 addresses.

```text
arp.dst.hw_mac
```

Shows ARP packets containing target MAC addresses.

Useful to inspect the target hardware address field.

```text
arp.src.proto_ipv4 == 192.168.1.10
```

Shows ARP packets where the sender IP is a specific address.

Useful to inspect which MAC address is claiming that IP.

```text
arp.dst.proto_ipv4 == 192.168.1.10
```

Shows ARP packets asking about a specific IP address.

Useful to identify who is trying to resolve that host.

```text
arp.src.hw_mac == 00:11:22:33:44:55
```

Shows ARP packets sent by a specific MAC address.

Useful to track ARP activity from one host.

```text
arp.isgratuitous
```

Shows gratuitous ARP packets.

Useful to identify hosts announcing or updating their own IP-to-MAC mapping.

```text
arp.duplicate-address-detected
```

Shows ARP packets where Wireshark detected a possible duplicate IP address.

Useful to identify IP conflicts or possible ARP spoofing.

```text
arp.duplicate-address-frame
```

Shows ARP packets related to a previously detected duplicate address frame.

Useful when following duplicate IP evidence across the capture.