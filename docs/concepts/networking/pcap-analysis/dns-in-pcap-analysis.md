# DNS Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

DNS is often one of the fastest ways to understand what a host is trying to reach.

In PCAP analysis, DNS can help identify contacted domains, suspicious lookups, failed resolutions, possible C2 infrastructure, malware callbacks, and domains that later appear in HTTP or TLS traffic.

> [!NOTE]
> DNS does not prove that a connection succeeded. It shows that a host tried to resolve a name.

## Core Concept

DNS translates domain names into IP addresses.

A client sends a DNS query to a DNS server.

The DNS server replies with one or more records, depending on the requested domain and record type.

Example:

```text
Client -> DNS Server: Query A example.com
DNS Server -> Client: Response A example.com = 93.184.216.34
```

In this case:

```text
Queried domain: example.com
Resolved IP: 93.184.216.34
```

After this, the client may try to connect to the resolved IP using another protocol such as HTTP, HTTPS, SMB, FTP, or something else.

## Main DNS Record Types

| Record type | Meaning |
|---|---|
| A | Resolves a domain to an IPv4 address |
| AAAA | Resolves a domain to an IPv6 address |
| CNAME | Alias from one domain name to another |
| MX | Mail server record |
| NS | Name server record |
| TXT | Text record, sometimes abused or useful during investigations |
| PTR | Reverse DNS lookup from IP to name |

## Common DNS Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| Query followed by A/AAAA response | Domain successfully resolved |
| Query followed by NXDOMAIN | Domain does not exist |
| Many failed queries | Possible typo, broken configuration, malware, or DGA-like behavior |
| Many random-looking domains | Possible DGA or suspicious automated activity |
| Repeated queries to the same domain | Possible beaconing, retry behavior, or normal application behavior |
| DNS query followed by HTTP/TLS connection to resolved IP | Domain may be related to later network activity |
| TXT queries with unusual content | Possible tunneling, validation, or suspicious data exchange |
| Long subdomains | Possible tracking, tunneling, malware, or encoded data |
| Internal host querying external suspicious domains | Possible compromised host activity |

## DNS Response Codes

| Response code | Meaning |
|---|---|
| NoError | Query completed successfully |
| NXDOMAIN | Domain does not exist |
| ServFail | DNS server failed to complete the request |
| Refused | DNS server refused the query |
| FormErr | Malformed DNS query |

## Wireshark Filters

```text
dns.flags.response == 0
```

Shows DNS queries.

Useful to identify which domains a host is trying to resolve.

```text
dns.flags.response == 1
```

Shows DNS responses.

Useful to identify resolved IP addresses and DNS result codes.

```text
dns.qry.name
```

Shows packets containing queried domain names.

Useful when focusing on requested domains.

```text
dns.qry.name contains "example"
```

Shows DNS queries containing a specific string.

Useful when searching for a known domain, keyword, or suspicious pattern.

```text
dns.flags.rcode != 0
```

Shows DNS responses with non-success result codes.

Useful to find failed, refused, or problematic DNS lookups.

```text
dns.flags.rcode == 3
```

Shows NXDOMAIN responses.

Useful to identify domains that do not exist.

```text
dns.a
```

Shows DNS packets containing IPv4 A records.

Useful to identify domains resolved to IPv4 addresses.

```text
dns.aaaa
```

Shows DNS packets containing IPv6 AAAA records.

Useful to identify domains resolved to IPv6 addresses.

```text
dns.cname
```

Shows DNS packets containing CNAME records.

Useful to identify aliases and redirection chains between domain names.

```text
dns.qry.type == 1
```

Shows A record queries.

Useful to focus on IPv4 name resolution.

```text
dns.qry.type == 28
```

Shows AAAA record queries.

Useful to focus on IPv6 name resolution.

```text
dns.qry.type == 16
```

Shows TXT record queries.

Useful when checking for unusual TXT activity or possible abuse.

```text
dns.qry.type == 12
```

Shows PTR queries.

Useful when checking reverse DNS lookups.