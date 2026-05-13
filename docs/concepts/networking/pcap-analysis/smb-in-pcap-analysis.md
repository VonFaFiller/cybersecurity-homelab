# SMB Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

SMB is common in Windows network environments.

In PCAP analysis, SMB can help identify file share access, authentication attempts, file reads and writes, named pipe activity, administrative share usage, and possible lateral movement.

> [!NOTE]
> SMB is especially useful when investigating Windows host interaction, file transfer, remote administration, and PsExec-like activity.

## Core Concept

SMB allows clients to access files, directories, printers, named pipes, and shared resources on a remote system.

A client usually connects to an SMB server, authenticates, accesses a share, and then performs actions such as opening, reading, writing, or deleting files.

Example:

```text
Client -> Server: Negotiate Protocol
Client -> Server: Session Setup
Client -> Server: Tree Connect to \\SERVER\Share
Client -> Server: Create/Open file
Client -> Server: Read or Write
```

In this case:

```text
Session Setup: authentication phase
Tree Connect: share access phase
Create/Open: file or object access
Read/Write: data transfer phase
```

## Main SMB2 Commands

| Command | Practical meaning |
|---|---|
| NEGOTIATE | Client and server negotiate SMB capabilities |
| SESSION_SETUP | Authentication and session establishment |
| LOGOFF | Session termination |
| TREE_CONNECT | Client connects to a share |
| TREE_DISCONNECT | Client disconnects from a share |
| CREATE | Opens or creates a file, directory, or named pipe |
| CLOSE | Closes an opened object |
| READ | Reads data from a file or object |
| WRITE | Writes data to a file or object |
| IOCTL | Performs device, pipe, or control operations |
| QUERY_DIRECTORY | Lists directory contents |
| QUERY_INFO | Queries metadata about files or objects |
| SET_INFO | Changes metadata or attributes |

## Common SMB Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| SESSION_SETUP followed by success | SMB authentication succeeded |
| Repeated SESSION_SETUP failures | Failed login attempts, wrong credentials, brute force, or access issue |
| TREE_CONNECT to normal share | User or service accessing shared resources |
| TREE_CONNECT to `ADMIN$` | Possible remote administration or lateral movement |
| TREE_CONNECT to `C$` | Administrative access to the system drive |
| TREE_CONNECT to `IPC$` | Named pipe access or remote procedure communication |
| CREATE followed by WRITE | File creation or upload to a share |
| CREATE followed by READ | File access or file download from a share |
| QUERY_DIRECTORY activity | Directory listing or share browsing |
| Access to named pipes | Possible remote administration, service control, or lateral movement |
| File copy followed by service-related activity | Possible PsExec-like behavior |
| SMB errors after access attempts | Permission issue, missing file, failed authentication, or blocked action |

## Main SMB Fields

| Field | Practical use |
|---|---|
| Command | Shows the SMB operation being performed |
| Filename | Shows the accessed file, directory, or named pipe |
| Tree | Shows the accessed share |
| Session ID | Helps correlate activity within the same SMB session |
| Tree ID | Helps correlate actions against the same share |
| NT Status | Shows success or error codes |
| Create Action | Shows whether a file was opened, created, or overwritten |
| Share Type | Identifies disk share, pipe, or printer access |

## Wireshark Filters

```text
smb2
```

Shows SMB2 traffic.

Useful to focus on modern SMB activity.

```text
smb || smb2
```

Shows SMB1 and SMB2 traffic.

Useful when the capture may contain both protocol versions.

```text
smb2.cmd == 0
```

Shows SMB2 NEGOTIATE commands.

Useful to identify SMB negotiation between client and server.

```text
smb2.cmd == 1
```

Shows SMB2 SESSION_SETUP commands.

Useful to inspect authentication-related SMB activity.

```text
smb2.cmd == 3
```

Shows SMB2 TREE_CONNECT commands.

Useful to identify which shares were accessed.

```text
smb2.cmd == 5
```

Shows SMB2 CREATE commands.

Useful to identify opened or created files, directories, and named pipes.

```text
smb2.cmd == 8
```

Shows SMB2 READ commands.

Useful to identify file or object reads.

```text
smb2.cmd == 9
```

Shows SMB2 WRITE commands.

Useful to identify file writes or uploads.

```text
smb2.cmd == 14
```

Shows SMB2 QUERY_DIRECTORY commands.

Useful to identify directory listing activity.

```text
smb2.filename
```

Shows SMB2 packets containing filenames.

Useful to inspect accessed files, directories, or named pipes.

```text
smb2.filename contains "exe"
```

Shows SMB2 packets where the filename contains a specific string.

Useful when searching for executable files or suspicious file names.

```text
smb2.filename contains "pipe"
```

Shows SMB2 packets involving named pipe paths.

Useful when checking for remote administration or lateral movement behavior.

```text
smb2.tree
```

Shows SMB2 packets containing share/tree information.

Useful to identify accessed shares.

```text
smb2.tree contains "ADMIN$"
```

Shows SMB2 traffic involving the `ADMIN$` administrative share.

Useful when checking for remote administration or PsExec-like behavior.

```text
smb2.tree contains "C$"
```

Shows SMB2 traffic involving the `C$` administrative share.

Useful when checking for administrative access to the system drive.

```text
smb2.tree contains "IPC$"
```

Shows SMB2 traffic involving the `IPC$` share.

Useful when checking for named pipe or remote procedure activity.

```text
smb2.nt_status
```

Shows SMB2 packets containing NT status codes.

Useful to inspect whether SMB actions succeeded or failed.

```text
smb2.nt_status != 0x00000000
```

Shows SMB2 packets with non-success status codes.

Useful to identify failed authentication, denied access, missing files, or other SMB errors.

```text
smb2.nt_status == 0xc0000022
```

Shows SMB2 access denied errors.

Useful to identify permission failures.

```text
smb2.nt_status == 0xc000006d
```

Shows SMB2 logon failure errors.

Useful to identify failed authentication attempts.

```text
smb2.create.action
```

Shows SMB2 create action information.

Useful to determine whether a file was opened, created, overwritten, or superseded.

