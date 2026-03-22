# Windows Local User Creation and Group Audit Test

## Objective
Verify that local account creation and local group membership changes on Windows can be traced through Security log events.

## Why this matters
This exercise shows that administrative actions performed on a Windows system leave identifiable traces in the Security log. It also introduces the use of Event IDs as a way to search for specific types of events instead of reading the full log manually.

## Observations
- A new local user named `analyst1` was created successfully.
- The new user was added to the local `Users` group.
- The user account was visible through PowerShell after creation.
- Event ID `4720` was observed and corresponded to local user creation.
- Event ID `4732` was observed and corresponded to a member being added to a local security-enabled group.
- PowerShell filtering was more practical than Event Viewer for locating the relevant events quickly.
- Event Viewer was still useful for confirming the same events through the graphical interface.

## Result
The exercise confirmed that local account creation and local group membership changes can be traced through Windows Security logs, and that specific Event IDs can be used to identify precise administrative actions.

## Notes
- Event IDs should be interpreted as references to specific event types, not as broad generic categories.
- For targeted log review, PowerShell provided a cleaner workflow than manually searching through Event Viewer.
