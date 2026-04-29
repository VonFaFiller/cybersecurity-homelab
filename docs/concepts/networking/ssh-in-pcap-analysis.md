# SSH Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

SSH is useful in PCAP analysis when investigating remote administration, encrypted shell access, brute-force attempts, lateral movement, server exposure, and suspicious command-line access.

SSH traffic is encrypted after the handshake, so the actual commands, usernames, passwords, and terminal output are usually not visible in Wireshark.

> [!NOTE]
> SSH is still useful in PCAP analysis because metadata such as protocol banners, key exchange behavior, session timing, packet volume, and repeated connection attempts can reveal suspicious remote access activity.

## Core Concept

SSH provides encrypted remote access between a client and a server.

Before encrypted communication begins, the client and server exchange protocol information and negotiate cryptographic parameters.

Example:

```text
Client -> Server: SSH protocol version
Server -> Client: SSH protocol version
Client <-> Server: Key exchange negotiation
Client <-> Server: New keys
Client <-> Server: Encrypted packets
```

In this case:

```text
Protocol version: identifies SSH software/version when visible
Key exchange: negotiates encryption parameters
New keys: encryption is established
Encrypted packets: SSH session content is no longer readable
```

After encryption starts, Wireshark can usually show that SSH communication occurred, but not what commands were executed inside the session.

## Main SSH Phases

| Phase | Practical meaning |
|---|---|
| Protocol version exchange | Client and server announce SSH version/software |
| Key exchange initialization | Client and server advertise supported algorithms |
| Key exchange | Cryptographic material is negotiated |
| New keys | Encrypted communication begins |
| Authentication | User authentication occurs inside the encrypted session |
| Channel activity | Shell, command execution, port forwarding, or file transfer may occur inside the encrypted session |
| Disconnect | SSH session is closed |

## Common SSH Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| SSH version exchange followed by encrypted packets | SSH session was established |
| SSH version exchange with no encrypted packets | Failed, interrupted, or incomplete SSH negotiation |
| Repeated SSH connections to one host | Possible brute force, repeated admin access, automation, or retry behavior |
| Many hosts connecting to one SSH server | Shared administration point, scanning, or exposed service |
| One host connecting to many SSH servers | Possible lateral movement, scanning, or administrative activity |
| Short repeated SSH sessions | Failed login attempts, probing, automation, or unstable access |
| Long SSH session with steady traffic | Interactive shell, tunneling, file transfer, or active remote administration |
| Large SSH data transfer | Possible SCP/SFTP transfer, tunneling, or bulk encrypted activity |
| SSH from unusual internal host | Possible compromised host or unauthorized administration |
| SSH to unusual external IP | Possible outbound remote access, tunneling, or C2-like behavior |

## Main SSH Fields

| Field | Practical use |
|---|---|
| Protocol version | Shows SSH version/software banner when visible |
| Key exchange algorithms | Shows supported key exchange methods |
| Server host key algorithms | Shows supported server host key types |
| Encryption algorithms | Shows supported encryption algorithms |
| MAC algorithms | Shows supported integrity algorithms |
| Compression algorithms | Shows supported compression methods |
| Encrypted packet | Indicates encrypted SSH payload after negotiation |
| Encrypted packet length | Helps estimate session activity and transfer size |
| Packet length | Shows SSH packet size |
| Padding length | Shows SSH padding information |

## Wireshark Filters

```text
ssh
```

Shows SSH traffic.

Useful to focus on SSH negotiation and encrypted SSH sessions.

```text
ssh.protocol
```

Shows SSH protocol version strings.

Useful to identify SSH software and version banners when visible.

```text
ssh.protocol contains "OpenSSH"
```

Shows SSH packets where the protocol banner contains a specific string.

Useful when searching for OpenSSH clients or servers.

```text
ssh.kex_algorithms
```

Shows SSH key exchange algorithm lists.

Useful to inspect supported key exchange methods during negotiation.

```text
ssh.server_host_key_algorithms
```

Shows SSH server host key algorithm lists.

Useful to identify supported host key types.

```text
ssh.encryption_algorithms_client_to_server
```

Shows encryption algorithms proposed for client-to-server traffic.

Useful to inspect SSH cryptographic negotiation.

```text
ssh.encryption_algorithms_server_to_client
```

Shows encryption algorithms proposed for server-to-client traffic.

Useful to inspect SSH cryptographic negotiation in the opposite direction.

```text
ssh.mac_algorithms_client_to_server
```

Shows MAC algorithms proposed for client-to-server traffic.

Useful to inspect integrity algorithm negotiation.

```text
ssh.mac_algorithms_server_to_client
```

Shows MAC algorithms proposed for server-to-client traffic.

Useful to inspect integrity algorithm negotiation in the opposite direction.

```text
ssh.compression_algorithms_client_to_server
```

Shows compression algorithms proposed for client-to-server traffic.

Useful to inspect whether compression was offered.

```text
ssh.compression_algorithms_server_to_client
```

Shows compression algorithms proposed for server-to-client traffic.

Useful to inspect whether compression was offered in the opposite direction.

```text
ssh.encrypted_packet
```

Shows encrypted SSH packet data.

Useful to confirm that encrypted SSH communication occurred after negotiation.

```text
ssh.encrypted_packet_length
```

Shows encrypted SSH packet length.

Useful to estimate session activity, repeated interaction, or large transfers.

```text
ssh.packet_length
```

Shows SSH packet length.

Useful to inspect packet sizing during SSH negotiation and encrypted activity.

```text
ssh.padding_length
```

Shows SSH padding length.

Useful when inspecting SSH packet structure.