# FTP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

FTP is useful in PCAP analysis when investigating cleartext authentication, file transfers, directory browsing, uploads, downloads, and attacker movement through exposed file services.

Because classic FTP is not encrypted, usernames, passwords, commands, filenames, and transfer activity may be visible directly in Wireshark.

> [!NOTE]
> FTP separates control traffic from data transfer traffic. Commands usually appear in the FTP control channel, while file contents and directory listings may appear in FTP-DATA.

## Core Concept

FTP uses a command-and-response model.

A client connects to an FTP server, authenticates, sends commands, and then reads, uploads, downloads, or lists files.

Example:

```text
Client -> Server: USER analyst
Server -> Client: 331 Password required
Client -> Server: PASS password123
Server -> Client: 230 Login successful
Client -> Server: LIST
Server -> Client: 150 Opening data connection
Server -> Client: 226 Transfer complete
```

In this case:

```text
USER / PASS: authentication
230: login succeeded
LIST: directory listing
150 / 226: data transfer started and completed
```

## Main FTP Commands

| Command | Practical meaning |
|---|---|
| USER | Sends username |
| PASS | Sends password |
| SYST | Requests system type |
| PWD | Prints current directory |
| CWD | Changes directory |
| CDUP | Moves to parent directory |
| LIST | Lists directory contents |
| NLST | Lists filenames only |
| RETR | Downloads a file from the server |
| STOR | Uploads a file to the server |
| DELE | Deletes a file |
| MKD | Creates a directory |
| RMD | Removes a directory |
| RNFR | Selects file to rename |
| RNTO | Provides new filename |
| PORT | Active mode data connection setup |
| PASV | Passive mode data connection setup |
| EPSV | Extended passive mode data connection setup |
| QUIT | Ends the FTP session |

## Common FTP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| USER followed by PASS | Login attempt |
| USER / PASS followed by 230 | Successful authentication |
| USER / PASS followed by 530 | Failed authentication |
| Repeated login failures | Brute force, wrong credentials, or access issue |
| LIST or NLST after login | Directory browsing |
| CWD followed by LIST | User is navigating directories |
| RETR followed by 150 / 226 | File download started and completed |
| STOR followed by 150 / 226 | File upload started and completed |
| DELE after file access | File deletion activity |
| RNFR followed by RNTO | File rename activity |
| PASV or EPSV before transfer | Passive mode data transfer setup |
| PORT before transfer | Active mode data transfer setup |
| FTP-DATA after LIST | Directory listing content |
| FTP-DATA after RETR | Downloaded file content |
| FTP-DATA after STOR | Uploaded file content |

## Main FTP Response Codes

| Code | Practical interpretation |
|---|---|
| 220 | Service ready |
| 221 | Service closing control connection |
| 226 | Transfer complete |
| 227 | Entering passive mode |
| 229 | Entering extended passive mode |
| 230 | Login successful |
| 331 | Username accepted, password required |
| 425 | Cannot open data connection |
| 426 | Connection closed, transfer aborted |
| 500 | Syntax error or unknown command |
| 530 | Login failed or not authenticated |
| 550 | File unavailable, permission denied, or missing file |

## Wireshark Filters

```text
ftp
```

Shows FTP control traffic.

Useful to inspect authentication, commands, responses, filenames, and directory navigation.

```text
ftp-data
```

Shows FTP data channel traffic.

Useful to inspect transferred file data or directory listing content.

```text
ftp.request.command
```

Shows FTP packets containing client commands.

Useful to focus on actions performed by the FTP client.

```text
ftp.request.command == "USER"
```

Shows FTP username submissions.

Useful to identify login attempts.

```text
ftp.request.command == "PASS"
```

Shows FTP password submissions.

Useful to identify cleartext credentials when available.

```text
ftp.request.command == "LIST"
```

Shows FTP directory listing requests.

Useful to identify browsing of server-side directories.

```text
ftp.request.command == "RETR"
```

Shows FTP file download requests.

Useful to identify files retrieved from the server.

```text
ftp.request.command == "STOR"
```

Shows FTP file upload requests.

Useful to identify files sent to the server.

```text
ftp.request.command == "CWD"
```

Shows FTP directory changes.

Useful to reconstruct navigation through server directories.

```text
ftp.request.command == "DELE"
```

Shows FTP file deletion requests.

Useful to identify deletion activity.

```text
ftp.request.command == "PASV" || ftp.request.command == "EPSV"
```

Shows passive mode negotiation.

Useful to understand how the FTP data connection was established.

```text
ftp.request.command == "PORT"
```

Shows active mode negotiation.

Useful to identify client-provided data connection details.

```text
ftp.request.arg
```

Shows FTP command arguments.

Useful to inspect usernames, filenames, paths, and command parameters.

```text
ftp.request.arg contains ".exe"
```

Shows FTP commands where the argument contains a specific string.

Useful when searching for suspicious filenames or file extensions.

```text
ftp.response.code
```

Shows FTP packets containing response codes.

Useful to inspect server replies.

```text
ftp.response.code == 230
```

Shows successful FTP logins.

Useful to identify authenticated sessions.

```text
ftp.response.code == 530
```

Shows failed FTP authentication or access failures.

Useful to identify failed login attempts.

```text
ftp.response.code == 150
```

Shows file status okay and data connection opening.

Useful to identify the start of a transfer.

```text
ftp.response.code == 226
```

Shows completed transfers.

Useful to confirm that a transfer finished successfully.

```text
ftp.response.code == 550
```

Shows unavailable files, permission issues, or failed file operations.

Useful to identify denied or failed file access.