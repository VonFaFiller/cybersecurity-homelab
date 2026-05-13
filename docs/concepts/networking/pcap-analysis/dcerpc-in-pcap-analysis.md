# DCE/RPC Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

DCE/RPC is useful in PCAP analysis when investigating Windows remote administration, service control, named pipe activity, Active Directory interaction, lateral movement, endpoint mapping, and PsExec-like behavior.

DCE/RPC often appears inside SMB traffic through named pipes, but it can also appear directly over TCP.

> [!NOTE]
> In Windows investigations, DCE/RPC is important because many administrative actions are performed through RPC interfaces, even when the visible traffic looks like SMB or TCP activity at first.

## Core Concept

DCE/RPC allows a client to call functions on a remote system.

A client connects to a remote RPC service, binds to a specific interface, sends a request, and receives a response.

Example:

```text
Client -> Server: Bind to RPC interface
Server -> Client: Bind ACK
Client -> Server: RPC Request
Server -> Client: RPC Response
```

In this case:

```text
Bind: client selects the RPC interface
Bind ACK: server accepts the interface binding
Request: client calls a remote operation
Response: server replies to the operation
```

In Windows environments, DCE/RPC is commonly carried through SMB named pipes.

Example:

```text
Client -> Server: SMB Tree Connect to IPC$
Client -> Server: SMB Create/Open \PIPE\svcctl
Client -> Server: DCE/RPC Bind
Client -> Server: DCE/RPC Request
```

In this case:

```text
IPC$: SMB share used for inter-process communication
\PIPE\svcctl: named pipe related to Service Control Manager activity
DCE/RPC: remote procedure call traffic carried inside SMB
```

## Main DCE/RPC Packet Types

| Packet type | Practical meaning |
|---|---|
| Bind | Client binds to an RPC interface |
| Bind ACK | Server accepts the RPC bind |
| Bind NAK | Server rejects the RPC bind |
| Request | Client calls a remote procedure |
| Response | Server replies to a request |
| Fault | Server reports an RPC error |
| Alter Context | Client changes or adds a presentation context |
| Auth3 | Authentication-related continuation |
| Shutdown | Server indicates shutdown of RPC connection |
| Cancel | Client cancels an RPC call |
| Orphaned | Client abandons an RPC call |

## Common DCE/RPC Interfaces

| Interface / Pipe | Practical use |
|---|---|
| Endpoint Mapper / EPM | Maps RPC services to endpoints |
| `\PIPE\svcctl` | Service Control Manager remote operations |
| `\PIPE\samr` | Security Account Manager remote operations |
| `\PIPE\lsarpc` | Local Security Authority remote operations |
| `\PIPE\netlogon` | Domain logon and secure channel activity |
| `\PIPE\wkssvc` | Workstation service information |
| `\PIPE\srvsvc` | Server service and share enumeration |
| `\PIPE\spoolss` | Print spooler remote operations |
| `\PIPE\atsvc` | Legacy scheduled task interface |
| DRSUAPI | Active Directory replication-related operations |

## Common DCE/RPC Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| RPC Bind followed by Bind ACK | Client successfully bound to an RPC interface |
| RPC Bind followed by Bind NAK | RPC interface or negotiation was rejected |
| RPC Request followed by Response | Remote procedure call completed |
| RPC Request followed by Fault | Remote procedure call failed |
| RPC over SMB after access to `IPC$` | Named pipe-based Windows RPC activity |
| Access to `\PIPE\svcctl` followed by RPC requests | Possible remote service control |
| Access to `\PIPE\samr` followed by RPC requests | Possible account or group enumeration |
| Access to `\PIPE\lsarpc` followed by RPC requests | Possible policy, SID, or domain information lookup |
| Access to `\PIPE\srvsvc` followed by RPC requests | Possible share or server enumeration |
| Access to `\PIPE\wkssvc` followed by RPC requests | Possible workstation or host information lookup |
| Access to `\PIPE\netlogon` followed by RPC requests | Domain authentication or secure channel activity |
| RPC activity after SMB authentication | Authenticated remote administration or enumeration |
| RPC activity involving service control plus file copy | Possible PsExec-like behavior |
| Many RPC binds to different interfaces | Enumeration, administrative tooling, or lateral movement |
| Repeated RPC faults | Failed access, unsupported operation, permission issue, or malformed request |

## Main DCE/RPC Fields

| Field | Practical use |
|---|---|
| Packet type | Identifies Bind, Request, Response, Fault, or other RPC packet types |
| Interface UUID | Identifies the RPC interface being used |
| Operation number | Identifies the remote procedure being called within an interface |
| Call ID | Correlates RPC requests and responses |
| Context ID | Identifies the presentation context |
| Fragment length | Shows RPC fragment size |
| Authentication length | Shows authentication data length |
| Authentication level | Shows protection level when present |
| Stub data | Contains interface-specific request or response data |
| Fault status | Shows RPC error information |

## Wireshark Filters

```text
dcerpc
```

Shows DCE/RPC traffic.

Useful to focus on remote procedure call activity.

```text
dcerpc.pkt_type
```

Shows DCE/RPC packet type information.

Useful to identify Bind, Request, Response, Fault, and other RPC packet types.

```text
dcerpc.pkt_type == 11
```

Shows DCE/RPC Bind packets.

Useful to identify clients binding to RPC interfaces.

```text
dcerpc.pkt_type == 12
```

Shows DCE/RPC Bind ACK packets.

Useful to identify accepted RPC bindings.

```text
dcerpc.pkt_type == 13
```

Shows DCE/RPC Bind NAK packets.

Useful to identify rejected RPC bindings.

```text
dcerpc.pkt_type == 0
```

Shows DCE/RPC Request packets.

Useful to identify remote procedure calls sent by clients.

```text
dcerpc.pkt_type == 2
```

Shows DCE/RPC Response packets.

Useful to identify replies from RPC servers.

```text
dcerpc.pkt_type == 3
```

Shows DCE/RPC Fault packets.

Useful to identify failed RPC operations.

```text
dcerpc.cn_bind_to_uuid
```

Shows RPC interface UUIDs used during bind.

Useful to identify which RPC interface the client is trying to use.

```text
dcerpc.cn_bind_to_uuid == 367abb81-9844-35f1-ad32-98f038001003
```

Shows binds to the Service Control Manager Remote Protocol interface.

Useful when checking for remote service control or PsExec-like behavior.

```text
dcerpc.cn_call_id
```

Shows DCE/RPC call IDs.

Useful to correlate RPC requests and responses.

```text
dcerpc.cn_opnum
```

Shows DCE/RPC operation numbers.

Useful to identify which procedure is being called inside a specific RPC interface.

```text
dcerpc.cn_ctx_id
```

Shows DCE/RPC context IDs.

Useful to correlate activity within the same presentation context.

```text
dcerpc.cn_frag_len
```

Shows DCE/RPC fragment length.

Useful to inspect RPC message size and fragmentation.

```text
dcerpc.cn_auth_len
```

Shows DCE/RPC authentication data length.

Useful to identify RPC traffic carrying authentication information.

```text
dcerpc.auth_level
```

Shows DCE/RPC authentication level information.

Useful to inspect whether RPC traffic is using authentication, integrity, or privacy protection when visible.

```text
dcerpc.cn_stub_data
```

Shows DCE/RPC stub data.

Useful to inspect interface-specific request or response data when decoded.

```text
dcerpc.fault_status
```

Shows DCE/RPC fault status information.

Useful to identify why an RPC request failed.

```text
smb2.filename contains "svcctl"
```

Shows SMB2 traffic involving the `svcctl` named pipe.

Useful when checking for remote service control activity.

```text
smb2.filename contains "samr"
```

Shows SMB2 traffic involving the `samr` named pipe.

Useful when checking for account or group enumeration.

```text
smb2.filename contains "lsarpc"
```

Shows SMB2 traffic involving the `lsarpc` named pipe.

Useful when checking for domain, SID, or policy lookup activity.

```text
smb2.filename contains "srvsvc"
```

Shows SMB2 traffic involving the `srvsvc` named pipe.

Useful when checking for share or server enumeration.

```text
smb2.filename contains "wkssvc"
```

Shows SMB2 traffic involving the `wkssvc` named pipe.

Useful when checking for workstation or host information queries.

```text
smb2.filename contains "netlogon"
```

Shows SMB2 traffic involving the `netlogon` named pipe.

Useful when checking for domain logon or secure channel activity.