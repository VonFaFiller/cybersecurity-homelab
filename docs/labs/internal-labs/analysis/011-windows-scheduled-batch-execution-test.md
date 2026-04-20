# Windows Scheduled Batch Execution Test

## Objective
Verify that a scheduled task can execute a batch file and produce observable changes in the filesystem.

## Why this matters
This exercise connects three important elements: a scheduled task, a script-based action, and a visible effect on the system. 
It shows that task execution is not only a configured object in Windows, but also a mechanism that can launch scripts and leave traces.

## Observations
- A batch file was created successfully.
- A scheduled task was created to execute that batch file.
- Running the task caused the batch file to execute.
- The execution of the batch file created the expected output file in the target directory.
- The Task Scheduler Operational log recorded the execution of the scheduled task.

## Result
The exercise confirmed that a Windows scheduled task can launch a batch file, that the script can modify the filesystem as expected, and that the task execution can be observed through Task Scheduler logging.
