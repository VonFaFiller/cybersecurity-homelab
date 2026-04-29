# TCP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

TCP appears constantly in PCAP analysis.

Understanding TCP behavior helps determine whether a connection was established, refused, interrupted, unstable, or repeatedly attempted.

> [!NOTE]
> TCP analysis is often the first layer of confirmation before interpreting higher-level protocols such as HTTP, SMB, FTP, TLS, or database traffic.

## Core Concept

TCP is connection-oriented.

Before application data is exchanged, the client and server usually establish a connection through the three-way handshake:

```text
Client -> Server: SYN
Server -> Client: SYN/ACK
Client -> Server: ACK
```

If the handshake completes, the connection was established at the transport layer.

If it does not complete, the packet sequence can still explain what happened.

The host sending the first SYN is usually the client.

The host replying with SYN/ACK is usually the server.

This is useful because IP direction alone can be misleading when there are many conversations.

Example:

```text
10.0.0.5 -> 10.0.0.10: SYN
10.0.0.10 -> 10.0.0.5: SYN/ACK
10.0.0.5 -> 10.0.0.10: ACK
```

In this case:

```text
Client: 10.0.0.5
Server: 10.0.0.10
```

## Main TCP Flags

| Flag | Meaning |
|---|---|
| SYN | Starts a connection attempt |
| ACK | Acknowledges received data or connection state |
| FIN | Gracefully closes a connection |
| RST | Resets or refuses a connection |
| PSH | Pushes data to the application quickly |
| URG | Marks urgent data, rarely useful in normal PCAP analysis |

## Common TCP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| SYN → SYN/ACK → ACK | Connection established |
| SYN → RST/ACK | Port closed or connection refused |
| SYN repeated with no SYN/ACK | No response, filtered service, packet loss, or unreachable host |
| SYN → SYN/ACK repeated | Client may not be completing the handshake or packets may be missing |
| RST after data exchange | Session reset or application/server closed the connection abruptly |
| FIN → ACK → FIN → ACK | Graceful connection close |
| Retransmissions | Packet loss, delay, congestion, or unstable communication |
| Many SYNs to many ports | Possible port scan |
| Many SYNs to many hosts | Possible host discovery or network scan |

## Wireshark Filters


```text
tcp.flags.syn == 1
```

Shows packets with the SYN flag set.

This includes both initial SYN packets and SYN/ACK replies.

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Shows initial connection attempts only.

Useful to identify which host is trying to start a connection.

```text
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

Shows SYN/ACK replies.

Useful to identify services that are responding to connection attempts.

```text
tcp.flags.reset == 1
```

Shows TCP reset packets.

Useful to identify refused, reset, or abruptly closed connections.

```text
tcp.flags.fin == 1
```

Shows packets involved in graceful TCP connection closure.

```text
tcp.analysis.retransmission
```

Shows packets that Wireshark marks as TCP retransmissions.

Useful to spot packet loss, delay, congestion, or unstable communication.

```text
tcp.analysis.lost_segment
```

Shows cases where Wireshark believes a TCP segment is missing from the capture.

Useful when a stream looks incomplete or inconsistent.

```text
tcp.analysis.flags
```

Shows packets with TCP analysis warnings or notable conditions.

Useful as a broad filter for retransmissions, lost segments, duplicate ACKs, out-of-order packets, and similar issues.


