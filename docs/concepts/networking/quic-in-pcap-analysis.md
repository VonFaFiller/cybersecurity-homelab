# QUIC Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

QUIC is useful in PCAP analysis when investigating modern encrypted web traffic, HTTP/3 activity, UDP/443 connections, repeated encrypted callbacks, browser traffic, and malware or application communication that avoids classic TCP/TLS behavior.

QUIC commonly appears as encrypted traffic over UDP, especially on port 443.

> [!NOTE]
> QUIC does not use the TCP three-way handshake. It runs over UDP and includes its own connection establishment, TLS 1.3 handshake, encryption, stream handling, and connection management.

## Core Concept

QUIC is a transport protocol that runs over UDP.

It is commonly used to carry HTTP/3 traffic.

Unlike HTTPS over TCP, QUIC combines transport setup and TLS 1.3 encryption inside UDP-based communication.

Example:

```text
Client -> Server: QUIC Initial
Server -> Client: QUIC Initial / Handshake
Client <-> Server: QUIC Handshake
Client <-> Server: Protected QUIC traffic
```

In this case:

```text
QUIC Initial: starts the QUIC connection
Handshake: negotiates encryption and connection parameters
Protected traffic: application data is encrypted
```

QUIC can still expose useful metadata such as version, connection IDs, packet types, retry behavior, server name when visible, and HTTP/3 indicators when decoded.

## Main QUIC Packet Types

| Packet type | Practical meaning |
|---|---|
| Initial | Starts the QUIC connection and carries early handshake data |
| 0-RTT | Sends early data before handshake completion when supported |
| Handshake | Carries handshake data after Initial phase |
| Retry | Server asks the client to retry with a token |
| Version Negotiation | Server indicates that the client's QUIC version is unsupported |
| Short Header Packet | Protected packet used after connection establishment |

## Common QUIC Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| QUIC Initial followed by Handshake packets | QUIC negotiation started successfully |
| Initial packets with no server response | No response, blocked UDP/443, unsupported service, or incomplete capture |
| Retry packet after Initial | Server requires address validation before continuing |
| Version Negotiation packet | Client and server did not agree on QUIC version |
| Repeated Initial packets | Retry behavior, blocked communication, packet loss, or failed negotiation |
| Protected QUIC packets after handshake | Encrypted application data is being exchanged |
| UDP/443 traffic decoded as QUIC | Possible HTTP/3 or QUIC-based application traffic |
| UDP/443 traffic not decoded as QUIC | Could be unsupported QUIC version, encrypted/obfuscated UDP traffic, or non-QUIC UDP service |
| Long regular QUIC sessions | Browser activity, streaming, application traffic, or persistent encrypted communication |
| Short repeated QUIC sessions | Failed attempts, beaconing, probing, or application retries |
| QUIC to unusual external IPs | Possible suspicious encrypted communication |
| HTTP/3 over QUIC | Web traffic carried over QUIC instead of TCP/TLS |

## Main QUIC Fields

| Field | Practical use |
|---|---|
| Version | Shows the QUIC version used or attempted |
| Destination Connection ID | Identifies the connection from the receiver side |
| Source Connection ID | Identifies the connection from the sender side |
| Packet Type | Shows Initial, Handshake, Retry, or other QUIC packet types |
| Packet Number | Helps follow QUIC packet order |
| Token | Used during address validation and Retry behavior |
| Frame Type | Shows the type of QUIC frame when decoded |
| Stream ID | Identifies streams inside a QUIC connection when visible |
| Crypto Data | Carries TLS handshake data inside QUIC |
| Protected Payload | Indicates encrypted QUIC data |

## QUIC and HTTP/3

QUIC is often seen together with HTTP/3.

HTTP/3 uses QUIC as its transport layer instead of TCP.

Example:

```text
HTTP/1.1 or HTTP/2 over TLS: TCP -> TLS -> HTTP
HTTP/3 over QUIC: UDP -> QUIC -> HTTP/3
```

In PCAP analysis, this means that normal HTTP filters may not show the same readable request/response content.

Most of the useful information may come from QUIC metadata, TLS handshake metadata, DNS, SNI when visible, IP addresses, timing, and traffic volume.

## Wireshark Filters

```text
quic
```

Shows QUIC traffic.

Useful to focus on QUIC connections and encrypted UDP-based sessions.

```text
quic.version
```

Shows QUIC packets containing version information.

Useful to identify which QUIC version is being used or negotiated.

```text
quic.long.packet_type
```

Shows QUIC long-header packet type information.

Useful to identify Initial, 0-RTT, Handshake, Retry, or version negotiation behavior.

```text
quic.long.packet_type == 0
```

Shows QUIC Initial packets.

Useful to identify QUIC connection attempts.

```text
quic.long.packet_type == 1
```

Shows QUIC 0-RTT packets.

Useful to identify early data sent before full handshake completion.

```text
quic.long.packet_type == 2
```

Shows QUIC Handshake packets.

Useful to identify the handshake phase.

```text
quic.long.packet_type == 3
```

Shows QUIC Retry packets.

Useful to identify address validation or retry behavior.

```text
quic.dcid
```

Shows QUIC destination connection IDs.

Useful to follow QUIC connections even when IP/port tuples change.

```text
quic.scid
```

Shows QUIC source connection IDs.

Useful to correlate packets belonging to the same QUIC connection.

```text
quic.packet_number
```

Shows QUIC packet numbers.

Useful to inspect packet ordering inside a QUIC connection.

```text
quic.frame_type
```

Shows QUIC frame type information.

Useful to inspect decoded QUIC frame behavior.

```text
quic.stream.stream_id
```

Shows QUIC stream IDs.

Useful to inspect stream-level activity inside a QUIC connection when visible.

```text
tls.handshake.extensions_server_name
```

Shows visible SNI values inside TLS handshake data when Wireshark can decode them.

Useful to identify the requested server name when available.

```text
tls.handshake.extensions_server_name contains "example"
```

Shows QUIC/TLS handshake traffic where the visible SNI contains a specific string.

Useful when searching for a known domain or suspicious keyword.

```text
http3
```

Shows HTTP/3 traffic when Wireshark decodes it.

Useful to identify web traffic carried over QUIC.

