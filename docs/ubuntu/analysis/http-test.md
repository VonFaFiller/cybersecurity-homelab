# Ubuntu HTTP Test

## Objective
Verify how a simple service opens a listening port and receives connections from another machine.

## Why this matters
This exercise helps establish the link between process, listening port, incoming connection, and server-side trace.

## Observations
- A Python HTTP server was started on port 8000
- Ubuntu listened on port 8000 while the process was active
- The Windows VM successfully connected to the service
- The server recorded incoming HTTP requests
- After stopping the process, port 8000 was no longer listening

## Result
The exercise confirmed that starting a service creates a listening port, remote access generates a connection, and the activity can be observed from both client and server perspectives.
