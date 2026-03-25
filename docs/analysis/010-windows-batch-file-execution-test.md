# Windows Batch File Execution Test

## Objective
Verify that a simple `.bat` file can execute multiple commands, produce observable changes in the filesystem, and run through `cmd.exe`.

## Why this matters
This exercise shows that a batch file is not a special binary format, but a plain text script interpreted by `cmd.exe`.
It also demonstrates how a simple script can automate multiple actions and produce visible effects on the system.

## Observations
- A batch file named `test-script.bat` was created successfully.
- Executing the batch file created a new folder at `C:\Lab\bat-test`.
- Executing the batch file also created a file named `result.txt` inside that folder.
- The contents of `result.txt` confirmed that the batch file had run successfully.
- While the script was paused, `cmd.exe` appeared in the process list.
- The exercise confirmed that a `.bat` file runs through the Windows command interpreter rather than as a standalone executable format.

## Result
The exercise confirmed that a Windows batch file can automate multiple commands in sequence, produce visible filesystem changes, and execute through `cmd.exe`.
