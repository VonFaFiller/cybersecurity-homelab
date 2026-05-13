# SMTP, IMAP and POP3 Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

SMTP, IMAP, and POP3 are useful in PCAP analysis when investigating email delivery, mailbox access, phishing activity, credential exposure, attachments, suspicious mail clients, and communication with mail servers.

When these protocols are not encrypted, Wireshark may expose commands, usernames, passwords, sender addresses, recipient addresses, message headers, subjects, and email content.

> [!NOTE]
> SMTP is mainly used to send or relay email. IMAP and POP3 are mainly used to retrieve email from a mailbox.

## Core Concept

Email traffic usually involves different protocols depending on the action being performed.

SMTP is used to send email from a client to a mail server or between mail servers.

IMAP and POP3 are used by clients to access received email from a mailbox.

Example:

```text
Client -> SMTP Server: send email
SMTP Server -> Mail Server: relay or deliver email
Client -> IMAP/POP3 Server: retrieve email
```

In this case:

```text
SMTP: sending or relaying email
IMAP: mailbox access and synchronization
POP3: simple email download from mailbox
```

If the traffic is encrypted through TLS, the email content and credentials are usually not readable directly in Wireshark.

## Main Email Protocol Roles

| Protocol | Common ports | Practical role |
|---|---|---|
| SMTP | 25, 587, 465 | Sends or relays email |
| IMAP | 143, 993 | Accesses and synchronizes mailbox contents |
| POP3 | 110, 995 | Downloads email from a mailbox |

## Common Email Protocol Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| SMTP `HELO` / `EHLO` followed by `MAIL FROM` and `RCPT TO` | Email sending or relay activity |
| SMTP `AUTH` followed by message submission | Authenticated email sending |
| SMTP message with attachment content | Possible file delivery or phishing attachment |
| SMTP traffic with suspicious sender or recipient | Possible phishing, spam, or unauthorized mail activity |
| IMAP login followed by mailbox commands | Mailbox access occurred |
| IMAP `SELECT INBOX` followed by `FETCH` | Client opened inbox and retrieved messages |
| IMAP repeated `LOGIN` failures | Failed credential attempts or misconfigured client |
| POP3 `USER` followed by `PASS` | POP3 login attempt |
| POP3 `RETR` after successful login | Email message download |
| POP3 `DELE` after retrieval | Email deletion after download |
| Cleartext credentials | Unencrypted mail protocol authentication |
| STARTTLS command followed by TLS negotiation | Protocol switches from cleartext to encrypted traffic |
| Mail protocol over TLS port | Content and credentials are likely encrypted |

## Main SMTP Commands

| Command | Practical meaning |
|---|---|
| HELO | Client identifies itself to the SMTP server |
| EHLO | Extended SMTP greeting with feature negotiation |
| AUTH | Starts SMTP authentication |
| MAIL FROM | Defines sender address |
| RCPT TO | Defines recipient address |
| DATA | Starts email message content |
| RSET | Resets current mail transaction |
| VRFY | Verifies whether a mailbox exists |
| EXPN | Expands a mailing list |
| STARTTLS | Starts TLS encryption |
| QUIT | Ends the SMTP session |

## Main IMAP Commands

| Command | Practical meaning |
|---|---|
| LOGIN | Authenticates to the mailbox |
| CAPABILITY | Requests server capabilities |
| STARTTLS | Starts TLS encryption |
| SELECT | Selects a mailbox, commonly INBOX |
| EXAMINE | Opens a mailbox in read-only mode |
| LIST | Lists available mailboxes |
| SEARCH | Searches mailbox messages |
| FETCH | Retrieves message data |
| STORE | Changes message flags |
| COPY | Copies messages to another mailbox |
| UID | Performs operations using unique message IDs |
| LOGOUT | Ends the IMAP session |

## Main POP3 Commands

| Command | Practical meaning |
|---|---|
| USER | Sends username |
| PASS | Sends password |
| AUTH | Starts authentication |
| STAT | Requests mailbox message count and size |
| LIST | Lists messages and sizes |
| RETR | Retrieves a message |
| TOP | Retrieves message headers and first lines |
| DELE | Marks message for deletion |
| NOOP | Keeps the session alive |
| RSET | Cancels deletion marks |
| STLS | Starts TLS encryption |
| QUIT | Ends the POP3 session |

## Main Email Fields and Artifacts

| Field / Artifact | Practical use |
|---|---|
| Sender address | Identifies who sent the email |
| Recipient address | Identifies who received the email |
| Subject | Shows the email subject when visible |
| Message-ID | Helps correlate email messages |
| Date | Shows email timestamp from message headers |
| Received headers | Show mail relay path |
| User-Agent / X-Mailer | May identify the email client |
| MIME boundaries | Separate message parts and attachments |
| Content-Type | Identifies text, HTML, attachment, or multipart content |
| Content-Disposition | Often identifies attachment filename |
| Attachment filename | Identifies delivered files |
| Base64 content | May contain encoded body content or attachments |

## Wireshark Filters

```text
smtp
```

Shows SMTP traffic.

Useful to inspect email sending, relaying, authentication, and message content when unencrypted.

```text
imap
```

Shows IMAP traffic.

Useful to inspect mailbox access and message retrieval when unencrypted.

```text
pop
```

Shows POP3 traffic.

Useful to inspect mailbox downloads and POP3 authentication when unencrypted.

```text
smtp.req.command
```

Shows SMTP request commands.

Useful to inspect client-side SMTP actions.

```text
smtp.req.command == "EHLO"
```

Shows SMTP EHLO commands.

Useful to identify SMTP clients and advertised server capabilities.

```text
smtp.req.command == "AUTH"
```

Shows SMTP authentication attempts.

Useful to identify authenticated email submission.

```text
smtp.req.command == "MAIL"
```

Shows SMTP sender declaration.

Useful to identify sender addresses.

```text
smtp.req.command == "RCPT"
```

Shows SMTP recipient declaration.

Useful to identify recipient addresses.

```text
smtp.req.command == "DATA"
```

Shows SMTP DATA commands.

Useful to identify the start of email message content.

```text
smtp.req.command == "STARTTLS"
```

Shows SMTP STARTTLS commands.

Useful to identify when SMTP traffic switches to TLS.

```text
smtp.response.code
```

Shows SMTP response codes.

Useful to inspect whether SMTP commands succeeded or failed.

```text
smtp.response.code >= 400
```

Shows SMTP client or server error responses.

Useful to identify rejected messages, authentication issues, or mail delivery problems.

```text
imap.request.command
```

Shows IMAP request commands.

Useful to inspect mailbox actions performed by the client.

```text
imap.request.command == "LOGIN"
```

Shows IMAP login attempts.

Useful to identify mailbox authentication activity.

```text
imap.request.command == "SELECT"
```

Shows IMAP mailbox selection.

Useful to identify which mailbox was opened.

```text
imap.request.command == "FETCH"
```

Shows IMAP message retrieval.

Useful to identify email messages being retrieved.

```text
imap.request.command == "SEARCH"
```

Shows IMAP searches.

Useful to identify mailbox search activity.

```text
imap.request.command == "STARTTLS"
```

Shows IMAP STARTTLS commands.

Useful to identify when IMAP traffic switches to TLS.

```text
imap.response
```

Shows IMAP responses.

Useful to inspect mailbox server replies.

```text
pop.request.command
```

Shows POP3 request commands.

Useful to inspect POP3 client actions.

```text
pop.request.command == "USER"
```

Shows POP3 username submissions.

Useful to identify POP3 login attempts.

```text
pop.request.command == "PASS"
```

Shows POP3 password submissions.

Useful to identify cleartext credentials when available.

```text
pop.request.command == "RETR"
```

Shows POP3 message retrieval.

Useful to identify downloaded email messages.

```text
pop.request.command == "DELE"
```

Shows POP3 deletion requests.

Useful to identify messages marked for deletion.

```text
pop.request.command == "STLS"
```

Shows POP3 STLS commands.

Useful to identify when POP3 traffic switches to TLS.

```text
pop.response
```

Shows POP3 responses.

Useful to inspect server replies to POP3 commands.

```text
imf
```

Shows Internet Message Format email content when Wireshark decodes it.

Useful to inspect email headers and message content.

```text
imf.subject
```

Shows email subject fields.

Useful to identify message subjects.

```text
imf.from
```

Shows email sender fields.

Useful to identify sender addresses.

```text
imf.to
```

Shows email recipient fields.

Useful to identify recipient addresses.

```text
mime_multipart
```

Shows MIME multipart content.

Useful to inspect email body parts and attachments.

```text
mime_multipart.header.content-type
```

Shows MIME content type headers.

Useful to identify text, HTML, multipart sections, or attachment data.

```text
mime_multipart.header.content-disposition
```

Shows MIME content disposition headers.

Useful to identify attachment filenames and inline content.