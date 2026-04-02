# Web Investigation Lab (CyberDefenders)

## Objective
Analyze web traffic to identify SQL injection activity against the BookWorld application, reconstruct the attack sequence from initial probing to database enumeration and data access, and determine which web endpoint and backend data structures were involved.

## Scenario
The exercise is based on a PCAP containing suspicious HTTP traffic associated with abnormal database activity against the BookWorld website.

The goal is to reconstruct the attacker’s actions by identifying:
- the web endpoint targeted by the attacker
- the first observable SQL injection attempts
- the progression from probing to database enumeration
- the database identified as relevant to the website
- the table containing website user data
- the overall attack pattern shown in the web traffic

## References
- CyberDefenders: Web Investigation Lab
- MITRE ATT&CK tactics / techniques observed:
  - Initial Access
  - Discovery
  - Credential / Data Access
  - Exploitation of Public-Facing Application
  - SQL Injection

## Tools Used
- Wireshark
- HTTP packet inspection
- Follow TCP Stream
- Display filters
- Packet details pane
- URL decoding methods
  - Online URL decoder
  - PowerShell decoding with `[System.Uri]::UnescapeDataString(...)`

## Investigation Workflow
1. Reviewed the HTTP traffic to identify the web application endpoint repeatedly targeted by suspicious requests.
2. Focused on requests to `search.php`, because the same script and parameter appeared repeatedly with progressively abnormal values.
3. Distinguished normal-looking search activity from malicious probing by identifying encoded special characters, SQL-like operators, and values inconsistent with ordinary user searches.
4. Isolated the earliest suspicious requests in chronological order to determine where benign-looking usage ended and active backend manipulation began.
5. Decoded the request values to reconstruct the real SQL injection payloads, rather than relying only on the percent-encoded representation shown in the packets.
6. Examined the progression of the requests to understand whether the attacker was only testing syntax, verifying true / false behavior, or moving into schema enumeration.
7. Identified requests targeting `INFORMATION_SCHEMA` to confirm that the attacker had moved from simple probing into structured database discovery.
8. Confirmed that `bookworld_db` was the relevant website database based on enumeration results observed in the HTTP traffic.
9. Followed the later requests that enumerated tables within `bookworld_db`.
10. Verified that the table later used for user-related field extraction was `customers`, based on follow-up requests referencing user data fields such as email, first name, last name, phone number, and address.
11. Reconstructed the attack sequence as web probing followed by SQL injection, schema discovery, table discovery, and targeted extraction of website user information.

## Key Evidence
- Repeatedly targeted web script:
  - `search.php`
- Attack vector observed in traffic:
  - SQL injection through the `search` parameter
- Relevant application database identified through enumeration:
  - `bookworld_db`
- Table containing website user data:
  - `customers`
- Evidence of structured backend discovery:
  - `INFORMATION_SCHEMA.SCHEMATA`
  - `INFORMATION_SCHEMA.TABLES`
- Evidence of later organized extraction behavior:
  - use of SQL functions such as `CONCAT` and `JSON_ARRAYAGG`
- Decoding methods used during analysis:
  - Online URL decoder
  - PowerShell decoding

## Findings
- The traffic showed a clear shift from ordinary-looking web requests to crafted requests directed at the same application endpoint, indicating that the attacker had identified a promising input field and was iterating on it.
- `search.php` was the observed exploited web endpoint, because the malicious requests consistently targeted that script and altered the value of the same parameter in a way consistent with SQL injection testing and exploitation.
- The strongest observation was not simply that `search.php` appeared in the traffic, but that its parameter values changed from plausible search terms to encoded payloads containing syntax consistent with backend manipulation.
- The decoded payloads revealed a progression from initial probing into database enumeration, which strengthened the conclusion that the activity was not random malformed input but deliberate SQL injection.
- Requests involving `INFORMATION_SCHEMA` showed that the attacker had advanced from testing the injection point to discovering the database structure.
- Enumeration results identified `bookworld_db` as the database relevant to the web application.
- Later requests showed table discovery and direct targeting of `customers`, and that table was then associated with user-related fields, making it the defensible conclusion for the website users data table.
- In a realistic investigation, the statement “`search.php` is the vulnerable PHP script” should be handled carefully:
  - **Observed:** malicious requests repeatedly targeted `search.php`
  - **Strong inference:** `search.php` was the exploited web endpoint
  - **Not fully proven from PCAP alone:** whether the vulnerable code physically resided inside `search.php` itself or in included backend logic
- The lab was straightforward overall, but the request-decoding step introduced more friction than the wording of the question suggested.

## Result
- Exploited Web Endpoint: `search.php`
- Attack Type: `SQL injection via HTTP GET parameter`
- Relevant Database: `bookworld_db`
- Website Users Data Table: `customers`
- Attack Pattern: `Initial web probing → encoded SQLi attempts → backend behavior testing → schema enumeration → table enumeration → user data extraction`

## Mistakes / Friction Points
- The lab wording often encouraged direct answer extraction before clearly separating:
  - what was directly visible
  - what was strongly suggested
  - what was actually proven
- The exercise was easy overall, but the decoding stage was more operationally annoying than analytically difficult.
- There was a risk of treating “the script observed in the malicious request” as automatically identical to “the exact location of the vulnerable code,” which is acceptable for challenge scoring but weaker as real investigative language.
- The decoding requirement could feel disproportionately dispersive because the question looked simple, while the actual answer path involved export choices, decoding steps, and filtering out unnecessary text.

## Notes
- The most useful mindset was to identify the point where normal web usage stopped and controlled backend interaction began.
- For this reason, the key transition was not the presence of `search.php` alone, but the first abnormal value sent to its `search` parameter.
- In this lab, URL decoding was necessary because the meaningful payload content was hidden inside percent-encoded request values.
- Both an online decoder and PowerShell were used during analysis to reconstruct the readable payloads.
- In practical workflow terms, the decoding step could have been handled more directly if only the relevant `Request URI` values had been extracted first, instead of working from noisy full packet dissection exports.
- In a real investigation, stronger confirmation would still come from:
  - web server logs
  - PHP / application error logs
  - database logs
  - source code review
  - host-side forensic evidence
