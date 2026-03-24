# Ubuntu SSH Service Stop/Start Test

## Objective
Verify that stopping and starting the SSH service directly affects port availability and remote access.

## Why this matters
This exercise confirms the relationship between a running service, its listening port, and the ability of a remote client to connect.

## Observations
- While the SSH service was active, port 22 was listening and the Windows VM could connect.
- After stopping the SSH service, port 22 was no longer listening.
- While the service was stopped, the Windows VM could not connect through SSH.
- After starting the SSH service again, port 22 returned and SSH access worked again.
- The SSH logs reflected the corresponding connection attempts and service activity.

## Result
The exercise confirmed that SSH availability depends on the state of the SSH service, and that the listening port and remote access change accordingly.
