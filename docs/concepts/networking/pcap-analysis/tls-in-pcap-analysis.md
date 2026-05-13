# TLS Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

TLS is common in modern PCAP analysis because many application protocols are encrypted.

When the payload is encrypted, TLS can still provide useful metadata such as server names, certificates, handshake behavior, protocol versions, cipher suites, alerts, and timing.

> [!NOTE]
> TLS usually hides the application-layer content, but it can still help identify which host contacted which service and which domain or certificate was involved.

## Core Concept

TLS provides encrypted communication between a client and a server.

Before encrypted application data is exchanged, the client and server perform a TLS handshake.

During the handshake, some metadata may still be visible.

Example:

```text
Client -> Server: Client Hello
Server -> Client: Server Hello
Server -> Client: Certificate
Client <-> Server: Encrypted Application Data
```

In this case:

```text
Client Hello: starts the TLS negotiation
Server Hello: confirms selected TLS parameters
Certificate: may expose the server identity
Encrypted Application Data: application content is no longer readable
```

In many investigations, the most useful TLS artifact is the SNI value.

SNI can reveal the domain name the client wanted to reach.

## Main TLS Handshake Messages

| Message | Practical meaning |
|---|---|
| Client Hello | Client starts TLS negotiation |
| Server Hello | Server replies and selects TLS parameters |
| Certificate | Server provides certificate information |
| Certificate Request | Server asks the client for a certificate |
| Client Key Exchange | Client contributes key exchange material in older TLS versions |
| Finished | Handshake phase is completed |
| Alert | TLS warning or error |
| Application Data | Encrypted application content |

## Common TLS Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| Client Hello followed by Server Hello | TLS negotiation started successfully |
| Client Hello with visible SNI | Domain contacted by the client may be recoverable |
| Client Hello without SNI | Domain may be unavailable, hidden, or not used |
| Certificate visible in handshake | Server identity may be recoverable from certificate fields |
| TLS Application Data after handshake | Encrypted traffic is being exchanged |
| TLS Alert after Client Hello | TLS negotiation failed or was rejected |
| Repeated Client Hello messages | Retry behavior, failed negotiation, blocked service, or unstable communication |
| Short TLS session with little data | Failed connection, probing, or quick callback |
| Regular TLS sessions at fixed intervals | Possible beaconing or normal background traffic |
| TLS to unusual IPs or ports | Possible suspicious encrypted communication |
| Many hosts contacting the same TLS endpoint | Shared service, update infrastructure, proxy, or possible centralized C2 |

## Main TLS Fields

| Field | Practical use |
|---|---|
| SNI | Reveals the requested server name when visible |
| Certificate Subject | Identifies the certificate owner or service |
| Certificate Issuer | Shows who issued the certificate |
| Certificate Validity | Shows certificate time range |
| TLS Version | Shows the TLS protocol version being negotiated |
| Cipher Suite | Shows the selected or offered cryptographic algorithms |
| ALPN | Can reveal the intended application protocol, such as HTTP/2 |
| Alert Description | Explains TLS errors or handshake failures |
| Application Data | Indicates encrypted payload exchange |

## Wireshark Filters

```text
tls.handshake.type == 1
```

Shows TLS Client Hello messages.

Useful to identify TLS connection attempts and client-side negotiation.

```text
tls.handshake.type == 2
```

Shows TLS Server Hello messages.

Useful to identify servers that responded to TLS negotiation.

```text
tls.handshake.type == 11
```

Shows TLS Certificate messages.

Useful to inspect certificate information when visible.

```text
tls.handshake.extensions_server_name
```

Shows TLS packets containing SNI.

Useful to identify requested domain names.

```text
tls.handshake.extensions_server_name contains "example"
```

Shows TLS packets where the SNI contains a specific string.

Useful when searching for a known domain or keyword.

```text
tls.handshake.version
```

Shows TLS handshake version information.

Useful to identify the TLS version involved in the negotiation.

```text
tls.record.version
```

Shows TLS record version information.

Useful when checking how TLS records are represented in the capture.

```text
tls.handshake.ciphersuite
```

Shows TLS cipher suite information.

Useful to inspect offered or selected cipher suites.

```text
tls.handshake.extensions_alpn_str
```

Shows ALPN values when present.

Useful to identify the application protocol negotiated over TLS, such as HTTP/2.

```text
tls.alert_message
```

Shows TLS alert messages.

Useful to identify TLS errors, rejected handshakes, or abnormal session termination.

```text
tls.alert_message.desc
```

Shows TLS alert descriptions.

Useful to understand why a TLS negotiation or session may have failed.

```text
tls.app_data
```

Shows encrypted TLS application data.

Useful to confirm that encrypted payload exchange occurred after the handshake.
