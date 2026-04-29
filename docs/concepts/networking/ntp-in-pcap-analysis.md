# NTP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

NTP is useful in PCAP analysis when investigating time synchronization, repeated outbound traffic, local infrastructure services, suspicious external time servers, and timestamp consistency across hosts.

NTP is usually background traffic, but it can help identify which systems are synchronizing time and whether a host is contacting expected or unexpected time sources.

> [!NOTE]
> NTP is rarely the main evidence in an investigation, but time synchronization can matter when correlating activity across multiple hosts or logs.

## Core Concept

NTP synchronizes system time between a client and a time server.

A client sends an NTP request to a server.

The server replies with timing information that helps the client adjust or validate its clock.

Example:

```text
Client -> NTP Server: NTP request
NTP Server -> Client: NTP response
```

In this case:

```text
NTP request: client asks for time synchronization data
NTP response: server provides time synchronization data
```

NTP normally uses UDP.

## Main NTP Modes

| Mode | Meaning | Practical use |
|---:|---|---|
| 1 | Symmetric Active | Peer-to-peer time synchronization |
| 2 | Symmetric Passive | Reply to symmetric active mode |
| 3 | Client | Client request to an NTP server |
| 4 | Server | Server response to an NTP client |
| 5 | Broadcast | Server broadcasts time information |
| 6 | Control | NTP control message |
| 7 | Private | Implementation-specific or legacy/private use |

## Common NTP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| Client mode followed by Server mode | Normal NTP request/response |
| Repeated NTP requests to same server | Normal time synchronization, retry behavior, or polling |
| NTP requests with no response | Server unreachable, blocked traffic, packet loss, or incomplete capture |
| Many hosts using same NTP server | Shared enterprise time source or gateway-provided configuration |
| One host contacting unusual external NTP servers | Misconfiguration, user-configured time source, malware behavior, or suspicious outbound traffic |
| NTP traffic at regular intervals | Normal polling or scheduled synchronization |
| NTP control/private mode traffic | Administrative, diagnostic, legacy, or suspicious NTP activity |
| Large or unusual NTP packets | Possible malformed traffic, abuse, amplification-related behavior, or non-standard use |
| NTP timestamps inconsistent with capture timeline | Possible clock drift, bad host time, or timestamp interpretation issue |

## Main NTP Fields

| Field | Practical use |
|---|---|
| Mode | Shows whether the packet is client, server, control, or another NTP mode |
| Version | Shows the NTP version |
| Stratum | Shows distance from the reference time source |
| Poll interval | Shows requested polling interval |
| Precision | Shows clock precision |
| Root delay | Shows delay to the primary reference source |
| Root dispersion | Shows estimated error relative to the reference source |
| Reference ID | Identifies the reference clock or upstream time source |
| Reference timestamp | Last time the server clock was set or corrected |
| Origin timestamp | Time from the client request copied into the server response |
| Receive timestamp | Time when the server received the client request |
| Transmit timestamp | Time when the packet was sent |

## Wireshark Filters

```text
ntp
```

Shows NTP traffic.

Useful to focus on time synchronization activity.

```text
ntp.flags.mode == 3
```

Shows NTP client-mode requests.

Useful to identify hosts requesting time synchronization.

```text
ntp.flags.mode == 4
```

Shows NTP server-mode responses.

Useful to identify time servers replying to clients.

```text
ntp.flags.mode == 6
```

Shows NTP control messages.

Useful to identify administrative or diagnostic NTP activity.

```text
ntp.flags.mode == 7
```

Shows NTP private-mode messages.

Useful to identify legacy, implementation-specific, or suspicious NTP behavior.

```text
ntp.flags.version
```

Shows NTP version information.

Useful to inspect which NTP version is being used.

```text
ntp.stratum
```

Shows NTP stratum values.

Useful to understand how close the time server is to a reference source.

```text
ntp.stratum == 0
```

Shows invalid or unspecified stratum values.

Useful to identify unsynchronized, invalid, or special NTP responses.

```text
ntp.stratum > 15
```

Shows invalid high stratum values.

Useful to identify unusable or abnormal time synchronization responses.

```text
ntp.refid`
```

Shows NTP reference ID information.

Useful to identify reference clocks or upstream time sources.

```text
ntp.xmt`
```

Shows NTP transmit timestamp information.

Useful to inspect when an NTP packet was sent.

```text
ntp.recv`
```

Shows NTP receive timestamp information.

Useful to inspect when the server received the request.

```text
ntp.org`
```

Shows NTP origin timestamp information.

Useful to correlate server responses with client requests.

```text
ntp.rootdelay`
```

Shows NTP root delay information.

Useful to inspect delay to the primary reference source.

```text
ntp.rootdispersion`
```

Shows NTP root dispersion information.

Useful to inspect estimated time error relative to the reference source.