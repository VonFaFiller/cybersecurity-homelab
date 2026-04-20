# TCP Connection Behavior in PCAP Analysis

## Why This Matters

TCP appears constantly in PCAP analysis.

Understanding TCP behavior helps determine whether a connection was established, refused, interrupted, unstable, or repeatedly attempted.

> [!NOTE]
>TCP analysis is often the first layer of confirmation before interpreting higher-level protocols such as HTTP, SMB, FTP, TLS, or database traffic.
## Core Concept

TCP is connection-oriented.

Before application data is exchanged, the client and server usually establish a connection with <u>the three-way handshake:</u>

```text
Client -> Server: SYN
Server -> Client: SYN/ACK
Client -> Server: ACK
````

If the handshake completes, the connection was established.

If it does not complete, the packet sequence can still explain what happened.

## Main TCP Flags

| Flag | Meaning                                             |
| ---- | --------------------------------------------------- |
| SYN  | Starts a connection attempt                         |
| ACK  | Acknowledges received data or connection state      |
| FIN  | Gracefully closes a connection                      |
| RST  | Resets or refuses a connection                      |
| PSH  | Pushes data to the application quickly              |
| URG  | Marks urgent data, rarely useful in normal analysis |

## Common TCP Patterns in PCAPs

| Pattern                      | Practical interpretation                                           |
| ---------------------------- | ------------------------------------------------------------------ |
| SYN → SYN/ACK → ACK          | Connection established                                             |
| SYN → RST/ACK                | Port closed or connection refused                                  |
| SYN repeated with no SYN/ACK | No response, filtered service, packet loss, or unreachable host    |
| SYN → SYN/ACK repeated       | Client may not be completing the handshake or packets are missing  |
| RST after data exchange      | Session reset or application/server closed the connection abruptly |
| FIN → ACK → FIN → ACK        | Graceful connection close                                          |
| Retransmissions              | Packet loss, delay, congestion, or unstable communication          |
| Many SYNs to many ports      | Possible port scan                                                 |
| Many SYNs to many hosts      | Possible host discovery or network scan                            |

## Wireshark Filters

```text
tcp
```

```text
tcp.flags.syn == 1
```
Shows TCP packets with the SYN flag set.

Useful to identify connection attempts.

```text
tcp.flags.reset == 1
```

Shows TCP packets with the RST flag set.

Useful to identify refused, reset, or abruptly closed connections.

```text
tcp.analysis.retransmission
```

Shows packets that Wireshark marks as TCP retransmissions.

Useful to spot packet loss, delay, congestion, or unstable communication.

```text
tcp.analysis.lost_segment
```

```text
tcp.analysis.flags
```

Shows packets with TCP analysis warnings or notable conditions.

Useful as a broad filter for retransmissions, lost segments, duplicate ACKs, out-of-order packets, and similar issues.

```text
tcp.port == 80
```

```text
tcp.stream eq 0
```

## Client and Server Direction

The host sending the first SYN is usually the client.

The host replying with SYN/ACK is usually the server.

This is useful because IP direction alone can be misleading when there are many conversations.

## Important Caution

A packet sent to a service does not prove that communication succeeded.

For example:

```text
Client -> Server: SYN
Client -> Server: SYN
Client -> Server: SYN
```

This only shows repeated connection attempts.

It does not prove that the service accepted the connection.

To confirm that the connection succeeded, I need to see the handshake complete or later application-layer data.




