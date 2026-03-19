# Windows 11 VM Setup

## Goal
Create a Windows VM for controlled testing and lab usage.

## Configuration
- OS: Windows 11 Pro
- CPU: 1 processor / 4 cores
- RAM: 6 GB
- Disk: 80 GB (thin provisioned)
- Network: NAT
- Firmware: UEFI + Secure Boot

## Decisions
- Chose Windows 11 Pro over Enterprise for a simpler initial environment
- Used a local account instead of a Microsoft account
- Selected NVMe disk for improved I/O performance

## Notes
- VM resources (CPU and RAM) are adjustable depending on the lab phase
- This VM will be used as a controlled sandbox, not as the primary endpoint
- The Windows host system will serve as the main realistic endpoint
- Windows Update was completed after installation
- VMware Tools were installed after the initial setup

## Issues
- VM initially attempted PXE boot instead of loading the ISO
- Fixed by correctly attaching the ISO in CD/DVD settings
