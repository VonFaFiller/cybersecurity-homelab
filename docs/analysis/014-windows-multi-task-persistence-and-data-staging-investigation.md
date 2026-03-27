#Windows Multi-Task Persistence and Data Staging Investigation

## Objective
Investigate a more complex Windows-only scenario without reviewing the setup beforehand.
Identify the relevant artifacts.
Determine how the activity executed.
Determine whether persistence was present.
Reconstruct the main activity chain.
Identify the earliest observable origin of that chain.

## Why this matters
This exercise is useful because it moves beyond single-artifact analysis.
The scenario required correlating multiple scripts, multiple scheduled tasks, execution context, and output artifacts across different directories.
It also required separating relevant components from background noise and non-central supporting elements.
This is closer to a defensive investigation workflow than earlier exercises based on one obvious file or one obvious trigger.

## Observations
- Multiple suspicious artifacts were identified under `C:\ProgramData`.
- The activity was not limited to one folder or one script.
- Several directories contained PowerShell scripts, text files, and output artifacts that appeared related.
- One cluster of artifacts was associated with telemetry-style collection and output staging.
- Another cluster was associated with support collection and archive creation.
- Another scheduled task was associated with maintenance-style activity and appeared less central to the main chain.
- Multiple scheduled tasks were identified in Task Scheduler.
- Some of those tasks were grouped by creation time and appeared linked.
- At least one scheduled task used recurring triggers.
- At least one scheduled task used logon-based execution.
- The tasks executed PowerShell with `-NoProfile` and `-ExecutionPolicy Bypass`.
- Several tasks ran under `NT AUTHORITY\SYSTEM`.
- Script review showed that some components collected local system information.
- Observed collection included items such as hostname, user context, and service-related information.
- Part of the chain staged data in temporary locations.
- Part of the chain compressed staged data and wrote archive output to disk.
- Event Viewer and Task Scheduler history showed relevant task events.
- Event ID 106 was identified as the task registration event.
- Event ID 140 showed task update or modification activity.
- Event IDs 200 and 201 showed execution-related activity for scheduled tasks.
- The earliest observable origin identified during the investigation was the registration of scheduled tasks under the user context `GuineaPig`.
- Later execution occurred under the `SYSTEM` context.
- This indicated that task creation and task execution did not occur under the same effective context.
- The exercise contained both central components and less relevant supporting or noisy components.
- The scenario was more complex than previous exercises because the relevant chain was distributed and not limited to one obvious artifact.

## Result
The scenario was successfully investigated to a useful analytical depth.
The main activity chain was identified as a set of linked PowerShell scripts executed through multiple scheduled tasks.
The tasks provided persistence through recurring and logon-related triggers.
Relevant scripts collected local system information, staged data, and produced output artifacts including compressed diagnostic-style content.
Execution of the main operational components occurred under `NT AUTHORITY\SYSTEM`.
The earliest observable origin of the chain was the task registration activity associated with the user context `GuineaPig`.
This allowed the investigation to distinguish between the initial observable creation context and the later privileged execution context.

## Notes
This exercise highlighted an important methodological distinction.
Files, scripts, scheduled tasks, and logs should not be treated as equivalent types of evidence.
The more useful order of analysis in this scenario would have been:
Event Viewer and task history first.
Then scheduled tasks as execution mechanisms.
Then scripts as behavior.
Then files as outputs or possible noise.

This exercise also confirmed that a realistic investigation can involve several linked components without any single artifact being sufficient on its own.
The main analytical value came from correlation rather than from finding one obviously suspicious file.

Because AI-generated scenarios had started to become "too constrained" and less useful for the level of realism required, future practical exercises will rely more heavily on external and more specialized training sources.
AI support will remain useful for debriefing, reasoning review, and documentation, but not as the primary source for the most realistic scenarios.

## Issues Encountered
- The main mistake in this exercise was not tracking the investigation state while working.
- Relevant artifacts, task names, event relationships, and temporal ordering were not written down as they were found.
- Because of that, part of the exercise had to be mentally reconstructed a second time near the end.
- This especially affected the final requirement to identify the origin of the chain.
- A second issue was conceptual rather than technical.
- Files, scripts, tasks, and logs were initially treated too equally instead of being separated by investigative role.
- A third issue was terminology.

## Resolution
- The final part of the exercise was completed by identifying Event ID 106 as the most relevant creation point for scheduled tasks.
- The associated user context `GuineaPig` was treated as the earliest observable origin of the chain.
- Later execution under `SYSTEM` was then interpreted correctly as execution context rather than original entry context.
- The main procedural lesson is that future investigations should maintain a running artifact list and a rough chronological chain during the exercise.
