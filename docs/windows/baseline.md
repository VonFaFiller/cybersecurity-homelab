# Windows Baseline

## Objective
Record the initial state of the Windows VM before making configuration changes or starting lab exercises.

## Why this matters
This baseline is needed to identify what is normal in the system before introducing new users, services, scheduled tasks, ports, or additional network activity. It provides a reference point for future comparison.

## Observations
- Windows 11 Pro running on VMware
- Standard local user account active
- NAT network configured and working correctly
- Default Windows services active immediately after installation
- Multiple background processes present in a clean default installation
- Several active network connections observed after installation
- Several listening ports active by default (e.g. 135, 445)
- Default local accounts include built-in system accounts in addition to the active user
- The system shows significantly more background activity than the Ubuntu VM baseline

## Result
Initial system state recorded successfully.

## Notes
- Compared to a stripped or customized Windows install, the default installation includes substantially more background activity and pre-enabled components
