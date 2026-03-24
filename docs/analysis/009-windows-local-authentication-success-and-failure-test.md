# Windows Local Authentication Success and Failure Test

## Objective
Verify that failed and successful local authentication attempts on Windows can be identified through Security log events.

## Why this matters
This exercise shows that Windows records authentication attempts in the Security log and that successful and failed logons can be distinguished through specific Event IDs.
It also highlights the importance of filtering noisy events correctly.

## Observations
- A local user named `operator1` was created successfully.
- A failed authentication attempt with `runas` generated a Security log event with ID `4625`.
- A successful authentication attempt with `runas` generated Security log events with ID `4624`.
- The successful authentication opened a new command prompt running as `operator1`.
- Event ID `4624` produced a very noisy result set and included many successful logons unrelated to the exercise.
- Event ID `4625` was easier to isolate because the failed authentication event was much less noisy.

## Issue Encountered
- The initial `runas` syntax using `.\operator1` did not work as expected in this context.
- Using `operator1` directly resolved the problem.
- The initial expectation was that Event ID `4624` would clearly isolate the successful authentication from the exercise, but the result set included many unrelated successful logons.

## Result
The exercise confirmed that Windows Security logs can distinguish between failed and successful local authentication attempts.
It also showed that some Event IDs, especially `4624`, are highly noisy and require more careful filtering to be useful in practice.
