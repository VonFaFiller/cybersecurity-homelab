# PsExec Hunt Lab (CyberDefenders)

## Objective
Analyze network traffic to identify PsExec-based lateral movement, reconstruct the attack sequence across internal hosts, and determine how the attacker authenticated, installed the remote service, and moved to additional machines.

## Scenario
The exercise is based on a PCAP containing suspicious internal Windows traffic consistent with remote administration abuse.

The goal is to reconstruct the attacker’s actions by identifying:
- the initially compromised internal machine
- the first pivot target
- the account used during authentication
- the network shares used for service deployment and communication
- the later pivot activity to additional hosts
- the overall lateral movement pattern

## References
- CyberDefenders: PsExec Hunt Lab
- MITRE ATT&CK tactics observed:
  - Lateral Movement
  - Credential Access
  - Execution
  - Discovery

## Tools Used
- Wireshark
- Statistics → Conversations
- Statistics → Endpoints
- SMB / SMB2 packet inspection
- NTLMSSP authentication inspection
- Follow TCP Stream
- Display filters
- Packet details pane

## Investigation Workflow
1. Identified the most active internal conversations using Statistics → Conversations.
2. Focused on SMB / SMB2 traffic because the activity pattern was consistent with Windows remote administration and possible PsExec use.
3. Distinguished the likely attacker-controlled internal host from the remote target hosts by checking traffic direction, SMB session setup, and chronology.
4. Isolated the suspicious SMB authentication traffic to determine which machine initiated the connections.
5. Inspected NTLMSSP fields to extract the username and the source workstation name shown in authentication metadata.
6. Reviewed SMB tree connections to determine which administrative shares were accessed during the remote operation.
7. Confirmed that `ADMIN$` was used to deploy the service and `IPC$` was used for inter-machine communication.
8. Reconstructed the first pivot from the initially compromised host to the first target machine.
9. Followed the later SMB activity to identify the next machine targeted during additional lateral movement.
10. Rebuilt the sequence as an internal compromise followed by PsExec-based remote execution and further pivoting.

## Key Evidence
- Initially compromised internal host:
  - `10.0.0.130`
- Source workstation hostname observed in authentication:
  - `HR-PC`
- Account used in SMB / NTLM authentication:
  - `ssales`
- First pivot target IP:
  - `10.0.0.133`
- Administrative share used to install the service:
  - `ADMIN$`
- Share used for communication between the two machines:
  - `IPC$`
- Second machine later targeted for pivoting:
  - `MARKETING-PC`
- PsExec-related remote execution artifact observed:
  - `PSEXESVC`

## Findings
- The earliest suspicious internal activity originated from `10.0.0.130`, making it the most likely attacker-controlled machine already under unauthorized control.
- SMB / SMB2 was the most useful protocol to investigate because the attack behavior was not web-oriented or user-browsing-oriented; it was consistent with Windows remote administration and lateral movement.
- NTLM authentication data revealed the username `ssales` and the source workstation name `HR-PC`, which helped distinguish the initiating machine from the remote target.
- Access to `ADMIN$` indicated remote service deployment behavior, while use of `IPC$` was consistent with PsExec communication and remote execution workflow.
- The traffic pattern showed the attacker moving first from the compromised source host to `10.0.0.133`, then later pivoting again to another host identified as `MARKETING-PC`.
- The sequence was consistent with PsExec abuse for authenticated lateral movement inside the network rather than an exploit against an exposed external service.

## Result
- Initial Compromised Host: `10.0.0.130`
- Source Hostname: `HR-PC`
- Username Used: `ssales`
- First Pivot Target IP: `10.0.0.133`
- Service Installation Share: `ADMIN$`
- Communication Share: `IPC$`
- Later Pivot Target Hostname: `MARKETING-PC`
- Attack Pattern: `Initial internal compromise → SMB/NTLM authentication → ADMIN$ service deployment → IPC$ communication → PsExec lateral movement → further pivoting`

## Mistakes / Friction Points
- The lab wording pushed early attribution before the full chain was fully reconstructed, even though in a real investigation the initial compromised host would normally be confirmed later in the process.
- It was necessary to distinguish between:
  - the machine already under attacker control
  - the first remote target of lateral movement
  - later machines targeted afterward
- The `Host` field inside NTLM authentication could easily be misread as the remote destination host, when in context it pointed to the source workstation presenting the authentication.

## Notes
- Statistics → Conversations was useful for triage, but it was not enough on its own to prove which machine was initially compromised.
- The decisive step was reading SMB / SMB2 directionality together with NTLM authentication details and share usage.
- In PsExec-style investigations, SMB / SMB2 is especially valuable because it can expose:
  - authentication context
  - administrative share access
  - service deployment behavior
  - named-pipe-based communication
- The reconstruction worked because the PCAP preserved the activity in a readable order: source authentication, remote share access, service-related actions, and later pivoting to another host.
- In a real case, stronger confirmation would still be needed from endpoint logs, process creation evidence, service creation logs, and host-level forensic artifacts.
