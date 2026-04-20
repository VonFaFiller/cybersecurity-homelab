# Defensive Cybersecurity Home Lab & PCAP Analysis

## Overview

This repository documents my practical cybersecurity learning path, focused on defensive investigation, PCAP analysis, Windows/Linux lab exercises, and progressive blue team methodology.

The goal is to build evidence of practical skills through documented investigations, not just notes or theoretical summaries.

The repository includes:

- external PCAP analysis labs
- internal Windows and Linux lab exercises
- packet analysis workflows
- defensive investigation notes
- concepts studied when they became relevant during analysis
- future Security Onion and monitoring work

## Main Focus

The current focus is practical defensive analysis, especially:

- PCAP triage
- Wireshark investigation
- attacker activity reconstruction
- network protocol interpretation
- Windows artifact analysis
- Linux and Windows lab testing
- basic detection and investigation methodology

## Repository Structure

```text
docs/
  labs/
    external/
      malware-analysis/
      pcap-analysis/
    internal/
      windows/
      linux/
      analysis/

  concepts/
    networking/
    protocols/
    endpoint/
    malware-traffic/
````

## External Labs

External labs are used to practice investigation on prepared datasets, mainly PCAP-based challenges.

These write-ups document:

* initial triage
* traffic filtering
* protocol inspection
* attacker and victim identification
* payload or command reconstruction
* reasoning process
* lab limitations when the scenario is unrealistic or overly guided

The purpose is not only to recover answers, but to document how the investigation was approached.

## Internal Labs

Internal labs are controlled exercises built in my own environment.

They are used to test and observe specific behaviors such as:

* Windows user creation
* scheduled tasks
* authentication events
* service state changes
* batch execution
* Linux permissions
* SSH activity
* HTTP and network behavior

These exercises are intentionally simple at first, because they are used to understand what specific actions produce at the system, log, and network level.

## Concepts

The `concepts/` section is not a generic theory dump.

It contains technical concepts studied because they appeared during labs or investigations.

Examples:

* TCP connection behavior in PCAP analysis
* DNS resolution during traffic reconstruction
* HTTP requests, POSTs, uploads, and webshell behavior
* TLS metadata and encrypted traffic analysis
* SMB and Windows lateral movement basics
* IP addressing, NAT, and traffic scope

Each concept should explain:

* why I studied it
* what the core idea is
* what it looks like during analysis
* how it helps avoid wrong conclusions

## Tools Used

* Wireshark
* VMware Workstation
* Windows
* Ubuntu
* PowerShell
* Linux shell
* CyberDefenders labs
* Security Onion planned for later monitoring work

## Documentation Standard

For lab write-ups, the goal is to preserve the actual reasoning process rather than producing a polished report that hides how the answer was reached.

## Current Status

Completed / active:

* PCAP analysis write-ups
* CyberDefenders external labs
* Windows internal lab tests
* Ubuntu internal lab tests
* basic investigation documentation

Planned:

* Security Onion deployment
* more realistic internal traffic generation
* malware traffic analysis
* endpoint and network visibility correlation
* reusable investigation methodology

## AI Usage Disclosure

AI may be used to assist with wording, grammar, formatting, and structure.

The technical analysis, lab work, investigation process, troubleshooting, conclusions, and documentation decisions are my own.

## Disclaimer

This repository is for educational and defensive cybersecurity learning only.

All internal exercises are performed in a controlled personal lab environment.
