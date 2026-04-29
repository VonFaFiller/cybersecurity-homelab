# RDP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

RDP is useful in PCAP analysis when investigating remote desktop access, Windows administration, exposed remote services, brute-force attempts, lateral movement, and suspicious interactive access.

RDP traffic is usually encrypted after negotiation, so the actual desktop activity, typed commands, credentials, and screen content are normally not visible in Wireshark.

> [!NOTE]
> RDP analysis in Wireshark is mostly based on metadata, connection behavior, negotiation details, timing, packet volume, and session direction.

## Core Concept

RDP provides remote graphical access to a Windows system.

A client connects to an RDP server, negotiates the connection, establishes security, and then exchanges encrypted session data.

Example:

```text
Client -> Server: TCP connection
Client -> Server: RDP negotiation request
Server -> Client: RDP negotiation response
Client <-> Server: Security negotiation
Client <-> Server: Encrypted RDP session data
```

In this case:

```text
TCP connection: transport-layer connection to the RDP service
RDP negotiation: client and server agree on connection/security parameters
Security negotiation: encryption and authentication-related setup
Encrypted session data: remote desktop activity is no longer readable
```

The client is usually the host initiating the connection.

The server is usually the host listening for RDP access.

## Main RDP Components

| Component | Practical meaning |
|---|---|
| TPKT | Transport wrapper commonly used by RDP |
| COTP / X.224 | Connection setup layer used before higher RDP negotiation |
| RDP Negotiation | Determines selected security protocol |
| TLS / CredSSP | Common security/authentication layer for modern RDP |
| MCS / T.125 | Multipoint communication setup used by RDP |
| Virtual Channels | Carry redirected devices, clipboard, drive mapping, audio, or other RDP features |
| Encrypted Data | Remote desktop session content after security setup |

## Common RDP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| TCP connection to RDP service followed by RDP negotiation | RDP connection attempt |
| RDP negotiation followed by encrypted traffic | RDP session likely established |
| Repeated short RDP connections | Failed login attempts, probing, brute force, or unstable access |
| Many clients connecting to one RDP server | Exposed shared remote access point or possible brute-force activity |
| One host connecting to many RDP servers | Possible lateral movement, scanning, or administrative activity |
| Long RDP session with steady traffic | Interactive remote desktop session or active remote administration |
| Large RDP traffic volume | Active desktop usage, file transfer through redirection, clipboard use, or heavy GUI activity |
| RDP connection from unusual internal host | Possible compromised host or unauthorized access |
| RDP connection from external IP | Exposed remote access, VPN-less administration, or attack surface |
| RDP negotiation failure or disconnect | Failed setup, rejected security protocol, policy issue, or interrupted connection |

## Main RDP Fields

| Field | Practical use |
|---|---|
| Negotiation Type | Shows the type of RDP negotiation message |
| Selected Protocol | Shows the security protocol selected by the server |
| Encryption Method | Shows negotiated encryption method when visible |
| Encryption Level | Shows negotiated encryption level when visible |
| Server Random | Part of older RDP security negotiation |
| Encrypted Client Random | Part of older RDP security negotiation |
| Domain | May expose domain value in older or less protected negotiation |
| Username | May expose username value in older or less protected negotiation |
| Client Name | May expose the client hostname |
| Desktop Width / Height | Shows requested remote desktop resolution |
| Virtual Channel | May indicate redirected devices or additional RDP features |
| Encrypted Data | Indicates encrypted RDP payload |

## Wireshark Filters

```text
rdp
```

Shows RDP traffic when Wireshark identifies the protocol.

Useful to focus on RDP negotiation and decoded RDP fields.

```text
t125
```

Shows T.125 / MCS traffic.

Useful to include conference setup and virtual channel establishment related to RDP.

```text
cotp
```

Shows COTP / X.224 traffic.

Useful to inspect early connection setup used before higher RDP negotiation.

```text
cotp && !(cotp.type == 0x06 || cotp.type == 0x0f)
```

Shows COTP traffic excluding normal data and acknowledge TPDUs.

Useful to focus on special COTP packets related to connection setup or control behavior.

```text
tpkt
```

Shows TPKT traffic.

Useful because RDP is commonly carried through TPKT before higher-level RDP structures.

```text
rdp.negType
```

Shows RDP negotiation type information.

Useful to inspect RDP negotiation messages.

```text
rdp.selectedProtocol
```

Shows the protocol selected by the server during RDP negotiation.

Useful to identify whether standard RDP security, TLS, or CredSSP-style security was selected when visible.

```text
rdp.encryptionMethod
```

Shows RDP encryption method information.

Useful to inspect older RDP security negotiation.

```text
rdp.encryptionLevel
```

Shows RDP encryption level information.

Useful to inspect older RDP security settings.

```text
rdp.clientName
```

Shows RDP packets containing a client name value.

Useful to identify the connecting workstation when visible.

```text
rdp.userName
```

Shows RDP packets containing a username value.

Useful to identify the account name when visible.

```text
rdp.domain
```

Shows RDP packets containing a domain value.

Useful to identify the domain or local authentication context when visible.

```text
rdp.desktop.width
```

Shows requested desktop width.

Useful to inspect RDP client display settings when visible.

```text
rdp.desktop.height
```

Shows requested desktop height.

Useful to inspect RDP client display settings when visible.

```text
rdp.encryptedData
```

Shows encrypted RDP data.

Useful to confirm that protected RDP session data is being exchanged.