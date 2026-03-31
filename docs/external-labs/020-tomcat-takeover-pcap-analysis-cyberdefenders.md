# Tomcat Takeover Lab (CyberDefenders)

## Objective
Analyze network traffic to identify suspicious access to a Tomcat web server, reconstruct the attack sequence from enumeration to post-exploitation, and determine how the attacker gained administrative access and attempted persistence.

## Scenario
The exercise is based on a PCAP containing traffic related to suspicious activity against a Tomcat web server.

The goal is to reconstruct the attacker’s actions by identifying:
- the external source IP
- the web server being targeted
- the enumeration activity
- the administrative interface discovered
- the valid credentials used
- the malicious file upload
- the post-compromise behavior, including persistence attempts

## References
- CyberDefenders: Tomcat Takeover Lab
- MITRE ATT&CK tactics observed:
  - Reconnaissance
  - Credential Access
  - Privilege Escalation
  - Persistence
  - Discovery
  - Command and Control

## Tools Used
- Wireshark
- Statistics → Conversations
- HTTP packet inspection
- Follow TCP Stream
- Display filters
- Packet details pane

## Investigation Workflow
1. Identified the most active conversations using Statistics → Conversations.
2. Distinguished internal and external hosts based on observed addressing and communication patterns.
3. Isolated the suspicious external host communicating with the internal web server.
4. Filtered HTTP traffic to inspect requests, response codes, and access patterns.
5. Observed repeated unauthorized access attempts and requests to unusual or administrative paths.
6. Identified web enumeration activity and extracted the tool name from the HTTP User-Agent.
7. Confirmed the admin-related directory and the port exposing the administrative functionality.
8. Inspected authentication-related HTTP headers to identify valid credentials used by the attacker.
9. Traced the upload request to identify the malicious payload delivery.
10. Followed the post-upload traffic to reconstruct reverse shell activity and persistence behavior.

## Key Evidence
- Suspicious external host:
  - `14.0.0.120`
- Internal web server:
  - `10.0.0.112`
- Administrative web port:
  - `8080`
- Admin-related directory discovered:
  - `/admin-console`
- Enumeration tool identified:
  - `gobuster/3.6`
- Repeated unauthorized responses observed:
  - `401 Unauthorized`
- Successful credential use observed in HTTP authorization data:
  - `admin:tomcat`
- Malicious upload endpoint observed:
  - `POST /manager/html/upload`

## Findings
- One external host performed targeted HTTP enumeration against the Tomcat server.
- The traffic showed directory and file discovery activity rather than normal browsing behavior.
- The attacker located an admin-related path and targeted Tomcat management functionality exposed on port 8080.
- Multiple unauthorized responses indicated failed or restricted access attempts before valid administrative access was observed.
- The HTTP authorization data revealed working credentials used to authenticate successfully.
- After authentication, the attacker uploaded a malicious file consistent with post-exploitation activity.
- Follow-on traffic indicated execution of the uploaded payload and an attempt to maintain access on the compromised host.

## Result
- Attacker IP: `14.0.0.120`
- Victim Web Server: `10.0.0.112`
- Admin Panel Port: `8080`
- Admin Directory: `/admin-console`
- Enumeration Tool: `gobuster`
- Valid Credentials: `admin:tomcat`
- Attack Pattern: `Web enumeration → admin interface discovery → authentication → malicious upload → shell access → persistence attempt`

## Mistakes / Friction Points
- The exercise was relatively easy because the traffic volume was low and the suspicious HTTP activity stood out quickly.
- The wording of some questions pushed toward identifying the suspicious host early, even though in a real investigation that would initially remain a working hypothesis rather than a final attribution.
- It was important to distinguish between early indicators and confirmed conclusions.

## Notes
- In a small PCAP, reading the HTTP Info column directly can be enough to orient the investigation before applying more specific filters.
- Statistics → Conversations was useful as a triage step, but the real value came from validating the HTTP sequence and request details.
- The reconstruction worked because the attack chain was visible in order: enumeration, unauthorized access attempts, admin path discovery, authentication, upload, and post-exploitation.
- In a real case, the same logic would still be useful as triage, but stronger confirmation would be required from a broader timeline and additional logs.
