# XXE Infiltration PCAP Analysis (CyberDefenders)

## Objective
Analyze HTTP/XML and MySQL traffic in a PCAP to reconstruct an XXE attack chain against a web application, identify what the attacker accessed through XML external entity abuse, determine when the exposed database credentials were first used, and identify the later web shell activity.

## Scenario
The exercise is based on a PCAP containing suspicious HTTP/XML traffic associated with a suspected XXE injection attack against a web application.

The goal is to reconstruct the attacker’s actions by identifying:
- the highest-numbered open port exposed on the target host
- the vulnerable PHP endpoint
- the first malicious XML file uploaded
- the web app configuration file read through XXE
- the compromised database password
- the first MySQL login timestamp after credential exposure
- the uploaded web shell used for command execution and persistence

## References
- CyberDefenders: XXE Infiltration Lab
- Main activity types observed during analysis:
  - HTTP/XML file upload abuse
  - XXE-based local file read
  - application configuration disclosure
  - MySQL access after credential exposure
  - web shell use through HTTP GET parameters

## Tools Used
- Wireshark
- Statistics → Conversations
- HTTP/XML packet inspection
- Display filters
- Packet details pane

## Investigation Workflow
1. Reviewed the traffic at a high level and quickly recognized that the lab was mostly HTTP-centered, with XML uploads driving the compromise and only a later MySQL step.
2. For the open-port question, used the fact that the target host was communicating on port 80 and 3306, then answered with the highest-numbered one instead of overthinking the "web server" wording.
3. Isolated the XML-related attacker traffic and immediately found repeated `POST` requests to `/review/upload.php`, which made the vulnerable PHP endpoint easy to identify.
4. For the first malicious XML file, used the fact that the upload requests were already in chronological order and simply took the first multipart `filename` value.
5. For the configuration-file question, reduced the ambiguity by separating "generic sensitive file" from "file that belongs to the web app configuration", then relied on the payload clue that pointed to `config.php`.
6. The database password did not require inference: it was directly visible in the exfiltrated configuration content.
7. The MySQL question was the main friction point because the requested method was inflated compared to the actual task: after the credential exposure, the only thing that mattered was the first `mysql.login_request` that followed it.
8. The web-shell question was straightforward once command execution appeared directly in the URI through requests to `/uploads/booking.php?cmd=...`.

## Key Evidence
- Open ports relevant to the target host during analysis:
  - `80`
  - `3306`
- Highest-numbered open port:
  - `3306`
- Vulnerable PHP endpoint:
  - `/review/upload.php`
- First malicious XML file uploaded:
  - `TheGreatGatsby.xml`
- Web app configuration file read through XXE:
  - `config.php`
- Database password exposed in the payload:
  - `Winter2024`
- Timestamp used as the pivot for credential exposure during analysis:
  - `2024-05-31 12:03:12`
- First MySQL login after credential exposure:
  - `2024-05-31 12:08:49 UTC`
- Web shell identified from command-execution requests:
  - `booking.php`
- Clear command-execution evidence:
  - `/uploads/booking.php?cmd=whoami`
  - `/uploads/booking.php?cmd=uname%20-a`

## Findings
- The lab was easy overall.
- Most answers were visible almost directly once each question was stripped down to its actual technical meaning.
- The main analytical effort was not packet complexity but wording cleanup.
- The vulnerable-endpoint question was simple because XML traffic immediately narrowed the search space:
  - Observed: repeated attacker `POST` requests carrying XML reached `/review/upload.php`
  - Conclusion: that URI was the vulnerable PHP script
- The first-malicious-file question did not need deep timeline work:
  - Observed: the uploads were already visible in chronological order
  - Conclusion: the first multipart `filename` was enough to answer it
- The configuration-file question was badly phrased:
  - Observed: "sensitive file" and "web app configuration file" could be conflated if read too fast
  - Conclusion: the real task was to find the file belonging to the application’s configuration, not just any interesting file
- The password question was trivial once the right payload had been opened:
  - Observed: the password was plainly visible in the leaked configuration content
  - Conclusion: this was extraction, not investigation
- The MySQL question was the most irritating part of the lab:
  - Observed: the real logic was only "find the first MySQL login after the credentials were exposed"
  - Conclusion: the long explanation around it made a simple temporal correlation look more complex than it was
- The web-shell question was also obvious:
  - Observed: `booking.php` was called with a `cmd=` parameter and returned command-execution output
  - Conclusion: no serious deduction was needed once those requests were visible

## Result
- Highest-Numbered Open Port: `3306`
- Vulnerable PHP Script: `/review/upload.php`
- First Malicious XML File: `TheGreatGatsby.xml`
- Web App Configuration File: `config.php`
- Compromised Database Password: `Winter2024`
- First MySQL Login After Credential Exposure: `2024-05-31 12:08:49 UTC`
- Uploaded Web Shell: `booking.php`

## Mistakes / Friction Points
- The main weakness of the lab was question wording, not traffic complexity.
- Some questions were more ambiguous than they needed to be:
  - `highest-numbered port open on the victim's web server` could be misread as "web port" instead of "highest port on the same host"
  - `web app configuration file` could be misread as "any sensitive file"
- The MySQL question was the most inflated one:
  - the actual work was just finding the first `mysql.login_request` after the leak
  - the challenge explanation made that simple step sound more investigative than it really was
- Because the lab is closed and the traffic is very direct, several answers could be reached almost immediately without broad reconstruction.
- Overinvestigating some steps would have added noise rather than value.

## Notes
- In a real investigation, stronger confirmation would still come from:
  - web server logs
  - application logs
  - database logs
  - host-side forensic evidence
  - source code review of the vulnerable upload handler
- In this lab, many answers were directly exposed in packet payloads, so the most useful mindset was to reduce each question to its core technical request and ignore the inflated narrative around it.
- The most annoying part was not the exploit chain itself, but the mismatch between how simple some answers were and how overexplained some hints or writeups tried to make them look.
