# Ubuntu SSH Login Test

## Objective
Verify that the Ubuntu VM can accept SSH connections from the Windows VM and that successful and failed authentication attempts are traceable through system logs.

## Why this matters
This exercise establishes the relationship between a real network service, its listening port, remote authentication, and the resulting log traces.

## Observations
- The SSH service was configured to listen on port 22
- The Windows VM successfully connected to the Ubuntu VM through SSH
- A successful login generated log entries showing the source IP, the authenticated user, and the opened session
- A failed login attempt generated log entries showing the source IP, the target user, and the authentication failure
- The logs recorded the event and its context, but not the password entered

## Result
The exercise confirmed that SSH provides a real remote access service on port 22 and that both successful and failed authentication attempts can be identified through system logs.

## Issue Encountered
Initial SSH connection attempts were aborted.

## Resolution
SSH was switched from socket activation to the standard service mode, which allowed the connection to complete successfully.
