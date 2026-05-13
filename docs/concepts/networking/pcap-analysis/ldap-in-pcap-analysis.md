# LDAP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

LDAP is useful in PCAP analysis when investigating Active Directory queries, domain enumeration, authentication attempts, user and group lookups, computer object lookups, service discovery, and directory-based reconnaissance.

LDAP traffic can expose searched base paths, filters, object names, attributes, usernames, domain information, group membership, and query results when the traffic is not encrypted.

> [!NOTE]
> Plain LDAP can expose readable directory queries and responses. LDAPS or LDAP protected through TLS hides most useful content from direct packet inspection.

## Core Concept

LDAP is used to query and modify directory services.

In Windows environments, LDAP is commonly used to interact with Active Directory.

A client connects to an LDAP server, usually a domain controller, binds to the directory, performs searches or other operations, and receives directory responses.

Example:

```text
Client -> LDAP Server: Bind Request
LDAP Server -> Client: Bind Response
Client -> LDAP Server: Search Request
LDAP Server -> Client: Search Result Entry
LDAP Server -> Client: Search Result Done
```

In this case:

```text
Bind Request: client starts LDAP authentication or session setup
Search Request: client queries the directory
Search Result Entry: directory object returned by the server
Search Result Done: LDAP search completed
```

LDAP is especially useful when a host is enumerating users, groups, computers, domain objects, or service-related information.

## Main LDAP Operations

| Operation | Practical meaning |
|---|---|
| Bind Request | Client authenticates or starts an LDAP session |
| Bind Response | Server replies to the bind attempt |
| Search Request | Client searches the directory |
| Search Result Entry | Server returns a matching object |
| Search Result Done | Server marks the search as complete |
| Modify Request | Client modifies a directory object |
| Add Request | Client adds a new directory object |
| Delete Request | Client deletes a directory object |
| Modify DN Request | Client renames or moves a directory object |
| Compare Request | Client compares an attribute value |
| Extended Request | Client performs an extended LDAP operation |
| Unbind Request | Client closes the LDAP session |

## Common LDAP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| Bind Request followed by successful Bind Response | LDAP bind succeeded |
| Bind Request followed by error | Authentication or bind failed |
| Repeated Bind failures | Wrong credentials, brute force, misconfiguration, or blocked access |
| Search Request after Bind | Client is querying the directory |
| Search Request with broad base DN | Possible domain-wide enumeration |
| Search Request with user-related filters | User enumeration or account lookup |
| Search Request with group-related filters | Group or membership enumeration |
| Search Request with computer-related filters | Computer or host enumeration |
| Search Request involving `servicePrincipalName` | SPN enumeration or Kerberos-related reconnaissance |
| Many Search Requests in sequence | Automated enumeration, application lookup behavior, or directory reconnaissance |
| Large number of Search Result Entries | Broad directory dump or large query result |
| LDAP errors after searches | Invalid base DN, insufficient rights, missing object, or malformed query |
| LDAP activity before SMB, Kerberos, or RDP | Directory lookup may be supporting authentication, service access, or lateral movement |

## Main LDAP Fields

| Field | Practical use |
|---|---|
| Message ID | Correlates LDAP requests and responses |
| Protocol operation | Shows the LDAP operation type |
| Bind name | Shows the account or DN used during bind when visible |
| Authentication type | Shows simple, SASL, or other authentication method |
| Base object | Shows where the LDAP search starts |
| Search scope | Shows whether the query targets base, one level, or subtree |
| Search filter | Shows the LDAP query filter |
| Result code | Shows whether the LDAP operation succeeded or failed |
| Matched DN | Shows the matched directory path when available |
| Error message | Shows LDAP failure details |
| Object name | Shows returned directory object names |
| Attributes | Shows returned object attributes when visible |

## Common LDAP Attributes

| Attribute | Practical use |
|---|---|
| sAMAccountName | Windows logon name |
| userPrincipalName | User logon name in UPN format |
| cn | Common name |
| distinguishedName | Full LDAP path of an object |
| objectClass | Object category such as user, group, or computer |
| memberOf | Group membership |
| member | Members of a group |
| servicePrincipalName | Services registered for Kerberos authentication |
| dNSHostName | Computer DNS hostname |
| operatingSystem | Operating system information for computer objects |
| lastLogon | Last logon timestamp |
| pwdLastSet | Password last set timestamp |
| userAccountControl | Account flags and properties |

## Wireshark Filters

```text
ldap
```

Shows LDAP traffic.

Useful to focus on directory service activity.

```text
ldap.bindRequest
```

Shows LDAP Bind Requests.

Useful to identify LDAP authentication or session setup attempts.

```text
ldap.bindResponse
```

Shows LDAP Bind Responses.

Useful to inspect whether LDAP bind attempts succeeded or failed.

```text
ldap.searchRequest
```

Shows LDAP Search Requests.

Useful to identify directory queries.

```text
ldap.searchResEntry
```

Shows LDAP Search Result Entries.

Useful to inspect objects returned by directory searches.

```text
ldap.searchResDone
```

Shows LDAP Search Result Done messages.

Useful to identify completed LDAP searches and result status.

```text
ldap.unbindRequest
```

Shows LDAP Unbind Requests.

Useful to identify LDAP session termination.

```text
ldap.messageID
```

Shows LDAP message IDs.

Useful to correlate requests and responses inside LDAP traffic.

```text
ldap.protocolOp
```

Shows LDAP protocol operation information.

Useful to identify the type of LDAP operation being performed.

```text
ldap.bindRequest.name
```

Shows LDAP bind names.

Useful to identify the account or distinguished name used during bind when visible.

```text
ldap.bindRequest.authentication.simple
```

Shows LDAP simple authentication values.

Useful to inspect simple bind authentication when visible in unencrypted LDAP traffic.

```text
ldap.searchRequest.baseObject
```

Shows LDAP search base objects.

Useful to identify where a directory search starts.

```text
ldap.searchRequest.baseObject contains "DC="
```

Shows LDAP searches using a domain component path.

Useful when identifying Active Directory domain search bases.

```text
ldap.searchRequest.filter
```

Shows LDAP search filters.

Useful to inspect what the client is searching for.

```text
ldap.searchRequest.filter contains "sAMAccountName"
```

Shows LDAP searches involving the `sAMAccountName` attribute.

Useful when checking for user account lookup or enumeration.

```text
ldap.searchRequest.filter contains "objectClass=user"
```

Shows LDAP searches involving user objects.

Useful when checking for user enumeration.

```text
ldap.searchRequest.filter contains "objectClass=group"
```

Shows LDAP searches involving group objects.

Useful when checking for group enumeration.

```text
ldap.searchRequest.filter contains "objectClass=computer"
```

Shows LDAP searches involving computer objects.

Useful when checking for host or machine account enumeration.

```text
ldap.searchRequest.filter contains "servicePrincipalName"
```

Shows LDAP searches involving service principal names.

Useful when checking for SPN enumeration or Kerberos-related reconnaissance.

```text
ldap.searchResEntry.objectName
```

Shows LDAP returned object names.

Useful to inspect directory objects returned by searches.

```text
ldap.resultCode
```

Shows LDAP result codes.

Useful to inspect whether LDAP operations succeeded or failed.

```text
ldap.resultCode != 0
```

Shows LDAP operations with non-success result codes.

Useful to identify failed binds, failed searches, permission issues, missing objects, or malformed requests.

```text
ldap.errorMessage
```

Shows LDAP error messages.

Useful to inspect failure details returned by the LDAP server.