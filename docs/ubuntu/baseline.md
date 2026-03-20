# Ubuntu Baseline

## Objective
Record the initial state of the Ubuntu VM before making configuration changes or starting lab exercises.

## Why this matters
This baseline is needed to identify what is normal in the system before introducing new users, services, ports, or network activity. It provides a reference point for future comparison.

## Observations
- Ubuntu 24.04 LTS running on VMware
- Standard local user with sudo privileges
- NAT network configured on interface ens33
- SSH service active
- Logging services active
- No unexpected services observed
- Expected listening ports include SSH and local DNS-related services

## Result
Initial system state recorded successfully.
