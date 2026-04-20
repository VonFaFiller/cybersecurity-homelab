# Windows Scheduled Task Test

## Objective
Verify that a scheduled task can be created, executed, and traced through Windows Task Scheduler logs.

## Why this matters
This exercise shows that Windows scheduled tasks are not only executable system objects, but also observable through dedicated event logs.
It also highlights the difference between the component that manages tasks and the component that displays recorded events.

## Observations
- A scheduled task named `Lab-Notepad-Test` was created successfully.
- The task was executed manually.
- The action associated with the task launched `notepad.exe`.
- The Task Scheduler Operational log recorded multiple related events, including task launch, action launch, and completion.
- Event Viewer and PowerShell both showed the relevant Task Scheduler events once the correct log was used.

## Issue Encountered
- Initial log checks failed because the wrong log path was opened first.
- The correct log was `Microsoft-Windows-TaskScheduler/Operational`.
- The `Operational` log was also disabled initially, so no relevant events were being recorded.
- After enabling the correct Task Scheduler Operational log and running the task again, the expected events appeared.

## Result
The exercise confirmed that a Windows scheduled task can be created and executed successfully, and that its activity can be traced through the Task Scheduler Operational log. 
It also confirmed that missing visibility may depend on the wrong log being checked or the relevant log being disabled.
