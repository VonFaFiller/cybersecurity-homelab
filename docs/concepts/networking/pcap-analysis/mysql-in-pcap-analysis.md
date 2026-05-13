# MySQL Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

MySQL is useful in PCAP analysis when investigating database access, login attempts, SQL queries, enumeration, data extraction, and suspicious interaction with backend services.

When MySQL traffic is visible, usernames, selected databases, queries, errors, and command flow may be readable directly in Wireshark.

> [!NOTE]
> MySQL authentication does not usually expose the plaintext password, but login attempts, usernames, database names, and SQL queries can still be extremely useful during analysis.

## Core Concept

MySQL uses a client-server model.

A client connects to a MySQL server, receives a server greeting, sends a login request, and then issues commands such as selecting a database or executing SQL queries.

Example:

```text
Server -> Client: Server Greeting
Client -> Server: Login Request
Server -> Client: OK / Error
Client -> Server: COM_QUERY
Server -> Client: Query Response
```

In this case:

```text
Login Request: authentication attempt
OK / Error: login result
COM_QUERY: SQL query sent by the client
Query Response: server response to the query
```

## Main MySQL Commands

| Command | Code | Practical meaning |
|---|---:|---|
| COM_QUIT | 1 | Client closes the MySQL session |
| COM_INIT_DB | 2 | Client selects a database |
| COM_QUERY | 3 | Client sends a SQL query |
| COM_FIELD_LIST | 4 | Client requests table field information |
| COM_PING | 14 | Client checks whether the server is alive |
| COM_STMT_PREPARE | 22 | Client prepares a SQL statement |
| COM_STMT_EXECUTE | 23 | Client executes a prepared statement |
| COM_STMT_CLOSE | 25 | Client closes a prepared statement |

## Common MySQL Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| Server greeting followed by login request | Client attempted to authenticate to MySQL |
| Login request followed by OK response | MySQL authentication succeeded |
| Login request followed by error response | MySQL authentication failed |
| Repeated login requests | Possible brute force, misconfiguration, or repeated application retries |
| COM_INIT_DB after login | Client selected a database |
| COM_QUERY after login | Client executed SQL queries |
| SHOW DATABASES / SHOW TABLES | Database or table enumeration |
| SELECT from INFORMATION_SCHEMA | Schema enumeration or SQL injection-style reconnaissance |
| SELECT with many returned rows | Possible data retrieval or dump |
| INSERT / UPDATE / DELETE | Data modification activity |
| LOAD_FILE | Possible file read from the database server |
| INTO OUTFILE | Possible file write to the database server |
| Error responses after queries | Failed SQL syntax, permission issue, or invalid object access |
| Database queries after suspicious web requests | Possible backend interaction caused by web exploitation |

## Main MySQL Fields

| Field | Practical use |
|---|---|
| Command | Shows the MySQL command being executed |
| Username | Shows the user attempting to authenticate |
| Schema | Shows the selected database when visible |
| Query | Shows SQL queries sent by the client |
| Response code | Shows whether a response is OK, EOF, or Error |
| Error code | Shows database-side failure details |
| Server greeting | Identifies MySQL server handshake activity |
| Packet number | Helps follow packet order inside the MySQL exchange |

## Wireshark Filters

```text
mysql
```

Shows MySQL traffic.

Useful to focus on database protocol activity.

```text
mysql.login_request
```

Shows MySQL login requests.

Useful to identify authentication attempts and submitted usernames.

```text
mysql.login_response
```

Shows MySQL login responses.

Useful to identify whether authentication succeeded or failed.

```text
mysql.user
```

Shows MySQL packets containing usernames.

Useful to identify accounts used during database access.

```text
mysql.user contains "root"
```

Shows MySQL traffic where the username contains a specific string.

Useful when searching for privileged or known accounts.

```text
mysql.schema
```

Shows MySQL packets containing selected database names.

Useful to identify which database the client attempted to use.

```text
mysql.command
```

Shows MySQL command packets.

Useful to inspect the type of action performed by the client.

```text
mysql.command == 2
```

Shows COM_INIT_DB commands.

Useful to identify database selection.

```text
mysql.command == 3
```

Shows COM_QUERY commands.

Useful to identify SQL queries sent by the client.

```text
mysql.command == 14
```

Shows COM_PING commands.

Useful to identify connection checks or keepalive-like database activity.

```text
mysql.command == 22
```

Shows COM_STMT_PREPARE commands.

Useful to identify prepared SQL statements.

```text
mysql.command == 23
```

Shows COM_STMT_EXECUTE commands.

Useful to identify execution of prepared SQL statements.

```text
mysql.query
```

Shows MySQL packets containing SQL queries.

Useful to inspect database activity directly.

```text
mysql.query contains "SELECT"
```

Shows SQL SELECT queries.

Useful to identify data retrieval.

```text
mysql.query contains "INFORMATION_SCHEMA"
```

Shows queries involving INFORMATION_SCHEMA.

Useful to identify database structure enumeration.

```text
mysql.query contains "SHOW"
```

Shows SHOW queries.

Useful to identify database, table, variable, or status enumeration.

```text
mysql.query contains "INSERT"
```

Shows INSERT queries.

Useful to identify database write activity.

```text
mysql.query contains "UPDATE"
```

Shows UPDATE queries.

Useful to identify database modification activity.

```text
mysql.query contains "DELETE"
```

Shows DELETE queries.

Useful to identify deletion activity.

```text
mysql.query contains "LOAD_FILE"
```

Shows queries using LOAD_FILE.

Useful to identify possible file-read attempts from the database server.

```text
mysql.query contains "INTO OUTFILE"
```

Shows queries using INTO OUTFILE.

Useful to identify possible file-write attempts from the database server.

```text
mysql.response_code
```

Shows MySQL response code information.

Useful to inspect OK, EOF, or Error responses.

```text
mysql.response_code == 255
```

Shows MySQL error responses.

Useful to identify failed authentication, failed queries, permission issues, or invalid SQL operations.