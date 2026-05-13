# NTLM Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

NTLM is useful in PCAP analysis when investigating Windows authentication, SMB access, HTTP authentication, proxy authentication, failed logons, lateral movement, and legacy authentication behavior.

NTLM traffic can expose usernames, domains, hostnames, authentication flow, challenge-response exchanges, and which services were involved in the authentication attempt.

> [!NOTE]
> NTLM does not expose plaintext passwords. It uses a challenge-response mechanism, but the username, domain, workstation, and authentication sequence may still be visible.

## Core Concept

NTLM is a challenge-response authentication protocol.

The client does not send the password directly.

Instead, authentication usually follows a three-message exchange:

```text
Client -> Server: NTLM Negotiate
Server -> Client: NTLM Challenge
Client -> Server: NTLM Authenticate
```

In this case:

```text
Negotiate: client proposes NTLM capabilities
Challenge: server sends a challenge value
Authenticate: client responds using credential-derived material
```

NTLM is commonly seen inside other protocols, especially SMB and HTTP.

Example over SMB:

```text
Client -> Server: SMB Session Setup + NTLM Negotiate
Server -> Client: SMB Session Setup + NTLM Challenge
Client -> Server: SMB Session Setup + NTLM Authenticate
```

In this case:

```text
SMB: carrier protocol
NTLMSSP: authentication mechanism inside the SMB session
```

## Main NTLMSSP Message Types

| Message type | Value | Practical meaning |
|---|---:|---|
| NEGOTIATE_MESSAGE | 1 | Client starts NTLM authentication and proposes options |
| CHALLENGE_MESSAGE | 2 | Server replies with challenge and target information |
| AUTHENTICATE_MESSAGE | 3 | Client submits authentication response |

## Common NTLM Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| Negotiate → Challenge → Authenticate | NTLM authentication exchange occurred |
| Authenticate followed by protocol success | NTLM authentication likely succeeded |
| Authenticate followed by access denied or logon failure | NTLM authentication failed |
| Repeated NTLM authentication failures | Wrong password, brute force, misconfiguration, or blocked account |
| NTLM inside SMB Session Setup | Windows file share or remote service authentication |
| NTLM inside HTTP 401 flow | Web or proxy authentication |
| Same user authenticating to multiple hosts | Possible lateral movement, administration, or automated access |
| Many users authenticating from one host | Possible credential testing or shared system behavior |
| Machine account authentication ending with `$` | Computer account authentication |
| NTLM used where Kerberos was expected | Possible fallback, IP-based access, missing SPN, or legacy configuration |
| NTLM followed by SMB access to `ADMIN$`, `C$`, or `IPC$` | Possible remote administration or lateral movement |

## Main NTLM Fields

| Field | Practical use |
|---|---|
| Message type | Identifies Negotiate, Challenge, or Authenticate |
| Username | Shows the account used during authentication |
| Domain | Shows the domain or local authentication context |
| Hostname / Workstation | Shows the client workstation name when visible |
| Challenge | Shows the server challenge value |
| NTLM response | Shows challenge-response authentication material |
| NTLMv2 response | Shows NTLMv2 authentication response material |
| Target name | Shows the server or domain target name |
| Negotiation flags | Shows requested or supported NTLM capabilities |
| Session key | May appear as encrypted session-related material |

## Wireshark Filters

```text
ntlmssp
```

Shows NTLMSSP traffic.

Useful to focus on NTLM authentication exchanges.

```text
ntlmssp.messagetype == 1
```

Shows NTLM Negotiate messages.

Useful to identify clients starting NTLM authentication.

```text
ntlmssp.messagetype == 2
```

Shows NTLM Challenge messages.

Useful to identify servers responding to NTLM authentication attempts.

```text
ntlmssp.messagetype == 3
```

Shows NTLM Authenticate messages.

Useful to identify submitted NTLM authentication responses.

```text
ntlmssp.auth.username
```

Shows NTLM Authenticate messages containing usernames.

Useful to identify accounts used during NTLM authentication.

```text
ntlmssp.auth.username contains "admin"
```

Shows NTLM authentication where the username contains a specific string.

Useful when searching for administrator accounts or known users.

```text
ntlmssp.auth.domain
```

Shows NTLM Authenticate messages containing domain values.

Useful to identify the domain or local authentication context.

```text
ntlmssp.auth.hostname
```

Shows NTLM Authenticate messages containing client workstation names.

Useful to identify the host performing authentication.

```text
ntlmssp.challenge
```

Shows NTLM Challenge values.

Useful to inspect server challenge messages.

```text
ntlmssp.ntlmv2_response
```

Shows NTLMv2 response data.

Useful to identify NTLMv2 authentication material.

```text
ntlmssp.target_name
```

Shows NTLM target name information.

Useful to identify the server, domain, or authentication target when visible.

```text
ntlmssp.neg_flags
```

Shows NTLM negotiation flags.

Useful to inspect negotiated NTLM capabilities.

```text
smb2 && ntlmssp
```

Shows NTLM authentication inside SMB2 traffic.

Useful to investigate Windows file share or remote administration authentication.

```text
http && ntlmssp
```

Shows NTLM authentication inside HTTP traffic.

Useful to investigate web or proxy authentication using NTLM.