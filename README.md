# Defensive Cybersecurity Home Lab (WIP)

## Overview
This repository documents the build and development of a personal **defensive cybersecurity home lab** created for **hands-on learning**, **technical experimentation**, and **progressive blue team practice**.

The primary purpose of this project is practical study through direct interaction with systems, networks, logs, and defensive tooling.  
A secondary purpose is to maintain clear technical documentation that can also serve as evidence of skill development over time.

This repository reflects the **actual implementation state** of the lab and does not present unfinished components as completed.

## Learning Focus
This lab is being developed to support practical learning in:

- Linux system administration
- Windows system fundamentals
- virtualization
- basic networking
- system and network visibility
- log analysis
- troubleshooting
- defensive investigation workflows

## Current Status
### Completed
- VMware environment prepared
- Ubuntu virtual machine installed
- Windows virtual machine installed

### In Progress / Planned
- Ubuntu baseline configuration and basic administration practice
- Security Onion Standalone deployment
- Virtual network design and validation
- Initial logging, telemetry, and analysis exercises

## Current Study Strategy
The lab is being built in stages.

### Stage 1 — Foundations
The initial phase focuses on:
- Ubuntu VM
- Windows VM
- VMware networking
- users, groups, permissions, services, processes, and logs
- repeatable exercises using snapshots and reversible changes

This stage is intended to build operational familiarity before introducing the central monitoring platform.

### Stage 2 — Central Visibility
After the foundation phase, the lab will expand to include:
- Security Onion Standalone
- Windows host as a realistic endpoint
- Ubuntu VM as a lightweight auxiliary system used when needed

At this stage, the lab will support both:
- **host visibility**
- **network visibility**

### Stage 3 — Controlled and Realistic Observation
The environment will then be used in multiple ways depending on the exercise:

- **Ubuntu VM + Windows VM**
- **Security Onion + Windows host**
- **Security Onion + Ubuntu VM**
- **Security Onion + Windows host + Ubuntu VM**
- **Security Onion + Windows VM** for more controlled and reversible testing

The Windows VM is intended to remain a **sandbox system**, not the primary endpoint.

## Planned Architecture

### Host System
- Host OS: Windows 11 Pro
- CPU: AMD Ryzen 7 7800X3D
- Memory: 32 GB RAM
- Storage: 1.8 TB NVMe SSD
- Hypervisor: VMware Workstation

### Virtual Machines
- **Ubuntu VM**
  - currently installed
  - used for Linux administration, services, networking, and controlled test traffic

- **Windows VM**
  - currently installed
  - used as a reversible sandbox for controlled Windows exercises

- **Security Onion Standalone**
  - planned
  - used as the main monitoring and analysis platform

## Design Rationale
This lab is intentionally structured around two different goals:

1. **controlled experimentation**
   - performed mainly in virtual machines
   - suitable for repeatable exercises, snapshots, and rollback

2. **realistic observation**
   - performed using the actual Windows host as an endpoint
   - useful for observing real telemetry, background noise, and normal system activity

This distinction is central to the project:
- **VMs** are used for controlled learning and safe experimentation
- the **Windows host** is used for realistic endpoint visibility
- **Security Onion Standalone** is used as the central analysis platform

## Practical Constraints
This lab is being designed around a single host system with **32 GB RAM**.

Because of that, the project is structured to avoid treating all components as permanently active at the same time. VM resources (CPU and RAM) are dynamically reallocated depending on the active lab components, with Security Onion Standalone expected to require the majority of available resources. Ubuntu and Windows VM usage will be scaled accordingly based on the current exercise.

## Planned Use Cases
The lab is intended to support exercises such as:

- basic Linux and Windows administration
- service creation, modification, and observation
- process and connection analysis
- firewall and network visibility checks
- host-to-host communication testing
- log collection and review
- controlled generation of observable system events
- defensive monitoring and introductory investigation workflows

## Documentation Standard
This repository is intended to track the real build process, including:

- setup steps
- configuration choices
- VM resource allocation
- networking decisions
- technical issues encountered
- troubleshooting steps
- fixes applied
- screenshots
- lessons learned
- future improvements

The documentation is meant to show actual progression, including incomplete phases and implementation constraints.

## Planned Repository Structure
```text
- README.md
docs/
  ubuntu/
    analysis/
  windows/
```
## Documentation Approach
Each technical note or setup document should aim to include:

- Objective
- Why this matters
- Observations
- Result
- Notes (if needed)

## AI Usage Disclosure
For transparency: AI was used only to assist with the wording, grammar, formatting, and structure of this documentation.

The technical setup, installation work, configuration, troubleshooting, analysis, design decisions, and project progression are my own.

## Disclaimer
This project is built strictly for educational purposes in an isolated personal lab environment.
