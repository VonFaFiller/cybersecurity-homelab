# Windows Scheduled Task Artifact Investigation

## Objective
Investigate a suspicious Windows artifact without reviewing the setup beforehand.
Identify what executed it.
Determine whether it represented persistence.
Document the reasoning used to reconstruct the event chain.

## Why this matters
This exercise shifts the workflow from known setup validation to artifact-led investigation.
The analysis began from a suspicious filesystem artifact.
It then moved toward execution context, task configuration, and impact assessment.
This better reflects a defensive workflow in which an analyst first notices an artifact and then reconstructs what happened.

## Observations
- The investigation began from the filesystem rather than from Windows logs.
- `C:\ProgramData` was made visible and then reviewed by date and folder naming.
- The folder considered most suspicious was `C:\ProgramData\WinCache`.
- Inside that folder, the files `inventory.ps1` and `inventory.log` were found.
- The PowerShell script was opened in a text editor and reviewed without being executed.
- The script collected `whoami`, `hostname`, and `Get-Date`.
- The script wrote its output to `inventory.log` using `Out-File` with `-Append`.
- The log contained only one execution block.
- This showed that the script was designed to allow repeated execution.
- The available evidence, however, showed only a single observed run.
- The investigation then pivoted from the artifact to a plausible execution mechanism.
- Task Scheduler was checked as the most likely source.
- In Task Scheduler Library, a task named `WinCache Inventory` was identified.
- The task action launched `powershell.exe`.
- The command line included `-ExecutionPolicy Bypass`.
- The action pointed directly to `C:\ProgramData\WinCache\inventory.ps1`.
- The trigger was configured as `One time`.
- The task history showed that the task had been triggered.
- The action had started.
- The task had completed successfully.
- The execution context shown by the task was `NT AUTHORITY\SYSTEM`.
- No additional related artifacts were found in the same folder.
- No broader linked task chain was identified during the exercise.

## Result
The suspicious artifact in `C:\ProgramData\WinCache` was successfully traced to a Scheduled Task named `WinCache Inventory`.
The task executed `inventory.ps1` through PowerShell with `ExecutionPolicy Bypass`.
It ran under `NT AUTHORITY\SYSTEM`.
It completed successfully.
The trigger configuration and the available evidence showed a one-time execution rather than persistence.
No additional linked artifacts or wider execution chain were identified.

## Issue Encountered
There was an analytical risk of treating the task's `Enabled` state as evidence of persistence.
That would have been incorrect.
A one-time task can remain enabled without functioning as a recurring mechanism.
There was also an analytical risk of concluding too early that the script was harmless.
That would have been too strong.
The observed behavior was low-impact.
However, the script still represented basic local reconnaissance activity.
The investigation also reached a natural limit once no broader chain was found beyond the task and the local log file.

## Resolution
The task was interpreted through its trigger and execution history rather than through its enabled state alone.
The `One time` trigger, together with the successful completion history, supported the conclusion that this was a single scheduled execution.
It did not support persistence.
The script was classified as low-impact local reconnaissance.
It was not automatically classified as benign.
The investigation was closed once the execution mechanism, context, timing, and impact had been explained.
It was also closed once no further linked artifacts were found.

## Notes
This exercise was useful because it required reconstruction rather than simple validation.
The main practical lesson was to pivot from a discovered artifact to its execution mechanism.
Task properties then made it possible to answer what executed, when it executed, under which account it executed, and whether it was persistent or only a single execution.
The exercise also reinforced that not finding a larger chain is still a valid result when the available evidence has been reasonably checked.
