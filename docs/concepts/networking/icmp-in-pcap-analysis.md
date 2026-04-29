# ICMP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

ICMP is useful in PCAP analysis when investigating host reachability, ping activity, scanning behavior, unreachable services, routing issues, traceroute-like behavior, and network error messages.

ICMP often appears when a host is testing connectivity or when another protocol fails to reach its destination.

> [!NOTE]
> ICMP does not use TCP or UDP ports. It uses message types and codes instead.

## Core Concept

ICMP is used to send network control and error messages.

A host can send an ICMP Echo Request to test whether another host is reachable.

If the destination replies, it sends an ICMP Echo Reply.

Example:

```text
Host A -> Host B: Echo Request
Host B -> Host A: Echo Reply
```

In this case:

```text
Echo Request: reachability test
Echo Reply: host responded
```

ICMP can also report failed communication.

Example:

```text
Client -> Server: UDP packet
Server -> Client: ICMP Destination Unreachable
```

In this case:

```text
UDP packet: original attempted communication
ICMP Destination Unreachable: network or host reported failure
```

## Main ICMP Message Types

| Type | Message | Practical meaning |
|---:|---|---|
| 0 | Echo Reply | Response to a ping request |
| 3 | Destination Unreachable | Destination, port, protocol, or route was unreachable |
| 5 | Redirect | Router suggests a better route |
| 8 | Echo Request | Ping request |
| 11 | Time Exceeded | TTL expired, often seen in traceroute-like activity |
| 12 | Parameter Problem | Invalid or problematic IP header |
| 13 | Timestamp Request | Legacy timestamp request |
| 14 | Timestamp Reply | Legacy timestamp reply |

## Common ICMP Codes

| Type | Code | Meaning |
|---:|---:|---|
| 3 | 0 | Network unreachable |
| 3 | 1 | Host unreachable |
| 3 | 2 | Protocol unreachable |
| 3 | 3 | Port unreachable |
| 3 | 4 | Fragmentation needed |
| 3 | 13 | Communication administratively prohibited |
| 11 | 0 | TTL exceeded in transit |
| 11 | 1 | Fragment reassembly time exceeded |

## Common ICMP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| Echo Request followed by Echo Reply | Host is reachable |
| Echo Request with no Echo Reply | No response, filtered ICMP, unreachable host, or missing capture data |
| Many Echo Requests to many hosts | Possible ping sweep or host discovery |
| Repeated Echo Requests to one host | Connectivity test, monitoring, retry behavior, or suspicious probing |
| Destination Unreachable after UDP traffic | UDP service may be closed, unreachable, or blocked |
| Port Unreachable after UDP packet | Destination UDP port is likely closed |
| Time Exceeded messages | Possible traceroute-like behavior or routing loop |
| ICMP Redirect | Possible routing change, misconfiguration, or suspicious network behavior |
| Large ICMP payloads | Possible tunneling, data transfer, or abnormal ICMP usage |
| Regular ICMP traffic at fixed intervals | Monitoring, keepalive behavior, or possible beaconing |

## Main ICMP Fields

| Field | Practical use |
|---|---|
| Type | Identifies the ICMP message category |
| Code | Adds detail to the ICMP type |
| Identifier | Helps match Echo Requests and Echo Replies |
| Sequence number | Helps track ICMP request/reply order |
| Checksum | Used to detect corruption |
| Data | Payload carried inside the ICMP message |
| Response time | Helps measure time between request and reply when available |

## Wireshark Filters

```text
icmp.type == 8
```

Shows ICMP Echo Requests.

Useful to identify ping requests and host discovery attempts.

```text
icmp.type == 0
```

Shows ICMP Echo Replies.

Useful to identify hosts that responded to ping requests.

```text
icmp.type == 3
```

Shows ICMP Destination Unreachable messages.

Useful to identify failed communication, unreachable hosts, blocked paths, or closed UDP ports.

```text
icmp.type == 3 && icmp.code == 3
```

Shows ICMP Port Unreachable messages.

Useful to identify UDP traffic sent to closed ports.

```text
icmp.type == 3 && icmp.code == 13
```

Shows communication administratively prohibited messages.

Useful to identify traffic blocked by filtering or policy.

```text
icmp.type == 11
```

Shows ICMP Time Exceeded messages.

Useful to identify traceroute-like behavior, TTL expiration, or routing issues.

```text
icmp.type == 5
```

Shows ICMP Redirect messages.

Useful to identify route redirection behavior.

```text
icmp.ident
```

Shows ICMP packets containing identifier values.

Useful to correlate Echo Requests and Echo Replies.

```text
icmp.seq
```

Shows ICMP packets containing sequence numbers.

Useful to reconstruct ping order and repeated ICMP activity.

```text
icmp.data
```

Shows ICMP packets containing payload data.

Useful to inspect ICMP payload content.

```text
icmp.data contains "example"
```

Shows ICMP packets where the payload contains a specific string.

Useful when searching for cleartext indicators, tunneling, or abnormal ICMP payloads.