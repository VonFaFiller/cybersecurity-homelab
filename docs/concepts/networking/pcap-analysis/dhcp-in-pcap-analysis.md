# DHCP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

DHCP is useful in PCAP analysis when investigating IP address assignment, host identification, local network configuration, DHCP servers, gateways, DNS servers, lease activity, and suspicious or unexpected device behavior.

DHCP can expose useful host details such as MAC address, assigned IP address, hostname, requested IP address, DHCP server identifier, gateway, DNS server, and lease time.

> [!NOTE]
> In Wireshark, DHCP fields are commonly shown under `bootp` because DHCP is based on BOOTP.

## Core Concept

DHCP assigns network configuration to clients automatically.

A typical DHCP exchange follows the DORA sequence:

```text
Client -> Broadcast: DHCP Discover
Server -> Client: DHCP Offer
Client -> Server: DHCP Request
Server -> Client: DHCP ACK
```

In this case:

```text
Discover: client is looking for a DHCP server
Offer: server offers an IP configuration
Request: client requests the offered configuration
ACK: server confirms the lease
```

This can help identify which host received which IP address and from which DHCP server.

## Main DHCP Message Types

| Message type | Value | Practical meaning |
|---|---:|---|
| Discover | 1 | Client is looking for a DHCP server |
| Offer | 2 | Server offers an IP configuration |
| Request | 3 | Client requests an offered or previously used IP address |
| Decline | 4 | Client rejects the offered address |
| ACK | 5 | Server confirms the lease |
| NAK | 6 | Server rejects the client request |
| Release | 7 | Client releases the assigned address |
| Inform | 8 | Client requests configuration without asking for an IP lease |

## Common DHCP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| Discover → Offer → Request → ACK | Normal DHCP lease assignment |
| Discover with no Offer | No DHCP server responded, server unavailable, or capture is incomplete |
| Offer with no Request | Client did not accept or continue the lease process |
| Request followed by ACK | Lease was accepted or renewed |
| Request followed by NAK | Server rejected the requested configuration |
| Repeated Discover messages | Client cannot obtain network configuration |
| Repeated Request messages | Client may be retrying lease renewal or requesting a previous address |
| Multiple DHCP servers offering addresses | Possible rogue DHCP server or multi-server environment |
| Client hostname visible in DHCP options | Host can be identified through DHCP metadata |
| Requested IP address visible | Client may be asking for a previous lease |
| DHCP server identifier visible | DHCP server IP can be identified |
| Gateway and DNS options visible | Local network configuration can be reconstructed |

## Main DHCP Options

| Option | Practical use |
|---|---|
| DHCP Message Type | Identifies Discover, Offer, Request, ACK, NAK, and other DHCP messages |
| Requested IP Address | Shows the IP address requested by the client |
| DHCP Server Identifier | Identifies the DHCP server |
| Host Name | Shows the client hostname when provided |
| Client Identifier | Identifies the DHCP client |
| Vendor Class Identifier | Can expose OS, device, or client implementation hints |
| Parameter Request List | Shows which configuration options the client requested |
| IP Address Lease Time | Shows the lease duration |
| Subnet Mask | Shows the assigned subnet mask |
| Router | Shows the default gateway offered to the client |
| Domain Name Server | Shows DNS servers offered to the client |
| Domain Name | Shows the domain name assigned by DHCP |
| Relay Agent Information | Shows DHCP relay information when present |

## Wireshark Filters

```text
bootp
```

Shows BOOTP/DHCP traffic.

Useful to inspect DHCP exchanges and network configuration assignment.

```text
bootp.type == 1
```

Shows BOOTP/DHCP requests from clients.

Useful to identify client-side DHCP activity.

```text
bootp.type == 2
```

Shows BOOTP/DHCP replies from servers.

Useful to identify DHCP server responses.

```text
bootp.option.dhcp == 1
```

Shows DHCP Discover messages.

Useful to identify clients looking for a DHCP server.

```text
bootp.option.dhcp == 2
```

Shows DHCP Offer messages.

Useful to identify DHCP servers offering IP configuration.

```text
bootp.option.dhcp == 3
```

Shows DHCP Request messages.

Useful to identify clients requesting an offered or previous IP address.

```text
bootp.option.dhcp == 5
```

Shows DHCP ACK messages.

Useful to confirm that a DHCP lease was accepted.

```text
bootp.option.dhcp == 6
```

Shows DHCP NAK messages.

Useful to identify rejected DHCP requests.

```text
bootp.option.dhcp == 7
```

Shows DHCP Release messages.

Useful to identify clients releasing an IP address.

```text
bootp.hw.mac_addr
```

Shows DHCP packets containing client MAC address information.

Useful to associate a DHCP client with a MAC address.

```text
bootp.hw.mac_addr == 00:11:22:33:44:55
```

Shows DHCP packets involving a specific client MAC address.

Useful to track DHCP activity from one host.

```text
bootp.ip.your
```

Shows DHCP packets containing the assigned client IP address field.

Useful to identify the IP address offered or assigned to a client.

```text
bootp.option.requested_ip_address
```

Shows DHCP packets containing a requested IP address.

Useful to identify which address the client is asking for.

```text
bootp.option.dhcp_server_id
```

Shows DHCP packets containing the DHCP server identifier.

Useful to identify which server offered or confirmed the lease.

```text
bootp.option.hostname
```

Shows DHCP packets containing a client hostname.

Useful to identify hosts by DHCP metadata.

```text
bootp.option.hostname contains "DESKTOP"
```

Shows DHCP packets where the hostname contains a specific string.

Useful when searching for a known device name or naming pattern.

```text
bootp.option.vendor_class_id
```

Shows DHCP packets containing vendor class information.

Useful to identify OS, device type, or DHCP client implementation hints.

```text
bootp.option.parameter_request_list
```

Shows DHCP packets containing requested DHCP options.

Useful to understand what configuration the client asked for.

```text
bootp.option.lease_time
```

Shows DHCP packets containing lease duration information.

Useful to inspect how long the IP assignment is valid.

```text
bootp.option.router
```

Shows DHCP packets containing default gateway information.

Useful to identify the gateway assigned to clients.

```text
bootp.option.domain_name_server
```

Shows DHCP packets containing DNS server information.

Useful to identify DNS servers assigned through DHCP.