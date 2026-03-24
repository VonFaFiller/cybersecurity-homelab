# Windows Service State Change Logging Test

## Objective
Verify whether stopping and starting a Windows service produces easily observable events in the System log.

## Why this matters
This exercise was intended to confirm the relationship between a service state change, the System log, and the ability to trace that change through standard Windows event logging tools.

## Observations
- The tested services changed state correctly when stopped and started through PowerShell.
- Service state changes were confirmed directly through `Get-Service`.
- The expected service-related events were not easily observable through the initial PowerShell query using a fixed Event ID filter.
- Manual inspection through Event Viewer proved more useful than relying only on `Get-WinEvent` with a narrow filter.
- The Task Scheduler Operational log behavior from the previous exercise reinforced the idea that event visibility in Windows depends heavily on the correct channel, provider, and log configuration.
- In this specific test, active logging did not guarantee a clear or easily discoverable event for the service state change that was being investigated.

## Issue Encountered
- The exercise was based on the expectation that service stop/start operations would produce obvious and easily retrievable events in the System log.
- That expectation did not hold reliably on this machine for the tested services.
- Filtering too early by a specific Event ID made the investigation less useful.
- Event Viewer turned out to be more effective than the initial PowerShell query for exploring what was actually present.

## Result
The exercise confirmed that a service can stop and start correctly without necessarily producing an immediately obvious or easily retrievable event in the expected Windows log view.
It also showed that Windows event visibility depends on the actual provider/channel behavior and that manual inspection through Event Viewer can be more useful than a premature narrow query.
