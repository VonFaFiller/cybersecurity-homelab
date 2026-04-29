# HTTP Behavior in PCAP Analysis

## Practical Use in PCAP Analysis

HTTP is one of the most useful protocols in PCAP analysis when the traffic is not encrypted.

It can expose visited hosts, requested paths, parameters, uploaded files, downloaded files, user agents, cookies, credentials, webshell interaction, exploitation attempts, and attacker-controlled commands.

> [!NOTE]
> HTTP is especially useful because request and response content can often be inspected directly in Wireshark.

## Core Concept

HTTP is request-based.

A client sends an HTTP request to a server.

The server replies with an HTTP response.

Example:

```text
Client -> Server: GET /index.html HTTP/1.1
Server -> Client: HTTP/1.1 200 OK
```

In this case:

```text
Request method: GET
Requested path: /index.html
Response code: 200
```

HTTP traffic is usually structured around:

```text
Method
Host
URI
Headers
Body
Response code
Response content
```

## Main HTTP Methods

| Method | Meaning |
|---|---|
| GET | Requests a resource from the server |
| POST | Sends data to the server |
| PUT | Uploads or replaces a resource |
| DELETE | Requests deletion of a resource |
| HEAD | Requests headers without the response body |
| OPTIONS | Requests supported methods or server capabilities |

## Main HTTP Status Codes

| Status code | Practical interpretation |
|---|---|
| 200 OK | Request succeeded |
| 301 / 302 | Redirect |
| 400 Bad Request | Malformed or invalid request |
| 401 Unauthorized | Authentication required or failed |
| 403 Forbidden | Access denied |
| 404 Not Found | Resource does not exist |
| 500 Internal Server Error | Server-side error |

## Common HTTP Patterns in PCAPs

| Pattern | Practical interpretation |
|---|---|
| Repeated GET requests | Browsing, enumeration, crawling, or automated activity |
| Repeated POST requests | Form submission, brute force, upload, API interaction, or webshell use |
| POST followed by 200 / 302 | Submitted data was accepted or redirected after processing |
| POST followed by 401 / 403 | Authentication failed or access denied |
| Suspicious parameters in URI | Possible command injection, SQL injection, traversal, or exploit attempt |
| Encoded characters in URI | Payload may require URL decoding |
| File upload request | Possible legitimate upload or malicious file delivery |
| File download response | Possible payload download or data retrieval |
| Unusual User-Agent | Possible script, tool, malware, scanner, or custom client |
| Commands inside parameters or body | Possible webshell or command execution |
| Large response body after suspicious request | Possible file download, data dump, or command output |

## Main HTTP Fields

| Field | Practical use |
|---|---|
| Host | Identifies the requested domain or virtual host |
| Request URI | Shows the requested path and parameters |
| User-Agent | Identifies the client, browser, script, tool, or malware |
| Referer | Shows where the request came from |
| Cookie | Can expose sessions, authentication state, or tracking values |
| Authorization | Can expose authentication data if not protected |
| Content-Type | Helps identify submitted or returned data type |
| Content-Length | Helps identify large uploads, downloads, or unusual payloads |
| Server | May expose the web server software |
| Location | Shows redirect destination |

## Wireshark Filters

```text
http.request
```

Shows HTTP requests.

Useful to focus on client-side actions.

```text
http.response
```

Shows HTTP responses.

Useful to inspect server replies and response codes.

```text
http.request.method == "GET"
```

Shows HTTP GET requests.

Useful to inspect requested resources, browsing, enumeration, and downloads.

```text
http.request.method == "POST"
```

Shows HTTP POST requests.

Useful to inspect submitted data, logins, uploads, API calls, and possible webshell interaction.

```text
http.host
```

Shows HTTP packets containing a Host header.

Useful to identify requested domains or virtual hosts.

```text
http.host contains "example"
```

Shows HTTP traffic where the Host header contains a specific string.

Useful when searching for a known domain or keyword.

```text
http.request.uri
```

Shows HTTP packets containing a request URI.

Useful to inspect requested paths and parameters.

```text
http.request.uri contains "cmd"
```

Shows HTTP requests where the URI contains a specific string.

Useful when searching for command execution, webshell parameters, or suspicious keywords.

```text
http.user_agent
```

Shows HTTP packets containing a User-Agent header.

Useful to identify browsers, scripts, scanners, tools, or malware clients.

```text
http.user_agent contains "curl"
```

Shows HTTP traffic where the User-Agent contains a specific string.

Useful when searching for known tools or unusual clients.

```text
http.response.code == 200
```

Shows successful HTTP responses.

Useful to identify requests that received a normal server response.

```text
http.response.code == 302
```

Shows HTTP redirects.

Useful to identify login redirects, application flow, or redirection chains.

```text
http.response.code >= 400
```

Shows HTTP client or server errors.

Useful to identify failed requests, forbidden access, missing resources, or server-side issues.

```text
http.cookie
```

Shows HTTP packets containing cookies.

Useful to inspect session-related traffic.

```text
http.authorization
```

Shows HTTP packets containing Authorization headers.

Useful to identify authentication data when visible.

```text
http.content_type
```

Shows HTTP packets containing Content-Type headers.

Useful to identify submitted or returned data types.

```text
http.file_data
```

Shows HTTP packets containing file or body data.

Useful to inspect transferred content, command output, payloads, or submitted data.