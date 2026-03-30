# PacketDetective – PCAP Analysis (CyberDefenders)

## Objective
Analyze multiple PCAP segments to identify suspicious authentication activity, remote administrative behavior, event log tampering, named pipe usage, and covert account setup across Windows network traffic.

## Scenario
The exercise is based on packet captures containing SMB, SMB2, NTLMSSP, DCE/RPC, and related Windows administrative traffic.

The goal is to reconstruct suspicious activity by identifying:
- authentication behavior
- event log clearing attempts
- named pipe usage
- covert username creation or usage
- communication duration between specific hosts

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/achievements/VonFaFiller/packetdetective/
- https://cyberdefenders.org/walkthroughs/packetdetective/

## Tools Used
- Wireshark
- Statistics → Conversations
- SMB / SMB2 packet inspection
- NTLMSSP field analysis
- DCE/RPC packet inspection
- Packet Bytes pane
- raw byte filters using `frame contains`

## Investigation Workflow
1. Identified suspicious SMB and SMB2 authentication traffic and extracted usernames from NTLM authentication messages.
2. Used Wireshark statistics to review communication duration between the relevant hosts.
3. Investigated event log clearing activity by isolating the corresponding request and checking the real UTC timestamp.
4. Traced remote administrative behavior across SMB, SMB2, and DCE/RPC traffic.
5. Pivoted into named pipe analysis when standard packet summaries were not sufficient.
6. Applied a raw byte filter for `\PIPE` to locate the actual pipe name inside Packet Bytes.
7. Distinguished between transport, named pipe, RPC interface, operation name, and username.

## Key Evidence
- Suspicious host pair observed in one communication flow:
  - `172.16.66.1`
  - `172.16.66.36`
- Named pipe value identified in raw packet bytes:
  - `\PIPE\atsvc`
- Pipe token used as the answer:
  - `atsvc`
- DCE/RPC traffic was observed after the SMB and SMB2 transport stage.
- NTLM authentication messages exposed a non-standard username used in suspicious requests.
- Event log clearing activity was visible through `ClearEventLog request`.
- Duration-related questions were answered through Wireshark statistics rather than packet-by-packet inspection.

## Findings
- Most of the exercise was relatively direct once the correct protocol or packet type was selected.
- Username-related questions were answered by pivoting to:
  - `Session Setup`
  - `NTLMSSP_AUTH`
- Duration-related questions were answered more efficiently from:
  - `Statistics`
  - `Conversations`
- Event log clearing questions required reading the real packet timestamp rather than the relative `Time` column.
- The most difficult part of the exercise was identifying the named pipe correctly.
- That question was objectively more awkward than the rest of the lab because the answer was not clearly exposed in the higher-level dissections.
- The wording also made the pivot less obvious, because the expected answer was the pipe token rather than the RPC interface name.
- The useful string had to be extracted from raw frame content and Packet Bytes instead of from the more convenient protocol summary fields.

## Result
- The lab reinforced practical packet analysis for:
  - SMB / SMB2
  - NTLMSSP authentication
  - DCE/RPC communication
  - event log tampering
  - named pipe identification
- The hardest friction point was the named pipe question.
- The rest of the lab was significantly more linear and easier to map from question to field.
- Time spent struggling on the named pipe was not wasted:
  - it led to a much better understanding of how SMB and SMB2 transport DCE/RPC traffic
  - it also made later username identification much faster because the NTLM authentication flow became clearer

## Mistakes / Friction Points
- The main error was not technical inability, but choosing the wrong evidence layer for the question.
- I initially treated the named pipe question as if the answer should come directly from:
  - SMB2 summaries
  - DCE/RPC context items
  - RPC interface names
- This led to confusion between:
  - named pipe name
  - RPC interface name
  - requested operation
- From now on, named pipe questions will be handled with filters first, not by manually sorting protocols or guessing from dissections.

## Notes
- Spending too much time on the named pipe question still had a useful outcome.
- It forced a deeper inspection of DCE/RPC than I would normally have done in a simple easy-level lab.
- Because of that, I now understand the chain more clearly:
  - `TCP`
  - `SMB / SMB2`
  - `named pipe`
  - `DCE/RPC`
  - `authentication / request / response`
- That extra effort did not help only for the pipe question.
- It also made later username identification much faster because I understood where NTLM authentication details actually appeared.
