# Windows User, Batch File, and Scheduled Task Scenario Test

## Objective
Verify that a scheduled task can execute a batch file, produce observable filesystem changes, and be linked to a specific execution context.

## Why this matters
This exercise connects multiple elements into a single scenario: local user creation, batch script execution, scheduled task creation, and visible output files. 
It is useful because it shows how several ordinary Windows components can be chained together and then analyzed through their effects and logs.

## Observations
- A local user named `Utente` was created successfully.
- A batch file was created to write output files inside `C:\Lab\scenario`.
- A scheduled task was created to execute that batch file.
- The batch file was intended to create output files and record execution context through `whoami` and `hostname`.
- The scheduled task mechanism was successfully configured and tested as part of the scenario.
- The scenario showed how a user account, a script, and a scheduled task can be combined into a single chain of actions.

## Issue Encountered
- The scheduled task created with the user `Utente` produced a warning stating that the task might fail to start because the batch logon privilege was not enabled for that task principal.
- In practice, the account `Utente` could not execute the batch file through the scheduled task in this configuration.
- To continue the exercise, the task was executed using the main account already in use on the virtual machine instead of the newly created `Utente` account.

## Result
The exercise confirmed that a Windows scheduled task can be used to launch a batch file and produce visible system changes, but it also showed that the execution context matters.
In this case, the newly created account `Utente` did not have the required privilege to run the task as configured, so the scenario had to be completed with the main VM account instead.
