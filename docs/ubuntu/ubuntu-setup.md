# Ubuntu VM Setup

## Goal
Set up a Linux virtual machine for system administration, networking exercises, and controlled traffic generation within the lab.

## Configuration
- OS: Ubuntu (Desktop)
- CPU: 1 processor / 2 cores
- RAM: 4 GB
- Disk: 40 GB (thin provisioned)
- Network: NAT

## Decisions
- Chose Ubuntu for simplicity and wide documentation support
- Kept resource usage moderate to preserve host performance
- Used NAT for initial connectivity and ease of setup

## Notes
- VM resources (CPU and RAM) can be adjusted depending on the lab phase
- Ubuntu will be used as a secondary system for generating traffic and testing interactions with Windows
- System packages were updated after installation
- One package was temporarily deferred due to Ubuntu phased updates
- open-vm-tools and open-vm-tools-desktop installed for improved VM integration (display, input, clipboard)
