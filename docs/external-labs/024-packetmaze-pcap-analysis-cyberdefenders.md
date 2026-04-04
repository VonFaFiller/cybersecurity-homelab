# PacketMaze PCAP Analysis (CyberDefenders)

## Objective
Analyze a mixed-protocol PCAP to identify relevant client and server activity across FTP, DNS, TLS, and web traffic, reconstruct the meaning of challenge questions that were often written ambiguously, and recover the requested answers without confusing protocol roles or traffic layers.

## Scenario
The exercise is based on a PCAP containing several different types of traffic rather than a single narrowly scoped service interaction.

The goal is to reconstruct the relevant activity by identifying:
- the FTP credentials and file-transfer activity
- the FTP server and its MAC-based vendor information
- the IPv6 DNS server associated with the FTP host
- the first TLS 1.3 client random used in a connection to `protonmail.com`
- the non-standard folder timestamp visible through FTP directory listing activity
- the URL associated with a connection to the IP address `104.21.89.171`
- other protocol-specific details embedded in the traffic

## References
- CyberDefenders: PacketMaze Lab
- Main protocol families observed during analysis:
  - FTP / FTP-DATA
  - DNS
  - IPv6 / ICMPv6
  - HTTP / HTTPS
  - TLS 1.3
  - Ethernet / MAC addressing

## Tools Used
- Wireshark
- Statistics → Conversations
- Statistics → Endpoints
- Packet details pane
- Follow TCP Stream
- Display filters
- OUI / MAC vendor lookup
- External lookup for vendor-country enrichment when required

## Investigation Workflow
1. Reviewed the traffic at a high level to understand that the PCAP was not limited to a single service role, and that the client was interacting with FTP, DNS, and web services in parallel.
2. Identified the FTP host and treated it as one node within a broader mixed-traffic scenario rather than assuming that every question referred only to FTP activity.
3. Used FTP control traffic to identify commands, authentication behavior, and transfer intent.
4. Distinguished FTP control traffic from FTP-DATA, because commands such as `LIST` only described the action, while the actual directory contents were carried on the associated data channel.
5. Used passive-mode FTP logic only when necessary: starting from `LIST`, checking the preceding `227 Entering Passive Mode`, and then following the matching FTP data stream.
6. Interpreted DNS-related questions carefully to avoid confusing the IPv4 client host with the IPv6 address of the DNS server it was using.
7. Tracked MAC-level evidence separately from IP-level evidence, because some questions required vendor enrichment from the MAC OUI rather than any direct IP-based attribution.
8. Used OUI resolution to identify the FTP server NIC vendor and noted that the final country-of-registration question required external enrichment rather than packet evidence alone.
9. Examined TLS handshake traffic to clarify where `Client Hello`, `Server Hello`, and the `Client Random` actually appear, instead of answering by guesswork.
10. For the `protonmail.com` question, used the `Client Hello` and TLS 1.3 context to isolate the right request chronologically, while also noting that some extra validation steps were curiosity-driven rather than strictly required by the lab.
11. Interpreted IP-to-URL questions by treating them as web-traffic reconstruction problems rather than as questions about the FTP server itself.
12. Explored transferred image content and metadata out of curiosity to confirm where file bytes and EXIF-style data actually travel inside FTP sessions.

## Key Evidence
- FTP server identified in the analysis:
  - `192.168.1.26`
- Important protocol distinction during analysis:
  - FTP control traffic showed commands such as `USER`, `PASS`, `LIST`, `RETR`, `STOR`
  - FTP-DATA carried the real file bytes and directory listing contents
- Evidence relevant to passive FTP listing analysis:
  - `227 Entering Passive Mode (...)`
  - subsequent `FTP Data ... (PASV) (LIST)` stream
- Evidence that the non-standard folder timestamp had to be read from data-channel output:
  - the useful directory listing was not contained in the control-channel `LIST` request itself
- FTP server MAC address identified during analysis:
  - `c8:09:a8:57:47:93`
- Vendor resolution derived from MAC OUI:
  - `Intel`
- DNS question clarification required during analysis:
  - the challenge asked for the IPv6 address of the DNS server used by `192.168.1.26`
  - it did **not** ask for the IPv6 address of `192.168.1.26` itself
- TLS handshake clarification required during analysis:
  - the `Client Random` was read from `Client Hello`
  - the question wording did not require deeper validation of full connection establishment, although that possibility was considered
- Web-traffic reconstruction clarification required during analysis:
  - the IP `104.21.89.171` had to be mapped back to the URL/site visited by the client
- Curiosity-driven validation about transferred files:
  - image bytes and metadata were expected on FTP-DATA rather than on the FTP control channel

## Findings
- The main analytical difficulty in this lab was not packet complexity but question wording.
- The traffic itself was mostly easy to interpret once the protocol role of each flow was understood.
- The first major structural point was that the PCAP was mixed-protocol:
  - **Observed:** FTP, DNS, TLS, and web-related traffic were all present
  - **Conclusion:** not every question referred to the same host role or the same protocol context
- A recurring source of friction was vague wording such as `the user`, `the server`, or `the URL visited`:
  - **Observed:** several questions omitted the precise technical subject
  - **Conclusion:** the real task was often to first translate the challenge wording into a precise network question before filtering packets
- FTP required a specific control-versus-data distinction:
  - **Observed:** `LIST` appeared on the control channel, but the actual directory contents did not
  - **Observed:** the relevant listing appeared on the associated `FTP Data ... (PASV) (LIST)` stream
  - **Conclusion:** following the control stream alone was insufficient for timestamp and folder-name questions
- The MAC-vendor question also required a layer distinction:
  - **Observed:** the packet evidence gave the MAC address and OUI-derived vendor
  - **Conclusion:** the final country-of-registration answer required an external lookup and was not recoverable from the PCAP alone
- The TLS 1.3 question was straightforward once the handshake structure was understood:
  - **Observed:** the requested value was the `Random` field in `Client Hello`
  - **Conclusion:** understanding the handshake was useful, but some extra reasoning about whether the connection was fully established went beyond what the challenge strictly required
- The IP-to-URL question looked more confusing than it actually was:
  - **Observed:** the scenario included both FTP and external web traffic
  - **Conclusion:** the question was asking for the site associated with that IP, not for anything about the FTP server
- Some of the deeper reasoning done during the lab was curiosity-driven rather than required:
  - understanding why a host talks to DNS
  - understanding where image metadata appears in FTP transfers
  - understanding the structure of the TLS handshake
- The rest of the lab was easy overall, and many answers were found immediately once the correct protocol perspective was chosen.

## Result
- FTP Server: `192.168.1.26`
- FTP Server MAC Address: `c8:09:a8:57:47:93`
- FTP Server MAC Vendor: `Intel`
- FTP Password: `<insert confirmed password>`
- IPv6 DNS Server Used by `192.168.1.26`: `<insert confirmed IPv6 address>`
- First TLS 1.3 Client Random for `protonmail.com`: `<insert confirmed client random>`
- Non-Standard FTP Folder Timestamp on April 20: `<insert confirmed time>`
- URL Connected to `104.21.89.171`: `<insert confirmed URL>`
- Manufacturer Country for FTP Server MAC Vendor: `<insert confirmed country>`

## Mistakes / Friction Points
- The main difficulty was the ambiguity of the challenge wording:
  - several questions did not clearly specify whether they referred to a client, a server, an IP, a domain, or a protocol role
- A second friction point was the lack of explicit context:
  - the exercise did not clearly tell which host played which role
  - this made some questions look more obscure than they really were
- The FTP directory-listing part was made more tedious by the way it is usually explained:
  - the control `LIST` request alone was not enough
  - the real answer had to be read from the associated FTP data stream
- Some additional time was spent on mechanism-level understanding out of curiosity rather than necessity:
  - DNS communication logic
  - TLS handshake structure
  - metadata
- The remaining parts that are not emphasized here were mostly easy and were often solved immediately once the protocol context was clear.

## Notes
- In a real investigation, stronger context would normally already exist in the form of:
  - asset inventory
  - host naming
  - network diagrams
  - DHCP / DNS logs
  - proxy or firewall logs
- Because of that, many of the ambiguities present in this lab would usually be clarified much earlier in a real workflow.
- For FTP-specific questions, server-side logs would confirm uploads, listings, and directory timestamps more directly than reconstructing them only from passive-mode packet traces.
- For MAC-vendor questions, the PCAP can support vendor identification through OUI resolution, but not all enrichment details are contained in the packets themselves.
- The curiosity-driven review of image transfers confirmed a useful practical rule:
  - FTP control traffic shows commands and filenames
  - FTP-DATA carries the actual file bytes and embedded metadata
