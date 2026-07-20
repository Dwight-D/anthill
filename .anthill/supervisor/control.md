# Supervisor control flag

Updated: <date> (<who / session>)

> **RUNTIME ARTIFACT — durable control channel.** A pause/stop/resume order the
> supervisor reads on EVERY wake-up drain, before other steering. Durable so a
> control order survives message loss or cross-tier misrouting. State first,
> signal second: the user sets the state here, THEN optionally pings; the
> supervisor reads it and acknowledges by landing state. Default state is `run`.

## State

- **state:** `run`
  <!-- run | pause | stop -->
- **set-by:** <who / session>
- **reason:** <why — shown in the supervisor's stand-down / wrap-up>

## Semantics

- **run** — normal operation.
- **pause** — stop issuing new work: land worker state where possible, stand
  down without tearing down the team, keep draining on each wake-up, resume
  steering only when the state returns to `run`.
- **stop** — run the `Wrap-up` sequence (workers land state, commit+push
  supervisor artifacts, final summary) and end.
