# MITRE T1595 – PCAP Analysis (CyberDefenders)

## Objective
Analyze network traffic to identify reconnaissance activity, determine the scanning behavior, identify the targeted network range, and map the observed activity to the correct MITRE ATT&CK technique.

## Scenario
The exercise is based on a PCAP containing network traffic generated during a scanning phase.

The goal is to reconstruct the reconnaissance activity by identifying:
- the scanning host
- the targeted network range
- the ports being probed
- the overall scanning pattern
- the associated MITRE ATT&CK technique

## References
- https://cyberdefenders.org/blueteam-ctf-challenges/achievements/VonFaFiller/t1595/
- https://attack.mitre.org/techniques/T1595/

## Tools Used
- Wireshark
- Statistics → Endpoints
- Statistics → Conversations
- TCP packet inspection
- Display filters

## Investigation Workflow
1. Identified the most active IP addresses using Statistics → Endpoints.
2. Isolated the suspicious external host interacting with multiple internal targets.
3. Observed repeated connection attempts across different IP addresses within the same subnet.
4. Identified the range of targeted IP addresses based on destination patterns.
5. Inspected TCP traffic to determine which ports were being probed.
6. Confirmed that the connections were short-lived and repetitive, indicating scanning behavior.
7. Mapped the observed behavior to a reconnaissance technique using MITRE ATT&CK.

## Key Evidence
- Scanning source IP:
  - `185.245.85.178`
- Targeted network range:
  - `192.168.196.0/24`
- Multiple internal hosts contacted within the same subnet.
- Repeated TCP connection attempts across different hosts.
- Short-lived connections without full session establishment.
- Consistent probing behavior across multiple targets.

## Findings
- One external host initiated connections to multiple internal IP addresses.
- The traffic pattern showed systematic probing across a full subnet.
- The behavior was repetitive and lacked signs of exploitation or payload delivery.
- The focus was on identifying reachable hosts and open services.
- This activity corresponds to reconnaissance rather than intrusion.

## Result
- Attacker IP: `185.245.85.178`
- Target Network: `192.168.196.0/24`
- Activity Type: `Network Scanning`
- MITRE Technique: `T1595 – Active Scanning`
- Sub-technique: `T1595.001 – Scanning IP Blocks`

## Mistakes / Friction Points
- Misleading hint suggesting external threat intelligence lookup, which was unnecessary for this exercise.

## Notes
- The key to solving the exercise was recognizing the pattern rather than analyzing single packets in isolation.
- Mapping behavior to MITRE requires interpretation, not direct extraction from traffic.
- Using Statistics views in Wireshark significantly simplifies identifying scanning patterns.
