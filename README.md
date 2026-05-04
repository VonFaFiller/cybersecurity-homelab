# Cybersecurity Homelab

Practical cybersecurity documentation focused on defensive analysis, network traffic investigation, malware triage, threat intelligence, and controlled lab testing.

The goal of this repository is to show how I approach investigations: what I check first, which artifacts I use, how I confirm or reject assumptions, and how I document the reasoning behind each answer.

## Focus Areas

- PCAP analysis and Wireshark-based investigation
- Malware analysis and malware traffic interpretation
- Threat intelligence and IOC pivoting
- Windows and Linux lab testing
- Network protocol notes for practical analysis
- Defensive methodology and evidence-based reconstruction

## Repository Structure

```text
docs/
  labs/
    external-labs/
      pcap-analysis/
      malware-analysis/
      threat-intel/

    internal-labs/
      windows/
      ubuntu/
      analysis/

  concepts/
    networking/
````

## External Labs

External labs are used to practice investigation on prepared datasets, mainly from CyberDefenders.

The write-ups focus on the reasoning process, not only on the final answer.

Typical workflow:

```text
initial triage
→ relevant artifacts
→ filters / pivots
→ suspicious behavior
→ confirmation
→ answer
```

Main lab categories:

* `pcap-analysis/` — network traffic reconstruction, protocol analysis, attacker/victim identification
* `malware-analysis/` — malware behavior, dropped files, payloads, execution chains
* `threat-intel/` — hash pivoting, VirusTotal, ANY.RUN, registrars, C2s, threat actor attribution

## Internal Labs

Internal labs are controlled tests built in my own environment.

They are used to understand what specific actions produce at the system, log, and network level.

Examples:

* Windows user and process activity
* PowerShell and batch execution
* Linux permissions and SSH behavior
* service changes
* scheduled tasks
* basic network behavior

## Concepts

The `concepts/` section contains short technical notes linked to what appears during labs.

It is not meant to be a generic theory dump.

Current focus:

* TCP, UDP, DNS, HTTP, TLS
* SMB, Kerberos, LDAP, NTLM
* ICMP, ARP, DHCP
* RDP, SSH, QUIC
* mail protocols
* DCE/RPC and Windows network behavior

Each page is written for quick consultation during analysis.

## Tools

* Wireshark
* VirusTotal
* ANY.RUN
* MalwareBazaar / threat intel sources
* VMware Workstation
* Windows
* Ubuntu
* PowerShell
* Linux shell
* CyberDefenders labs

## Documentation Style

The documentation preserves the investigation flow.

I prefer to show:

* what I observed
* what looked suspicious
* what I filtered or searched
* what confirmed the conclusion
* where the lab was unrealistic or too guided

This is intentional: the objective is to document analytical process, not only clean final results.

## Current Status

Completed / active:

* PCAP analysis labs
* malware analysis labs
* threat intelligence labs
* Windows and Ubuntu internal lab tests
* networking concepts for analysis

Planned:

* Security Onion deployment
* more realistic internal traffic generation
* endpoint and network visibility correlation
* reusable investigation workflows

## AI Usage Disclosure

AI may be used to assist with wording, grammar, formatting, and structure.

The technical analysis, lab work, investigation process, troubleshooting, conclusions, and documentation decisions are my own.

## Disclaimer

This repository is for educational and defensive cybersecurity learning only.

All internal exercises are performed in a controlled personal lab environment.

[1]: https://github.com/VonFaFiller/cybersecurity-homelab/tree/main/docs/labs/external-labs "cybersecurity-homelab/docs/labs/external-labs at main · VonFaFiller/cybersecurity-homelab · GitHub"
[2]: https://github.com/VonFaFiller/cybersecurity-homelab/tree/main "GitHub - VonFaFiller/cybersecurity-homelab: Cybersecurity home lab for blue team practice using VMware, Ubuntu, Windows and Security Onion. · GitHub"
