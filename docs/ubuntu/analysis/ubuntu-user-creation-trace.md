# Ubuntu User Creation Trace

## Objective
Verify whether user creation activity can be traced through system logs.

## Why this matters
This helps establish whether administrative changes can be linked to specific actions and privileged sessions.

## Observations
- The `adduser testuser` command appeared in the logs
- A root session opened through the local user account
- The system recorded user creation, group creation, home directory creation, and password setup

## Result
User creation activity was traceable through system logs, including the command used, the privileged context, and the affected account.
