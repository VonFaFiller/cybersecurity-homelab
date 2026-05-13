# Kerberos Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

Kerberos is useful in PCAP analysis when investigating Windows domain authentication, Active Directory activity, service access, failed logons, lateral movement, and suspicious authentication patterns.

Kerberos traffic can expose usernames, realms, requested services, ticket requests, authentication errors, encryption types, and domain controller interaction.

> [!NOTE]
> Kerberos does not usually expose plaintext passwords. It shows authentication flow, ticket activity, service access, and failure reasons.

## Core Concept

Kerberos is a ticket-based authentication protocol.

A client authenticates to a Key Distribution Center, usually a domain controller in Windows environments, and receives tickets that allow access to services without sending the password directly to those services.

A simplified Kerberos flow looks like this:

```text
Client -> KDC: AS-REQ
KDC -> Client: AS-REP
Client -> KDC: TGS-REQ
KDC -> Client: TGS-REP
Client -> Service: AP-REQ
Service -> Client: AP-REP
```

In this case:

```text
AS-REQ / AS-REP: client requests and receives a Ticket Granting Ticket
TGS-REQ / TGS-REP: client requests and receives a service ticket
AP-REQ / AP-REP: client presents the service ticket to the target service
```

In Active Directory investigations, Kerberos can help identify which user requested access to which service.

## Main Kerberos Message Types

| Message type | Value | Practical meaning |
|---|---:|---|
| AS-REQ | 10 | Client requests initial authentication from the KDC |
| AS-REP | 11 | KDC replies with initial authentication result |
| TGS-REQ | 12 | Client requests a service ticket |
| TGS-REP | 13 | KDC replies with a service ticket |
| AP-REQ | 14 | Client presents a ticket to a service |
| AP-REP | 15 | Service replies to the client |
| KRB-ERROR | 30 | Kerberos error response |

## Common Kerberos Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| AS-REQ followed by AS-REP | Initial Kerberos authentication succeeded |
| AS-REQ followed by KRB-ERROR | Initial authentication failed or requires additional steps |
| AS-REQ with pre-authentication required | Normal Kerberos behavior in many Windows environments |
| Repeated AS-REQ failures | Failed logons, wrong password, disabled account, or brute force |
| TGS-REQ followed by TGS-REP | Client obtained a ticket for a service |
| TGS-REQ for many services | Service enumeration, lateral movement, or normal enterprise activity |
| TGS-REQ for `krbtgt` | Ticket Granting Ticket-related activity |
| TGS-REQ for `cifs` | SMB/file share access |
| TGS-REQ for `ldap` | LDAP/domain query activity |
| TGS-REQ for `host` | Host-based service access |
| TGS-REQ for `HTTP` | Web service authentication |
| Kerberos errors after service ticket requests | Missing SPN, access issue, time skew, or authentication problem |
| High volume of Kerberos failures from one host | Possible brute force, misconfiguration, or compromised host activity |

## Main Kerberos Fields

| Field | Practical use |
|---|---|
| Message type | Identifies AS-REQ, AS-REP, TGS-REQ, TGS-REP, AP-REQ, AP-REP, or KRB-ERROR |
| Client name | Shows the user or machine account requesting authentication |
| Service name | Shows the requested service principal |
| Realm | Shows the Kerberos realm or AD domain |
| Error code | Shows why Kerberos authentication or ticket request failed |
| Encryption type | Shows the encryption type used or requested |
| Pre-authentication data | Shows whether pre-authentication data is present |
| Ticket | Indicates ticket-related Kerberos data |
| KDC options | Shows requested Kerberos ticket options |

## Common Kerberos Error Codes

| Error code | Meaning |
|---:|---|
| 6 | Client not found in Kerberos database |
| 7 | Server not found in Kerberos database |
| 18 | Client credentials revoked |
| 23 | Password expired |
| 24 | Pre-authentication failed |
| 25 | Pre-authentication required |
| 32 | Ticket expired |
| 37 | Clock skew too great |

## Wireshark Filters

```text
kerberos
```

Shows Kerberos traffic.

Useful to focus on domain authentication and ticket activity.

```text
kerberos.msg_type == 10
```

Shows AS-REQ messages.

Useful to identify initial authentication requests.

```text
kerberos.msg_type == 11
```

Shows AS-REP messages.

Useful to identify KDC replies to initial authentication requests.

```text
kerberos.msg_type == 12
```

Shows TGS-REQ messages.

Useful to identify service ticket requests.

```text
kerberos.msg_type == 13
```

Shows TGS-REP messages.

Useful to identify service ticket replies.

```text
kerberos.msg_type == 14
```

Shows AP-REQ messages.

Useful to identify tickets being presented to services.

```text
kerberos.msg_type == 15
```

Shows AP-REP messages.

Useful to identify service replies to Kerberos authentication.

```text
kerberos.msg_type == 30
```

Shows Kerberos error messages.

Useful to identify failed authentication, failed ticket requests, or Kerberos configuration issues.

```text
kerberos.CNameString
```

Shows Kerberos packets containing client name strings.

Useful to identify users or machine accounts involved in Kerberos activity.

```text
kerberos.CNameString contains "admin"
```

Shows Kerberos traffic where the client name contains a specific string.

Useful when searching for a known user, administrator account, or naming pattern.

```text
kerberos.SNameString
```

Shows Kerberos packets containing service name strings.

Useful to identify requested services.

```text
kerberos.SNameString contains "cifs"
```

Shows Kerberos traffic involving CIFS service names.

Useful when checking for SMB or file share authentication.

```text
kerberos.SNameString contains "ldap"
```

Shows Kerberos traffic involving LDAP service names.

Useful when checking for domain query authentication.

```text
kerberos.SNameString contains "krbtgt"
```

Shows Kerberos traffic involving the Ticket Granting Ticket service.

Useful when checking TGT-related activity.

```text
kerberos.realm
```

Shows Kerberos packets containing realm information.

Useful to identify the AD domain or Kerberos realm.

```text
kerberos.error_code
```

Shows Kerberos packets containing error codes.

Useful to inspect Kerberos failures.

```text
kerberos.error_code == 24
```

Shows Kerberos pre-authentication failed errors.

Useful to identify failed password attempts.

```text
kerberos.error_code == 25
```

Shows Kerberos pre-authentication required errors.

Useful to identify normal pre-authentication negotiation or accounts requiring pre-authentication.

```text
kerberos.error_code == 37
```

Shows Kerberos clock skew errors.

Useful to identify time synchronization problems affecting authentication.

```text
kerberos.etype
```

Shows Kerberos encryption type information.

Useful to inspect encryption types used or requested during Kerberos authentication.